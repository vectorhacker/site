---
title: "Value Objects in Go"
date: 2020-10-23T03:12:35-04:00
toc: false
images:
tags:
  - DDD
  - Go
  - Domain Driven Design
---

Recently I've been thinking a lot about how to apply some DDD principles in Go in idiomatic and simple ways, and I've come accross a few patterns I'd like to share. Mainly we'll be talking about how to apply the concept of Value Objects in Go in two ways. These are examples of how I apply them in my projects and actually use them so these are not just toy examples.

## Description

What is a value object first of all? It's an object that isn't quite an entity, but represents data that entities might use to complete their tasks. These are, as the name sugests, objects that contain the values of something useful to the domain being worked on. Examples of these objects include Social Security Numbers, Names of people, Locations, Dates, Routing information, etc. Two of the key characteristics for value is that they are 1. Immutable and 2. not littering our domain with just strings.

## Always Valid

On top of what the concept of a value object, I'd like to introduce the concept of an always valid value object. For value objects that represent something more complex or have strict validation rules, I often introduce the concept of them always being valid once created. That is to say if when constructing the object all the inputs are valid then the object is always in a valid state, otherwise it is never created. Put another way, you cannot create and use a value object that isn't valid.

### Immutability

In order to be always valid, the object must be immutable. To do that in Go we use encapsulation by using package level hiding of fields. 

## Implementing in Go

The first and and most common kind of value object I create in Go is the simple type alias. 

```go
type ID string
type City string
```

If the object does not need complex validation and is just a simple field then it can be a type alias. Examples include things like ids, names that don't need to be checked if empty or not, and sometimes certain numbers. Some value objects are even simpler and come with the standard library, such as `time.Time` (tho admitidly internally internally it is more complex) and `bool`. 


The second kind is one that I also use often, if not more often considering that it represents complex concepts, is the immutable field value object.

As an example, take a name field that need to be within a certain length of characters. We'd implement them in Go like this:

```go

type Name struct {
  // explicitly not public field
  name string
}


// NewName creates a new valid name object
func NewName(name string) (Name, error) {
  if len(name) > 20 || name == ""  {
    return Name{}, fmt.Errorf("invalid name, must be withn 20 charcters and non-empty")
  }

  return Name{name: name}, nil
}

// String implements the fmt.Stringer interface.
func (n Name) String() string {
  return n.name
}

// MarshalText used to serialize the object
func (n Name) MarshalText() ([]byte, error) {
  return []byte(n.name), error
}

// UnmarshalText deserializes the object and returns an error if it's invalid. 
func (n *Name) UnmarshalText(d []byte) error {
  var err error
  *n, err = NewName(string(d))
  return err
}
```


A few things to note are that first, the only way to create the object is through the constructor; there's no other way, this way we gaurantee it's valid once created. If it's invalid to create it, we quickly return an error. The other thing to note is the implementation of the Stringer, TextMarshaller, and TextUnmarshaller interfaces; we do this so that it is easy to serialize and deserialize the objects. 

With all those combined, we've created our value objects and this has several advantages. Firstly when we use these objects we can rest assured that they are always valid. When created, say in our application layer we can quickly validate them. The aggregates need not worry about checking them nor should the application code, because they are never in an invalid state nor in any risk of being set to something invalid. The only check we should enforce in our aggregates and service layers is to check if one of the simpler types are empty when using them if that's something we need to have not happen, other than that it's simple to just assume they aren't empty.

We can of course create value object out of value objects, but that's another story.

## Conclusion

In Go, value object are simple to implement and we can reap numerous rewards by applying the always valid concept. With that, I'll let you Go and may your objects always be valid.