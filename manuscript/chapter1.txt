# Why Historical Modeling?

At the beginning of this century, a friend and I, working on building a web development business, attended a day-long conference on networking equipment and services. We saw routers designed to balance the traffic in any given network, and testing equipment designed to saturate that network and bring it down. We learned about the OSI 7-layer stack, and where different vendors and components fit in and supported that model.

This conference was eye-opening for me. But not because of all the innovative solutions that I saw before me. My eyes were opened to the absurdity of the problem itself. It seemed that we were putting far too much effort, money, time, and brainpower into solving a single problem: moving data from where it was to where it is needed.

What occurred to me is that we have invested a huge amount of effort into making sure that information flows from one computer to another. For example, it flows from a web browser on an end-user’s computer to a database in a datacenter halfway around the world. Along the way it passes through network switches, load balancers, web farms, and application servers before it finally arrives.

After the data takes that long arduous journey, that same user will refresh the page and pull all of that data back again.

How absurd this all looked to me! Surely, we can find a model of computing that lets the data reside where it is most needed. If information originates from a user, why must he wait for it to cross all those layers only to be re-rendered and displayed right back to him? If he wants to collaborate with someone else, why can’t they exchange just the common subset of information between themselves, rather than mixing their data together in a central database?

This observation set me on a journey to discover a model of computing that uses data where it resides, rather than relying upon the interactions between connected systems. I found hidden assumptions that we impose upon ourselves when building distributed systems: a client must wait for a response from a server in order to get the most recent data; a record can be identified by a key generated upon insertion; a URI – despite containing a host name – can identify a resource independent of its location. And the biggest assumption of them all: changes to individual resources can appear sequential, even though the aggregate changes asynchronously and in parallel.

The systems that we had always built depended upon these assumptions. And so the protocols and languages supporting those systems had to ensure that these assumptions were true. But they weren’t. And so those protocols and languages were unreliable. Which leaves vendors at a networking conference selling plugs for a leaky dyke.

One-by-one I removed these assumptions until I was left with a fundamental core: a set of laws that govern information, how it interrelates, how it flows, and how it can be traversed. Then piece-by-piece I constructed a model of computing that relies only upon these fundamental laws. The result has clearly defined behaviors, whether a system is connected or disconnected, synchronous or asynchronous. It makes no promises that it can’t keep.

The model removes concepts that we used to take for granted. Concepts like changing the value of a variable over time. Concepts like locking a resource so that it cannot be changed by two actors simultaneously. Concepts like looking up the value of a resource by its ID. The challenge, then, is to build software systems under a new set of rules. Can the systems that we need be built without the assumptions that we are used to? 

The answer, I discovered, is no. You still need to return to the old regime to solve specific problems. But amazingly, the larger portion of the software systems that you need can be constructed without them. These portions can be built using only the core rules of information: Historical Modeling. And when you do so, you reap the rewards of reliability, scalability, and autonomy.

In this book, I will introduce you to the rules of Historical Modeling. I will demonstrate that these rules are easily satisfiable in a distributed system. I will show how you can use Historical Modeling to build the larger portion of a software system, and how to integrate that portion with the old regime when necessary. But before I do that, I must first expose the hidden assumptions under which we have traditionally written software.
