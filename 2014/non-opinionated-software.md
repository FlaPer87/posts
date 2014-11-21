<!---
$"metadata"$
{
  "md": true,
  "title": "Non-opinionated software can't exist",
  "draft": false,
  "slug": "non-opinionated-software-cant-exist",
  "tags": [
    "philosophy",
    "software",
    "python",
    "openstack",
    "zaqar"
  ]
}
$"metadata"$
-->

Here's a thing. I don't believe there's such a thing like "non opinionated" software and I think we should all be more careful when we communicate what the goals of our projects are. The later may not be new to you, probably not even the former but yet, I keep hearing the former everywhere and I keep seeing the later being ignored.

Before I get into why I think non-opinionated software *doesn't* exist, I'd like to define some of the things that I'll argue about in this post. Let's start with `opinions`.

What's an opinion?
==================

I'll start by emphasizing that opinions are not **facts**, therefore they do not represent the absolute truth and they are not verifiable. An opinion that can be verified becomes a fact, which means it's always been a fact and it was not an opinion in the first place. In fact, opinions are considered to be `subjective` and this prevents them from being absolute. Nonetheless, opinions can be supported by facts.

`Opinions` have been studied and argued about for a long time. Plato's analogy of the [Divided Lines](http://en.wikipedia.org/wiki/Analogy_of_the_Divided_Line) explains the difference between `knowledge` and `belief` but even before writing [The Republic](http://en.wikipedia.org/wiki/The_Republic_%28Plato%29), Plato and other philosophers had already argued about `opinions`. Protagoras, for example, claimed that all men's opinions are true. The meaning of this claim and the contradiction that lies within itself were thoroughly discussed in Plato's [Theaetetus](http://en.wikipedia.org/wiki/Theaetetus_%28dialogue%29) dialogue. The high-level result of this dialogue is that `opinions` don't hold `truth`. I really encourage you to read the dialogue, I consider it to be enlightening.

Another common mistake with regards to opinions is that people commonly claim opinions are *relative* without claiming what their opinion is relative in terms of. Opinions ought to be subjective, they express something that is relative to the person providing such opinion in the context they are expressed. This subtle distinction is as important as understanding that opinions don't hold truth. The context an opinion is expressed in could affect the opinion itself.

One last thing about opinions, perhaps not so relevant for the content of this post, is that opinions have a weight. That is, depending on the source of the opinion an opinion may be more relevant than the ones coming from other less reliable sources. There are many things that can be argued about this last note. For example, if opinions don't hold truth and they are subjective, why should some opinions have more value than others? My personal **opinion** is that it all depends on the context where the opinion was provided. I'd probably give more value to a distributed system experts' opinion about my distributed software than I'd give to a web designer's.

What's a non-opinionated software?
==================================

Now that we've gone through some of the aspects related to opinions, lets get into what opinions mean when they're applied to software.

