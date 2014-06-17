<!---
$"metadata"$
{
  "md": true,
  "title": "Real-Time Systems: High level introduction",
  "draft": false,
  "slug": "high-level-real-time-systems",
  "tags": [
    "openstack",
    "real-time",
    "distributed",
	"python"
  ]
}
$"metadata"$
-->

I've been working with `real-time` distributed systems for quite some time
now. This one of the topics I like the most. In this post I'd like to
spend some time explaining at a high level what `real-time` systems are, where
they're used, some of their requirements and I'd also like to conclude the post
with a small section explaining why I think the term `real-time` is neither
accurate nor correct to describe these systems.


Real Time Systems
=================

A system is said to be `real-time` , when it's subjected to execution time
constraints. What this means is that whatever it takes to complete the
execution of a specific task will determine whether the task ended successfully
or not. There are different groups of `real-time` systems.  The first group is
composed by systems that are fully dependent of a specific deadline. Any
deviation from the time constraint will be considered a total failure. This
systems are said to be *hard* real time systems. The second group is composed
by systems whose quality may be affected by missed deadlines but those misses
won't be considered failures. Nonetheless, missed deadlines will invalidate the
usefulness of the result provided. This group is usually called *firm* and it's
not so common. The third group, though, is said to be *soft* real time because
the deadlines misses are not considered failures and the usefulness of their
results will decrease when a deadline is missed but it won't be invalidated.

The different levels of constraints exist to satisfy multiple scenarios. For
instance, *hard* `real-time` systems are commonly used for stock quotas
transactions, airplane systems, car systems etc, whereas *soft* `real-time`
systems are used when the availability of the result is important but not as
much to make it mission critical. In other words, *hard* `real-time` systems
main goal is to meet all deadlines, whereas *soft* `real-time` systems just
need to meet a subset.

