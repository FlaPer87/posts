<!---
$"metadata"$
{
  "md": true,
  "title": "Glance wants to go public",
  "draft": false,
  "slug": "glance-going-public",
  "tags": [
    "public",
    "image",
    "cloud",
    "service",
    "glance",
    "openstack"
  ]
}
$"metadata"$
-->

Havana development started, some folks just came back from April's summit and many things have been discussed - or are still under discussion. As for OpenStack's Image, one of the targets for this release is to make it ready for public environments. Unfortunately, some important features are missing, and that for, Glance is not cloud-ready, yet.

Does OpenStack Image needs too go public?
=========================================

As for now, I can only see good things coming out of this. Glance is doing its job and has grown gradually  in the last 2 releases but it does lacks of some features that could improve its integration within different environments.

One thing that should be kept in mind is that this project is meant to provide images, and either it does that publicly with strong enforcements or privately in some hidden data center doesn't change that.

That being said, there are some ideas that come to mi mind where a public image service could be used (I'm pretty sure there are way better use cases for it):

1. Remote image download for distributions' installer
2. Vagrant boxes distribution
3. Public image service within a cloud service (allowing users to do mor things that what they're allowed now)
4. ISO images distribution
5. ... add yours

Some features missing
=====================

*Quotas*
--------

Maybe not the most important but definitely required. As for now, OpenStack's Image service lacks of any kind of quotas support, which means it is not possible to set limits on the many operations available - neither globally nor per user. Once Glance will reach the "outside world", it will be mandatory to moderate the resources usage in multiple ways: per user, per instance, per action, per tenant and per region.

This is still under discussion, and current thoughts go around supporting it internally or as an separated service.

*Robust user roles*
-------------------

Perhaps the most important one, without it, many of the other features can't be implemented. Currently, roles and policies haven't been used heavily throughout Glance, which allow users to execute some actions without any enforcement. Good thing is the code is there - most of it - and what's really missing is the presence of new policies and roles.

*Protected image properties*
----------------------------

Images have properties, and those properties are public. However, it is necessary to assign hidden properties to images for other purposes like (following items are under discussion): billing, permissions, roles. Current properties model doesn't allow this:

    class ImageProperty(BASE, ModelBase):
        """Represents an image properties in the datastore"""
        __tablename__ = 'image_properties'
        __table_args__ = (UniqueConstraint('image_id', 'name'), {})

        id = Column(Integer, primary_key=True)
        image_id = Column(String(36), ForeignKey('images.id'),
                          nullable=False)
        image = relationship(Image, backref=backref('properties'))

        name = Column(String(255), index=True, nullable=False)
        value = Column(Text)


*Rate Limits*
--------------

Under some views this might look like something related to quotas, whether it is or not is not what we'll discuss here. As for Glance, we're treating it as a separate task and different things are being taken under consideration. One of those is to leave this outside Glance and let third party tools - regardless they are part of OpenStack.

Performance and latency
=======================

There are some other discussions going around OpenStack's Image performance and more precisely about improving uploads and downloads. Although this is not a blocker task, it would be nice to see it going forward and being able to reduce the time and bandwidth needed for both operations. Since this is a long topic, I'd like to start sharing some insightful posts and blueprints that some folks already wrote:

* [Image Transfer Service](http://tropicaldevel.wordpress.com/2013/01/11/an-image-transfers-service-for-openstack/) by John Bresnahan

* [Upload / Download Workflow](https://blueprints.launchpad.net/glance/+spec/upload-download-workflow) by Iccha Sethi

Open Discussions
================

Most of this things are under discussion, feel free to chime. I'd like to thank [Iccha Sethi](http://www.icchasethi.com) for bringing most of this things up and her great contributions to the project.

* [Glance Blueprints](https://blueprints.launchpad.net/glance/)
* [Exposing Glance for public clouds](https://blueprints.launchpad.net/glance/+spec/exposing-glance-for-public-clouds)
* [Havana getting Glance ready for public clouds](https://etherpad.openstack.org/havana-getting-glance-ready-for-public-clouds)

