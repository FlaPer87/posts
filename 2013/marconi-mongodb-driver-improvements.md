<!---
$"metadata"$
{
  "md": true,
  "title": "Marconi's mongodb driver improvements",
  "draft": false,
  "slug": "marconi-mongodb-driver-improvements",
  "tags": [
    "mongodb",
    "python",
    "openstack",
    "marconi"
  ]
}
$"metadata"$
-->

Big changes have happened in [Marconi](https://wiki.openstack.org/wiki/Marconi)'s [mongodb's driver](http://blog.flaper87.org/post/5161e76f0f06d378ae482adc/) lately.

### The why

When we kicked Marconi off, we focused a lot on the API definition, we tried to sort out as many things as we could about it. In order to do this, we needed to have something working so we could test it and find possible corner cases; that's exactly what we did. We created 2 storage drivers - one for sqlite and one for mongodb - both full-featured and updated with the latest changes and decisions we made. Although the API is still far from complete, we did manage to do what we wanted but we accumulated some technical debt that we now have to pay.

### How big is it?

* Changes to the schema
* Changes to the indexes
* Changes to the sort keys
* Give queries a hint
* Reduce counts, embrace buffering.

### Choose your schema, do it wisely

There are a couple of things that should always be kept in mind when working with databases, one of those is: "Think about your schema and do it wisely".

At the beginning, we had a rather simple model for messages. Among other fields it had a 'q' field that held the queue's ID it belonged to. Since our API sends / receives queues names instead of IDs, we had to look-up the queue's ID in the queue collection for every query we had to to on our messages collection. We also needed to verify the existence of every queue and this model was handy. Unfortunately, it quickly became a problem, performance wise. We ended up doing way more queries than we should and the penalty was high.

After some reconsiderations we decided that:

* Getting messages doesn't verify whether the queue exist. Think about it as a half-lazy queue, if the queue doesn't exist, it'll return nothing.

* Posting messages verifies whether the queue exist. We don't have a fully-lazy queue API - I'm not sure we'll go down that road - and we want to keep the queue creation as an explicit process in the messaging API. The other reason is that we want to avoid data inconsistency.

* Claiming messages doesn't verify whether the queue exist. Same logic from message's get applies here. If you try to claim messages from a non existent queue, you'll get nothing.

All the above means the user is responsible for its own mistakes, meaning, if there's a typo in the queue's name, the message ID or whatsoever, the user will have to figure that out itself.

These reconsiderations basically meant we didn't need to verify the queue existence for any of the message's operations but posts. Since our message's schema required a queue id instead of its name, we changed it and used the queue's name + project instead.

This is what the message's schema looks like now:

        -------------------
        project      ->   p
        queue_name   ->   q
        expires      ->   e
        ttl          ->   t
        uuid         ->   u
        claim        ->   c
        marker       ->   k
        body         ->   b


### Changes to the indexes

Another thing that should always be kept in mind: "Pick your indexes, do it early and do it wisely."

We did, indeed, the indexes were being correctly used but we made some mistakes anyway, plus, some changes were made to the schema and the indexes needed to reflect those changes as well.

The first [bug](https://bugs.launchpad.net/marconi/+bug/1206153) we found was related to sort operations not using the index. Two things caused this, the first was that the field being used for sorting was at the end of our compound index, the second was that we recently changed the field we're using to guarantee FIFO - using a self-kept counter instead of mongodb's _id - but some sort operations were still using the old field.

#### Sorts come before range filters

There are plenty of good blog posts ( [1](http://blog.mongolab.com/2012/06/cardinal-ins/) ,  [2](http://snmaynard.com/2012/10/17/things-i-wish-i-knew-about-mongodb-a-year-ago/) ) explaining the thing behind range filters and sorts, nevertheless we failed at this. :P

We've got some excuses about why we got there but, lets get to the fix. The fix consisted in:

1. Move 'k' after the equality indexes.
2. Ensure the presence of 'k' in the query, either by sorting or filtering.

### Changes to the sort keys

After implementing our self-maintained counter to guarantee FIFO, some queries weren't updated to reflect such change. Fixing that made some queries almost fully covered by the index.

### Give queries a hint

I mentioned before that all queries were using the indexes, thing is, not all of them were using the *right* index :D.

The fix for this was quite simple, just use hint were necessary and make sure that query uses the right index. To be more precise, we enforced the 'active' index usage for listing operations in the MessageController.

**Side Note:** pymongo's hint doesn't accept indexes' names, you'll have to pass the spec instead ;)

### Don't use count, embrace buffering

There's one last [change](https://bugs.launchpad.net/marconi/+bug/1207759) we did. We were counting messages before trying claims as a way of asserting there were messages available. Besides the fact that counting added an extra query in the whole workflow, count operations are [not as fast](https://jira.mongodb.org/browse/SERVER-1752) as expected in MongoDB - some issues have been fixed, others still remain -, unfortunately.

The fix consisted on removing that specific count and running the query right away. Once the results of the query are buffered, we can verify the list containing them is not empty.

### Debugging the whole thing

The debugging process for all this performance issues consisted in:

1. Enable the query profiler
2. Get an account on MMS and enabled the profiler there as well
3. Sort by slowest queries and start debugging each query:
  1. Run the query in the shell using explain.

There are a couple things that might come handy, though:

* MMS's query-profiler doesn't show all the fields you get from the profiler itself. If you query the "system.profiler" collection in your database, you'll get more information about that operation.

* Run explain for every query and try different strategies on the shell (backup everything).
