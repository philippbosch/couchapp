# CouchApps and DesktopCouch

In **version 0.7**, Couchapp has a new feature to couchapp allowing you to
push, clone and browse CouchApps in the local CouchDB installed with 
[desktopcouch](http://freedesktop.org/wiki/Specifications/desktopcouch), 
so ubuntu users (or other linux distributions where desktopcouch has
been ported) won't have to install another CouchDB to test and will be able to
pair it with other desktop.

# How it works?

to push to your local couchdb installed with desktopcouch:

    
    
    couchapp push desktopcouch://testdb 
    

To clone :

    
    
    couchapp clone desktopcouch://testdb/_design/test test1 
    

To browse and use your application:

    
    
    couchapp browse . desktopcouch://mydb 
    

and with push option :

    
    
    couchapp push --browse . desktopcouch://mydb
    

