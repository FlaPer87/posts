<!---
$"metadata"$
{
  "md": true,
  "title": "glance-gridfs-store",
  "draft": false,
  "slug": "glance-gridfs-store",
  "tags": [
    "openstack",
    "glance",
    "mongodb"
  ]
}
$"metadata"$
-->

Recently, [GridFS](http://docs.mongodb.org/manual/core/gridfs/) support has been added to Glance, which means, it is now possible to store images inside GridFS. For those of you that don't know what GridFS is, it is a convention for storing files inside MongoDB.

About the implementation
========================

The implementation was pretty much straight-forward. It implements all the required methods and it just needed few lines of code. Some cool things about this store is that GridFS already had most (or all of them) things needed: md5, length, clean API. No rocket science!

Some Benefits
=============

* It's fast
* It's distributed
* Can be mounted (using gridfs-fuse)
* Can be shared with other environments and / or applications
* Fits perfectly in deployments where mongodb already exists.
* Reads from secondaries when using replica sets.
* Sharding support.

Random Thoughts
===============

1) Even though Glance uses gridfs mostly as a bucket to store images, it can still be extended - either changing the store code or from outside it -  with other features like tracking accesses to images, for example.

2) When using replica sets, an interesting deployment could be to have a replica, where Glance can read from, in every glance-api node. This would speed reads but it would definitely use more space.

3) When using shards, would it be possible to have glance-api running on the shard nodes and "shard" glance requests as well? (sounds like a crazy idea)

I don't expect this store to be widely used but I do find interesting how MongoDB fits well in so many environments and such different use cases.