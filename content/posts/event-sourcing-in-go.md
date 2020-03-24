---
title: "Event Sourcing in Go"
date: 2020-03-23T21:13:59-04:00
draft: false
---

I've recently gone into doing CQRS with event sourcing along with DDD (Domain Driven Design) principles. I've been doing it in Go and want to share how I do it. 

To begin with I've researched this topic thouraly. I've probably watched and rewatched hundreds of videos and read many posts, articles, and books on it. I am by no means a cqrs/es expert, but I have gained some insight into how to do it. First thing I want to put out there is that you don't need a framework to do this. In fact, ddd and cqrs/es are best done without a framework getting in the way and instead you should create a [Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html) where the framework you choose later on becomes a sort of plugin into your application. 

So what is event sourcing? Simply put, event sourcing is a pattern by which current state is derived from previous facts (events). That is to say, to get the current state of an object you replay all the events of an object in memory to rebuild it from the actions you took previously. And you must accept these previous actions as fact, that is they already happened and you can't change it. You can change your mind about what a fact means to you in the future, but you can't change the fact or that it happened. New events are emmited when new actions are taken and that gets saved into the event store. The only unit of persitance in an event sourced application is the events, everything else is a projection of those events or more correctly a cache.

Events are emitted by aggregates who are composed of one or more entities. The repository for those aggregates are responsible for saving saving those events into the event store. 

Where do these events come from? They come from the initial design of the system through various excercises such as event storming, a process by which the overall process or subprocesses of an application are discovered and modeled as events that have happened. Keep in mind, these aren't fine grained low-level events like you might be used to in GUI programming, such as a user clicked event, they're more coarse than that. Take the example of a system that deals with hospital admisions of a patient, let us model the interactions the patient has in the system with events. A patient might first be admited into the hospital, we might say we have a "Patient Admited" event; notice the past perfect tense. Later on, when a patient is transfered to another ward we say that we have a "Patient Transfered" event. Finally when a patient gets discharged we have "Patient Discharged". These events arose out of the need to describe what has happened to a patient. 

These events are created through the use of commands in the command query responsibilty segregation pattern. Commands signify intent that an action should be taken, but with no gaurantee that they will. A command handler has the ability to reject commands and produce no events. There are many ways to model commands in a system. They can be as simple as imperetive methods that tell an object what to do or as DTOs that have the data in them needed to carry out the command. In my case, I chose to make it as simple as a method on an object.

So with all this, what does it all look like in Go? To begin let's look at what an event might look like in Go. Let's use our patient example.

```go
package patient

// Event is a domain event marker.
type Event interface {
	isEvent()
}

func (e PatientAdmited) isEvent()    {}
func (e PatientTransfered) isEvent() {}
func (e PatientDischarged) isEvent() {}

// PatientAdmited event.
type PatientAdmited struct {
	ID   ID         `json:"id"`
	Name Name       `json:"name"`
	Ward WardNumber `json:"ward"`
	Age  Age        `json:"age"`
}

// PatientTransfered event.
type PatientTransfered struct {
	ID            ID         `json:"id"`
	NewWardNumber WardNumber `json:"new_ward"`
}

// PatientDischarged event.
type PatientDischarged struct {
	ID ID `json:"id"`
}
```

Notice how the events are simply go structs which reference the id of the aggregate they're related to. There is also a marker interface that is implemented by them by way of a single unexported method; this is to prevent outside structs from being flaged as possible events from this package and also to limit the scope of what counts as an event for this application. 

Next we turn to our aggregate, which looks like this.


