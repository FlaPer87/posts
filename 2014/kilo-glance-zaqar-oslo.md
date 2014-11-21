<!---
$"metadata"$
{
  "md": true,
  "title": "What's coming in Kilo for Glance, Zaqar and Oslo?",
  "draft": false,
  "slug": "kilo-glance-zaqar-oslo",
  "tags": [
    "glance",
    "oslo",
    "python",
    "openstack",
    "zaqar"
  ]
}
$"metadata"$
-->

As usual, here's a write up of what happened last week during the OpenStack Summit. More than a summary, this post contains the plans we discussed for the next 6 months.

Glance
======

Lots of things happened in Juno for Glance. Work related to [artifacts](https://etherpad.openstack.org/p/MetadataRepository-ArtifactRepositoryAPI) was done, [async workers](https://blueprints.launchpad.net/glance/+spec/async-glance-workers) were implemented and [glance_store](https://blueprints.launchpad.net/glance/+spec/create-store-package) was created. If none of these things excite you, I'm sorry to tell you that you're missing the big picture.

The 3 features mentioned above are the bases of many things that will happen in Kilo. For long time, we've been waiting for async workers to land and now that we have them we can't but use them. One of the first things that will consume this feature is [image introspection](https://blueprints.launchpad.net/glance/+spec/introspection-of-images), which will allow glance to read image's metadata and extract useful information from them. In addition to this, we'll messing with images a bit more by implementing basic support for [image conversion](https://blueprints.launchpad.net/glance/+spec/basic-import-conversion) to allow for automatic conversion of images during uploads and also as a manual operation. There are many things to take care of here and tons of subtle corner cases so please, keep an eye on these things and help us out.

The work on [artifacts](https://etherpad.openstack.org/p/MetadataRepository-ArtifactRepositoryAPI) is not complete, there are still many things to do there and lots of patches and code are being written. This still seems to be the path the project is going down to for Kilo to allow more generic catalogs and support for storing data assets.

One more thing on Glance, all the work that happened in glance_store during Juno, will finally pay off in Kilo. We'll start refactoring the library and it'll likely be adopted by Nova in [K-2](https://etherpad.openstack.org/p/kilo-nova-glance). Noticed I said likely? That's because before we get there, we need to clean up the messy glance wrapper nova has. In that same [session](https://etherpad.openstack.org/p/kilo-nova-glance) we discussed what to do with that code and agreed on getting rid of it and let nova consume glanceclient directly, which will happen in kilo-1 before the glance_store adoption. Here's the [spec](https://review.openstack.org/#/c/133485/).

Zaqar
=====

When thinking about Zaqar and Kilo, you need to keep 3 things in mind:

1. [Notifications](https://review.openstack.org/#/c/129192/)
2. [Persistent Transport](https://etherpad.openstack.org/p/kilo-zaqar-summit-persistent-transports)
3. [Integration](https://etherpad.openstack.org/p/kilo-zaqar-summit-integration-with-services) with other services

[Notifications](https://review.openstack.org/#/c/129192/) is something we've wanted to work on since Icehouse. We talked about them back in Hong Kong, then in Atlanta and we finally have a good plan for them now. The team will put lots of efforts on this feature and we'd love to get as much feedback as possible on the implementation, use cases and targets. In order to implement notifications and mark a fresh start, the team has also decided to [bump the API](https://etherpad.openstack.org/p/kilo-zaqar-summit-v2) version number to 2 and use this chance to clean up the technical debt from previous versions. Some of the things that will go away from the API are:

- Get messages by id
- FIFO will become optional
- Queue's will be removed from the API, instead we'll start talking about topics. Some notes on this [here](http://blog.flaper87.com/post/people-dont-like-to-queue-up/).

One of the projects goal is to be easily consumed regardless of the device you're using. Moreover, the project wants to allow users to integrate with it. Therefore, the team is planning to start working on a [persistent Transport](https://etherpad.openstack.org/p/kilo-zaqar-summit-persistent-transports) in order to define a message-based protocol that is both stateless and persistent as far as the communication between the peers goes. The first target is websocket, which will allow users to consume Zaqar's API from a browser and even using a library without having to go down to raw TCP connections, which was highly discouraged at the summit. This falls perfectly in the projects goals to be easily consumable and to reuse existing technologies and solutions as much as possible.

Although the above two features sound exciting, the ultimate goal is to integrate with other projects in the community. The team has long waited for this opportunity and now that it has a stable API, it is the perfect time for this integration to happen. At our [integration session](https://etherpad.openstack.org/p/kilo-zaqar-summit-integration-with-services) folks from Barbican, Trove, Heat, Horizon showed up - **THANKS** - and they all shared use-cases, ideas and interesting opinions about what they need and about what they'd like to see happening for Kilo with regards to this integration. Based on the results of this session Heat and Horizon are likely to be the first targets. The team is thrilled about this and we're all looking forward for this collaboration to happen.

Oslo
====

No matter what I work on, I'll always have time for Oslo. Just like for the other projects I mentioned, there will be exciting things happening in Oslo as well.

Let me start by saying that new libraries will be [released](https://etherpad.openstack.org/p/kilo-oslo-library-proposals) but not many of them. This will give the team the time needed to focus on the existing ones and also to work on the other, perhaps equally important, items in the list. For example, we'll be moving away from using [namespaces](https://etherpad.openstack.org/p/kilo-oslo-namespace-packages) - YAY!, which means we'll be updating all the already released libraries. Something that's worth mentioning is that the already released libraries won't be renamed and the ones to be released will follow the same standard for names. The difference is that they won't be using namespaces internally at all.

Also related to the libraries maintenance, the team has decided to stop using [alpha versions](https://etherpad.openstack.org/p/kilo-oslo-alpha-versioning) for the libraries. One of the points against this is that we currently don't put caps on stable branches, however this will change in Kilo. *We will pin to MAJOR.MINOR+1 in stable, allowing bug fixes in MAJOR.MINOR.PATCH+1.*

I unfortunately couldn't attend all the Oslo sessions and I missed one that I really wanted to attend about [oslo.messaging](https://etherpad.openstack.org/p/kilo-oslo-oslo.messaging). By reading the etherpad, it looks like great things will happen in the library during kilo that will help with growing its community. Drivers will be kept in tree, zmq won't be deprecated, yet. Some code de-duplication will happen and both the rabbit and qpid driver will be merged into a single one now that kombu has support for qpid. Just like other projects throughout OpenStack, we'll be targeting full Py3K support like CRAZY!


Hopefully I didn't forget anything or even worse said something stupid. Now, if you may excuse me, I gotta go offline for the next 6 month. Someone has to work on these things.