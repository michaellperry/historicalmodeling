# Why Historical Modeling?

At the beginning of this century, a friend and I, working on building a web development business, attended a day-long conference on networking equipment and services. We saw routers designed to balance the traffic in any given network, and testing equipment designed to saturate that network and bring it down. We learned about the OSI 7-layer stack, and where different vendors and components fit in and supported that model.

This conference was eye-opening for me. But not because of all the innovative solutions that I saw before me. My eyes were opened to the absurdity of the problem itself. It seemed that we were putting far too much effort, money, time, and brainpower into solving a single problem: moving data from where it was to where it is needed.

What occurred to me is that we have invested a huge amount of effort into making sure that information flows from one computer to another. For example, it flows from a web browser on an end-user's computer to a database in a datacenter halfway around the world. Along the way it passes through network switches, load balancers, web farms, and application servers before it finally arrives.

After the data takes that long arduous journey, that same user will refresh the page and pull all of that data back again.

How absurd this all looked to me! Surely, we can find a model of computing that lets the data reside where it is most needed. If information originates from a user, why must he wait for it to cross all those layers only to be re-rendered and displayed right back to him? If he wants to collaborate with someone else, why can't they exchange just the common subset of information between themselves, rather than mixing their data together in a central database?

This observation set me on a journey to discover a model of computing that uses data where it resides, rather than relying upon the interactions between connected systems. I found hidden assumptions that we impose upon ourselves when building distributed systems: a client must wait for a response from a server in order to get the most recent data; a record can be identified by a key generated upon insertion; a URI – despite containing a host name – can identify a resource independent of its location. And the biggest assumption of them all: changes to individual resources can appear sequential, even though the aggregate changes asynchronously and in parallel.

The systems that we had always built depended upon these assumptions. And so the protocols and languages supporting those systems had to ensure that these assumptions were true. But they weren't. And so those protocols and languages were unreliable. Which leaves vendors at a networking conference selling plugs for a leaky dyke.

One-by-one I removed these assumptions until I was left with a fundamental core: a set of laws that govern information, how it interrelates, how it flows, and how it can be traversed. Then piece-by-piece I constructed a model of computing that relies only upon these fundamental laws. The result has clearly defined behaviors, whether a system is connected or disconnected, synchronous or asynchronous. It makes no promises that it can't keep.

The model removes concepts that we used to take for granted. Concepts like changing the value of a variable over time. Concepts like locking a resource so that it cannot be changed by two actors simultaneously. Concepts like looking up the value of a resource by its ID. The challenge, then, is to build software systems under a new set of rules. Can the systems that we need be built without the assumptions that we are used to? 

The answer, I discovered, is no. You still need to return to the old regime to solve specific problems. But amazingly, the larger portion of the software systems that you need can be constructed without them. These portions can be built using only the core rules of information: Historical Modeling. And when you do so, you reap the rewards of reliability, scalability, and autonomy.

In this book, I will introduce you to the rules of Historical Modeling. I will demonstrate that these rules are easily satisfiable in a distributed system. I will show how you can use Historical Modeling to build the larger portion of a software system, and how to integrate that portion with the old regime when necessary. But before I do that, I must first expose the hidden assumptions under which we have traditionally written software.

## CRUD

When you look at how software is constructed today, much of it is concerned with computing the current state. Entire business applications are built on the operations Create, Read, Update, and Delete – known as the CRUD operations. In this mission to compute the current state, we have lost track of a vital piece of information: how the system came to be in this state in the first place.

CRUD destroys information about past decisions. Let's dig a little deeper into the CRUD operations to see where that information got lost. The first two operations do not destroy information.

-	Create – record new information where there was none
-	Read – retrieve information already recorded

Creating does not destroy any information that used to be there; it simply adds to it. Reading doesn't change the system at all. So neither of these two operations lose information.

But now comes Update. It sounds innocuous: bring some information up to date. But what happens to the information that used to be in its place? When you update a customer's shipping address, the old address is overwritten. It is destroyed. And with it, we destroy some potentially important features. Are there any orders being shipped to the old address that we need to re-route? Is there a general demographic shift that we can mine from this data?

And what was the context of that change? Who made it, and what other information was available to them at the time? This might be important for assessing data quality, and for correcting errors after they are discovered.

When performing an Update operation, all of this information is lost. Even if the system captures auditing attributes – who made the change and when – these attributes themselves will be overwritten by the next update. Updates, because they occur in situ, are destructive.

Then we come to Delete, which is destructive by its very nature. When a record is deleted, not only is the information contained within the record destroyed, but so is the audit information that might have been saved along with it. If subsequent records had referred to the deleted record, then at best those records are orphaned. At worst, the deletion cascades through the system, deleting wave upon wave of related data.

And so, while Create and Read preserve information, Update and Delete destroy it. The first step toward modeling a system historically, therefore, is to purge Update and Delete from our toolset.