```go
package patient

// Patient aggregate.
type Patient struct {
	id         ID
	ward       WardNumber
	name       Name
	age        Age
	discharged bool

	changes []Event
	version int
}

// NewFromEvents is a helper method that creates a new patient
// from a series of events.
func NewFromEvents(events []Event) *Patient {
	p := &Patient{}

	for _, event := range events {
		p.On(event, false)
	}

	return p
}

// Ward returns the patient's ward number.
func (p Patient) Ward() WardNumber {
	return p.ward
}

// Name returns the patient's name.
func (p Patient) Name() Name {
	return p.name
}

// Age returns the patient's age.
func (p Patient) Age() Age {
	return p.age
}

// Discharged returns wether or not the patient has been discharged.
func (p Patient) Discharged() bool {
	return p.discharged
}

// ID returns the id of the patient. Duh.
func (p Patient) ID() ID {
	return p.id
}

// New creates a new Patient from id, name, age and ward number.
func New(id ID, name Name, age Age, ward WardNumber) *Patient {
	p := &Patient{}

	p.raise(&PatientAdmited{
		ID:   id,
		Name: name,
		Age:  age,
		Ward: ward,
	})

	return p
}

// Transfer transfers a patient to a new ward.
func (p *Patient) Transfer(newWard WardNumber) error {
	if p.discharged {
		return ErrPatientDischarged
	}

	p.raise(&PatientTransfered{
		ID:            p.id,
		NewWardNumber: newWard,
	})

	return nil
}

// Discharge discharges a patient
func (p *Patient) Discharge() error {
	if p.discharged {
		return ErrPatientDischarged
	}

	p.raise(&PatientDischarged{
		ID: p.id,
	})

	return nil
}

// On handles patient events on the patient aggregate.
func (p *Patient) On(event Event, new bool) {
	switch e := event.(type) {
	case *PatientAdmited:
		p.id = e.ID
		p.age = e.Age
		p.ward = e.Ward

	case *PatientDischarged:
		p.discharged = true

	case *PatientTransfered:
		p.ward = e.NewWardNumber

	}

	if !new {
		p.version++
	}
}

// Events returns the uncommited events from the patient aggregate.
func (p Patient) Events() []Event {
	return p.changes
}

// Version returns the last version of the aggregate before changes.
func (p Patient) Version() int {
	return p.version
}

func (p *Patient) raise(event Event) {
	p.changes = append(p.changes, event)
	p.On(event, true)
}
```

You'll notice a few things. First the use of private fields in the aggregate, this is to protect our invariants. Then you'll notice that each method that raises an event does not change state directly, but instead raises an event first and that gets handled by the aggregate's `On` method. The `On` method is in charge of rebuilding the state from an event. It does a type switch on the event and caries out the behaviour accordingly. Let's look at a single method closely.

```go
// Transfer transfers a patient to a new ward.
func (p *Patient) Transfer(newWard WardNumber) error {
	if p.discharged {
		return ErrPatientDischarged
	}

	p.raise(&PatientTransfered{
		ID:            p.id,
		NewWardNumber: newWard,
	})

	return nil
}
```

Notice how the `Transfer` method first checks if the patient has already been discharged before emitting the event. We first have to check the business rules are satisfied before emitting the appropriate event, otherwise we reject the command and return an error. If all is well, we simply raise the event which gets handled by the `On` method and we return nil. We do not change state unless all of the conditions are satisfied. 

Let's look at the `On` and `raise` methods more closely. 


```go
// On handles patient events on the patient aggregate.
func (p *Patient) On(event Event, new bool) {
	switch e := event.(type) {
	case *PatientAdmited:
		p.id = e.ID
		p.age = e.Age
		p.ward = e.Ward

	case *PatientDischarged:
		p.discharged = true

	case *PatientTransfered:
		p.ward = e.NewWardNumber

	}

	if !new {
		p.version++
	}
}

func (p *Patient) raise(event Event) {
	p.changes = append(p.changes, event)
	p.On(event, true)
}
```

Notice, the On method first does a type switch on the event and selects the case for each event type. This is where state change happends. Once an event is emited and saved we do not throw an error, we simply process the event and carry on. We can change here if we decide that an event is no longer relavant or if it means something different, but we can't return an error and say an event is invalid. Then we check if this is a new event, if it isn't we increment the version number of our aggregate. The `raise` method does two things, it appends the event into our changes slice and calls the event handler `On` saying that this is a new event and we should not increment the version number. Wait, what's this about the version? The version is an optimistic concurrency pattern used to help us avoid database locks in order to change our aggregate.

So let's look at how we might use this aggregate. First we have a repository that saves and retrives these's aggregate from the event store and we also have a service that represents a particular use case. 

One method on the serivce might look like this. We first load up the aggregate by replaying the events, we execute the command, and save it to the repository. Simple, no?
```go 
func (s *service) TransferPatient(
	ctx context.Context,
	id patient.ID,
	newWard patient.WardNumber,
) error {
	p, err := s.repo.Load(ctx, id)
	if err != nil {
		return err
	}

	if err := p.Transfer(newWard); err != nil {
		return err
	}

	if err := s.repo.Save(ctx, p); err != nil {
		return err
	}
	return nil
}
```

