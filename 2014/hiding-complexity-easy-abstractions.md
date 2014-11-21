<!---
$"metadata"$
{
  "md": true,
  "title": "Hiding unnecessary complexity",
  "draft": true,
  "slug": "hiding-unnecessary-complexity",
  "tags": [
    "mongodb",
    "openstack",
    "python"
  ]
}
$"metadata"$
-->

This post does not represent a strong opinion but something I've been thinking about for a bit. The content could be completely wrong or it could even make some sense. Regardless, I'd like to throw it out there and hopefully gather some feedback from people interested in this topic.

Before I get into the details, I'd like to share why I care. Since I started programming, I've had the opportunity to work with experienced and non experienced folks in the field. This allowed me to learn from others the things I needed and to teach others the things they wanted to learn that I knew already. Lately, I've dedicated way more time to teaching others and welcoming new people to our field. Whether they already had some experience or not is not relevant. What is indeed relevant, though, is that there's something that needed to be taught, which required a base knowledge to exist.

As silly as it may sound, I believe the process of learning, or simply the steps we follow to acquire new knowledge, can be represented in a directed graph. We can't learn everything at once, we must follow an order. When we want to learn something, we need to start somewhere and dig into the topic of our interest one step at a time.

The thing I've been questioning lately is how deep does someone need to go to consider something as *learned*? When does the **required** knowledge to do/learn `X` ends? Furthermore, I'm most interested in what we - as developers or creators of these abstractions that will then be consumed - can do to define this.

Learning new things is fascinating, at least for me. When I'm reading about a topic I know nothing about, I'd probably read until I feel satisfied with what I've discovered whereas when I'm digging into something I **need** to know to do something else, I'd probably read until I hit that `a-ha` moment and I feel I know enough to complete my task. Whether I'll keep digging afterwards or not depends on how interesting I think the topic is. However, the important bit here is that I'll focus on what I need to know and I leave everything else aside.

I believe the same thing happens when we're consuming an API - regardless it's a library, a RESTFul API, RPC API, etc. We'll read the documentation - or just the API - and then we'll start using it. There's no need to read how it was implemented and, hopefully, no further reading will be necessary either. If we know enough and/or the API is simple enough - in terms of how it exposes the internal implementation, vocabulary, pattern, etc - we won't need to dig into any other topics that we may not know already.

Whenever we are writing an API, we tend to either expose too many things or too few things. Finding the right balance between the things that should be kept private and the ones that should be made public is a never-ending crusade. Moreover, keeping the implementation simple and yet flexible becomes harder as we move on writing the API. Should we expose all the underlying context? What is the feeling a consumer of this API should have?

By now, you are probably thinking that I just went nuts and this is all nonsense and you're probably right but I'll ignore that and I'll keep going. Let me try to explain what I mean by using some, hopefully more realistic, examples.

Imagine you're writing an API for a messaging system - you saw this example coming, didn't you? - that is supposed to be simple, intuitive and yet powerful in terms of features and semantics. Now, before thinking about the API you should think about the things you want this service to support. As a full featured messaging service, you probably want it to support several messaging patterns. For the sake of this post, lets make a short list:

