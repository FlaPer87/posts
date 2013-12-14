<!---
$"metadata"$
{
  "md": true,
  "title": "How Monblog uses Mongodb",
  "draft": false,
  "slug": "how-monblog-uses-mongodb",
  "tags": [
    "mongodb",
    "monblog",
    "gridfs"
  ]
}
$"metadata"$
-->


Recently, I released the first version of monblog. Monblog is a half-static blog engine I wrote based on some ideas taken from static blog engines and dynamic ones. In this post, I'd like to show some highlights about how monblog uses mongodb and most important, gridfs.

The idea behind monblog, which uses mongodb, is to provide a blog engine capable of interpreting static files (txt, markdown, html, ...) as posts entries while storing them into something more dynamic and easy to update like mongodb.

Some of the advantages of this implementation are:

* Fast render
* Distributed files that can be remotely accessed
* Statistics storage
* Automatic replication (depends on you're mongodb deployment)

I'm pretty sure that each one of those bullets could be implemented in both, dynamic and static engines but I do think there are some extra advantages when using them together.


About Mongodb
=============

In order to keep that fs deployment fashion style and to give some extra atomicity to each post, monblog stores posts entries inside [gridfs](http://docs.mongodb.org/manual/applications/gridfs/) under the `posts` collection and uses each file's `metadata` field to save all its attributes. Let me spend some words here.

Metadata
--------

Each post entry can define (usually at the very top of the file) a metadata section containing its attributes by using the following format:


    $"metadata"$
    {
      "md": true,
      "title": "How Monblog uses Mongodb",
      "draft": false,
      "slug": "how-monblog-uses-mongodb",
      "tags": [
        "mongodb",
        "monblog",
        "gridfs"
      ]
    }
    $"metadata"$


This section is a simple json-serialized map between 2 `$"metadata"$` tags and it's parsed during the post's upload process. In it, it is possible to define attributes like tags, upload dates, post format or even mark it as draft.


Posts
-----

In order to retrieve the posts list to show monblog does:

    def _posts(query=None):
        try:
            page = int(request.values.get("page", 0))
        except ValueError:
            page = 0

        posts = db.find("posts.files", query or {}).\
            sort([("uploadDate", -1)]).\
            skip(page * PAGE_SIZE).limit(PAGE_SIZE)
        return render_template('%s/posts.html' % conf.TEMPLATE_THEME,
                                posts=posts, query=query)

One of the things to notice here is the fact that it is possible to use gridfs' `.files` collection for querying, updating and storing data as well. In this case, monblog uses it for storing metadata and stats for each post, i.e: every time a post is read it increases the `reads` counter in posts' `metadata` field:

    @app.route('/post/<post_id>/', methods=["GET"])
    def get_post(post_id):
      post = db.fs.get(ObjectId(post_id))
      db.update("posts.files",
                {"_id": post._id},
                {"$inc": {"metadata.reads": 1}})
      return render_template('%s/post.html' % conf.TEMPLATE_THEME,
                             post=post, **post.metadata)

In the above example monblog uses both, normal collection access for updating reads counter and gridfs instance for getting the post from mongodb.

In the previous example, monblog uses mongodb's `ObjectId` to retrieve the post from gridfs but, what If you'd like to use the post's slug instead? Since slug is being used as filename, it is possible to:

    @app.route('/post/file/<filename>/', methods=["GET"])
    def get_post_by_filename(filename):
        post = db.fs.get_last_version(filename=filename)
        db.update("posts.files",
                  {"_id": post._id},
                  {"$inc": {"metadata.reads": 1}})
        return redirect(url_for('get_post', post_id=post._id))


Static Files
------------

Monblog currently keeps static files outside mongodb, but it could be a good idea to store them inside mongodb. Since monblog could be installed in multiple nodes, it would also be necessary to have static resources distributed as well and instead of doing that by copying them all around, we could save some space and keep them inside gridfs.

A drawback here is that it would be necessary to think about an easy way to updating those files, even though they wont be updated frequently.