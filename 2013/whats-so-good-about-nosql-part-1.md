<!---
$"metadata"$
{"md": true, "upload_date": "2010-05-09 18:57:09", "title": "What's so good about nosql? - Part 1 - What does nosql stand for?", "draft": false, "slug": "whats-so-good-about-nosql-part-1", "tags": []}
$"metadata"$
-->
Hey Ho,

This is the first in a series of posts that aim to explain why nosql databases are so great (in some areas) over relational ones. In this post I'll explain what nosql is and list some of the main nosql families and databases.

I'll start quoting the <a href="http://nosql-database.org/">nosql-database</a> definition about NoSQL databases:

<p><i>"Next Generation Databases mostly addressing some of the points: being <strong>non-relational, distributed, open-source and horizontal scalable</strong>. The original intention has been modern web-scale databases. The movement began early 2009 and is growing rapidly. Often more characteristics apply as: <strong>schema-free, easy replication support, simple API, eventually consistent / BASE (not ACID), and more</strong>. So the misleading term "nosql" (the community now translates it mostly with "not only sql") should be seen as an alias to something like the definition above."</i></p>

Let me explain some of the main nosql databases characteristics.

* <strong>schema-less</strong>: This means that it can store anything in any order, no matter what type it is or how it is structured. Think of it as a toys box that can be used to store anything you want, yeah, just like that, there's not going to be a box for G-I Joes and another one for cars, you just put anything you want to store in there and that's it, there are no moms saying "You told me this box was for cars and that one for G-I Joes, you can't put your cars in the G-I Joes box, there're not of the same type"... Who cares? At the end, they're all toys, Racist!

* <strong>horizontal scalable</strong>: Didn't you ever had to carry many books at the same time when you were at school? Well, I didn't, but, this is a good example to explain horizontal scalable systems. If I would had to do something like that I'm sure I would had called my friends and asked them for help, woudn't you? Well, that's what horizontal scalable systems do, they share operations/data horizontally between the many nodes running and release single nodes from back pains.

* <strong>Simple API</strong> Last but not Least.  A database is not just about how data is managed, indexed or processed, it also has to guarantee an easy way to access data from outside (the so called drivers). When we talk about simple API we're not talking about well defined methods with understandable names that can be learnt easily, this part is important too but, it is also important how data gets exposed by databases. Most of the nosql databases use simple json encoded messages and simple protocols like REST, Simple HTTP, TCP Sockets and so on. What else can we need?

Now that we know what a nosql database is we should also know how they're grouped, let me list some of the families and some of the well known databases:

<h4><strong>Wide Column Store</strong></h4>

* <a href="http://cassandra.apache.org/">Cassandra</a>: I'm sure you know this one, no? hmm, lets see, have you ever used Facebook? Well, They started this project.

* <a href="http://hadoop.apache.org/">Hadoop</a>: Another apache project, I've never used it before but it really looks awesome. Take a look to <a href="http://hadoop.apache.org/pig/">PIG</a> and you'll know what I mean.


<h4><strong>Document Store</strong></h4>

* <a href="http://www.mongodb.org">Mongo</a>: I love it and I use it all the time so stop reading this blog post and start reading about mongodb if you haven't done yet.

* <a href="http://couchdb.apache.org/">CouchDB</a>: Another apache project. God, this guys have many projects. This one is written in erlang.

<h4><strong>Key Value / Tuple Store</strong></h4>

* <a href="http://memcachedb.org/">MemcacheDB</a>: Well known and used by many people. We, for example, use it as Django cache back-end.

* <a href="http://code.google.com/p/redis/">Redis</a>: Well known and used by many people as well.


<h4><strong>Graph Databases</strong></h4>

* <a href="http://neo4j.org/">Neo4j</a>: Wow, you have to see it running...


<h3>References: </h3>
* <a href="http://nosql-database.org/">nosql-database</a>

<h3>Update: </h3>

For further reading take a look at:

* <a href="http://www.jroller.com/dmdevito/entry/thinking_about_nosql_databases_classification">Thinking about NoSQL databases (classification and use cases)</a>