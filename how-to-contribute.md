# Contributing

This repository holds all of the code in the project. The Python `couchapp`
script is the bulk of the repository, but the JavaScript stuff is in there to.

The jquery.couch.app.js and jquery.evently.js files are both 
[in the vendor directory](http://github.com/couchapp/couchapp/tree/master/vendor/_attachments/).

If you have a commit to one of the CouchApp files (JavaScript or otherwise) please let us
know on the [mailing list](http://groups.google.com/group/couchapp) as we don't
always get the messages in our Github inbox.

Also, documentation and blog posting is **very much appreciated**. Don't be
afraid to tell us how CouchApp sucks. We want it to be very easy to use, so
giving us a high bar to reach is important.

If you prefer developing mobile apps with Titanium, @pegli maintains a module
which wraps Couchbase Lite for that platform.

If you've built a sync powered app and are starting to hit the point where
Apache CouchDB filtered replication doesn't scale for you, you might want to
check out the Couchbase Sync Gateway which uses the same sync protocol but is
designed to give efficient subsets of a big data corpus to sync clients. So
you can sync to CouchDB or Couchbase Lite (nee TouchDB).

Or simply use rcouch a custom distribution of Apache CouchDB with a bunch of
new features that offers since a while incremental view changes (indexed on
the disk) and replication support using a view and allows you to replicate in
an efficient manner subsets of your databases.

Last thing, I think PouchDB is the future of browser based sync apps. It can
sync with the same sync protocols but uses the built-in storage of HTML5.
