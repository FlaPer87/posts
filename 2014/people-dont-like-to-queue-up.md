<!---
$"metadata"$
{
  "md": true,
  "title": "People don't like to queue up",
  "draft": false,
  "slug": "people-dont-like-to-queue-up",
  "tags": [
    "python",
    "openstack",
    "marconi"
  ]
}
$"metadata"$
-->

Lately, I've been thinking a lot about what queues represent in Marconi and how they could be improved. It's not a secret that I'm not a fan of the way we're currently thinking about queues. Throughout Marconi, a queue is a first-citizen resource that is used to group messages under a specific context. Other than that, a queue has no other responsibility and it doesn't do anything useful from a user perspective.

In addition to the aforementioned, queues are not as lazy as I would like them to be. In order to post a message, a queue must exist, which means it's necessary to create the queue before posting a message.

If we look at how some messaging tools and protocols work nowadays, we'll see that many of them have shift away from this concept to a more message oriented one. For instance, qpid-proton, which is implemented on top of amqp 1.0, needs no queue to send a message. Another good example is zmq, which doesn't have the concept of queue either.

At this point, you could already guess what my proposal is:

Let's get rid of queues
=======================

In my opinion, queues have reached the end of they're lives. The idea of having a first-citizen resource that piles up a bunch of messages and allows clients to subscribe to them is not affordable nor necessary anymore.

From a user perspective, the most important resource in a queuing system is the message. The user expects the queuing system to be reliable in terms of message delivery not queue's existence. Whether the queue plays an important role in this whole process is not as important to the user as being able to send and receive messages fast and reliably.

This is what keeping a queue means in Marconi:

**API:** Queue's have their own set of RESTFul endpoints. In order to create a queue, you need to send a `POST` request to the queue's endpoint. The same thing is necessary to get the metadata of that queue.

**Storage:** As of now, Marconi has just database like storage drivers. Regardless of what the driver is, Marconi can't simply trust that the queue exists. Therefore, it is necessary to verify the queue existence *before* doing any operation on the message. Depending on the storage, this may require either an extra query, an extra join or an extra call.

What If we just think about Topics?
===================================

I'd like to introduce the knowledge of `Topic` in Marconi. A topic describes the context a message belongs to - basically the way queues did. The difference between a topic like the one proposed here and queues is that the former ought not to be a first-citizen resource -  instead it should be an attribute of the message - whereas the later is a first-citizen resource that acts as a categorizer of messages.

Note that this does not mean that drivers for 'queue-aware' system can't be implemented. The difference is that the concept of queue would be lazily hidden behind a topic.

What about queue's metadata?
============================

When queue's metadata was first introduced in Marconi, the idea was to allow users to add custom properties for that queue. Behind that generosity hides the idea of re-using the queue metadata to add private properties like per-queue limits, flavors etc.

I don't think this necessarily needs to go away. The idea would be to make the queue resource - now called topic - completely optional. It would still be possible to query it, create it and delete it but it won't play such an important role as it does now.

What about existing code?
=========================

To be fully honest, I don't think this change would require any code changes in the client side. The existing library requires the user to have a queue instance to which the messages are posted. Those `Queue`s instances could be made lazy without impacting the way the user code works.

For instance, this example won't need to be changed:

```python
    cli = client.Client(URL)
    queue = cli.queue(queue_name)
    queue.post(messages)

    for msg in queue.messages(echo=True):
        print(msg.body)
        msg.delete()
```

Although it would also be possible to write it as:

```python
    cli = client.Client(URL)
    cli.post(messages, topic=queue_name)

    for msg in cli.messages(topic=queue_name, echo=True):
        print(msg.body)
        msg.delete()
```


Conclusion
==========

This post doesn't introduce anything that hasn't been heard before. The concept of topics has been around for quite some time and it's already been adopted by various of the existing queuing systems - it may mean something different in some cases. Many of those systems are still bound to queues but others - like the ones mentioned in this post - have abandoned it. I think this is the right moment for Marconi to make such a choice without brutally disrupting existing deployments.

I'm sorry for making this post short, rough and not as detailed as I'd have liked. This is just a proposal and it requires way more discussion and thoughts but I wanted to throw it out here.

Thoughts?