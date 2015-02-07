# What the HTTP is CouchApp?

This blog post is in response to a lot of well-deserved confusion in the
community around CouchApps. We haven't been clear enough in the past (either
in technical description or in the notion of the project). I hope to change
all that (with your help). This is just the beginning.

## The Basics

A CouchApp is just a JavaScript and HTML5 app that can be served directly to
the browser from CouchDB, without any other software in the stack. There are
many benefits (and some constraints) to doing it this way. The first section
of this article will address these tradeoffs.

In the bad old days (2008 and earlier), if you wanted to write a dynamic
database-backed web application, you had to have an architecture that looked
like a layer cake:

    
    
    Browser (UI and links between pages)
    -------------- HTTP ---------------
    Application server (business logic, templates, etc)
    --------- custom binary -----------
    Persistence: MySQL, PostgreSQL, Oracle
    

In fact, the bad old days are still with us, as most applications still rely
on fragile custom code, running in an application server like Ruby on Rails,
Python's Django, or some kinda Java thing. The pain-points of the 3 tier
architecture are well known: application developers must understand the
concept of shared-nothing state, or else clients can see inconsistent results
if they are load balanced across a cluster of app servers. The application
server is usually a memory hog. And at the end of the day, when you've finally
gotten the app layer to horizontal scalability, it turns our that your
database-tier has fatal scalability flaws...