Well, things aren't so simple for our repository implementation. Here, I've come up with a way to save them to dynamodb using some helper libraries from a project called [eventsource](https://github.com/altairsix/eventsource) by using embeded structs to serialize and deserialize the event objects to and from json. (The LINQ like syntax comes from a library called [go-linq](https://github.com/ahmetb/go-linq))

```go
func (r *repository) Load(
	ctx context.Context,
	id patient.ID,
) (*patient.Patient, error) {
    // load up all events
	records, err := r.store.Load(ctx, id.String(), 0, 0)
	if err != nil {
		return nil, err
	}

	events := []patient.Event{}
	linq.From(records).
		SelectT(func(record eventsource.Record) patient.Event {
			if err != nil {
				return nil
			}

			var typed struct {
				Type string `json:"event_type"`
			}
			err = json.Unmarshal(record.Data, &typed)
			if err != nil {
				return nil
			}

			var e patient.Event
			switch typed.Type {
			case eventName(&patient.PatientAdmited{}):
				e = &patient.PatientAdmited{}
			case eventName(&patient.PatientTransfered{}):
				e = &patient.PatientTransfered{}
			case eventName(&patient.PatientDischarged{}):
				e = &patient.PatientDischarged{}
			}

			err = json.Unmarshal(record.Data, e)
			if err != nil {
				return nil
			}

			return e
		}).
		ToSlice(&events)
	if err != nil {
		return nil, err
	}

	return patient.NewFromEvents(events), nil
}

func (r *repository) Save(ctx context.Context, p *patient.Patient) error {
	records := make([]eventsource.Record, len(p.Events()))

	var err error
	linq.From(p.Events()).
		SelectT(func(event patient.Event) eventsource.Record {
			var data []byte
			switch e := event.(type) {
			case *patient.PatientAdmited:
				data, err = json.Marshal(admited{
					Type:           eventName(e),
					PatientAdmited: e,
				})
			case *patient.PatientDischarged:
				data, err = json.Marshal(discharged{
					Type:              eventName(e),
					PatientDischarged: e,
				})
			case *patient.PatientTransfered:
				data, err = json.Marshal(transfered{
					Type:              eventName(e),
					PatientTransfered: e,
				})
			}
			return eventsource.Record{
				Data: data,
			}
		}).
		ToSlice(&records)
	if err != nil {
		return err
	}

	for i := range records {
		expectedVersion := i + p.Version()
		records[i].Version = expectedVersion
	}

	return r.store.Save(ctx, p.ID().String(), records...)
}

type admited struct {
	Type string `json:"event_type"`
	*patient.PatientAdmited
}

type transfered struct {
	Type string `json:"event_type"`
	*patient.PatientTransfered
}

type discharged struct {
	Type string `json:"event_type"`
	*patient.PatientDischarged
}

func eventName(event patient.Event) string {
	t := reflect.TypeOf(event)
	if t.Kind() == reflect.Ptr {
		t = t.Elem()
	}

	return t.Name()
}
```

In the `Save` method, you'll notice we extact the events from the aggregate, conver them to json and give them a version number, starting from the last version the aggregate was at. The version number is important for optimistic concurrency as we check before we save that there are not events with those versions before we save. If there are, we've been beaten by another command handler and need to reject, because we are stale.
```go
	for i := range records {
		expectedVersion := i + p.Version()
		records[i].Version = expectedVersion
	}
```


When we load up the aggregate, we first load up all events in order and replay them back into the aggregate. 

```go
func (r *repository) Load(ctx context.Context, id patient.ID) (*patient.Patient, error) {
     // load up all events
	records, err := r.store.Load(ctx, id.String(), 0, 0)
	if err != nil {
		return nil, err
	}

	events := []patient.Event{}
	linq.From(records).
		SelectT(func(record eventsource.Record) patient.Event {
            // convert from record to domain event...
		}).
		ToSlice(&events)
	if err != nil {
		return nil, err
	}

    // Replay the events
	return patient.NewFromEvents(events), nil 
}
```

In the aggregate's pacakge I've chosen to have a helper method that replays all events for us. 

```go
// NewFromEvents is a helper method that creates a new patient
// from a series of events.
func NewFromEvents(events []Event) *Patient {
	p := &Patient{}

	for _, event := range events {
		p.On(event, false)
	}

	return p
}
```

All this does is call our `On` method by passing false to the new flag. 

With all that we now have an event sourced aggregate in Go. We didn't need a framework to do this, the go standard library was enough and I think you can figure out how to change or add what you need in the future. 

## Conclusion
Should you use event sourcing? That depends on your project and if the added complexity of adding event sourcing is warrented. I will say, that most simple applications don't, but that many will have some breakthroughs by applying this pattern. Let me hear your thoughts on this pattern. I've seen incredible success with this pattern, especially in systems of various microservices where another microservice and get notifications of things that have happend in another by just reading the event stream of another service instead of asking them directly. So I see this going a lot of places and think I'll be doing this more in the future.