### Immutability

There is a word for models that don't allow Update or Delete operations: immutable. The challenge is to represent a fluid, dynamic system with nothing but immutable data structures. That challenge at first seems impossible. If the purpose of the system is to change over time, then how can we accomplish this with data structures that don't change?

The insight, however, is to recognize that the Create operation, even though it creates an immutable record, does indeed change the state of the system. If we define the immutable record thus created in just the right way, we can simulate Update and Delete operations. The difference is that, since our simulation uses only Create, no information is lost. The benefits, as we will see shortly, are tremendous.

How shall we, then, define our immutable data structures so that they simulate Update and Delete operations? It turns out that people have been doing just that for centuries. People have been recording information with immutable data structures long before computers were invented. It's just that they didn't call them "data structures"; they called them "documents". Historical Modeling simply recalls, and in some cases formalizes, these old techniques.

Let's look at a common example: accounting. Business is transacted not in the writing and rewriting of the current balance, but in the accretion of historical records.

Accountants practice the process of writing to a ledger all of the transfers of value from one account to another. At any point, the values of the transfers can be added together to obtain the current balance. But this balance is secondary. The important information is in the historical ledger of transfers themselves. It's in the history. These records capture the context of each transaction: who was involved, what was exchanged, and for what purpose. They record the knowledge that was present at the time, and upon which the business decision was made. They capture the transient relationships among the parties doing business with one another.

This information is valuable apart from its role in helping us determine current state. It helps us to reconcile different versions of history. It helps us to audit the behaviors of the past. It helps us to correct errors, not by changing history, but by compensating. And it helps record the resolutions of disputes, so that they are not raised again in the future.

Accountants' pencils have no erasers.

We understand the value of history when dealing with money. Why then do we build software on databases that overwrite history in an effort to maintain the current state?

## SOAP and RPC

The year after the networking conference, I joined a team working on a gift card system for use in restaurants. It was an exciting and challenging project. At the time, most businesses were only connected to the Internet over a slow and unreliable dial-up connection. Restaurants in larger cities sometimes had DSL, but even those networks were not always connected. A key selling point of this new Gift Card system would be that it would continue to work when the network was out. To accomplish this, we chose to cache card balances at the restaurant, and exchange messages when the network was available.

For our message exchange, we decided to use a new protocol called "SOAP". SOAP, if you don't recall, stood for the Simple Object Access Protocol. One of my teammates pointed out that it was not Simple, it was not used to Access Objects, and it was not (at least at the time) a clearly specified Protocol. We were building the client side in C++, and the server side in Java, but the standard was still several years away from making such interoperability simple.

Nevertheless, we set ourselves the challenge of using SOAP to exchange data between clients in the restaurant and servers in our datacenter. We all read a book (Understanding SOAP, by Kenn Scribner, Mark C. Stiver) that taught SOAP while building a library that generates C++ function calls from XML schemas. The code in this book actually generated machine code on-the-fly to extract parameters from a stack frame, generate a SOAP call, and then turn the resulting XML back into a stack frame containing the binary structure of the result. There were several reasons we discovered that this was a poor choice for the basis of our application message interchange. But the biggest one was that this put us in the RPC mindset.

RPC, or Remote Procedure Call, is the practice of turning a procedure invocation complete with its parameters into a network request. The call blocks until the response is received, and then the response is converted into the return value. The book on which we based our implementation did this literally at the binary level.

Because we focused on the library included in this book, we only considered the RPC capabilities of SOAP. But SOAP also had another mode of operation. It could be used as a document interchange instead of remote procedure calls. If we had focused on this style of use, perhaps things would have turned out better.

### Works on localhost

Thinking, as we were, in terms of remote procedures, we came up with a set of function signatures that solved our particular problem. That was to bring down a copy of gift card values to the restaurant, and to send up a set of financial transactions. These transactions were Add Value, Redeem, and Request Seed. Request Seed was a special transaction that told the server to send a fresh set of card values to the restaurant.

When we started, things seemed to be going well. But as we continued building out this system, we noticed more and more ways in which remote messages were not like procedure calls. At first, we got interoperability between C++ and Java working, at least for the limited set of SOAP messages that we were exchanging. Everything appeared to work just fine in development. But then we deployed to test.

The test machines were not running both the client and server on the same box. They were configured to run the server on one machine, and clients on several others. Once the system was in the test environment, we started to lose transactions.

One of the tests involved running a batch of automated transactions from the various clients and comparing card balances at the end. These tests ran overnight, and in the morning we observed the card balances. Some cards still retained their higher original balance on the server, despite being redeemed on one of the clients.

We diagnosed the problem and determined that transactions were being removed from the client's outgoing queue before being sent to the server. If there was an intermittent network failure, then the client would retry sending the transaction a few times before giving up. It would log the error, but the transaction was lost.

