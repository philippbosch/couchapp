# Using couchapp with multiple design documents

Here is what I did to use couchapp with multiple design documents. I want to
setup a project for a new database test6 with a design doc called design_doc1

Make sure couchdb and couchapp are installed and that couchdb is started.

First check that test6 doesn't exist

    
    
    $ curl http://127.0.0.1:5984/test6
    
    {"error":"not_found","reason":"no_db_file"}
    

OK. That was expected.

Generate a new couchapp.

    
    
    $ couchapp generate test6
    

On my machine (Debian squeeze), couchapp seems to generate the directory with
owner root. Change owner to john

    
    
    $ chown -R john ./test6
    $ cd test6
    $ ls
    
    _attachments   evently  language  shows    vendor
    couchapp.json  _id      lists     updates  views
    
    $ vi .couchapprc
    

Now edit .couchapprc as follows so it looks like

    
    
    { "env":
      { "default":
        {  "db":"http://127.0.0.1:5984/test6" }
      }
    }
    

Check that it was saved correctly:

    
    
    $ cat  .couchapprc
    
    { "env":
      { "default":
        {  "db":"http://127.0.0.1:5984/test6" }
      }
    }
    

NOTE: It looks like couchapp doesn't pick up the default db in what follows
when I do couchapp push

Make a directory for design documents:

    
    
    $ mkdir _design
    

Make a directory for design_doc1

    
    
    $ mkdir _design/design_doc1
    

move the design doc files created with couchapp generate to the design_doc1
directory:

    
    
    $ mv _attachments evently lists shows updates vendor views ./_design/design_doc1
    

review the directory structure

    
    
    $ ls  _design/design_doc1
    
    _attachments  evently  lists  shows  updates  vendor  views
    

now push design_doc1. Note that I have to include the url of the database as a
parameter. Couchapp doesn't seem to pick up the default db when I push from
the _design directory.

    
    
    http://127.0.0.1:5984/test6     is the url of the new db
    
    $ couchapp push _design/design_doc1 http://127.0.0.1:5984/test6
    2010-08-23 15:47:45 [INFO] Visit your CouchApp here:
    http://127.0.0.1:5984/test6/_design/design_doc1/index.html
    

Now check to see if db test6 was created:

    
    
        $ curl http://127.0.0.1:5984/test6
    {"db_name":"test6","doc_count":1,"doc_del_count":0,"update_seq":1,"purge_seq":0,"compact_running":false,"disk_size":106585,"instance_start_time":"1282603665650439","disk_format_version":5}
    

Now go into a browser and take a look at the test6 db

    
    
    http://127.0.0.1:5984/_utils/database.html?test6
    

You should see "_design/design_doc1" listed on the html page. That's good, it
means that design_doc1 was created.

Take a look at design_doc1 in the futon web admin. Open this URL in your
browser:

    
    
    http://127.0.0.1:5984/_utils/document.html?test6/_design/design_doc1
    

You should see a nice listing of the design_doc1. Try opening the index page
in your browser:

    
    
    http://127.0.0.1:5984/_utils/document.html?test6/_design/design_doc1/index.html
    

This should serve up index.html from the _attachments subdirectory
test6/_design/design_doc1/_attachments/index.html

Couchapp generate had created a sample view called recent-items. Try querying
it:

    
    
        $ curl http://127.0.0.1:5984/test6/_design/design_doc1/_view/recent-items
    
    {"total_rows":0,"offset":0,"rows":[]}
    

That's it. Multiple design can be used to create different interfaces for
users with different roles. For example, consider some data and the different
ways that and admin versus a regular user interacting with it.