[Read More](http://en.wikipedia.org/wiki/Real-time_computing)

Real time systems implications
==============================

The hardest thing about `real-time` systems are not the systems themselves but
the things they imply. A `real-time` system, for instance, implies some level
of `determinism`, `scale`, `fault-tolerance` etc. This all depends on the level
of strictness the system has. A system that needs to meet all the deadlines
will need to have `fault-tolerance` as well. In the event of a node failure,
the system has to send the result back before the deadline, otherwise it will
be considered a total failure.

Lets dive into some of those implications.


Time Constraint
---------------

At this point it is clear that `real-time` systems are not the same thing as
`really-fast` systems. A system is said to be `real-time` when its results are
tight to an execution time constraint. Therefore, it is necessary to establish
what that time constraint is and how strict it is. There's not just 1 time
constraint that has to be established, though. If your `real-time` system is
composed by more than 1 component, you'll need to establish a time constraint
for the inter-communications of your system. Each one of the inner constraints
have to be smaller than the constraint applied to the whole system. What that
time constraint is, depends on the system itself and its purpose.

Besides defining the time constraint, it's also necessary to establish how it
will be enforced, measured and what actions will be taken in case of
failure. Furthermore, it is necessary to determine where this enforcement will
happen. More about this later.


Integration Requirements
-------------------------

This pretty much falls into what systems integration is. However, since a
`real-time` system is not necessarily distributed, this will also apply to
non-distributed systems.

*Integration requirements* refer to the semantics, technology and methods used
by the system to enforce both the execution and the distribution of the
task. Things like whether the execution needs to be synchronous or asynchronous
need to be sorted out in this step. Therefore, it is also necessary to
establish what the components of the system are, what those components do and
how they interact with the rest of the environment.

This section has different applications. Depending on the system, it may be
implemented differently. An example of this are non distributed `real-time`
system - perhaps applications would be more accurate here - where the
integration with other systems through a non-deterministic environment is not
required. However, it is necessary to integrate the different components of
that single, most likely multi-threaded, system. Although this may seem
obvious, and perhaps implicit in every system aiming to run concurrently.


Determinism
-----------

I won't go deep into `Determinism`, the concepts and rules behind this topic
are big and out of the scope of this post. Visit [wikipedia](http://en.wikipedia.org/wiki/Determinism_%28disambiguation%29#Computer_science) for more information.

Determinism is not a strict requirement for every `real-time` system. It is
possible to have `real-time` systems that don't behave
deterministically. Although this is certainly the least common case for
`real-time` systems, the previous statement could be argued as a whole given
the fact that these systems would benefit from a deterministic behavior.

Determinism brings predictability to the system, which allows it to be more
reliable and lower the difficulties of meeting the goals of the tasks being
executed.


Fault Tolerance
---------------

Just as with `Determinism` I won't go deep in the concepts behind
`fault-tolerance`, for more information check [wikipedia](http://en.wikipedia.org/wiki/Fault_tolerance) out.

Unlike determinism, fault-tolerance is applicable just to multi-component
systems. That is, in most of the cases, a distributed system.

Fault tolerance is perhaps a stronger requirement for `real-time` systems that
what determinism itself is. A system willing to respect the imposed time
constraint has to survive possible failures and complete the task.

It is also worth mentioning that deterministic systems have to be
fault-tolerant, which is not necessarily true the other way around. Failing to
survive failures will introduce non-deterministic behaviors throughout the
system, therefore making the whole system behave non-deterministically.


Requirements Enforcement
========================

We've made it clear that every `real-time` system has intrinsic requirements
that should be met in order for it to meet its goals. The list of requirements
is far from being complete but it introduces some of the most relevant ones.

Some of the requirements described above need to be enforced at some point in
time during the execution of every task and the life of the system. In order to
do that, it is necessary to determine where this enforcement will happen and
when.

This enforcement is usually implemented along side with the system itself. That
means, if the system is distributed, the enforcement of the time constraint,
support for determinism and support for fault-tolerance will be distributed as well.

This step adds more complexity to the system. For instance, determining whether
the system is behaving deterministically, whether the system nodes' health is
fine or even whether the goals are being met is often the most critical task of
a successful `real-time` system.



`Real-Time` applicability
=========================

This post has been mostly focused on distributed systems. However, the term
`real-time` is not bounded to those systems. Here's a list of other type of
applications for this term:

* Programming languages
* Operating Systems
* Network protocols


Real time systems misconception
===============================

At this point, I don't expect you to be an expert on this field. In fact, I
think some of the topics explained above could certainly have been explained
more in detail. However, I do expect you to know that `real-time` does not mean
fast nor it means immediately. A `real-time` system is a system tight to time
constraints, which in most cases are very low. Therefore, I believe the term is
wrong and doesn't describe the real goal of the system.

Given the fact that there's no such thing as `real-time` and that computers are
governed by the laws of physics, it'd be accurate to say that unless the point
of reference for `real-time` systems is explicitly defined, the measurement
could be relative to any point of the system since the task was
executed. Common sense leads us to use the time when the task started as a
reference point to measure the success of the execution. This, though, implies
that no matter how fast the system is, the result won't ever be immediate and
the execution time is not actually real.

In my humble opinion, a more accurate term for this kind of systems would be
one that explicitly specifies the time constraints of the system itself. For
example: `time-bound`, `time-constrained`, etc.

Unfortunately, the misconception around the term has led people to use it
erroneously to describe things that are supposed to be fast as `real-time`.


Some References
===============

* [Real Time Computing - Wikipedia](http://en.wikipedia.org/wiki/Real-time_computing)
* [Programming Distributed Computing Systems: A Foundational Approach](http://www.amazon.com/gp/product/B00G9U9MBI/ref=kinw_myk_ro_title)
* [Fault-Tolerant Real-Time Systems: The Problem of Replica Determinism](http://www.amazon.com/Fault-Tolerant-Real-Time-Systems-Determinism-International/dp/1475770286/)