This was a simple fix. Just keep the transaction in queue until the client heard back from the server. Once the client knew that the server had received the transaction, it could remove it from its queue.

We made the change, deployed to the test environment, and again ran it overnight. In the morning, several of the cards had lower balances on the server than on the client. We had the opposite problem!

We tried to reproduce the issue in development, but the balances always turned out to be correct. We coined a phrase to express our frustration: "Works on localhost".

The problem, we finally deduced, was that a request was making it from the client to the server, but its response was not making it back. The server was decrementing the card's value, but the client was not removing the transaction from the queue. The client would continue to re-send the transaction until the network connection was restored, at which point the effect would be duplicated and the balances would disagree.

This problem arose because we were thinking of network requests and responses as procedure calls. Could a procedure ever fail to return to the caller? It would be as if the procedure's operations completed successfully, only for the "return" statement to thrown an exception. This failure condition never happens in procedure calls. RPC tricked us into thinking that it wouldn't happen during communications either.

### The Two Generals Problem

I would later learn about the Two Generals Problem. This is a story that illustrates a fundamental truth about distributed computing. The story begins with two generals of cooperating armies whose mission it is to capture a besieged city. They must both attack on the same day. If only one attacks, they will certainly loose. They are also separated by hostile territory. The generals themselves must not cross the gap to coordinate with one another, as they would be captured. So they must send messengers through enemy territory to agree upon a day for the attack. Any messenger might be lost, but the generals must together choose a day, and both must know for certain that the other agrees with that choice.

What then is the protocol? What messages and responses can the generals send through enemy territory in order to be sure that they reach a mutually understood agreement?

If one general sends a messenger to the other with the message to attack tomorrow, how will he be sure that the message will be delivered successfully? Perhaps he should send two. Will one of them make it? Send ten? The probability increases, but this isn't a question of chance; it's a question of certainty. If the second general never receives the message and the first attacks alone, then all will be lost.

So perhaps the protocol includes confirmation. The second general sends back a message confirming that the plan to attack was received. Now the first general only attacks if he receives confirmation.

But how can the second general be sure that the confirmation made it back to the first? If it doesn't, the first will not attack. If the second general attacks alone, then all is lost.

So perhaps the protocol includes acknowledgement. The first general sends back a message acknowledging that confirmation was received. The second general can then attack with confidence, but only if he receives acknowledgement.

You can see where this is going: a stalemate of indecision. At some point, no matter what protocol you choose, there is a final message that influences the recipient's behavior. If that message is received, the recipient will act one way; if not, he will act another way. There may be later messages in this protocol, but this is the last one that influences behavior.

Given that this is the last message that influences behavior, what is the sender's plan? The sender does not know whether the message will arrive. Nevertheless, he must already have determined his own behavior. So based on whether this particular message arrives or not, the sender and the recipient may or may not agree.

Thus, there is no solution to the Two Generals Problem. There is no message passing protocol that can guarantee that two parties reach an agreement, such that both know that agreement has been reached. And this was precisely the problem that we the Gift Card team believed we needed to solve. No wonder we were frustrated!

### The decision maker

Even though we didn't know about the Two Generals Problem at the time, we believed that we had to solve that problem to complete a gift card transaction. The client had to know that there were sufficient funds on the card to complete the transaction. The server had to agree that the transaction had taken place, and deduct the balance accordingly. Either the transaction took place on both sides, or it took place on neither. Both sides had to reach a mutually understood agreement.

But in fact, that problem statement is overconstrained. Yes, indeed, both sides need to reach an agreement. However, they do not need to both understand that agreement had been reached. By relaxing this constraint, we turned the problem from an impossible Two Generals into something that we could solve.

The key to solving this problem was in appointing a decision maker. In our case, the client was ultimately responsible. The client would do its best to determine the available balance, but no matter what information the server might have, the client would make the final go/no-go decision. Once the decision was made, the server would eventually have to agree with it. But the client did not need to know that agreement had been reached.

I should point out that our business customer was not pleased with this result. The server was ultimately aware of the balance of each card. Yet we, the development team, were arguing that the client would have the final say. If the client decided that the card should be redeemed, even if the server knew that it had insufficient funds, the transaction could not be rejected. How could it be any other way? If the server made the ultimate decision, then there would be no performance or resiliency benefit gained by caching card balances on the client. A key selling point of our solution was that it worked while disconnected, and so this was the only choice permitted us. Eventually, our business customer understood that this must be the case.

Once we had a decision maker, the Two Generals Problem disappeared. No longer must both sides understand that an agreement had been made before acting. One decides, and then continues the conversation until he knows that the other is aware of that decision.

### Idempotence

Having reframed the problem to make it solvable, we set forth implementing the solution. Remember that we faced first the problem of lost transactions, and then the problem of duplicated transactions. We had to be sure that a transaction would not be duplicated. The word for that, I would later learn, is "idempotence". 

