<!---
$"metadata"$
{
  "md": true,
  "title": "How and When - OpenStack Services Integration",
  "draft": false,
  "slug": "openstack-services-integration",
  "tags": [
    "openstack",
    "distributed",
    "python"
  ]
}
$"metadata"$
-->

As many of you know, OpenStack's is a fully distributed system. As such, it keeps its services as decoupled as possible and tries to stick to most of the distribution paradigms, deployments strategies and architectures. For example, one of the main tenets throughout OpenStack is that every module should be using 'Shared Nothing Architecture'. The Shared Nothing Architecture principle states that each node should be independent and self-sufficient. In other words, all nodes in a SNA are completely isolated from each other in terms of space and memory.

There are other distribution principles that are also part of OpenStack's tenets, however, this post is not about what principles OpenStack as a whole tries to follow. What I would like to discuss in this post is how OpenStack sticks together such a heavily distributed architecture and makes it work as one. The first thing we need to do is evaluate some of the integration methods that exist out there and how they're being used within OpenStack.

The integration of a distributed system relies on the capability of its components to communicate between them. This communication can happen in several ways and could go through different protocols. For example, we could integrate 2 services by sharing a file that contains the data we want to send from one service - source - to the other - destination. Although it seems obvious that there are many drawbacks from using this approach, it is still useful in scenarios where things like databases, rpc libraries and messaging, which are good replacements for the file-based method, can't be used.

There are many things besides communication that have to be taken under consideration. Nonetheless, establishing a channel between distributed systems is the first step to towards an integrated system.

Just like everything else, each one of this integration methods have some benefits and drawbacks. Some of them are consistent whereas others are more reliable and scalable.

I mentioned above that databases, rpc libraries and messaging could be good replacements for the file-based integration approach. Lets dig a bit more into those.

Databases
=========

I'd dare to say databases are the most common way of integration nowadays. As mentioned before, integrating 2 or more services is about making those nodes communicate with each other, regardless the scope of that communication. The services able to communicate with each other will use that ability to send data from point A to B, regardless what that data is considered to be.

The data travelling between those services could be anything: events, messages, notifications, database records etc. It doesn't matter, at the very end it's data and what matters is how it is being generated, how it's being sent, how it's being serialized and how it's being consumed.

The reason I mentioned all that is because databases conceptually are not message brokers, although they allow 2 or more services communicate and share data between them. Databases are collections of structured - and don't play the unstructured NoSQL card here - data. This data is usually organized and can be created, read, updated and deleted at any time.

It is pretty common to see different services relying on a database as a way to share data. It is even more common to see that happening for a horizontally scaled service. Some people would argue saying that nodes of the same service that rely on a database are not actually being integrated by it, although they are. If a service relies on a database, it means it uses it to store information that will be useful to other nodes of the same service running in parallel. Otherwise, each node of that service would be a rogue, isolated instance and that would break the consistency of the whole distributed system.

This is the first level of integration that services throughout OpenStack use. Most of the services depend on a database. However, none of them share the database access with services that doesn't belong to their cluster. For example, Nova instances won't ever access Cinder's database directly, instead, they'll go through Cinder's API service. The motivation for this is pretty simple, Nova doesn't have any knowledge of how volumes should be handled, nor what state they are in or when their state should be changed. That's something Cinder has to take care of.

Nevertheless, different nodes of the same service do share access to the same database. For example, `cinder-volume`, `cinder-scheduler` and `cinder-api` access the same database concurrently to operate on volumes' data, among other things.

Remote Procedure Calls
======================

As I already mentioned, I believe databases are the most common way to integrate services nowadays, many people do it without even knowing they are. However, as far as OpenStack is concerned, I believe RPC is the most used method. Almost 90% of the integrated projects - those that are part of OpenStack's release cycle - rely on RPC calls to communicate with other services.

But, what is RPC exactly?

Remote Procedure Call is a way of inter-process communication that allows clients to trigger the execution of subroutines in remote locations. There are many different implementations of it, although there's an - expired - RFC for it. Different patterns have been created based on those implementations.

RPC subroutines can do anything. In many cases, they're responsible for storing and retrieving data. In other cases, though, they just execute code and return the result to the caller. Likewise, the channel through which RPC calls are sent could be anything - message brokers, raw TCP connections etc - as long as it's possible to pack and send the message through it[0].