A quick google search returned [this](http://stackoverflow.com/questions/802050/what-is-opinionated-software) StackOverflow link where this same exact question was asked. Among the answers provided there, this is the one I think makes more sense:

> Non-opinionated software, on the other hand, leaves lots of flexibility to the user (developer). It doesn't proscribe one method of solving a problem, but provides flexible tools that can be used to solve the problem in many ways. The downside of this can be that because the tools are so flexible, it may be relatively hard to develop any solution. Much more of the solution may have to be hand-coded by the user (developer) because the framework doesn't provide enough help. You also have to think much more about how to provide a solution and mediocre developers may end up with poorer solutions than if they had bought into some opinionated software. PERL is probably the classic example of non-opinionated software.
>
> [partial quote](http://stackoverflow.com/a/802093)

Before I get into more details, I'd like to say that I'm not criticizing the answer provided on StackOverflow as such but the general misuse of the term "opinionated" in software development. That is to say that it is not, by any means, my intention to finger-point anyone.

I'm going to break the above down into several separate claims:

> Non-opinionated software, on the other hand, leaves lots of flexibility to the user (developer). It doesn't proscribe one method of solving a problem, but provides flexible tools that can be used to solve the problem in many ways.

The fun thing about "non-opinionated software" is that it never claims where the opinion is not being provided by claiming it does have an opinion on something. As expressed in the StackOverflow answer, the non-opinionated software provides the necessary **tools** to fix a **problem** by leaving enough **flexibility** to the consumer to decide what the best thing to do is. There are 3 important things here:

1. The software is meant to solve a specific problem, therefore it has an opinion on what the final goal is, what the problem it aims to solve is, etc.

2. The software provides the tools to solve such problem, therefore the software has a very specific opinion about what the right tools to solve such problem are.

3. The software leaves lots of flexibility to the developer, therefore it is of the opinion the developer knows best how to use the tools provided by itself. This could also be interpreted as the software doesn't have an opinion on how the tools should be used, therefore it leaves it up to the user.

The above describes a pretty opinionated software with regards to a specific problem it aims to solve. It tells the user what problems it is meant to solve, what tools should be used and it claims the user should know best as of how these tools should be used.

Later on in his answer, [tvanfosson](http://stackoverflow.com/users/12950/tvanfosson), describes one of the downsides of non-opinionated software:

> The downside of this can be that because the tools are so flexible, it may be relatively hard to develop any solution. Much more of the solution may have to be hand-coded by the user (developer) because the framework doesn't provide enough help. You also have to think much more about how to provide a solution and mediocre developers may end up with poorer solutions than if they had bought into some opinionated software

The difficulties described above are not an effect caused by the hypothetical lack of opinion but an excessively flexible abstraction that lacks of [pragmatism](http://en.wikipedia.org/wiki/Pragmatism). The absence of opinion doesn't make an implementation any more flexible. On the contrary, it is the author's opinion itself of keeping the abstraction flexible that generates such complexity. Every software reflects its authors' opinions.

While I'm aware that the above was excessively nitpicky, I still hope to have made a point with regards to the misconception about what "non-opinionated" software is or even better about why non-opinionated software **can't** be.

What's opinionated software then?
=================================

For the sake of consistency, I'm going to refer to the same answer I used in the previous section. In the above section, I partially quoted [tvanfosson](http://stackoverflow.com/users/12950/tvanfosson)'s answer and kept the part of it that refers to non-opinionated software. I'll now do the same with the part that refers to opinionated software.

> Opinionated software means that there is basically one way (the right way™) to do things and trying to do it differently will be difficult and frustrating. On the other hand, doing things the right way™ can make it very easy to develop with the software as the number of decisions that you have to make is reduced and the ability of the software designers to concentrate on making the software work is increased. Opinionated software can be great to use, if done well, if your problem maps onto the solution nicely. It can be a real pain to solve those parts of your problem that don't map onto the tools provided. An example here would be Ruby on Rails.

I don't think there's anything wrong about this part of the answer. However, I'd like to highlight the quite sarcastic mention of `right way™` therein. It's important to understand - I probably can't stress this enough - that opinions don't hold truth. In a software this means that opinionated software does not represent the right way of doing things - and I'm glad he used the ™ symbol there - but one way to do them, which may or may not be a good fit for the user.

Depending on the problems a software wants to solve, the choices made by the author may be the right ones. However, this does not mean the author's opinion is right. What this means is that based on external, proven, facts the choices the author made are the right ones to solve a specific problem. Remember that opinions can be supported by facts but the opinions themselves don't hold any truth.

With all the above said, I think opinionated software exists in the context of its own goals and regardless of what the opinions of the authors are, the software will be proved to be good or bad based on external facts within specific contexts. Opinionated software that follows existing principles and standards that have been proved to be good or bad carries the opinion of its author with regard to those principles and standards. Regardless of whether those principles the software has been based on are good or bad, the author certainly thinks they are valid, hence the software is being based on them. Still, the opinions of the author hold no truth and the facts these opinions are supported by are the ones that will determine the quality of the software.


Non-opinionated software unveiled
=================================

It's not my intention to go after authors that claim their software is non-opinionated but as a member of a community that supports such a claim about its own product, I'd like to take a few minutes and unveil how opinionated such product is.

I'm of course talking about OpenStack. For a long time - and I'm guilty of this myself - we claimed to be working on a non-opinionated cloud provider. The truth is that OpenStack is **very** opinionated in so many different areas and ways. From it's API to the kind of technologies it sits on to of. In OpenStack we don't even consider running it in something that is not Linux - besides the obvious technical limitations of other operating systems. Moreover, all the services supported by OpenStack have strong opinions on what they provide, how they provide it and what the yet-to-be-proved Right Way™ of doing things is.

To make the analysis more granular, let me go deeper into one of the services that exists within OpenStack. [Zaqar](https://wiki.openstack.org/wiki/Zaqar), for example, claimed to be a non-opinionated messaging service akin to [SQS](http://aws.amazon.com/sqs/). (Un)Fortunately, this is intrinsically wrong. A messaging service can't lack of opinion because it *has* to provide certain guarantees to its users, therefore it has to have an opinion on what those guarantees are. The service claimed to have a lack of opinion on what storage you could bake it with and again, this is wrong. The guarantees made by the API pretty much define what kind of storage you can or should use for this service. The fact that this service allows you to create your own driver doesn't mean the service lacks of opinion. It just means it's flexible enough to allow for a custom implementation on the storage layer. However, it has a strong opinion on how the driver should be implemented, how it should behave, etc. These opinions could make the implementation of such custom driver difficult or even impossible.

The same high-level analysis can be done on every single piece of OpenStack and I don't think this is wrong. I actually think that software that claims to lack of opinion is bad. If the author of such software does not have an opinion on what the best way to reach the goal is, then I think the result of his work has very few things that can be trusted. Note that I'm not suggesting that software shouldn't be **flexible**, what I'm stressing here is that flexibility should be based on opinions that are supported by facts. These opinions ought to be pragmatic and simple - I encourage you to watch this [talk](http://www.infoq.com/presentations/Simple-Made-Easy) from [Rich Hickey](https://twitter.com/richhickey) about simplicity - to allow for specific problems to be solved.

In case it wasn't clear, the point I wanted to make with this post is that non-opinionated software can't exists because from the moment a developer chooses a problem to solve and tries to solve it in a certain way, the developer's opinion will be reflected in the software, therefore the software will have an opinion on the way things should be done. Writing software is as important as knowing how to talk about it and it's our responsibility as authors of software to express precisely what the software is about, the opinions reflected there and what we think the best way to solve a problem is. If we fail to communicate this, we'll be simply fooling ourselves and even worse, we'll be trying to fool others.

**P.S:** I purposely avoided talking about strong or weak opinions. I think it goes without saying that there has to be a balance between them both and that we should all keep an opened mind all times.