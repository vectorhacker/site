---
title: "Don't Use Frameworks"
date: 2020-03-24T04:32:51-04:00
draft: false
---

Stop using frameworks for everything. Just stop. You don't need a framework to write good code and deliver products. Don't get me wrong, frameworks are useful, but they are all-consuming and hide the application. So if you shouldn't depend on frameworks what should you do instead? Clean architecture. More specifically, there are some patterns you can follow to better architect your application and think about the structure of your application and remove the need for frameworks to express your application. The patterns I suggest you use are [Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html), [Domain Driven Design](https://www.infoq.com/minibooks/domain-driven-design-quickly/), and the [SOLID](https://www.youtube.com/watch?v=t86v3N4OshQ) principles. Having a maintainable application is a matter of applying certain design principles and professionalism. In particular, applying [TDD](https://blog.cleancoder.com/uncle-bob/2014/12/17/TheCyclesOfTDD.html) or BDD and Domain Driven Design principles. Think about your application before you write a single line of code, design it, and test before you implement it.

What should you do about frameworks then? You can still use them, but only as plugins to your application. The use of a framework should be done by implementing internal interfaces for your applications' inner layers. Let me ask you this, how many times have you answered the question "what is your architecture?" with a phrase like "We have a ruby on rails back-end persisted by postgres using an angular front-end?" If you have, what you have just described is not an architecture, it is an application stack. And what is the application stack? The stack is a detail! No one cares what your application stack is, well no one who wants to get the work done cares what it is, and you shouldn't either. To be fair, I'm not saying your technology stack isn't important at some level, I'm saying that it's a detail. So what is an application's architecture then? It is the system of patterns that we use to carry out our application's functionality. Your architecture could be a simple CRUD system, or it might be a complex trading application that receives trade orders as incoming messages and delivers them by reserving credits and emitting order events depending on what state you're in. One thing is clear, your application's architecture is driven by the domain you're in and the complexity of the problem you are solving, a framework is not architecture.

So what does a frameworkless application look like? Here's an example in Go. Let us start by the business rules and implement entities. In this example we're using a Cargo shipping application. Notice that our entities are free of any database dependencies and external concerns. 

### Entities
```go
// TrackingID uniquely identifies a particular cargo.
type TrackingID string

// Cargo is the central class in the domain model.
type Cargo struct {
	TrackingID         TrackingID
	Origin             UNLocode
	RouteSpecification RouteSpecification
	Itinerary          Itinerary
	Delivery           Delivery
}

// SpecifyNewRoute specifies a new route for this cargo.
func (c *Cargo) SpecifyNewRoute(rs RouteSpecification) {
	c.RouteSpecification = rs
	c.Delivery = c.Delivery.UpdateOnRouting(c.RouteSpecification, c.Itinerary)
}

// AssignToRoute attaches a new itinerary to this cargo.
func (c *Cargo) AssignToRoute(itinerary Itinerary) {
	c.Itinerary = itinerary
	c.Delivery = c.Delivery.UpdateOnRouting(c.RouteSpecification, c.Itinerary)
}

// DeriveDeliveryProgress updates all aspects of the cargo aggregate status
// based on the current route specification, itinerary and handling of the cargo.
func (c *Cargo) DeriveDeliveryProgress(history HandlingHistory) {
	c.Delivery = DeriveDeliveryFrom(c.RouteSpecification, c.Itinerary, history)
}

// NewCargo creates a new, unrouted cargo.
func NewCargo(id TrackingID, rs RouteSpecification) *Cargo {
	itinerary := Itinerary{}
	history := HandlingHistory{make([]HandlingEvent, 0)}

	return &Cargo{
		TrackingID:         id,
		Origin:             rs.Origin,
		RouteSpecification: rs,
		Delivery:           DeriveDeliveryFrom(rs, itinerary, history),
	}
}
```