Idempotence is expressed informally as the property of a system that it will only apply a message once, even if it receives it multiple times. Formally, it is expressed as:

```
f(x) = f(f(x))
```

The function `f` is the behavior of the system as it receives a particular message. You could say that the function **is** the message, or at least the effect of the message on the state of the recipient.

The variable `x` is the state of the recipient before the message is received. And so `f(x)` is the state of the recipient afterward. The equation above simply says that a message is idempotent if the effect that it has on a recipient that already received the message is ... well, nothing. The state of the recipient is left unchanged.

Let's take a look at some messages to see if they are idempotent. For example, suppose that the message `f` is a sale: that Bob Jones just bought a latte for $5. The variable `x` is the state of the system before the message was received. This state captures the balance of each person's gift card.

```
x = {
  Sally Jane: $15,
  Bob Jones: $12,
  James Tan: $7
}
```

The value of `f(x)` would then be the state of the recipient after applying the message. Bob Jones' balance is decreased by $5, but everyone else's remains the same:

```
f(x) = {
  Sally Jane: $15,
  Bob Jones: $7,
  James Tan: $7
}
```

Now if we receive the message a second time, we apply it again:

```
f(f(x)) = {
  Sally Jane: $15,
  Bob Jones: $2,
  James Tan: $7
}
```

This message is not idempotent, because `f(x) ≠ f(f(x))`. And Bob is not happy about that.

In this example, the message `f` is a sale, and the state `x` is the set of card balances. This choice of message and state leads to a system that is not idempotent.

Idempotence is important in a distributed system, because the sender cannot know whether a message was received. So it must have the option of sending it again. Only if the message was idempotent can it know that resending it is safe.

### First move it, then process it

In the above example, because the state was the set of card balances, receipt of the message was the same as processing the message. The function f modified the state to calculate the new balance. Our overnight experiments on the Gift Card project showed us that this was a problem. We computed the balance of the card upon receipt of the message, and in doing so lost valuable information. We had to preserve that information to make our messages idempotent.

Our first step was to separate transmission of a message from its processing. After doing so, the state of the system was no longer about the result of the transactions. It was only about the receipt of those transactions. For example, the initial state of the server might be the following:

```
x = {
  MessagesReceived: [ 'abc', '123' ]
}
```

Now we are no longer talking about the balance of cards. We are only talking about the IDs of messages received. These IDs, in our case, were GUIDs generated in the restaurant by the originating client. Today I would make a different decision, but we'll get to that later. For now, let's continue with the GUIDs.

Suppose the client gives Bob's purchase the GUID '456'. It sends the transaction in a message which we will call `s` for "send".

```
s(x) = {
  MessagesReceived: [ 'abc', '123', '456' ]
}
```

Notice here that the recipient's state has changed to include the GUID of the sent transaction. If the sender is unsure whether the recipient received it, then it can retry the message. The result on the recipient's site would be:

```
s(s(x)) = {
  MessagesReceived: [ 'abc', '123', '456' ]
}
```

Notice that `s(s(x)) = s(x)`. The collection of messages received did not change, because the GUID was already present. In other words, the "send" message is idempotent. The processing of this transaction – reducing Bob's balance – can happen later.

The sender at this point has no idea whether the message was received, so it must continue to try indefinitely. That is, unless the recipient confirms receipt of the transaction. Let's make it do so in the form of a return message. For all the same reasons that we've already discussed, the confirm message must also be idempotent. We'll therefore take a look at the sender's state before this conversation started.

```
y = {
  MessagesToSend: [ '456' ]
}
```

This is the sender's outgoing queue. The sender periodically checks the queue and sends (or re-sends) the transactions it finds there. When the recipient receives that transaction, it responds with a confirmation message, `c`. The sender computes its state after confirmation:

```
c(y) = {
  MessagesToSend: [ ]
}
```

Just as with any message, the confirmation can be duplicated. If it is, the sender simply ignores it. Thus the state after receiving confirmation twice is:

```
c(c(y)) = {
  MessagesToSend: [ ]
}
```

Since `c(y) = c(c(y))`, the confirmation message is idempotent. The recipient can confirm repeatedly without causing any change on the sender. The protocol we chose here is simple: the recipient responds with a confirmation any time a transaction is sent, even if it had already been received. The recipient assumes that the sender will stop once it receives confirmation.

At this point, the two parties have reached an agreement. They both know that the transaction has been sent. However, the recipient doesn't know that agreement has been reached. This is what makes this problem different from the Two Generals Problem. The recipient doesn't need to know.

The processing of transactions was not idempotent. This lead to lost and duplicated transactions. But by separating the transmission of a transaction from its processing, we can impose idempotence where there was none, and fix those problems.
