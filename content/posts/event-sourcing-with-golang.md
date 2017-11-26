---
title: "Event Sourcing With Go"
date: 2017-11-26T03:38:23-04:00
draft: true
description: I talk about how to do event sourcing using Go.
---

Event sourcing is a technique in which all changes to data aren't persisted directly. Instead those changes get tracked in a log and all views of data get generated from that log. This can lead to some interesting results. For example, it is very hard to loose data because every change is tracked. It also allowes for better analytics, as you have all the data in every possible state. You can also recreate a state within a specific point in time.

Come to think of it, it sounds a little familiar. Yeah, that's right, Git does that! Every commit tracks the changes to the source code, you can replay commits to get the current state of the code and you can recreate states before certain commits were applied. 

> *The idea of event sourcing is to capture changes to state as a sequence of events and that all state can be derived from replaying those events.*

This is a very powerful idea, as it lends itself well to many domains. Often times it is very difficult to describe certain business processes with the use of CRUD operations. 

As an example lets look at a shoping cart. In a traditional scenario you'd repersent the cart as just a list of items associated to a user and would record the state of the cart within that list. Perhaps you have two tables in your database that look like this:

shopping_carts
--

| id        | user_id | state   |
| --------- |:-------:|:-------:|
| 1         | 1       | pending |

shoppint_cart_items

| id       | shopping_cart_id | item_id | count |
| -------- |:----------------:|:-------:|:-----:|
| 1        | 1                | 20      | 2     |
| 1        | 1                | 5       | 1     |
| 1        | 1                | 10      | 4     |

and you might have a list of items, of course that goes without saying. Now if you want to add an item to the shopping cart, you just create a new row to the table. But if you need to modify an item, say add one more to the item count, you need to update a row, no harm done right? To remove an item you just reduce the count or remove the row. So what's wrong with this model? Nothing, really. This model works perfectly fine, but it does have some problems. For one you can't track certain statistics with this model alone, like how many times a user added or removed an item from the list, how long after a user remves an item do they put it back... These things could be solved by just adding some additional logging mechanism to the code, but it risks lossing data or creating false data if a transaction fails. More over it is impossible to see what the list might have looked like at any given time.