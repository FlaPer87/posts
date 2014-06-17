<!---
$"metadata"$
{
  "md": true,
  "title": "Marconi to AMQP: See you later",
  "draft": false,
  "slug": "marconi-amqp-see-you-later",
  "tags": [
    "amqp",
    "python",
    "openstack",
    "marconi"
  ]
}
$"metadata"$
-->


In the last couple of weeks, Marconi's team has been doing lots of research around AMQP and the possibility of supporting traditional queuing systems. I have wanted this myself for quite some time. However, after digging more into what's needed and what supporting traditional brokers means for Marconi, I don't believe supporting such systems makes much sense anymore. Here's why.

Why is it important?
====================

It is, right?

One of Marconi's goals since the very beginning has been to fit into your stack. What that means is that you shouldn't need to change your application tier in order to use Marconi. If you've deployed MongoDB, you can simply install Marconi and point it to your MongoDB replica set. The same thing should happen if you've a traditional broker installed. We wanted you to be able to put Marconi on top of that broker.

The above boils down to adoption, we want people with "Messaging Needs" to adopt Marconi if it is a good option for their use case. This is, in my opinion, the main reason we decided to go down this road and work on the support for traditional queuing systems.

Another motivation is performance. Traditional queuing technologies are known and designed to be fast. This doesn't mean Marconi isn't, what it means is that depending on the storage technology in use, Marconi will perform differently.

The third motivation is something that Marconi brings to traditional brokers. With Marconi, it's possible to add per-queue sharding capabilities to traditional brokers. By using Marconi's pools and flavors it is possible to create separate clusters of storage that will be used cooperatively based on the pool settings.

Current API
===========

When the team started working on Marconi, it was decided to work on a unified api that would eventually support several message patterns. The API was developed based on feed semantics, which is very similar to the way other services like Azure Queues and SQS do it.

