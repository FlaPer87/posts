<!---
$"metadata"$
{
  "md": true,
  "title": "MongoDB 2.6 is out, Marconi will benefit from it",
  "draft": false,
  "slug": "mongodb2.6-marconi-benefits",
  "tags": [
    "mongodb",
    "python",
    "openstack",
    "marconi"
  ]
}
$"metadata"$
-->

Those of you following closely MongoDB's development know that the new stable version (2.6) is out and that it brings lots of improvements and new features.

Since there are already presentations, documentation and general information about this new release, I wanted to take a chance and evaluate those changes from Marconi's perspective. Specifically, I wanted to evaluate which of the changes of this new version will help improving Marconi's MongoDB storage driver.

[Index Intersection](http://docs.mongodb.org/master/core/index-intersection/)
==================

For a long time, the only way to have queries that would use an index for 2 or more fields was using compound indexes. Although compound indexes still exist and they are recommended for several scenarios, it is now possible to intersect 2 indexes per query, which means that queries like this one are now possible:

    > db.post.ensureIndex({a: 1})
    > db.post.ensureIndex({t: 1})
    > db.post.insert({t: "yasdasdasdasdaso", a: 673453})
    > db.post.find({t: "mmh", a: {"$lt": 5}}).explain() // Complex Plan

If you've followed Marconi's development, you may know that it depends heavily on compound indexes in order to have fully indexed-covered queries. With the addition of index intersection, it is now possible to relax some of the compound indexes. For example, [these](https://github.com/openstack/marconi/blob/master/marconi/queues/storage/mongodb/messages.py#L66-L90) two indexes `ACTIVE_INDEX_FIELDS` and the `COUNTING_INDEX_FIELDS` could be simplified into:

    ACTIVE_INDEX_FIELDS = [
        ('k', 1), # Used for sorting and paging, must come before range queries
    ]

    COUNTING_INDEX_FIELDS = [
        (PROJ_QUEUE, 1), # Project will be unique, so put first
        ('c.e', 1), # Used for filtering out claimed messages
    ]

Note that the index `{p_q: 1, 'c.e': 1}` is one of the most used ones in Marconi right now.


[New Bulk Semantics](http://docs.mongodb.org/master/release-notes/2.6/#new-write-operation-protocol)
==================

Marconi supports posting several messages at the same time. Bulk post, that is. Depending on the storage driver this has to be implemented differently. In the case of MongoDB's driver, Marconi relies on MongoDB's bulk inserts. Although we could have used `continueOnError` on previous MongoDB versions, we came up with a 2-step insert process that would ensure that either *all* or *none* of the messages are posted. This was done for several reasons, one of those being not having great semantics for bulk inserts and those not being extended to updates too.

In version 2.6, MongoDB introduced ordered bulk inserts. For Marconi, this means it can rely on a more deterministic behaviour when doing bulk-inserts. The determinism comes from the fact that with ordered bulk-inserts it'll be now possible to know the exact status of the insert in case of failures.

There are more things to analyse before being able to remove the 2-step inserts but this new feature definitely solves one of them.


[$min / $max](http://docs.mongodb.org/master/release-notes/2.6/#insert-and-update-improvements)
===========

This is another very cool feature to have. As of now, Marconi relies on ttl collections to delete expired messages and claims. Unfortunately, when creating a claim, there wasn't a way to update the message expiration date *if* it would've expired before the claim did. With the new `$min`/`$max` operators, it'll be now possible to do all this in a single operation.


    new_values = {'e': message_expiration, 't': message_ttl}
    collection = msg_ctrl._collection(queue, project)
            updated = collection.update({'_id': {'$in': ids},
                                         'c.e': {'$lte': now}},
                                        {'$set': {'c': meta,
                                                  '$max': new_values}},
                                        upsert=False,
                                        multi=True)['n']

In other words, we'll be able to simplify this piece of [code](https://github.com/openstack/marconi/blob/master/marconi/queues/storage/mongodb/claims.py#L160-L187)

Other things
============

There are several other new features and improvements that I'm very exited about. For instance, MongoDB 2.6 brings in [RBAC](http://docs.mongodb.org/master/release-notes/2.6/#security-improvements) (Role Based Access Control) down to a collection level. Although Marconi allows users to secure their databases, it doesn't directly rely on MongoDB's auth features. However, the new RBAC allows for a more secured distribution of messages throughout the database instance. It could be possible to create roles based on keystone roles and let the database enforce that for us. Whether or not this is a good idea, is out of the scope of this post, though.

MongoDB 2.6 also brings a brand new [write protocol](http://docs.mongodb.org/master/release-notes/2.6/#new-write-operation-protocol) that integrates write operations with write concerns. The default write concern is `safe-writes`, which means that write failures are reported immediately.

Overall, this is a really exiting MongoDB release for me and for Marconi's team. Besides bringing several fixes and enhancements, it also brings new features that will make the storage driver simpler and safer. Please, read the full [release notes](http://docs.mongodb.org/master/release-notes/2.6/) for more information.