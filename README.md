# Historical Modeling

Finding meaning in the relationships among partially ordered facts

## Building the Book

This is the repository for the Historical Modeling book, available on [LeanPub](https://leanpub.com/historicalmodeling). The purpose of this repository is primarily to feed that build cycle, but also to give you access to the content as it is developed. Please provide feedback on the book via [issues](https://github.com/michaellperry/historicalmodeling/issues).

I'm building the book in the open. But it does not have an open license. You may not create derivative works. [Creative Commons Attribution-NonCommercial-NoDerivitives 4.0 International License](https://github.com/michaellperry/historicalmodeling/blob/master/LICENSE.md).

## What is Historical Modeling

Historical Modeling is a model of software behavior based on partially ordered facts. It is a set of analysis, design, and implementation techniques. You can use Historical Modeling to build web applications, mobile applications, and enterprise systems. These patterns work well with relational databases, NoSQL databases, and other kinds of data stores. These are patterns. That said, you get the best experience when building solutions using tools built with Historical Modeling in mind.

In a historical model, a system records **facts**. These facts know about each other. If a fact knows about another fact, then that other fact is a **predecessor** - it came before. For example, invoices, purchase orders, and customer relationships are all partially ordered historical facts.

![Customer invoice model](manuscript/images/InvoiceFact.png)

Historical Modeling is a set of rules for defining historical facts. It's a set of patterns for constructing applications out of historical facts. And it's a set of tools for determining software behavior based on a history of facts. This book will introduce you to these concepts, and give you the tools you need to build distributed systems according to these rules.

### The nature of distributed systems

Most software systems that we build today are distributed systems. Rarely does all of the code and data reside on the end-user's machine. In distributed systems, the primary challenge is moving date from where it is stored to where it is used.

The most common models of sofware  behavior are based on current state. These include object-oriented, relational, non-relational (NoSQL), actors, and many other models. In these models of software behavior, user actions change the current stat, and the software displays that state. Unfortunately, these models do not reflect the reality of distributed systems.

In a distributed system, current state is an impossibility. At best, you can identify the state at a particular machine. But as you move away from that machine, you cannot know what that state is at any given time with any certainty. This is the reality of a distributed system, the most common type of system that we build today, and the models by which we build these systems do not acknowledge it.

There are a few models of software behavior that *do* acknowledge the truth of distributed systems. One of them is Event Sourcing. Another is Historical Modeling.

### Partial order

Both Event Sourcing and Historical Modeling record the actions of the users and other actors in the system, and derive the current state from those actions. The difference, however, is that Event Sourcing is based on a total order of events, while Historical Modeling is based on partial order.

A total order defines a sequence of events, each one coming before the next. Think about a game of chess. Each move has to occur in order. The current posision is based on the sequence of moves, but if these moves are played in a different order, the board wouldn't be the same.

A partial order defines a relationship between selected events. Some events occur before others. But some events are not related. A partial order is less restrictive than a total order.

Total ordering relies upon a container. Event Sourcing, for example, requires a sequential store. The event store puts all of the events in order. Partial order, however, captures the relationships within the events themselves. Each event knows specifically which other events occurred earlier. If there is no relationship recorded between two events, then either one could occur before the other.

When the container defines a total order, the distrubuted system built on top of it tends to be more centralized. But when events take on the responsibility of defining their own partial order, then the distributed system is allowed to be more decentralized.