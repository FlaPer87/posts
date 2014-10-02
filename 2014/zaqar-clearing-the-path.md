<!---
$"metadata"$
{
  "md": true,
  "title": "Zaqar's path going forward",
  "draft": false,
  "slug": "zaqar-path-going-forward",
  "tags": [
    "openstack",
    "zaqar",
    "messaging",
    "sqs"
]}
$"metadata"$
-->


Long time since I wrote my last post about Zaqar (ex Marconi) and I thought this one should be a summary of what has happened and what will, probably, happen going forward.

Let me start by sharing where Zaqar is in OpenStack.

At every end of the cycle - ~6 weeks before it ends, to be precise - every incubated project goes through a review process where things like API stability, integration with the OpenStack ecosystem, community etc - [full list](http://git.openstack.org/cgit/openstack/governance/tree/reference/incubation-integration-requirements.rst) - are revisited in order to evaluate the project and determine if it's ready to be part of the OpenStack integrated release. Despite Zaqar having met all those requirements, it was not accepted into the integrated release. The story is long and it's not the intent of this post to walk you through it. However, if you're interested in getting more details about what happened exactly please go here: [1st meeting](http://eavesdrop.openstack.org/meetings/tc/2014/tc.2014-09-02-20.04.log.html) [2nd meeting](http://eavesdrop.openstack.org/meetings/tc/2014/tc.2014-09-09-20.01.log.html) [3rd meeting](http://eavesdrop.openstack.org/meetings/tc/2014/tc.2014-09-16-20.02.log.html) [1st thread](http://lists.openstack.org/pipermail/openstack-dev/2014-September/044845.html) [2nd thread](http://lists.openstack.org/pipermail/openstack-dev/2014-September/045529.html) [Review](https://review.openstack.org/#/c/118592/).

One thing to get from the last review process, and definitely keep in mind, is that Zaqar is *ready* to be used in production environments. Technically speaking it met all the requirements imposed by the TC and as a project it's had a stable API for quite a bit already.

One of the discussions that happened during the last graduation review was related to whether Zaqar is a queuing service or a messaging service. To me, and as Gordon Sim mentioned in this [email](http://lists.openstack.org/pipermail/openstack-dev/2014-September/045560.html) there's no real difference between those 2 besides the later being a more generic term than the former. This discussion led to other discussions like whether things like `get-message-by-id` makes sense, whether keeping queuing semantics is necessary or even whether guarantees like FIFO should be kept.

All the above discussions have been interesting but I'd like to take a step back and walk you through a perhaps less technical topic but not less important. It's clear to me that not everyone knows what the project's vision is. So far, we've made clear what Zaqar's API goals are, what kind of service Zaqar is and the use-cases it tries to address but we haven't neither explicitly explained nor documented well-enough what Zaqar's scalability goals are, what guarantees from a storage perspective it gives nor how much value the project is putting on things like interoperability.

Zaqar has quite a set of features that give operators enough flexibility to achieve different scales and/or adapt it to their know-how and very specific needs. Something we've - or at least I have - always said about Zaqar - for better or for worse - is that you can play with its layers as if they were Lego bricks. I still think this is true and it doesn't mean Zaqar is trying to address *all* the use cases or making *everyone* happy. We want to give them flexibility to add functionality for additional use cases that aren't supported out of the box.. I know this has lots of implications, I'll dig into it a bit more later.

*Zaqar's vision is to provide a cross-cloud interoperable, fully-reliable messaging service at scale that is both, easy and not invasive, for deployers and users.*

It goes with no saying that the service (and team) has strived to achieve the above since the very beginning and I believe it does that, modulo bugs/improvements, right now.

Reliability
===========

Zaqar aims to be a fully-reliable service, therefore messages should never be lost under any circumstances except for when the message's expiration time (ttl) is reached - messages will not be around for ever (unless you explicitly request that). As of now, Zaqar's reliability guarantee relies on the storage ability to do so and on the service to be properly configured.

For example, if Zaqar is deployed on top of MongoDB - the current recommended storage for production - you'd likely do it by configuring a replica set or even a sharded cluster so that every message is replicated but if you use a single mongod instance, there's nothing the service can do to guarantee reliability. Well, there actually is, Zaqar could force deployers to configure either a replica set or a sharded cluster and die if they don't - we will likely force deployers.


Scalability
===========

Zaqar's design was thought at scale. Not all storage technologies will be able to perform the same way under massive loads, hence it's really important to choose a storage backend capable of supporting the expected user base.

That said, Zaqar also has some built-in scaling capabilities that aim to make scaling storage technologies easier and push their limits farther away. Zaqar's pools allow users to scale their storage layer by adding new storage clusters to it and balancing queues across them.

For example, if you have a zaqar+mongodb deployment and your mongodb cluster (regardless it is a replica set or a sharded cluster) reaches it's scaling limits, it'd be possible to setup a new mongodb cluster and add it as a pool in Zaqar. Zaqar will then balance queues based on the pools' weight across your storage clusters.

Although the above may sound like a quite naive feature, it is not. The team is aware of the limitations related to pools and the things left to do to make it less so. Let me walk you through some of these things.

One thing that you may  have spotted from the above is that pools work in a per-queue basis, which means there's no way to split queues across several storage clusters. This could be an issue for *huge* queues and it could make it more difficult to keep pools load balanced. Nonetheless, I still think it makes sense to keep it this way and here's why.

By balancing on queues and not messages, we're leaving the work of replicating and balancing messages to the technologies that have been doing it for years. This falls perfectly into Zaqar's will of relying as much as possible on the storage technology without re-inventing the wheel (nothing bad about the later, though). Though, I'd like to go a bit further than just "the service wants to rely on existing technologies".

Messages (data) distribution is not an easy task. I had the pleasure (?) to work on the core of these algorithms in the past and thankfully I know enough to want to be away from this while I can. For the sake of the argument, lets assume we add built-in message distribution in Zaqar. The way I think it would work is that we'd require a set of pools to exist so we can distribute messages across them. Then, the storage cluster itself will take care of the messages' replication. What this means is that deployers life would get more complicated since they'll be forced to create several storage clusters even for very basic Zaqar deployments in order to have messages replicated.

Now, to avoid forcing deployers to create several clusters, lets assume we implement message replication within Zaqar as well. This removes the need for deployers to create several clusters since even a single mongod instance - neither a replica set nor a sharded cluster is needed - would work perfectly as a pool since Zaqar would take care of replicating messages. Without getting into the technical details of how much logic we would need to move into Zaqar and the fact that we would be re-inventing things that had already been done elsewhere, I'd like us to ask ourselves why we should depend on external storage technologies if we already have everything needed to balance and replicate data ourselves? Lets not focus on the many tiny details but the overall picture. The service would be doing most of what's needed so why wouldn't we add the missing part and stop relying on external technologies?

All the above is to say that I'd rather spend time working on a swift driver - which we've discussed since the Icehouse summit - than working on having per-message balancing capabilities in Zaqar. Swift knows how to do this very well and it'd make perfectly sense to have zaqar on top of 1 swift pool and just scale that one. I'm not saying mongodb is not good for this job, although we (Zaqar team) should work on documenting better how to use a sharded mongodb cluster with Zaqar.

In other words, Zaqar's scaling focus most be balanced between the API, the guarantees it provides and the storage technology. I believe most of the focus should be invested in the later. The more pools you add, the more resources you'll need and the more complicated your deployment becomes.

There are definitely some issues related to balancing queues - a.k.a buckets, containers, toy's boxes, drawers, etc - and there's a very good reason why swift doesn't do it for containers. One of the things I'd like to see improved is the balancing algorithm Zaqar uses. As of now, it has a very naive and simple algorithm based on weights. The thing I like about this algorithm is that it gives the deployer enough control over the available clusters and the thing I don't like is that it gives the deployer enough control over the available clusters ;). I'd like to also have an algorithm that would make this balancing easier and that doesn't require the deployer to do anything besides adding new pools.

Again, I think pools are great but we should strive to scale the storage layer as much as possible before adding new pools.

Interoperability
================

Probably hard to believe but I think this is the most difficult task we have had and we will ever have. The project aims to preserve interoperability across clouds and to be able to do so, the features exposed through the API must be guaranteed to work on every cloud regardless of the storage technologies. As much as I'd like this to be true and possible, I think it's not and I also think this applies to every OpenStack service.

We cannot force deployers to deploy Zaqar in a way it'll preserve interoperability across clouds. Deployers are free to configure Zaqar (and any service) as they wish and install whatever driver they want (even non-official ones). If a deployer configures Zaqar on top of a single mongod instance or simply changes the write concern, Zaqar won't be reliable and message could be lost if the node goes down, hence the guarantee will broken.

In addition to the above, optional features, third-party drivers and custom settings - smaller maximum message size, for example - are neither part of this guarantee nor the team can do anything about them.

What we can do, though, is to make sure the features exposed through the API are supported by all the drivers we maintain, work on a set of deployment scenarios that would guarantee interoperability across clouds and make sure the default values of our configuration options are sane enough not to require any deployment to change them.

I'm sure there's a lot more to interoperability than what I'm stating here. What I want to get to is that we strive to make it easier for the service and deployers to preserve interoperability but I believe it cannot be guaranteed at 100%

As you may have noticed, Zaqar has been under a fire storm for the last ~4 weeks, which has been both exciting and stressful - nonetheless, please keep the fire coming.

Many people have many different expectations about Zaqar and it's impossible to make everyone happy so, if you're part of the unhappy group of people (or end up there), I'm sorry. The team has a clear vision of what this service has to provide and a mission to make that happen. I'm sure the service is not perfect and that you don't need to dig deep to find things that should work differently. If you do, please let us know, we're always looking forward to constructive feedback and making the service better.

Messaging is a broader enough field to cover tons of different tones of grey. While Zaqar is not trying to hold them all, it is definitely trying to provide enough to make the service worth it and suffice the use cases it has.