CouchDB is an HTTP server, capable of serving HTML directly to the browser. It
is also a database designed from the ground up for horizontal scalability. Did
I say silver bullet? ;) (Of course it is not a silver bullet -- if you can't
fit your app into CouchDB's constraints, you'll still have scaling issues.) If
you can build your app _with the grain_ of CouchDB's APIs, then you can
piggyback on all the work
[other](http://enda.squarespace.com/tech/2010/3/4/couchdb-at-scale-4-billion-requests-so-far.html) people have done to scale.

The fact is, 2-layer applications are simpler:

    
    
    Browser (UI and links between pages)
    -------------- HTTP ---------------
    CouchDB (persistence, business logic, and templating)
    

Because CouchDB is a web server, you can serve applications directly the
browser without any middle tier. When I'm feeling punchy, I like to call the
traditional application server stack "extra code to make CouchDB uglier and
slower."

Aside from simplicity and the scalability that comes with it, there is another
major benefit to creating a 100% pure CouchApp:
[Replication.](http://wiki.apache.org/couchdb/Replication)
When your app is hosted by just a CouchDB, that means it can be run from _any_
CouchDB, with no need to set up complex server-side dependencies. When your
app can run on _any_ CouchDB, you are free to take advantage of CouchDB's
killer feature: replicating the app and the data anywhere on the network.

Have you ever been frustrated by a slow website? Filling out forms and waiting
even a few seconds for the response can be infuriating. Many users will hit
the submit button over and over again, compounding whatever performance issues
that are effecting them, while introducing data integrity issues as well.
Google, Facebook, and other large competitive web properies know that
[perceived latency drives user engagement like nothing else, and they invest huge sums to make their sites seem faster.](http://perspectives.mvdirona.com/2009/10/31/TheCostOfLatency.aspx)

Your site can be faster than theirs, if you serve it from localhost. CouchDB
makes this possible. Here are 
[installers for OSX, Windows, and Linux](http://couch.io/get) 
and you can install [CouchDB on Android here](http://couch.io/android).

The take-home message from this section is: CouchDB can scale. If your app is
served by raw CouchDB, it can scale just the same. Also, there's no server
faster than the server running on your local device. And fast is what matters
for users.

In the next section we'll see what it takes to get your app to be served
directly from CouchDB, and what you can (and can't) do.

## CouchDB's built-in programming model

The CouchDB API is full featured and applicable to a lot of use cases. I can't
possible go in-depth here. Instead I'll focus only on the broad outline, and
on what is useful and necessary for CouchApps. If you want to learn more,
check out [the CouchDB wiki](http://wiki.apache.org/couchdb) or the 
[free CouchDB book](http://books.couchdb.org/).

The first thing to understand about CouchDB is that the entire API is HTTP.
Data is stored and retrieved using the protocol your browser is good at. Even
[the CouchDB test suite](http://jchris.couchone.com/_utils/couch_tests.html?script/couch_tests.js) is written in JavaScript and executed
from the browser. **It's all just HTTP.**

### HTML Attachments

A common question I get from people starting to write Ajax apps using CouchDB,
is "when I try to query the CouchDB with jQuery, it doesn't work." Usually it
turns out that they have an index.html file on their filesystem, which is
attempting to do an Ajax call against the CouchDB server. After I explain to
them [the same origin security policy](http://en.wikipedia.org/wiki/Same_origin_policy),
they start to understand this this means CouchDB needs to serve their HTML
(rather than loading it in the browser direct from the filesystem).

CouchDB documents may have [binary attachments.](http://stackoverflow.com/questions/1439371/creating-a-couchdb-standalone-attachment-using-curl) 
The easiest way to add an attachment to a document is via Futon.
We'll do that later in this blog post.

So, the simplest possible CouchApp is just an HTML file, served directly from
CouchDB, that uses Ajax to load and save data from the CouchDB.

### Map Reduce queries

What sets CouchDB apart from a simple key value store like Memcached or
[Amazon S3](http://aws.amazon.com/s3/), is that you can
query it by building indexes across the stored objects. You do this by writing
JavaScript functions that are passed each of your documents, and can pick from
them a set of keys under which you'd like to locate them.

So for a blog post, you might pick out all the tags, and make keys like 
`[tag, doc.created_at]`. 
Once you have a view like that, you can easily get a view of
all your blog posts with a given tag, in chronological order, no less. By
adding the reduce operator `_count` you can also see how many blog posts are
tagged `"foo"` or whatever.

I'm not gonna try to teach you all about views here. Try the 
[CouchDB book's guide to views](http://books.couchdb.org/relax/design-documents/views), the
[wiki](http://wiki.apache.org/couchdb/HTTP_view_API) and
this [chapter on advanced views](http://books.couchdb.org/relax/reference/views-for-sql-jockeys).

### Server Side Validations

The second thing people usually ask when they start to grok the CouchApp
model, is "How do I keep people from destroying all my data? How do I ensure
they only do what they are allowed to do?" The answer to that is 
[validation functions](http://books.couchdb.org/relax/design-documents/validation-functions). 
In a nutshell, each time someone saves or updates a
CouchDB document, it is passed to your validation function, which has the
option to throw an error. It can either throw `{"forbidden" : "no matter
what"}` or `{"unauthorized" : "maybe if you login as someone else"}` where, of
course, you are free to craft your own messages. If the function doesn't have
any errors, the save is allowed to proceed.

### Rendering Dynamic HTML

After a new user understands validation functions, they have begun to see that
perhaps CouchDB / CouchApps is a good candidate for their application. But
maybe something is missing... Search engines don't treat Ajax applications
with the same respect they do static HTML applications. Also, a fair
proportion of users have JavaScript disabled, or are using a screen-reader
type application, which may not understand Ajax.

These are all great reasons your application should ship the basic content of
a page as real-deal HTML. Luckily, CouchDB has an answer to that as well.

[Show functions](http://books.couchdb.org/relax/design-documents/shows) 
allow you to transform a document `GET` from JSON into the
format of your choice. On [this wiki application](http://pages.jchris.couchone.com/page/donk)
the main wiki content is rendered as server-side HTML, using a show function.
You can also use a show function to provide an XML, CSV, or even PNG version
of your original document. Some folks also use it to filter security-sensitive
fields from a JSON document, so that only public data is available to end-
users.

[List functions](http://books.couchdb.org/relax/design-documents/lists) are the analog of show functions, but for view results. A
view result is just a long list of JSON rows. A list function transforms those
rows to other formats. Here is 
[the JSON view of recent posts on my blog](http://jchrisa.net/drl/_design/sofa/_view/recent-posts?descending=true&limit=5), and here is 
[the HTML page that results from running that same view through a list function.](http://jchrisa.net/drl/_design/sofa/_list/index/recent-posts?descending=true&limit=5)

We added these capabilities to CouchDB because we knew that without the
ability to serve plain-old HTML, we wouldn't be completely RESTful.

Rounding out this group, is the ability to accept plain HTML form POSTs. (And
other arbitary input). For that, CouchDB uses [update functions](update-functions.md),
which can take arbitrary input and turn it in to JSON for saving to the
database.

### Pretty URLs

"All well and good", you may say, "but I can't really suggest to my clients
that their website should have URLs like
`[http://jchrisa.net/drl/_design/sofa/_list/index/recent-posts?descending=true&limit=5`](http://jchrisa.net/drl/_design/sofa/_list/index/recent-posts?descending=true&limit=5)!"

I used to respond with skepicism to such claims, _like a total moaf_. But I've
mended my ways, and seen the light. It also didn't hurt that
[Benoit](http://twitter.com/benoitc) committed an 
[awesome rewriter](http://blog.couchbase.com/what%E2%80%99s-new-apache-couchdb-011-%E2%80%94-part-one-nice-urls-rewrite-rules-and-virtual-hosts) 
to CouchDB, so we can provide nice pretty URLs like `/posts/recent`
instead of the above mess.

### Realtime Updates

Lastly, something folks don't usually ask for, but which is insanely useful:
[realtime notification about changes to the database.](http://books.couchdb.org/relax/reference/change-notifications) Essentially, CouchDB keeps a record of the order in
which operations were applied to a given database. This way, you can always
ask it "what's happened since the last time I asked?"

CouchDB implements this with the `_changes` feed, a JSON HTTP response, which
sends a single line, whenever something happens to the database. Since CouchDB
is implemented in Erlang, it is not expensive for it to hold open tens of
thousands of concurrent connections.

The `_changes` feed can be used to power realtime updates to a browser UI. For
instance, [this chat room](http://couchapp.org/example/_design/example/index.html) updates in realtime whenever a new message is
created.

The `_changes` feed is integral to CouchDB itself (not just a bolted on
feature), as it is used to power to replication itself. The replicator listens
to the changes feed of the source database, and writes changes to the target
database. This is what allows CouchDB to keep 2 database in sync in near
realtime.

You can also use `_changes` to drive asynchronous business logic. There will
be a webcast in August on this topic, as well as a blog post with more
details, from Couchio's [Jason Smith](http://twitter.com/_jhs).

### Filtered replication

One last part of the programming model. You can write a JavaScript function
that decides whether to include a given change in the `_changes` feed. The
possibilities are endless. See 
[Jan's blog post on new replication features](http://blog.couch.io/post/468392274/whats-new-in-apache-couchdb-0-11-part-three-new) 
for some interesting use-cases that might stimulate your imagination.

## Hello World

Now that I've described the theory of CouchApps to you, let's dig into the
practice. Before we get into the expert toolchain, let's see what we can do
with a little bit of HTML. I'll assume you have a CouchDB running at
localhost. If you don't, install one now (or signup for hosting at 
[Iris Couch](http://www.iriscouch.com/) or
[Cloudant](http://cloudant.com/)).

Quick, create a file called index.html, and put this in it:

    
    
    <!DOCTYPE html>
    <html>
      <head><title>Tiny CouchApp</title></head>
      <body>
        <h1>Tiny CouchApp</h1>
        <ul id="databases"></ul>
      </body>
      <script src="/_utils/script/jquery.js"></script>
      <script src="/_utils/script/jquery.couch.js"></script>
      <script>
        $.couch.allDbs({
          success : function(dbs) {
            dbs.forEach(function(db) {
              $("#databases").append('<li><a href="/_utils/database.html?'+db+'">'+db+'</a></li>');
            });
          }
        });
    
      </script>
    </html>
    

Now browse to your CouchDB's Futon at 
[http://localhost:5984/_utils](http://localhost:5984/_utils) 
and create a database called
"whatever". Now visit that database, and create a document. You will be
creating what is known as a, "Design Document", which is a special kind of
document in CouchDb that contains application code. The only thing you need to
know now is to set the document id to something that begins with "_design/"
and save it. Now click the button labeled "Upload Attachment" and choose the
index.html file you just created, and upload it. Now click the link in Futon
for index.html, and you should see a list of the databases on that CouchDB
instance.

(rengel, 2012-09-05: Because of the »Same Origin« policy the index.html file
has to be in the same directory, or a subdirectory thereof, as your whatever
database.)

You gotta admit there was nothing to that.

## Make it easy it with the CouchApp toolchain

Now that we've seen how you can build a basic CouchApp with the same set of
tools you'd use to do plain-old HTML, CSS, and JavaScript development, let's
learn how the experts (and the lazy!) do it.

Uploading each changed file to CouchDB via Futon would get tedious quick.
Alternatively, you could download the entire design document as JSON, and edit
that JSON in your editor... but keeping track of proper JSON escaping and
formattng is a task better done by a machine.

Back in the early days of CouchDB, I solved this problem with a Ruby script
that would update my map and reduce function from a folder. This way I could
open the folder in TextMate, and get all the proper JavaScript syntax
highlighting. To deploy the changes I'd run the Ruby script, and CouchDB would
have my new Map Reduce views.

That would have been the end of the story, except that for some reason, many
people had boatloads of trouble installing the Ruby script. I may have been
suffering from a bit of "grass is always greener," because my reaction was to
port the Ruby stuff to Python (with a little help from my friends), which I
thought would have a cleaner install story. (It almost does!)

Since then [the Python CouchApp script](http://github.com/couchapp/couchapp/) has grown in
capability. It boasts the ability to 
[push edits in real time](http://groups.google.com/group/couchapp/browse_thread/thread/67ccbdcdf9023106), 
import vendor modules, and more. Benoit Chesneau keeps it up to date
pretty agressively, it just got some [GeoCouch features today.](http://github.com/couchapp/couchapp/commit/9ff4ec09664a286f0c408ac76eb9c5589a56e208)

So let's use it!

### Installing CouchApp

There is a lot of documentation already out there about how to install the
CouchApp toolchain. I'll just link to it. The basic installation instructions
are [in the README](http://github.com/couchapp/couchapp)
and in [the CouchDB Book](http://books.couchdb.org/relax/example-app/design-documents).

Here are some hints about [installing on Windows.](http://wiki.apache.org/couchdb/Quirks_on_Windows)

Once you have CouchApp installed, the basic usage is simple. From within your
application directory, issue the following command.

    
    
    couchapp push . http://myname:mypass@localhost:5984/mydb
    

Replace `myname` and `mypass` with those you set up on your CouchDB using
Futon. If you didn't setup an admin password on Futon, you should do that --
until you do, your CouchDB can be administered by anyone. Also, if you are
running a CouchDB in the cloud, you'll need to replace `localhost:5984` with
something like `mycouch.couchone.com`. Also, of course, `mydb` should be
changed to the name of the database you want your program to live in.

All this is coverered in great detail in the CouchApp README and the book, as
linked above.

### The Standard Library

We've made it nearly to the end of this post. The last thing to cover are the
various JavaScript libraries for making CouchApps. I won't try to document
them, just name them, and say a little about their purpose.

I have a mental plan to clean up and consolidate some of these libraries, so
they are more modular. This should make it so that CouchApp code loads faster,
among other things.

#### The jQuery CouchDB Client API

We already used [jquery.couch.js](http://daleharvey.github.com/jquery.couch.js-docs/symbols/index.html) in the Tiny CouchApp example
HTML above. This is the basic CouchDB library for jQuery. It handles things
like saveDoc and openDoc, view queries, replication requests, etc. Essentially
it wraps the CouchDB API in Ajaxy goodness. This library ships as part of
CouchDB, as it is used by Futon.

#### The CouchApp Code Loader

The CouchApp toolchain ships with [jquery.couch.app.js](http://github.com/couchapp/couchapp/blob/master/vendor/_attachments/jquery.couch.app.js), 
which is tasked with one job -- loading your application code into
the page. This CouchApp jQuery plugin loads your design document (the JSON
saved as a result of a `couchapp push` command), so that the browser has
access to your view definitions, show and list functions, etc. It is invoked
like so:

    
    
    $.couch.app(function(app){
      // app.db is your jquery.couch.js object
      // app.require("lib/foo") gives you access to libraries
    });
    

Essentially, all this function does, is inspect the page you are on, determine
how to load the design document, load it, and gives you an object that
references it and allows you to require libraries from it. (There is some
legacy featuritis in there, but I'm working to remove that.)

#### Evently

[Evently](http://github.com/couchapp/couchapp/blob/master/vendor/_attachments/jquery.evently.js) 
is a convenience library I wrote for
myself. Essentially it does two things: One is it allows you to write complex
jQuery code in a declarative fashion. This makes code reuse easier, by
avoiding the tangled web of dependencies you often see in deeply nested jQuery
code. The other is that Evently knows a bunch of CouchDB tricks, so you can
get it to run a view query and hand you the results, without having to write
nested callbacks in the Ajax style.

As a coincidence, Evently's declarative structures happen to map onto JSON
objects nicely. It also happens that `couchapp push` maps filesystem
structures to JSON code as well. It was only after I'd written lots of Evently
code all in one file, that I realized I could nest the JSON structures into a
tree of folders and JavaScript files.

People think that Evently and the deeply-nested folders and files things must
go together, but it is just one way of doing things. For more 
[Evently docs, visit this link](http://couchapp.couchone.com/docs/_design/docs/index.html#/topic/evently).

#### Pathbinder

[Pathbinder](http://github.com/couchapp/couchapp/blob/master/vendor/_attachments/jquery.pathbinder.js) 
makes it so you can assign events
to be triggered when the hash part of the browser URL changes. Evently knows
how to use it, so you can declaratively link paths and events. 
[Pathbinder docs are here.](http://couchapp.couchone.com/docs/_design/docs/index.html#/topic/pathbinder)

### Examples

There is a [list of CouchApps here](list-of-couchapps.md)

