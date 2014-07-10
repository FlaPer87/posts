<!---
$"metadata"$
{
  "md": true,
  "title": "Juno preview for Glance and Marconi",
  "draft": false,
  "slug": "juno-preview-glance-marconi",
  "tags": [
    "openstack",
    "glance",
    "marconi"
  ]
}
$"metadata"$
-->

Yo!

You may probably know that I spend most of my time on OpenStack in general, I love tackling many things but I'm mostly focused on storage and queuing technologies - you can't do it all - so, I thought about giving you a heads up of what's being baked in the 2 projects I spend lot of my time.

Glance
======

Glance's team will focus on working on glance [Artifacts](https://etherpad.openstack.org/p/MetadataRepository-ArtifactRepositoryAPI). The plan for juno is to implemented models, API and everything needed to implement this feature without changing anything in the images API. That means images will remain the same during Juno and they'll be migrated later on during K or L depending on the status of the artifacts implementation. The artifacts work means Glance will move away from being a simply image registry to something more generic like a catalog of various data assets. In fact, the mission statement has already been [changed](https://review.openstack.org/#/c/98002/).

Another thing that will happen in Glance during Juno is that the code for store libraries will be pulled out from the code base into its own library. This work started during Icehouse and it's now almost complete. The new library - glance.store - contains the old, already supported, store drivers with a slightly different API to support random access to image data, remove the dependencies on global configuration objects and a couple of more things.

The goal behind this library is to remove from Glance part of the code that is reusable, and to allow external consumers to better support direct access to image data by using the same library Glance uses to manage such data.

There's one more thing that is worth mentioning about Glance's plans for Juno. The [async](https://blueprints.launchpad.net/glance/+spec/async-glance-workers) workers work is still moving forward. There's some support for it already - tasks base has been merged - and in the upcoming month the project will adopt taskflow as much as possible. There's still some work to do here and the feature is, unfortunately, moving slowly. An interesting thing about this new feature is that it'll allow Glance to do more things with the resources it has. For example, it'd be possible to do [image introspection](https://blueprints.launchpad.net/glance/+spec/introspection-of-images), convert and resize images without blocking requests.

Marconi
=======

As of Marconi, the plans are to complete the [API v1.1](https://blueprints.launchpad.net/marconi/+spec/api-v1.1). This version of the API is just like the previous one but it addresses some of the feedbacks gotten from the community. Some of the new things that will change are:

- Support for pop endpoints (get and delete)
- Queues are now lazy resources, which means they don't have to be created in advance.

On the storage side of Marconi, the team will add one new storage driver to support [redis](https://blueprints.launchpad.net/marconi/+spec/redis-storage-driver) and the support for [storage engines](https://blueprints.launchpad.net/marconi/+spec/marconi-queue-flavors) is on the [works](https://review.openstack.org/#/c/98777/). With storage engines (flavors) it'll be possible to create and tag clusters of storage and then use them based on their capabilities. This allows for a more granular billable and scalable deployments.

On top of the aforementioned storage engines, the team will add support for [queues migrations](https://blueprints.launchpad.net/marconi/+spec/queue-migration) between pools of the same type (flavor). It should be possible to do cross-type migrations but the team prefers to go with a more conservative approach and test the algorithm first and then improve it as needed.

Hope you find the above useful, any feedback is very welcome.