RPC has many benefits that databases don't. For example, it's possible to make synchronous calls and wait for responses whereas with databases the data transmission is always asynchronous. RPC also makes it easier for services to isolate operations and communications. However, just like databases, RPC is a very tightly coupled protocol that requires both endpoints to agree on a structure. This brings consistency and 'predictability' to the protocol at the cost of making it less flexible and obviously less decoupled.

Throughout OpenStack, modules using RPC rely on a messaging library - oslo.messaging - that takes care of the message serialization, transport communication and message acknowledgement. As for the time of this writing, the library has support for 3 different drivers - rabbit, qpid and zmq - that handle the underlying message bus this library sits on top of.

Some projects like Nova, Cinder and Neutron use RPC calls heavily. Almost everything that happens in those projects is triggered by RPC calls. For example, a volume creation request in Cinder is first sent from the `cinder-api` service to the `cinder-scheduler` which will then pick one of the `cinder-volume` nodes that are available and forward the request to it. All this happens in an asynchronous fashion.

Throughout OpenStack, asynchronous calls (cast) are used most of the time. One of the benefits behind this is that services can handle more load since they're not blocking on every call, therefore *-api services can return a response back to the client in less time.

[0] Notice we're not taking reliability, scalability under consideration here but just the ability to of the message channel to take the message from the sender to the receiver

Messaging
=========

NOTE: This section is not about message brokers. It refers to the use of messages as an atomic unit for sharing data between services, regardless they're homogeneous or not.

Messaging is perhaps the most decoupled method of integration. It's based on messages that travel through a channel which gets them from the producer to the consumer. Services sending and consuming messages don't necessarily need to agreed on a structure to use. Furthermore, consumers are not suppose to consume all messages, it all depends on the messaging pattern being used.

Messaging - loose coupling to be more precise - gives flexibility and scalability to the cost of being more complex in terms of implementations. Although services being integrated are not required to agreed on a structure, they do expect to get a message they can read the data from. Furthermore, the message channel - whatever it is - may need to have support for message routing, message transformation and message filtering among other things. All this is mostly coupled to the architecture but not to the services that are part of it.

Something interesting about messaging is that it can be the base for other integration methods. For example, it is possible to send messages containing RPC requests. In order to do this, though, it is necessary to apply all RPC requirements to messaging, for instance, both parts - producer and consumer - will need to agreed on the message structure and type.

Within OpenStack, messages are mostly used by the RPC library - oslo.messaging - itself. It packages all RPC requests into messages that are sent to a message broker which then forwards them to the consumers. There are cases, though, where messaging is used in a almost fully decoupled and asynchronous way. Notifications, for instance, are messages sent by OpenStack services **hoping** there'll be a consumer ready to process them. These notifications contain information about events that had happened at a very specific moment. In Glance, for example, when an image is downloaded a `image.download` notification is sent and it'll hopefully be consumed. Whatever it is that happens to that notification, Glance simply doesn't necessarily care about it. However, Ceilometer is a good example - perhaps the only one at the time of this writing - of a service interested in those notifications. It consumes all these events to meter resources usage and to allow users to bill based on that information.

It is now clear that messaging is heavily used in OpenStack and that most of the messages sent between homogeneous services are RPC calls. This calls travel through the message broker picked in the form of atomic messages and they are processed - at least most of them - asynchronously. The asynchronous nature of OpenStack's interoperability helps keeping developers focused on making distributed nodes faster, more scalable and more reliable.

Wrapping up
===========

This post covered 3 different integration methods. It also showed how they're used throughout OpenStack and how they're mixed together to reach great levels of integration. The post also covered some of the benefits of each one of the methods over the others and touched some of the drawbacks of each method as well.

One thing to keep in mind, though, is that OpenStack couldn't have been implemented in any other way. The fact that it relies on such a heavily distributed architecture has helped the project to succeed. The mixed integration styles it supports allow the project to have logically services - as opposed to functional ones - distributed in several nodes. Furthermore, it allows OpenStack to scale massively and dynamically. OpenStack's limits are set by other areas.

However, it's true that not everything is perfect and that there's definitely room for improvement. For instance, the fact that most of the operations throughout OpenStack rely on a message broker is not funny. We all know message brokers are hard - impossible - to scale. It is easy for message brokers to become a SPOF in the architecture, which means that a big part of your deployment will be fucked up if one of them would go down. I believe this is something that per-to-per protocols - amqp 1.0, for example - could alleviate.

I'll maybe cover this in one of my next posts