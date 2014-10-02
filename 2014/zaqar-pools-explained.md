<!---
$"metadata"$
{
  "md": true,
  "title": "Zaqar's pools explained",
  "draft": false,
  "slug": "zaqar-pools-explained",
  "tags": [
    "openstack",
    "zaqar",
    "messaging",
    "scaling"
]}
$"metadata"$
-->

Now that I've dedicated time to [explain](http://blog.flaper87.com/post/zaqar-path-going-forward/) what Zaqar's path going forward is (Zaqar being a messaging service akin to SQS/SNS), I can move on and spend some time diving into some of Zaqar's internals. For this post, I'd like to explain how Zaqar's pools work.

Zaqar's scalability is more than just adding web heads or scaling the storage backend. Although both sides can scale horizontally to support big scales, there's still a chance for the storage backend to hit a limit where it needs to offload traffic to other clusters in order to keep scaling. This is where pools come handy.

Essentially, pools are storage clusters[0]. You could think about pools as you'd think about shards. They are independent and isolated storage nodes that contain part of the data stored in the system. You can add as many pools as you need, although it is recommended to scale each pool as much as possible before adding new ones.

Pools can be more than just a way for scaling Zaqar's storage but for the sake of this post, I'll just explain how they work.

Let me start by explaining how data is split across pools.

Zaqar balances data across pools in a per-queue basis. That means, message distribution happens within the storage cluster and it's not done by Zaqar itself - there are some reasons to it (some of them explained [here](http://blog.flaper87.com/post/zaqar-path-going-forward/)) that I won't go through in this post.

As I've already mention in the past, distributing queues - buckets, containers, whatever - is not as effective as distributing messages. Doing distribution at a queue level has intrinsic limitations - like hard to balance storage nodes - that could be overcome by pushing distribution down to a message level. The later, however, brings in a whole lot of other issues that Zaqar is not willing to support just yet.

When pools were added, the team considered a set of algorithms that could be used to help balancing queues. Some of those algorithms didn't require much intervention from the operator side - like a hashring - whereas others - like a weight-based algorithm - require the operator to know it's loads, clusters distribution and capabilities. After having considered the available algorithms and the feedback from operators, the team chose to start with a weighted algorithm - we've been discussing supporting more algorithms in the future, but as of now there's just one - that would give deployers enough control over how data is distributed across pools and that would also make it easier to change the results of the algorithm easily and cheap. For example, if a pool wants to be dismissed, it's possible to set its weight to 0 and prevent it to get new queues.

The current weighted algorithm looks like this:


    def weighted(objs, key='weight', generator=random.randint):
        """Perform a weighted select given a list of objects.

        :param objs: a list of objects containing at least the field `key`
        :type objs: [dict]
        :param key: the field in each obj that corresponds to weight
        :type key: six.text_type
        :param generator: a number generator taking two ints
        :type generator: function(int, int) -> int
        :return: an object
        :rtype: dict
        """
        acc = 0
        lookup = []

        # construct weighted spectrum
        for o in objs:
            # NOTE(cpp-cabrera): skip objs with 0 weight
            if o[key] <= 0:
                continue
            acc += o[key]
            lookup.append((o, acc))

        # no objects were found
        if not lookup:
            return None

        # NOTE(cpp-cabrera): select an object from the lookup table. If
        # the selector lands in the interval [lower, upper), then choose
        # it.
        gen = generator
        selector = gen(0, acc - 1)
        lower = 0
        for obj, upper in lookup:
            if lower <= selector < upper:
                return obj
            lower = upper


**NOTE:** Something to note about the current algorithm is that it doesn't take into account the number of queues that exist in each pool, which is something that could be added to it. Also, if you've any feedback as to how this algorithm can be improved, please, let us know - `#openstack-zaqar @ freenode`.

The above algorithm is used just once per queue. When a queue is created, the pooling driver looks up for an available pool and then registers the queue there. A registry that maps queues and pools is kept in a catalogue that is then queried to lookup the pool a queue has been registered into.

Right after the queue is registered in a pool, all the operations on that queue will happen in that specific pool. However, global operations like getting statistics, examine cluster's health or even listing queues will happen across all the available pools.

Pools' concept is very simple and the implementation has lots of room for improvements that we'd love to explore. In the future, it'd be useful to have support for queue's migration with 0 downtime and obviously no data loss. Moreover, we'd also like to have support for other algorithms that would help balancing queue's as even as possible without depending on the operator.

This is all I've to say about Zaqar's pools. If there's anything that looks broken or could be improved, please let us know or even better, contribute ;)

[0] Note that cluster refers to a replicated, fully reliable storage deployment. For example, a mongodb cluster could be either a replica set or a sharded mongodb environment.
