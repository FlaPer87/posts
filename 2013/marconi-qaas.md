<!---
$"metadata"$
{
  "md": true,
  "title": "Marconi: QaaS baked by MongoDB",
  "draft": true,
  "slug": "marconi-qaas-baked-mongodb",
  "tags": [
    "openstack",
    "marconi",
    "mongodb",
    "queues"
  ]
}
$"metadata"$
-->

Marconi is a QaaS provider for OpenStack. It's in its early stages and under heavy development. It was kicked off a month ago and there's already a working version (not production ready and most likely buggy, but it works).

As part of the designing process, we had to choose what would the default persistence layer be backed by. TBH, it didn't took us more than 2 (or 3?) meetings to come to the conclusion that MongoDB was good for the job.

Pros / Cons
===========

As said before, MongoDB is certainly good for this job but it isn't perfect and some of those imperfections became kind of an implementation nightmare, even though, they were nothing we couldn't overcome.

Lets dig a bit more on this imperfections and how Marconi is handling them, for that, let me first show you Marconi's current data model.

Collections & Fields
====================

Marconi is using 2 collections to manage 3 different resources (queues, messages, claims) ([API DOCS](https://wiki.openstack.org/wiki/Marconi/specs/api/v1)). Queues and Messages have their own collection while Claims exist within the Messages' collection as part of claimed messages.

All fields where shortened to a single letter name in order to save space on disk and memory and they exist just when they're needed, for example, all messages lack of a claim field unless they were claimed.

Queues collection:

    {
        "_id" : ObjectId("515c65d7ce2ef01fa52574ae"),
        "m" : { "ttl": 60},
        "n" : "test_queue",
        "t" : "tenant"
    }

* **name**: n
* **tenant**: t
* **metadata**: m


Messages collection:

    {
        "_id" : ObjectId("515c65d70f06d314293083c8"),
        "b" : {
                "event" : "Event number 15"
        },
        "e" : 1365009959,
        "q" : ObjectId("515c65d7ce2ef01fa52574ae"),
        "u" : "client_uuid",
        "t" : 80
    }

* **ttl**: t
* **body**: b
* **uuid**: u
* **queue**: q
* **claim**: c
* **expires**: e

Limited updates
===============

Claims' controller is responsible for managing CRUD operations for claims. The implementation was pretty straight forward for most of the operations but for create. In order to create a claim we need to:

1. Get the queue
2. Get available messages
3. Claim available messages

As you might have noticed the find and claim of available messages is done in 2 separate operations, here's why:

* There's no way to execute an update over a limited number of records, yet [SERVER-1599](https://jira.mongodb.org/browse/SERVER-1599). You can do it either on 1 or all of the records matching your query

* findAndModify is not as fast as find + update, yet. [SERVER-1117](https://jira.mongodb.org/browse/SERVER-1117) so, an iterator that would call findAndModify N times wasn't a choice.

This is quite important since there are no transactions in mongodb but the 3 steps above seemed more reasonable instead of sacrificing performance and iterating N times.

Lack of Transactions
====================

TBH, I sometimes miss transactions. #confession

So, 3 separate queries must be executed and there's no way to ensure that they all will succeed and that no race conditions will affect these operations... Let's hope for the best!

Seriously, a best-effort fashion algorithm was implemented and it will ***try*** to claim as much messages as were requested but it doesn't ***guarantees*** so. Here's the piece of code that does that ([create method](https://github.com/stackforge/marconi/blob/master/marconi/storage/mongodb/controllers.py#L345)):

    # Get a list of active, not claimed nor expired
    # messages that could be claimed.
    msgs = msg_ctrl.active(queue, tenant=tenant, fields={"_id": 1})
    msgs = msgs.limit(limit).sort("_id")

    messages = iter([])

    # Lets respect the limit
    # during the count
    if msgs.count(True) == 0:
        return (str(oid), messages)

    ids = [msg["_id"] for msg in msgs]
    now = timeutils.utcnow_ts()

    # Set claim field for messages in ids
    updated = msg_ctrl._col.update({"_id": {"$in": ids},
                                    "$or": [
                                        {"c.id": None},
                                        {
                                            "c.id": {"$ne": None},
                                            "c.e": {"$lte": now}
                                        }
                                    ]},
                                   {"$set": {"c": meta}}, upsert=False,
                                   multi=True)["n"]

Basically, what this means is "Let the fastest win" which is (or not?) fair enough in this case. Under real heavy workload what most likely will happen is that most claims wont get as much messages as they asked for but, they will get enough to keep workers busy. Once all claimed messages are consumed, clients will be able to claim a new set of messages.

Soon I'll be able to do more tests under different workloads and share the results and stories.

Create Indexes
==============

Big issue, I mean, not Mongodb's fault. Marconi will handle gazillions of messages and queues (hopefully) and it must be fast as hell and it must save as much resources as possible. Choosing the wrong index (as for many applications out there) would end up slowing Marconi down and eating all available memory.

As for queues, the fields that will be used most in queries are `tenant` and `name`, that for, a unique (because queue names are unique) compound index on tenant and name was created. This index allow us for getting all queues that belong to a specific tenant and a queue name that belongs to a specific tenant.

Messages, instead, could be queried using the `queue` (because messages operations are always done within a queue), client's `uuid` (because clients can exclude their messages from queries), expiration date (to exclude expired messages), by claim id and claim expiration (to get claimed messages and / or exclude expired claims). This is the indexes map:

* (uuid, ASCENDING)
* (queue, ASCENDING)
* (expiration, DESCENDING)
* (claim.id, ASCENDING), (claim.expiration, DESCENDING)

Closing
=======

I won't say the issues listed above don't matter but there are definitely plenty of good reasons about why we chose MongoDB and most of this is all about knowing the technology you're relaying on.

Hopefully, you'll find the above information useful and it will help you on solving the same (or similar) issues faced in Marconi's implementation. This post is not meant to criticize MongoDB (which certainly is amazing but not perfect) but to give some insights and share the knowledge gained here. Last but not least, if you think the above is just bullshit and have a better way to do it just drop a comment or contribute back :)