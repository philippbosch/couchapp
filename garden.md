# The Garden

[The CouchApp Garden](http://couchapp.org/garden/_design/garden/index.html) 
is a CouchApp designed to make sharing other CouchApps easy.
Once you have the Garden installed on your CouchDB, you can use it to install
other CouchApps.

Note - links to couchapp.org here offline as of last edit.

## The basics

Currently, the Garden needs a lot of work, but the basic ideas are there.
Essentially, it can copy design documents from your other databases, into the
Garden database. As it copies them, it renames them so that they don't have
ids that start with `_design`. This means they can be replicated around
without the replicator having to run with admin privileges. Also, the apps
don't run code when they are just sitting in the Garden.

Once you have a local Garden database, you can install apps from it, into
databases on your CouchDB. The garden document will be copied to the target
database, as a design document again, and there will be a link to visit that
application.

## Sharing your app

To add your app to the Garden, install the Garden locally, and use its import
link, to add the app to your local garden database. Then replicate that
database to
[http://couchapp.org/garden](http://couchapp.org/garden),
and check out the updated [Garden](http://couchapp.org/garden/_design/garden/index.html).

## Contributing to the Garden

[The Garden code is on Github](http://github.com/jchris/garden), please fork and
contribute.