The cargo object is an entity that is concerned with the carrying out of business rules related to shipping cargo, such as route assignments and delivery progress. We abstract the saving and retrieving of the cargo using the repository pattern and creating an interface for the repository.

```go
// CargoRepository provides access a cargo store.
type CargoRepository interface {
	Store(cargo *Cargo) error
	Find(id TrackingID) (*Cargo, error)
	FindAll() []*Cargo
}
```

This interface is not given a concrete implementation at this step, instead, it is used in our use case through dependency injection in which the specific implementation will be chosen later. This is where our persistence technology will plug into. Notice how our application needs to know nothing about how cargo is persisted to use it, it is a detail we'll sort out later.

### Use cases
Our use cases are implemented by exposing a common use case interface for that particular use case and a hidden implementation that gets chosen using the factory pattern. That is to say, our use cases are expressed as interfaces with methods (actions) we can take. Use cases import only the inner layers of our application, such as the entities, and get injected with the concrete implementation of other interfaces like repositories and other uses cases. 

```go
// Service is the interface that provides booking methods.
type Service interface {
	// BookNewCargo registers a new cargo in the tracking system, not yet
	// routed.
	BookNewCargo(origin shipping.UNLocode, destination shipping.UNLocode, deadline time.Time) (shipping.TrackingID, error)

	// LoadCargo returns a read model of a shipping.
	LoadCargo(id shipping.TrackingID) (Cargo, error)

	// RequestPossibleRoutesForCargo requests a list of itineraries describing
	// possible routes for this shipping.
	RequestPossibleRoutesForCargo(id shipping.TrackingID) []shipping.Itinerary

	// AssignCargoToRoute assigns a cargo to the route specified by the
	// itinerary.
	AssignCargoToRoute(id shipping.TrackingID, itinerary shipping.Itinerary) error

	// ChangeDestination changes the destination of a shipping.
	ChangeDestination(id shipping.TrackingID, destination shipping.UNLocode) error

	// Cargos returns a list of all cargos that have been booked.
	Cargos() []Cargo

	// Locations returns a list of registered locations.
	Locations() []Location
}

type service struct {
	cargos         shipping.CargoRepository
	locations      shipping.LocationRepository
	handlingEvents shipping.HandlingEventRepository
	routingService shipping.RoutingService
}

// NewService creates a booking service with necessary dependencies.
func NewService(cargos shipping.CargoRepository, locations shipping.LocationRepository, events shipping.HandlingEventRepository, rs shipping.RoutingService) Service {
	return &service{
		cargos:         cargos,
		locations:      locations,
		handlingEvents: events,
		routingService: rs,
	}
}

func (s *service) AssignCargoToRoute(id shipping.TrackingID, itinerary shipping.Itinerary) error {
	if id == "" || len(itinerary.Legs) == 0 {
		return ErrInvalidArgument
	}

	c, err := s.cargos.Find(id)
	if err != nil {
		return err
	}

	c.AssignToRoute(itinerary)

	return s.cargos.Store(c)
}

func (s *service) BookNewCargo(origin, destination shipping.UNLocode, deadline time.Time) (shipping.TrackingID, error) {
	if origin == "" || destination == "" || deadline.IsZero() {
		return "", ErrInvalidArgument
	}

	id := shipping.NextTrackingID()
	rs := shipping.RouteSpecification{
		Origin:          origin,
		Destination:     destination,
		ArrivalDeadline: deadline,
	}

	c := shipping.NewCargo(id, rs)

	if err := s.cargos.Store(c); err != nil {
		return "", err
	}

	return c.TrackingID, nil
}

func (s *service) LoadCargo(id shipping.TrackingID) (Cargo, error) {
	if id == "" {
		return Cargo{}, ErrInvalidArgument
	}

	c, err := s.cargos.Find(id)
	if err != nil {
		return Cargo{}, err
	}

	return assemble(c, s.handlingEvents), nil
}

//...
// Location is a read model for booking views.
type Location struct {
	UNLocode string `json:"locode"`
	Name     string `json:"name"`
}

// Cargo is a read model for booking views.
type Cargo struct {
	ArrivalDeadline time.Time      `json:"arrival_deadline"`
	Destination     string         `json:"destination"`
	Legs            []shipping.Leg `json:"legs,omitempty"`
	Misrouted       bool           `json:"misrouted"`
	Origin          string         `json:"origin"`
	Routed          bool           `json:"routed"`
	TrackingID      string         `json:"tracking_id"`
}
```

