<!---
$"metadata"$
{
  "md": true,
  "title": "Glance Mitaka: Passing the torch",
  "draft": false,
  "slug": "glance-mitaka-passing-the-torch",
  "tags": [
    "openstack",
    "glance"
]}
$"metadata"$
-->

I'm not going to run for Glance's PTL position for the Newton timeframe.

There are many motivations behind this choice. Some of them I'm willing to discuss in private if
people are interested but I'll go as far as saying there are personal and professional reasons for
me to not run again.

As I've always done in my past cycles as PTL, I'd like to take some time to summarize what's
happened in the past cycle not only for the new PTL to know what's coming up but for the community
to know how things went.

Before I even start, I'd like to thank everyone in the Glance community. I truly believe this was a
great cycle for the project and the community has gotten stronger. None of this would have been
possible without the help of all of you and for that, I'm deeply in debt with you all. It does not
just take an employer to get someone to contribute to a project. Being paid, for those who are, to
do Open Source is not enough. It takes passion, motivation and a lot of patience to analyze a
technology, think out of the box and look for ways it can be improved either by fixing bugs or by
implementing new features. The amount of time and dedication this process requires is probably worth
way more than what we get back from it.

Now, with all that being said, here's Glance Mitaka for all of you:

Completed Features
==================

I think I've mentioned this already but I'm proud of it so I'll say it again. The prioritization and
scheduling of Glance Mitaka went so well that we managed to release M-3 without any feature freeze
exception (FFE) request. This doesn't mean all the features were implemented. In fact, at least 4
were pushed back to Newton. However, the team communicated, reviewed, sprinted and coded in such a
way that we were able to re-organize the schedule to avoid wasting time on things we new weren't
going to make it. This required transparency and hard decisions but that's part of the job, right?

