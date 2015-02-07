# Couchapp Command Line Usage

## [Installation Instructions](installing.md)

## Full command line usage

    
    
    $ couchapp help
    Usage: couchapp [OPTIONS] [CMD] [CMDOPTIONS] [ARGS,...]
    
    Options:
        -d, --debug 
        -h, --help 
        --version 
        -v, --verbose 
        -q, --quiet 
    
    Commands:
        autopush [OPTION]... [COUCHAPPDIR] DEST
            --no-atomic          send attachments one by one
            --no-inotify         Don't use inotify api
            --update-delay [VAL] time between each update
    
    
        browse   [COUCHAPPDIR] DEST
    
        clone    [OPTION]...[-r REV] SOURCE [COUCHAPPDIR]
            -r, --rev [VAL] clone specific revision
    
    
        generate [OPTION]... [app|view,list,show,filter,function,vendor] [COUCHAPPDIR] NAME
            --template [VAL] template name
    
    
        help     
    
        init     [COUCHAPPDIR]
    
        push     [OPTION]... [COUCHAPPDIR] DEST
            --no-atomic    send attachments one by one
            --export       don't do push, just export doc to stdout
            --output [VAL] if export is selected, output to the file
            -b, --browse   open the couchapp in the browser
            --force        force attachments sending
            --docid [VAL]  set docid
    
    
        pushapps [OPTION]... SOURCE DEST
            --no-atomic    send attachments one by one
            --export       don't do push, just export doc to stdout
            --output [VAL] if export is selected, output to the file
            -b, --browse   open the couchapp in the browser
            --force        force attachments sending
    
    
        pushdocs [OPTION]... SOURCE DEST
            --no-atomic    send attachments one by one
            --export       don't do push, just export doc to stdout
            --output [VAL] if export is selected, output to the file
            -b, --browse   open the couchapp in the browser
            --force        force attachments sending
    
    
        vendor   [OPTION]...[-f] install|update [COUCHAPPDIR] SOURCE
            -f, --force  force install or update
    
    
        version  
    

## Commands

### generate

Generates a basic CouchApp, also used to create function templates (ie. view,
list, show, filter, etc.).

    
    
        couchapp generate myapp
        cd myapp
        couchapp generate view someview
    

### init

Initialize a CouchApp. Run this command in your application's folder to create
two JSON encoded configuration files, `.couchapprc` for server configuration
and `.couchappignore` for a blacklist of files that will not be uploaded. This
command can be used to initialize a cloned application from external
repository (git, hg).

    
    
        cd mycouchapp
        couchapp init
    

### push

Push a CouchApp to one or more
[CouchDB](http://couchdb.apache.org/) servers

    
    
        cd mycouchapp
        couchapp push http://someserver:port/mydb
    

  * `\--no-atomic` sends attachments one by one. By default, all attachments are sent inline.
  * `\--export` only outputs JSON to `stdout`. Use in conjunction with `\--output` to save to a file.
  * `\--force` force attachment sending
  * `\--docid` sets a custom document id for this CouchApp. (ie. `_design/someid`)

### pushapps

Sends a folder of multiple CouchApps at once, similar to
[push](push.md).

    
    
        couchapp pushapps somedir/
    

### pushdocs

Sends a folder of JSON documents, used to populate your CouchDB database,
similar to (push apps)[pushapps] but for documents only. Alternatively, you
may create a top-level `_docs` folder.

### browse

Browse your CouchApp in the browser.

To browse with "official" CouchDBs

    
    
         couchapp browse . http://host:port/mydb
         or 
         couchapp browse . mydb (for default url) 
    

For desktopcouch on Ubuntu

    
    
         couchapp browse . desktopcouch://mydb
    

