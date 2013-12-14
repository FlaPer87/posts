<!---
$"metadata"$
{
  "md": true,
  "title": "Dynamic TTL Collection in Mongodb for Marconi",
  "draft": false,
  "slug": "mongodb-dynamic-ttl-collection",
  "tags": [
    "mongodb",
    "marconi",
    "openstack"
  ]
}
$"metadata"$
-->

One of the things that led the team towards choosing Mongodb as Marconi's - the queuing service for OpenStack - first and default storage back-end is its TTL Collections feature.

TTL Collections - perhaps it would be better to call it TTL Indexes - are normal collections with a special index type, which defines the time - in seconds - a record can last in the collection.

This was added in Mongodb version 2.2 and was rapidly accepted and integrated many deployments.

Implementation
==============

A TTL Index is created by specifying the expireAfterSeconds option in the ensureIndex method:

    db.ttl_col.ensureIndex({<field>: <direction>}, {expireAfterSeconds: <seconds>})

The above starts a background thread (if not already running) that will monitor the collection and scrub expired records every minute, which means a record can last at most `N + 60` where `N` is the number of seconds specified in the index and 60 the frequency of the background thread.

Marconi Usage
=============

Even though it is a great feature, it wasn't enough to cover Marconi's needs since the later supports per message TTL. In order to cover this, one of the ideas was to implement something similar to Mongodb's thread and have it running server-side but we didn't want that for a couple of reasons: it needed a separated thread / process and it had a bigger impact in terms of performance.

After digging into this a bit more and doing some tests, we found out it is possible to "fool" the ttl monitor and have dynamic TTL support without much effort. Let me explain this a bit more.

Mongodb's TTL monitor looks for records that have expired by checking if the date field specified in the index is less than the current time minus the seconds specified in the `expireAfterSeconds`.

    BSONObj query;
    {
        BSONObjBuilder b;
        b.appendDate( "$lt" , curTimeMillis64() - ( 1000 * idx[secondsExpireField]  .numberLong() ) );
        query = BSON( key.firstElement().fieldName() << b.obj() );
    }

Since the date set in the indexed field *must* be less than `time - ttl`, it is possible to have dynamic ttl by setting the index's ttl to 0 and adding it to the field date instead:

    > use ttl
    > db.ttl_col.ensureIndex({ttl: 1}, {expireAfterSeconds: 0})
    > var start  = new Date(new Date().getTime() + 60000)
    > db.ttl_col.insert({ttl: start})
    > while (true) {
        var count = db.ttl_col.count();

        print("# of records: " + count + " (" + (start - new Date()) + ")");

        if (count == 0)
            break;

        sleep(4000);
    }
    # of records: 1 (56783)
    # of records: 1 (52782)
    # of records: 1 (48781)
    # of records: 1 (44779)
    # of records: 1 (40778)
    # of records: 1 (36776)
    # of records: 1 (32775)
    # of records: 1 (28774)
    # of records: 1 (24773)
    # of records: 1 (20771)
    # of records: 1 (16770)
    # of records: 1 (12769)
    # of records: 1 (8768)
    # of records: 1 (4765)
    # of records: 1 (763)
    # of records: 1 (-3238)
    # of records: 1 (-7239)
    # of records: 1 (-11240)
    # of records: 1 (-15241)
    # of records: 1 (-19242)
    # of records: 0 (-23243)

In the above code, the record has a 1 min TTL and lasted ~1:20 - notice that it could have lasted at least 1 min and at most 2 min.

This made the implementation way easier and allowed us to use the same behavior for both messages and claims, even though claims expiration doesn't require removing records.

Current Marconi's message post looks like this:

    def post(self, queue, messages, client_uuid, tenant=None):
        qid = self._get_queue_id(queue, tenant)

        now = timeutils.utcnow()

        def denormalizer(messages):
            for msg in messages:
                ttl = int(msg["ttl"])
                expires = now + datetime.timedelta(seconds=ttl)

                yield {
                    "t": ttl,
                    "q": qid,
                    "e": expires,
                    "u": client_uuid,
                    "c": {"id": None, "e": now},
                    "b": msg['body'] if 'body' in msg else {}
                }

        ids = self._col.insert(denormalizer(messages))
        return map(str, ids)


Random thoughts
===============

Would it be possible / better to have this behavior in the database side? What would the cost of this task be? As for now, I can see it being implemented like this:

    > db.ttl_col.ensureIndex({datetime: 1}, {expireAfterSeconds: "ttl_field"})

This would require doing some in-query operations like adding the value of the record's `ttl` field to the current time and then check whether it is greater than `datetime`. They might be a better way, though.