In this example, we have a booking use case that handles the booking of new cargo. Notice how in the constructor we are expressing the repository dependencies as arguments and do no instantiate our implementations. Instead, concrete implementations of these interfaces are passed on to us, injected, in the main package. We also don't pass entities in and out of this layer to the outside, instead, we receive and output DTOs that are specific to this use case. These DTOS are just simple structs with no logic which we transform into and from our internal entities and value objects. We do not cheat and pass out our entities because that would couple them to the needs of the outside world, and we want to shield our innermost layers from change. 

Now, where do we put in the framework if we use one? We put them on the outside and include our use cases as dependencies. For example, in our booking cargo use case, we will have HTTP handlers for each method in the service interface. 

```go
type bookingHandler struct {
	s booking.Service

	logger kitlog.Logger
}

func (h *bookingHandler) router() chi.Router {
	r := chi.NewRouter()

	r.Route("/cargos", func(r chi.Router) {
		r.Post("/", h.bookCargo)
		r.Get("/", h.listCargos)
		r.Route("/{trackingID}", func(r chi.Router) {
			r.Get("/", h.loadCargo)
			r.Get("/request_routes", h.requestRoutes)
			r.Post("/assign_to_route", h.assignToRoute)
			r.Post("/change_destination", h.changeDestination)
		})

	})
	r.Get("/locations", h.listLocations)

	r.Method("GET", "/docs", http.StripPrefix("/booking/v1/docs", http.FileServer(http.Dir("booking/docs"))))

	return r
}

func (h *bookingHandler) bookCargo(w http.ResponseWriter, r *http.Request) {
	ctx := context.Background()

	var request struct {
		Origin          shipping.UNLocode
		Destination     shipping.UNLocode
		ArrivalDeadline time.Time
	}

	if err := json.NewDecoder(r.Body).Decode(&request); err != nil {
		h.logger.Log("error", err)
		encodeError(ctx, err, w)
		return
	}

	id, err := h.s.BookNewCargo(request.Origin, request.Destination, request.ArrivalDeadline)
	if err != nil {
		encodeError(ctx, err, w)
		return
	}

	var response = struct {
		ID shipping.TrackingID `json:"tracking_id"`
	}{
		ID: id,
	}

	w.Header().Set("Content-Type", "application/json; charset=utf-8")
	if err := json.NewEncoder(w).Encode(response); err != nil {
		h.logger.Log("error", err)
		encodeError(ctx, err, w)
		return
	}
}
```

In this example, we are using the chi package to implement routing and include our booking service as a dependency. This layer has no application logic and only calls the appropriate methods on our use case, converting the raw data into and from dtos for our use case. When we instantiate our server implementation we pass in the use case implementations as arguments to the constructor.

```go
// Server holds the dependencies for a HTTP server.
type Server struct {
	Booking  booking.Service
	Tracking tracking.Service
	Handling handling.Service

	Logger kitlog.Logger

	router chi.Router
}

// New returns a new HTTP server.
func New(bs booking.Service, ts tracking.Service, hs handling.Service, logger kitlog.Logger) *Server {
	s := &Server{
		Booking:  bs,
		Tracking: ts,
		Handling: hs,
		Logger:   logger,
	}

	r := chi.NewRouter()

	r.Use(accessControl)

	r.Route("/booking", func(r chi.Router) {
		h := bookingHandler{s.Booking, s.Logger}
		r.Mount("/v1", h.router())
	})
	r.Route("/tracking", func(r chi.Router) {
		h := trackingHandler{s.Tracking, s.Logger}
		r.Mount("/v1", h.router())
	})
	r.Route("/handling", func(r chi.Router) {
		h := handlingHandler{s.Handling, s.Logger}
		r.Mount("/v1", h.router())
	})

	r.Method("GET", "/metrics", promhttp.Handler())

	s.router = r

	return s
}
```

