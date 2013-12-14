<!---
$"metadata"$
{"md": true, "upload_date": "2011-03-22 08:06:27", "title": "Asterisk and 0MQ", "draft": false, "slug": "asterisk-and-0mq", "tags": ["asterisk", "zmq", "0mq", "call", "prediction"]}
$"metadata"$
-->
Hey ho!!!

Here's a little project I started last weekend.  It aims to replicate <a href="http://www.asterisk.org/" target="_blank">asterisk</a>'s manager's actions using <a href="http://www.zeromq.org/" target="_blank">zmq</a> and json. Right now it just handles the originate command which makes possible the execution of MANY calls (asynchronously). This might be useful for prediction algorithms that need to execute as many calls as possible in a short time. It also has a simple thread pool to handle concurrency, it makes possible to start as many workers as needed.

As I said, I just started it which means it needs a lot of work. Does any of you want to fork/help?

Some things that It should have:

* Bulk Originate.
* Call Status/Result Inf. (Maybe using a push socket)
* Callback when a originate ends (successfully or not) ?

Any other Idea?

Heres the github link: <a href="https://github.com/FlaPer87/asterisk-zmq-manager" target="_blank">https://github.com/FlaPer87/asterisk-zmq-manager</a>