The [API](https://wiki.openstack.org/wiki/Marconi/specs/api/v1) was designed to be browsable but without sacrificing the "messaging" feel of every queuing system. [New things](https://wiki.openstack.org/wiki/Marconi/specs/api/v1.1) are being added to make it more message based without sacrificing browsability. Note that it is important for the API to be HTTP-friendly.

Even though Marconi has been designed to support multiple transports, we consider the HTTP API the main product and that's were most of the focus has been put on in the last year and a half. So far, the feedback about the API has been good. Nevertheless, the team knows the API is not perfect and there will be changes.

What's up with AMQP?
====================

Let me start by saying there's no such thing as fully-non-opinionated software. Whatever it is the software is selling, it needs to have an opinion. As for Marconi, the API *is* the product, which means we *care* about its form, semantics and features. Moreover, we care about it being consistent, simple and interoperable.

After discussing the [unified API plans](http://lists.openstack.org/pipermail/openstack-dev/2014-June/037053.html) and digging into both AMQP 1.0 and [AMQP 0.9](http://lists.openstack.org/pipermail/openstack-dev/2014-June/037177.html) it's quite clear that many things would need to be changed to support queuing technologies that are based in any of these protocols. Moreover, these changes would be needed even to support technologies that have a streamed API.

**SIDE NOTE:** I don't think these API changes are the real problem, though. The real problem is how those changes will affect the existing API, where they would live and how they fit into Marconi's goals. Kurt Griffiths offered 3 possible plans in this [thread](http://lists.openstack.org/pipermail/openstack-dev/2014-June/037053.html) and I believe our only option as of now is C. Option A basically breaks the interoperability bit of the API whereas option B is basically a different product. With C, the number of storage technologies that Marconi will be able to support won't be high but I don't think this is a bad thing.

Back to AMQP. We wanted Marconi to be able to support brokers capable of speaking AMQP 1.0 - I won't go into the reasoning about this but I'll definitely say that the fact it's a standard played a big role here. Before I go into the details explaining whether AMQP 1.0 is a good fit or not for Marconi, I'd like to highlight that most of this research was done by [Victoria Martínez de la Cruz](http://vmartinezdelacruz.com/). Tomasz Janczuk did some experiments with AMQP 0.9, the results were published in this [thread](http://lists.openstack.org/pipermail/openstack-dev/2014-June/037177.html).

Lets get to it.

Store and forward
-----------------

Marconi relies on a store-forward message delivery. This means Marconi has no support for peer-to-peer communications. You send a message to Marconi, the message is then stored in the storage layer and it will then be available for consumption.

AMQP 1.0 is a protocol for message exchange between processes. Although the specification covers the presence of an intermediate process (a broker), it doesn't specify what functionalities such a broker should offer. Store and forward is also covered by the AMQP specification. For instance, Marconi could connect to an AMQP 1.0 capable broker to have store-forward message delivery. Since the AMQP 1.0 specification covers some of this areas but doesn't explicitly define how each one of them should be implemented, there's room for non-standard implementations on each broker, which means the interoperability of this storage driver could be broken.

While you go through the remaining points, remember that AMQP is just the protocol being used to talk to the broker, which means we need all the features to be supported not just by the protocol but the broker as well.

Message access by ID
--------------------

In AMQP 1.0, the message id is an optional field and it must be set by the producer. The producer is also responsible for enforcing the id uniqueness. This is actually fine, it'd be sad to have to depend on an external service to generate ids but it's probably not a big deal.

Unfortunately, though, in AMQP1.0 it's not possible to access messages directly by using the message-id or any other field. This makes it impossible to support random access to messages in the queue, which is one of Marconi's features. I won't discuss the usefulness of this feature here, although there was a recent [discussion](http://lists.openstack.org/pipermail/openstack-dev/2014-May/036131.html) arguing it.

Different Acknowledgement models
--------------------------------

AMQP's acknowledgement does not need to be immediate. However, it does need to go through the same session used to get the message. Since Marconi is not the final consumer of the message, it can't acknowledge it until the user does it through Marconi's API. This is an issue already because Marconi doesn't have support for persistent connections, which means a message may be pulled from one node and acknowledged on a different node.

A message that has been pulled out the queue is in an acquired state, which means no other consumer should get it. This is good except for the part where the protocol doesn't allow you to unlock a message based on its id. This doesn't play nice with Marconi's claim workflow. In Marconi, a message - or a set of messages - can be claimed and then deleted or released.

No support for queues
---------------------

AMQP 1.0 does not explicitly define what a queue is. It defines the state model for sending and acquiring messages but it doesn't mandates how it should be defined . Depending on how the broker implementation of AMQP 1.0 works, this feature may or may not be supported.

As of the time of this writing, queue's are a first-citizen resource in Marconi and I don't think that'll change in the near future. There have been some discussions and plans around getting rid of queues. Nonetheless, the team decided to keep them around at the last summit based on the feedback provided by the members of the community attending the sessions.

Conclusion
----------

To summarize, the points we've mentioned are:

* Store and forward
* Message access by ID
* Different Acknowledgement models
* No support for queues

There are other very valid points that have not been mentioned in the above list. In order to keep this post readable, I just highlighted the ones that I considered more relevant and that have a bigger impact in the current API. Based on the above mentioned points I don't believe the trade-off between changes required and things that would be gained by supporting traditional messaging systems is fair at all. This all had me thinking that we're probably trying to support AMQP in the wrong layer. What happens if instead of having support for traditional brokers we just add a new AMQP transport?

We'll definitely reconsider this at the K summit once we'll figure out what the API 2.0 should look like. Although this doesn't mean it'll be supported nor that the API will be overhauled, it does mean the team is always revisiting existing technologies and open to expand the project to the best of its possibilities.

One more thing. This post doesn't claim the Marconi API is perfect, what it states is that based on the current supported API, it's not possible to support traditional messaging systems. Whether it'll be possible to do so in the future or not remains to be seen.

I'd like to give a final thank to [Victoria Martínez de la Cruz](http://vmartinezdelacruz.com/) and Tomasz Janczuk for the work, passion and time dedicated to this analysis.