Notice, no logic relating to our use case is here, only logic to satisfy our framework an be a bridge between our use case and the framework. In this way, the core of our application knows nothing about what frameworks we use. It doesn't care at all. The framework could be changed two weeks from now and you wouldn't have to change a thing about the use case because of it. The only time use cases or entities might change is when business requirements change, never for technical reasons.

What about the repositories? Those are an outside-in thing that gets implemented elsewhere. When we choose a persistence technology we simply implement that interface and it gets passed into our use cases upon application startup. Here's a trivial, but important illustration, of an in-memory repository for our Cargo example.

```go
// Package inmem provides in-memory implementations of all the domain repositories.
package inmem

import (
	"sync"

	shipping "github.com/marcusolsson/goddd"
)

type cargoRepository struct {
	mtx    sync.RWMutex
	cargos map[shipping.TrackingID]*shipping.Cargo
}

func (r *cargoRepository) Store(c *shipping.Cargo) error {
	r.mtx.Lock()
	defer r.mtx.Unlock()
	r.cargos[c.TrackingID] = c
	return nil
}

func (r *cargoRepository) Find(id shipping.TrackingID) (*shipping.Cargo, error) {
	r.mtx.RLock()
	defer r.mtx.RUnlock()
	if val, ok := r.cargos[id]; ok {
		return val, nil
	}
	return nil, shipping.ErrUnknownCargo
}

func (r *cargoRepository) FindAll() []*shipping.Cargo {
	r.mtx.RLock()
	defer r.mtx.RUnlock()
	c := make([]*shipping.Cargo, 0, len(r.cargos))
	for _, val := range r.cargos {
		c = append(c, val)
	}
	return c
}

// NewCargoRepository returns a new instance of a in-memory cargo repository.
func NewCargoRepository() shipping.CargoRepository {
	return &cargoRepository{
		cargos: make(map[shipping.TrackingID]*shipping.Cargo),
	}
}
```
You can imagine that in a production environment you would implement this using SQL or some NoSQL database and build the entities up from serialized values. The repository implementation is responsible for rebuilding the objects from persistence, the entities don't care as long as they're built correctly. 

## Conclusion
So what should you take away from all this? That you don't need a framework. That simply applying these principles will get you far and frameworks are just plugins into your application. I recommend that you study these principles and practices more in-depth and stop thinking in terms of frameworks but in terms of your domain. What is it that you're trying to accomplish and build that, don't focus on the tools.

To see the entire code, go to the [GoDDD repository](https://github.com/marcusolsson/goddd/) where a complete example of DDD in Go lives. Fair warning, he does tend to ramble a lot during the beginning of those talks with interesting anecdotes, but they're always fun to watch. He's also written several books you should check out on clean code and architecture.

Also, check out the talks given by Uncle Bob on clean architecture. He's the OG when it comes to clean code and maintainability these days. 
* [ITkonekt 2019 | Robert C. Martin (Uncle Bob), Clean Architecture and Design](https://www.youtube.com/watch?v=2dKZ-dWaCiU&t=1290s)
* [Uncle Bob Martin - The Clean Coder](https://www.youtube.com/watch?v=NeXQEJNWO5w)
* [The Principles of Clean Architecture by Uncle Bob Martin](https://www.youtube.com/watch?v=o_TH-Y78tt4)
* [Robert C Martin - Clean Architecture and Design](https://www.youtube.com/watch?v=Nsjsiz2A9mg)

Finally, check out the topics of Domain-Driven Design and SOLID. They will change the way you architect your applications, I guarantee it.