* [CIM Namespace Metadata](http://specs.openstack.org/openstack/glance-specs/specs/mitaka/implemented/cim-namespace-metadata-definitions.html)
* [Support download from and upload to Cinder volumes](http://specs.openstack.org/openstack/glance-specs/specs/mitaka/implemented/cinder-store-upload-download.html)
* [Glance db purge utility](http://specs.openstack.org/openstack/glance-specs/specs/mitaka/implemented/database-purge.html)
* [Deprecate Glance v3 API](http://specs.openstack.org/openstack/glance-specs/specs/mitaka/implemented/deprecate-v3-api.html)
* [Implement trusts for Glance](http://specs.openstack.org/openstack/glance-specs/specs/mitaka/implemented/glance-trusts.html)
* [Migrate the HTTP Store to Use Requests](http://specs.openstack.org/openstack/glance-specs/specs/mitaka/implemented/http-store-on-requests.html)
* [Glance Image Signing and Verification](http://specs.openstack.org/openstack/glance-specs/specs/mitaka/implemented/image-signing-and-verification-support.html)
* [Supporting OVF Single Disk Image Upload](http://specs.openstack.org/openstack/glance-specs/specs/mitaka/implemented/ovf-lite.html)
* [Prevention of Unauthorized errors during upload/download in Swift driver](http://specs.openstack.org/openstack/glance-specs/specs/mitaka/implemented/prevention-of-401-in-swift-driver.html)
* [Add filters using an ‘in’ operator](http://specs.openstack.org/openstack/glance-specs/specs/mitaka/implemented/v2-add-filters-with-in-operator.html)
  
If the above doesn't sound impressive to you, let me fill you in with some extra info about Glance's
community.

Community
=========

Glance's community currently has 12 core members, 3 of which joined during Mitaka and 2 of those 3
members joined at the end of the cycle. That means the team ran on 9 reviewers for most of the cycle
except that out of those 9, 1 left the team and joined later in the cycle and 3 folks weren't super
active this cycle. That left the team with 5 constant reviewers throughout the cycle.

Now, the above is *NOT* to say that the success of the cycle is thanks to those 5 constant
reviewers. On the contrary, it's to say that we've managed to build a community capable of working
together with other non-core reviewers. This was a key thing for this cycle.

I don't think it's a secret to anyone that, at the beginning of the cycle, the community was fragile
and somewhat split. There were different opinions on what Glance should (or shouldn't) look like,
what new features Glance should (or shouldn't) have and where the project should be headed in the
next 6 months.

The team sat down, the team talked and the team agreed on what the project should be and that's what
the team did in the Mitaka cycle. Sharing one message with the rest of the OpenStack community (and
especially new Glance contributors) was a key for the community to become stronger.

What changed? What did the community do differently?

Priorities and Goals
--------------------

Mitaka was the first cycle that Glance strictly followed a
[list of priorities](http://specs.openstack.org/openstack/glance-specs/priorities/mitaka-priorities.html).
Funny enough, 2 of those priorities didn't make it in Mitaka but we'll get to that in a bit.

The list of priorities didn't do it all by itself. The list of priorities gave us a target, a goal.
It helped us to remain focused. It kept us on track. However, it did way more than that. The list of
priorities allowed us for:

* Sending a clear message of what the community has agreed on and where the community is headed
* Selecting a narrow list of features that we would be able to work on and review throughout the
  cycle
* Scheduling and splitting reviews to accommodate the priorities

Of those points, I believe the second one is the one that really did it for us. We kept the set of
new features small so that we could focus on what was important. We had more proposals than we
approved and we rejected the rest based on our priorities. This is something I'd like to see
happening again in Glance and I'd like to encourage the next PTL to do the same and  be *strict*
about it.

Reduce the review backlog
-------------------------

[We abandoned patches](http://stackalytics.com/?user_id=glancebot@mailinator.com)! We removed from
the review queue all the patches that, for 2 or more months, had been in merge conflict, had had
-1/-2 from cores or had had -1 from jenkins (hope I'm not missing something here). We did that and
we made the backlog shorter, we kept in the review list what was really relevant at that moment.

Something important about the above is that we didn't abandon patches that had stalled for lack of
reviews. We prioritized those, we bumped those to the top of our review list and we provided the
reviews those patches deserved. Some of them landed, some didn't but the important bit is that those
patches were reviewed. Glance's current backlog (verified patches, Workflow 0 and no -2s) is less
than 90 patches across all projects (likely way less than that but I just did a rough count) and the
most important thing is that *ALL* these patches have received reviews in 2016. Now, if you don't
think this is great, you should have seen our backlog before.

Now, there's no point in cleaning up the review queue if we're going to let it fill up again. Right?
This is where the community awesomeness comes to light. We created a [review dashboard](http://bit.ly/glance-dashboard),
which some folks used to organize their reviews. I found it super useful, I used it to prioritize my
reviews and help other folks to prioritize theirs. When you're given an organized list of reviews
rather than just a list of random reviews, it's *way* easier for you to know what to review. That
right there is the key. To know what to review. I believe, in Mitaka, the team knew what to focus on
and the team also knew someone in the community was ready to provide a fresher, cleaner, list of
reviews they could focus on. Some folks would prefer to go and make up a list themselves, others
will prefer to have one ready. Either way, having a clear story of where the focus should go is the
key to help reviews move faster. Remove the noise, it distracts from people from what's really
important.

Review Days
-----------

Not really a new thing. This has happened before and we just kept doing it. The difference, perhaps,
is that we increased the number of review days in the cycle. We tried to do at least 1 review day
per milestone and we're now doing a Review Monday until the end of the cycle to get as many bug
fixes as possible in before the release. RC1 is looking good already!

So, if you'd ask me, I believe what changed was the community. The community got together, polished
some things, and focused on what's important *the project*. If you read between lines, the above
shows one constant pattern, the community matured and it found what its placed in the OpenStack
community.

Single Team
-----------

The Glance team is now back to being a single reviewing machine rather than several, isolated, teams
with specific tasks, which sometimes ended up duplicated. The Glance Driver's team has been merged
into the Glance Core team and the Glare team (Artifacts) is not using the Fast Track anymore.

Having smaller teams has resulted in a very useful thing to do for other projects. Depending on the
size of the project, it'd be possible to map tasks to smaller teams and then reduce them once the
job is done ;). Unfortunately, given Glance's team size, this ended up adding *more* things to do to
members of those smaller teams that were also part of the other teams as well.

One reason to mention this is because we'll have the temptation to do this again in the future but,
as it's been proven thus far, Glance's community is not big enough to make such splits worth it and
those end up causing more harm to the community than good.

Spec Freeze
-----------

The team incorporated a spec freeze in this cycle. The dates that were picked were not the most
ideal ones but the freeze helped a lot to bring back focus on code reviews and coding. This freeze
put a timeline on folks to get their proposals ready, hence forcing them to have enough time to
implement such proposals. Having open milestones distracts the community from the schedule.
Announcing such milestones in advance and providing constant reminders helped with making sure folks
were prepared and ready to react.


Was it all rainbows?
====================

No, it was not. There were and there are *many* things we need to work on and improve. For instance,
2 of the priorities didn't make it this cycle. One of them (Nova's adoption of Glance's v2) simply
requires a bit of more work and it specifically requires a better alignment with the Nova
community's priorities. In other words, Nova needs to make this a priority for them.

The second priority that missed the deadline is the refactor of the image import workflow. Some of
you might be thinking "Guys, you had 1 job, *ONE* job and it was to discuss and implement that
refactor". Well, turns out that such refactor has an impact on *every* cloud and it's not something
the team can afford to change a third time (yes, this is the second time the image import workflow
is refactored). I'm actually happy it didn't make it in Mitaka because that gave the team more time
to evaluate the proposal that had been discussed at the summit, the issues around it and the
different alternatives. Nonetheless, I am a bit sad about how things evolved with this proposal
because at the very beginning of the cycle we were a bit naive in our planning of this work. That is
to say, that we should've probably known from the beginning that we wouldn't have had the time to
implement this spec and that it would have taken us the whole cycle to discuss it. The problem is
not that we didn't know it to begin with but the fact that we weren't able to communicate that to
the community from the beginning. I don't think this is a big deal, though. We realized soon enough
that we shouldn't rush this and that dedicating the cycle to discuss this spec was more better than
rushing it and then have a poor implementation of it.

We also experimented with a new process for lite specs and it was not a huge success. This impacted
some of the lite specs that had been proposed but we did our best to come out of that situation
without impacting other's people work. In fact, that situation not just highlighted the issues we
had with the process but the team responsible for it (the glance-drivers team), which ended up being
merged into the glance core team (as I mentioned in the previous section). This process is being
refactored and you can learn a bit more about it in [this review](https://review.openstack.org/#/c/282516/).

There's one more thing I wish we would have dedicated more time on. That's tempest. Unfortunately,
given the time available, size of the team and the priorities we had, tempest did not receive as
much love as we'd have loved to. There are several tempest tests that need to be cleaned up a bit,
especially on the V2 side.

To the Glance Community
=======================

All the credits for the above goes to you! As a PTL I don't think I can take *any* credit for what I
consider a successful cycle brought by the community itself. I instead recognize that it was all
possible because the community decided to go back to being awesome. I'm a believer that the PTL's
role is all about enabling the community to be awesome. Planning, prioritization, scheduling, etc.
it all serves a single goal, which is to allow the community for doing what they know best and focus
on that.

I've enjoyed every single of my stages in this community. Rushing through reviews, coding like
crazy, ranting like crazy, leading the community and back to reviewing like crazy. These years as a
member of Glance's community have taught me a lot about this project and how critical it is for the
rest of the community. As I always say, it's one of those projects that can take your whole cloud
down without you even noticing but I do hope you notice it.

Glance is often referred to as a simple project (true), as a small project (kinda true) and
sometimes as not super cool (false). I'd like to remind you that not only Glance is a "cool" project
to work on but it's also super critical for OpenStack. As I remind you this, I'd like to urge you to
help the project stay on track across the cycles. Glance (as every other projects) depends on the
ability of its community to dictate what's best for it.

Glance's interoperability has been compromised and there's a plan to help bringing it back. Let's
get that done. Glance's v1 is not considered secure and it must be deprecated. Let's do that as
well. Glance's stability and security has shown some weaknesses. Let's not ignore that. Working on
new features is always sexy. Working on the new cool stuff that other projects are doing might seem
like a must do task. I'd argue and say there's a time for everything and, while Glance shares
OpenStack's priorities, there are times where the project needs to take a step back, put itself
together again and start again. I don't believe Glance has left that self-healing period and I'd
like to urge the whole community to keep this in mind.

To the new PTL
==============

Listen! Listen to the things the OpenStack community has to say. Listen to the things external folks
have to say. Most importantly, listen to what the Glance community has to say. Glance is not a
playground for making random decisions. If you listen to what the community has to say, it'll be
easy enough to know what to do and what the next steps are. However, you should be ready for making
hard decisions and you need to have the courage to do so. During the last elections, I wrote a
[post](http://blog.flaper87.com/post/something-about-being-a-ptl/) about what being a PTL means and
I'd like to encourage you to read it, even if you've done so already.

If you look at the goals we set for Glance during Mitaka and the results we achieved, you'll soon
notice what the priorities for the next cycle should be. The community will help shaping those
priorities but the baseline is there already.

A great cycle is not measured on how many features the community is able to implement. Therefore, I
encourage you to not fall under the temptation of approving as many specs as possible. It is
*perfectly fine* to say no to specs because they conflict with the project's priorities. The more
specs the team approves, the more code there will be, the more people the project will need to
complete the feature (code wise and review wise). Keep the release small, keep it concise, keep it
focused. It's extremely important to communicate the intent of the release to the rest of the
community. Do not forget Glance *is* a critical piece of every cloud.

Glance's community is not formed by the core team. It's formed by every person willing to dedicate
time to the project either on reviews or code. Work with them, encourage them. They *are* helping
the project. Some folks simply don't want to do reviews, that's fine. They are still helping with
code and bug fixes. Recognize that and make sure they feel part of the community because they are.
Expanding the core team is great as long as you can ensure folks in the team are aligned with the
team's priorities. Welcome new members and do it gradually.

One more thing, learn to delegate. During my time as a PTL, I relied on other members as much as
possible for keeping up with some tasks. For instance, Erno Kuvaja helped immensely with releases
and stable maintenance, Nikhil Komawar kept the team updated about the cross-project initiatives,
Stuart Mclaren, Hemanth Makkapati and Brian Rosmaita worked with the vulnerability team on security
issues, etc. Thanks to all of them for their immense help and I do hope you'll keep up at what
you're doing :). In other words, burnout is real and you gotta take care of yourself too. Work with
the community, there's no need to take everything on your shoulders as you might end up dropping
some balls. When folks don't show up on reviews and they don't share their opinions, do not give
those as granted. Find them and ask for it.

And please, I beg you, let's get rid of v1!