* [Producer/Consumer](http://en.wikipedia.org/wiki/Producer%E2%80%93consumer_problem)
* [Publish/Subscribe](http://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern)

These are the 2 messaging patterns - probably the most common ones - that you'd like to have support for in your API. Now, think about how you'd implement them.

For the Producer/Consumer case you'd probably expose endpoints that will allow your users to post messages and get messages. So far so good, it's quite simple and straightforward. To make things a little bit more complicated, lets say you'd like to support grouping for messages. That is, you'd like to provide a simple way to keep a set of messages separated from another set of messages. A very simple way to do that is by supporting the concept of queues. However, [Queue](http://en.wikipedia.org/wiki/Queue_%28abstract_data_type%29) is probably a more complex type of resource which implicitly brings some properties into your system. For example, by adding queues to your API you're implicitly saying that messages have an order, therefore it's possible to walk through it - pagination, if you will - and these messages cannot - or shouldn't - be accessed randomly. You probably know all this, which makes the implementation quite simple and intuitive for you but, does the consumer of the API know this? will consuming the API be as simple and intuitive as implementing it was for you? Should the consumer actually care about what *queue* is? Keep in mind the only thing you wanted to add is `grouping` for messages.

You may argue saying that you could use lightweight queues or just call it something else to avoid bringing all these properties in. You could, for example, call them topics or even just groups. The downside of doing this is that you'd be probably reinventing a concept that already exists and assigning to it a different name and custom properties. Nothing wrong with that, I guess.

You've a choice to make now. Are you going to expose queues through the API for what they are? Or are you going to expose them in a simpler way and keep them as queues internally? Again, should your users actually care? What is it that they *really* need to know to use your API?

As far as your user is concerned, the important bit of your API is that messages can be grouped, posting messages is a matter of sending data to your server and getting them is a matter of asking for messages. Nonetheless, many messaging services with support for queues would require the user to have a **queue instance** where messages should be posted but again: should users actually care?

Would it be better for your API to be something like:

    MyClient.Queue('bucket').post('this is my message')

or would it be simpler and **enough** to be something like:

    MyClient.post('this is my message', group='bucket')


See the difference? Am I finally making a point? Leave aside CS and OOP technicality, really, should the *final user* care?

Lets move onto the second messaging pattern we would like to have support for, publish/subscribe. At this point, you've some things already implemented that you could re-use. For instance, you already have a way to publish messages and the only thing you have to figure out for the publishing part of the message pattern is how to route the message being published to the right class. This shouldn't be hard to implement, the thing to resolve is how to expose it through the API. Should the user know this is a different message pattern? Should the user actually know that this is a `publisher` and that messages are going to be routed once they hit the server? Is there a way all these concepts can be hidden from the user?

What about the subscriber? The simplest form of subscription for an messaging API is the one that does not require a connection to persist. That is, you expose an API that allows users to subscribe an external endpoint - HTTP, APN, etc - that will receive messages as they're pushed by the messaging service.

You could implement the `subscription` model by exposing a `subscribe` endpoint that users would call to register the above-mentioned receivers. Again, should this `subscriber` concept be hidden from the user? What about asking the user where messages published to group `G` should be forwarded to instead of asking the users to register subscribers for the publish/subscribe pattern?

Think about how emails - I hate myself for bringing emails as a comparison - work. You've an inbox where all your emails are organized. Your inbox will normally be presented as a list. You can also send an email to some user - or group of users - and they'll receive that email as you receive other's emails. In addition to this, your email service also provides a way to forward email, filter email and re-organize it. Do you see where I'm going with this? have you ever dug into how your email service works? have you ever wondered how all these things are implemented server side? Is your email provider using a queue or just a simple database? You may have wondered all these things but, were they essential for you to understand how to use your email client? I'd bet they weren't.

Does the above make any sense? Depending on how you read the above it may seem like a silly and unnecessary way of reinventing concepts, theories and things that already exist or it may be a way to just ask the users to know what they **really need** to know to use your service as opposed to forcing them to dig into things they don't really need - or even care about. The more you adapt your service - or API - to what the user is expected to know, the easier it'll be for them to actually use it.

If you got to this point, I'm impressed. I'm kinda feeling I may be really going nuts but I think this post has got me to sort of a fair conclusion and probably an open question.

As much as purists may hate this, I think there's no need to force 'knowledge' into users just for the sake of it. People curious enough will dig into problems, concepts, implementations, etc. The rest of the people will do what you expect them to do, they'll *use* your API - for better or for worse - and they shouldn't care about the underlying implementation, theories or complexities. All these things should be hidden from the user.

Think about newcomers and how difficult it could be for a person not familiar at all with messaging system to consume a library that requires `Producer`s and `Consumer`s to be instantiated separately. Think about this newcomer trying to understand why there are are `producers`, `consumers`, `publishers` and `subscribers`. What if this newcomer just wanted to **send** a message?

As a final note, I'd probably add that the whole point here is not to use silly names for every well-known concept just to make lazy people happy. If that were the case, we wouldn't have sets and everything would be an array with random properties attached to it. The point being made here is that we tend to expose through our APIs lots of unnecessary theories and concepts that users couldn't care less about. When working on the APIs our users will consume, we should probably ask ourselves how likely it is for them to know all this already and how we can hide unnecessary concepts from them without preventing them for digging deeper into it.

Although all this may sound like "Writing APIs 101", I don't believe it is as obvious for everyone as it seems.

