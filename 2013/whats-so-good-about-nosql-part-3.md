<!---
$"metadata"$
{"md": true, "upload_date": "2010-11-11 21:59:39", "title": "What's so good about nosql? - Part 3 - Fearless against schema-less dbs", "draft": false, "slug": "whats-so-good-about-nosql-part-3", "tags": ["mongodb", "nosql", "databases"]}
$"metadata"$
-->
See also:

* <a href="/post/file/whats-so-good-about-nosql-part-1/">Part 1 - What does nosql stand for?</a>
* <a href="/post/file/whats-so-good-about-nosql-part-2/">Part 2 - Talking about APIs</a>

People use to think about schema-less dbs as a mess which they're not; People use to talk about schema-less dbs as non structured dbs without order - which is half true. Schema-less dbs do have a structure and usually it is a JSON-like one dynamically ordered and not typed. Even if that looks as a list of features you wouldn't  like your db to have, I'll try to show you why and when they're great and can make developers work easier so, let's do a small list of great uses cases before taking a deeper look to schema-less dbs,

<h4>Use Cases:</h4>

* Dynamic data storage
* Logging (Asterisk, Syslogs...)
* Web Apps (Blogs, Web Sites, CMS...)
* Almost anything you want.

I'm not going to say why nosql databases are great for each of the cases listed above but I'd like to talk a little bit more about the third case, Web Apps.

<h3> Web Apps and NoSql </h3>

It might be difficult to accept that a non-typed and non-related db system could be good for web applications. Everybody knows that web apps success is not just based on those fancy views and the things they do, their success is also based on how well they perform and fast they can complete requestes and operations.

<h5> How should data be modeled </h5>

First of all, data will always be data, no matter what you do what changes is the way you access it and how easy it could be to find the information you need. When changing form sql dbs to nosql ones it is common that people keeps thinking about how they can relate the model A with the model B. This is good, I'm not complaining but it's true that this is as good as embedding and/or storing information of model B in model A.

A friend of mine came to visit me and we were talking about diferences between RDBMS and nosql dbs (Specifically  <a href="http://www.mongodb.org">mongodb</a>) and he asked me: <em>How would you store the information of a group of people that are related to a group fo cities and at the same time are related to some churches the are related to some of the those cities? </em> I honestly think that this could be a good example, lets figure it out:

In a RDBMS it would useful and easier to do something like this:

* 5 tables (people, cities, churches, people_cities, churches_cities)

In a nosql db (<a href="http://www.mongodb.org">mongodb</a> for example) I would suggest something like this:

* 1 collection cities with 2 attributes (people and churches) which should be a list of dictionaries? strings? that's up to you.

<h5> What about queries? </h5>

Querying for cities that are related to a person (lets say Mario) in a RDMS would require doing a query over the relations table and joining it to the cities table to get the city information, in the other hand, the same query in a nosql db would require doing a query over the cities containing Mario in the people attribute. The important thing here is that we are reducing the number of queries, iterations and processes that should be done to get the information we need.

<h5> What about modifying our models? </h5>

During the web applications evolving process it is common that the original models have to be changed to satisfice the app needs and normally new attributes are added/removed to/from the model. When working with RDBMS it would require you to modify your tables and add/remove the needed columns so they respect the model structure and this usually is a long process that needs to be done carefully so you don't mess your tables but, if we're using a nosql database this process would be as easy/simple as modifying your models adding/removing the attributes and in some cases (when needed) add som data normalization methods (or backward compatibility) to preserve the data that was already in the db and save it using the new model structure.

As I always say. Models were created to model data but models can be modeled too so, there's no reason to be afraid about schema-less databases and if we are going to use one of them it is important to know that our application performance/speed/dynamism will be improved as much as we adapt our models to the database features.
