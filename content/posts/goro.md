---
title: "Goro"
date: 2018-05-21T13:36:22-04:00
description: I created a Go library for the EventStore database and I wanted to share it with the world
draft: false
---

I know I haven't posted in a long time, but I finally felt I needed an update in my blog. I've recently updated some work on my own Go library for [EventStore](https://eventstore.org) called [goro](https://github.com/vectorhacker/goro). I made this for my own personal work and my job at [ALQMY](http://alqmy.io). 

I used to use [Go.GetEventStore](https://github.com/jetbasrawi/go.geteventstore) as my prefered library for working with EventStore. It was nice, but I felt it was lacking in many features and was ultimitely very cumbersome to work with, especially when it comes to reading events in a streaming fashion. I modeled my libary much like the sarama library for Kafka which has a similar api for streaming. On top of that I've added native support for reading streams, not only forwards, but backwards as well. This to me was a very important feature to have, as sometimes things like inlined snapshots in [Event Sourcing](https://martinfowler.com/eaaDev/EventSourcing.html) [[1]](https://martinfowler.com/eaaDev/EventSourcing.html)[[2]](http://microservices.io/patterns/data/event-sourcing.html)[[3]](https://eventstore.org/docs/event-sourcing-basics/index.html) require that you read a stream backwards.

So I think I should show you how the library actually works. First things first connecting to EventStore is as simple as:

```go
client := goro.Connect(
        "http://localhost:2113", 
        goro.WithBasicAuth("admin", "changeit"),
    )
```

Now here is the first major difference between my library and that of Go.GetEventStore. When you connect, you initially give it the connection options such as authentication. In the future I'll add more options, but each option is implemented via a simple interface:

```go
type ClientOption func(*Client)
```

This enables me to add options to it arbitrarily. This is very similar to how the [gRPC-Go](https://github.com/grpc/grpc-go) libraries add options to connections. In contrast Go.GetEventStore does this for authentication for example:

```go
client, err := goes.NewClient(nil, "http://youreventstore:2113")
if err != nil {
    log.Fatal(err)
}

client.SetBasicAuth("admin", "changeit")
```

I find this partial builder pattern kind of confusing and very fragile. Besdies, I'm usually very averse to the whole getter/setter pattern in general, but that's a topic for another blog post.

Another big thing I wanted was better streaming. I'm a big fan of using Go's channel feature to manage concurrency and to iterate through streaming data. It is very easy to iterate over a channel as if it where an array:

```go

stream := make(chan string)

go func(){
    for message := range stream {
        log.Println(message)
    }
}()

stream <- "hello"
stream <- "wow"

close(stream)
```

I think this is very nice, simple, clean, and elegant way of handling concurrency. There's also no risk of the channel giving an error when getting messages in this model because the go for loop automatically breaks when the channel is closed. So in that fashion, I modeled my subscriptions for EventStore in this manner:


```go
client := goro.Connect("http://localhost:2113", goro.WithBasicAuth("admin", "changeit"))

ctx := context.Background()
events := catchupSubscription.Subscribe(ctx)

for event := range events {
    //...
}
```

Each stream message has a few things inside it that help you manage the stream. This includes the event, an error, and some acknowledger for when you have persistent subscriptions.


Streaming style subscriptions isn't the only way to read events, you can also read arbitrarily using a reader:
```go
reader := client.FowardsReader("messages")
events, err := reader.Read(ctx, 0, 1)
if err != nil {
    panic(err)
}
```

as you can see, you can specify very easily the direction in which you want to read, this to me was very important.

## Improvements:
I'm still adding things to this library, and I welcome any pull requests that want to add features. A few of the features I want to add are:

- Support for the [Projections API](https://eventstore.org/docs/projections/api/index.html)
- Support for handling [users](https://eventstore.org/docs/http-api/swagger/Users.html).

These should be simple to implement, but I've been to lazy to build them yet and right now it's not a pressing concern.