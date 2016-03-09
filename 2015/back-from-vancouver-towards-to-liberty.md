<!---
$"metadata"$
{
  "md": true,
  "title": "Back from Vancouver, towards Liberty",
  "draft": false,
  "slug": "back-from-vancouver-towards-liberty",
  "tags": [
    "openstack",
    "glance",
    "zaqar"
]}
$"metadata"$
-->

Fever is gone (actual fever), energies are coming back and the next six months are blurried by all the things we have ahead. Besides the fever, I'd say this is what a normal summit feels like. Or well, what the feeling after the summit is like.

Just like in every other summit, we had fun, we discussed things, we brainstormed, we (kinda) fought, we enjoyed the excitement of being there and we came back with plans that we consider are a common ground between what we and others think is best for our projects and our community.

Here's a brain dump (not all pages were dumped) of what the summit brought me:

Zaqar (Messaging Service)
=========================

If you've followed Zaqar's drama, you know it's gone through several ups and downs (look for previous post and m-l discussions). Short before the summit, it went through another [down](http://lists.openstack.org/pipermail/openstack-dev/2015-April/061967.html). The community response turned out to be great and the good news is that it's [staying](http://lists.openstack.org/pipermail/openstack-dev/2015-May/064739.html).

Cross-project user-facing notifications
---------------------------------------

https://etherpad.openstack.org/p/liberty-cross-project-user-notifications

Besides brainstorming a bit on what things should/should not be notified and what format should be used, we also talked a bit about the available technologies that could be used for this tasks. Zaqar was among those and, AFAICT, at the end of the session we agreed on giving this a try. It'll likely not happen as fast as we want but the action item out of this session was to write a cross-project spec describing the things discussed and the technology that will be
adopted.

Heat + Zaqar
------------

The 2 main areas where Zaqar will be used in Heat are Software Config and Hooks. The minimum requirements (server side) for this are in place already. There's some work to do on the client side that the team will get to asap.


Sahara (or other guest agent based services) + Zaqar
----------------------------------------------------

We discussed 3 different ways to enable services to communicate with their guest agents using Zaqar:

1) Using notification hooks: Assuming the guest agents doesn't need to communicate with the controller, the controller can register a notification hook that will push messages to the guest agent.

2) Inject keystone credentials: The controller would inject keystone credentials into the VM to allow the guest agent to send/receive messages using Zaqar.

3) PreSigned URLs: The controller injects a PreSigned URL in the controller that will grant the guest agent access to a specific tenant/queue with either read or read&write access.


Hallway Discussions
-------------------

We had a chance to talk to some other folks from teams like Horizon that were also interested in doing some actual integration work with Zaqar as well. Not to mention that some other folks from the puppet team showed interest in helping out with the creation of puppet-manifests.


Glance (Image service)
=======================

V1 -> V2 -> V3... wait, WHAT?
-----------------------------

We've been talking about killing v1 for several cycles. For better or for worse, we haven't been able to do so. We still want, though. Nonetheless, the big news is that there'll be an experimental V3 of Glance API. You might be wondering what's wrong with Glance's team but hold your breath for a bit, we didn't pull this out of .... a black box.

Back in Atlanta, Alexander Tivelkov and other folks proposed something called Artifacts. Artifacts is - in a very poor definition - a data sets model. An object based API that describes resources that can be as simple as an image or as complicated as a template with dependencies, versions and other more complex features.

They have been working on that since then but there was some push back from the community during Kilo. Part of the community (included myself) felt that Glance was not the right place to do it. To some extent this was related to Glance being a simple image service and Artifacts were way more than that. Without going into the details of why and how these discussions happened, we found ourselves discussing again, at the Liberty summit, what the future of Glance would be. The resolution of this discussion is summarized in [this email](http://lists.openstack.org/pipermail/openstack-dev/2015-May/064603.html).

In other words, the work around artifacts will be merged in Glance's code base and it'll be exposed as part of an **experimental** V3 API. Or, as Jesse Cook put it in that thread, Artifacts is the *technical implementation* of Glance's V3, which is no more than an object API.

Now, what's important about the above is not the experimental V3 but the radical change in the type of API that Glance will expose. It'll go from being an *images* API to being an *objects* API. The resource type, properties and API are completely different.

The images API - v1 and v2 - will still be supported and the transition to v3 will not happen in L. It'll be material for the M summit.

I'll be dedicating time myself to this migration. That is, we'll have a dedicate set of people working on moving images to artifacts in the future and making sure existing deployments remain untouched. I'll be also working closely with the DefCore team to provide the required info about this transition.

However, I'd like to encourage people, at least during the L cycle, to keep considering Glance as an Image Service until the experimental V3 API has been released and the team decides to completely moves towards a fully objects API.

This change is huge and it'll require time, lots of tests, even more discussions and some other changes that are not technical at all. Fun times ahead.

CIS -> SearchLight
------------------

The CIS (Catalog Index Service) side of Glance announced during the summit that they'll split out off of Glance into its own project to satisfy not only Glance's needs but many other project's needs. Therefore, expect glance to shrink a bit from this side but don't be to happy, it'll get fatter as soon as the v3 machinery gets going.

CIS folks have done an amazing job and they deserve all the best and glory for it.

Misc
----

Definitely not least important but certainly less controversial.

We also had sessions for topics like:

1. Optimize image's cache ([link](https://etherpad.openstack.org/p/liberty-caching))
2. Image's uploads ([link](https://etherpad.openstack.org/p/liberty-glance-reliable-upload))
3. Support for OVF ([link](https://etherpad.openstack.org/p/OVF-support-in-glance))
4. Research on a NoSQL database driver ([link](https://etherpad.openstack.org/p/liberty-glance-nosql-backend))
5. Image Signing and Encryption ([link](https://etherpad.openstack.org/p/liberty-glance-image-signing-and-encryption))

If you're interested in any of the above, please, do not hesitate to jump into *#openstack-glance@freenode` and ask about them.

Oslo
====

Unfortunately, I wasn't able to attend as many Oslo sessions as I'd have liked. The discussions above and other commitments took part of the time I had scheduled for Oslo and well, there are always overlaps.

However, there are many interesting things for Oslo and I highly encourage you to look into [the etherpads](https://wiki.openstack.org/wiki/Design_Summit/Liberty/Etherpads#Oslo) and ask questions.

Personally, I'll dedicating more time to oslo.messaging during Liberty than other parts of it. We'll see.