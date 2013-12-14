<!---
$"metadata"$
{"md": true, "upload_date": "2010-06-24 21:09:41", "title": "What's so good about nosql? - Part 2 - Talking about APIs", "draft": false, "slug": "whats-so-good-about-nosql-part-2", "tags": []}
$"metadata"$
-->
See also:

* <a href="/post/file/whats-so-good-about-nosql-part-1/">Part 1 - What does nosql stand for?</a>

Disclaimer: This post is mainly oriented to Web Development. All the comments, opinions and features listed bellow exist for NoSQL databases and also for some SQL ones.

As I mentioned in the part 1 (above link) working with databases is not just about having a good place to store data, without a good and simple API to access the stored data we would have a sealed box full of information that we just can't use. Nowadays, there are many tools that make data-access process easier, for example, there are frameworks (like Django) that have <a href="http://en.wikipedia.org/wiki/Object-Relational_Mapping" title="orm" target="_blank">ORMs</a> that, as Wikipedia article says, <i>"create <strong>virtual object databases</strong> that can be used from within the programming language".</i> Ok, all this is great but:

<h4>How do we know that a database has a good API?</h4>

This might sound like personal opinion but, I'm sure we all agree with most of the things listed bellow:

* <strong>The less methods the better</strong>: It's a waste of code and time when APIs need to instantiate N objects (excluding connections objects) before being able to execute a simple query, come on, that process could be simplified with a single call.

* <strong>The fastest the better</strong>: If the above point is a waste of time, this one is even worse. There are queries that may take some seconds before returning the results, we can accept that, obviously, It depends on the query but, if the 40% / 50% of that delay is caused by the driver you should definitely find a better one.

* <strong>Comunication Methods matter</strong>: There are many communication protocols and methods so I'm not going to list them here but, you have to be aware that some of them may be faster than others so before saying that the communication method exposed by the API is good take a look to the next point and be sure that it fits your needs.

* <strong>Speed Tests</strong>: Benchmarking is always great and there are thousands of benchmarks around the net so, go to your favourite search engine and start looking for benchmarks related to the database that you're interested in.

<h4>Now what?</h4>

We now know that API is important and that it must be simple and fast, even though, that's not enough. A good API is also versatile and not app oriented, for example, If my database API is oriented to Web Applications then I'm doing something wrong because any kind of application has to be able to use that API to access my database unless the database itself is oriented to web applications (which would be really weird). That said, we should take a deeper look to what web applications need to be developed in short time and without affecting their performance. Most people use to think that ORMs are the solution to everyone trying to create a web application in short time without learning SQL or any other database syntax/driver, they're wrong, there're many tools around internet ready to be used that would make web development faster and easier too but we're not going to talk about that, what I'd  like to point is what are the common features that frameworks oriented to web development MUST have.

* <strong>Laziness</strong>: "Yes, because we're lazy people so we do lazy things." It is important for web apps to keep things lazy which means that every single module/object/data should be imported/loaded/instantiated when it is needed to process the request, in any other situation that specific module shouldn't be touched by any part of the application.

* <strong>Explicitness</strong>: Just like Python's "import this" says, "Explicit is better than Implicit". Nothing, and I really mean NOTHING should be written or considered as easy to understand. This (even being a boring process) will save lot of time to developers and people giving support to others.

* <strong>Simplicity</strong>: Explicit implicitly means that frameworks should be simple and understandable; "FOR DUMMIES!". Let's let frameworks off complexity.

* <strong>Documentation</strong>: If it is not well documented it just doesn't worth your effort/time.