# How To install Pages

If anyone else has issues understanding this whole vhosts thing i'll give a
'grandma-can-do-it' recount here of my troubles.

if you want to set up 'pages' on your machine and you are not very familiar
with what's what, here it goes:

get pages from github: [http://github.com/couchone/pages](http://github.com/couchone/pages)

navigate to your fav directory and do this in the terminal:

    
    
    git clone git://github.com/couchone/pages.git
    

make sure to install couchapp python helper app, i won't go into the details,
[the github instructions are great](http://wiki.github.com/couchapp/couchapp/manual-2)

Hope you have couchdbx installed on osx or the equivalent on your favourite
dev/production platform

do this from inside your pages directory:

    
    
    couchapp init  
    couchapp push . http://localhost:5984/pages
    

that will get you the app into your db, nothing new here, the git hub
instructions will tell you the same thing, what got me (besides a bad commit
;)) is the instruction on 
[http://blog.couch.io/post/443028592/whats-new-in-apache-couchdb-0-11-part-one-nice-urls](http://blog.couch.io/post/443028592/whats-new-in-apache-couchdb-0-11-part-one-nice-urls)

the section about vhosts is a bit ambiguous for those that aren't in the
know...

the instructions there are as follows:

"Each HTTP 1.1 request includes a mandatory header field Host: hostname.com
with the server name it is trying to reach. You can tell CouchDB to look for
that Host header and redirect all requests that match to any URL inside
CouchDB by adding this to your configuration file local.ini:

    
    
    [vhosts]  
    couch.io = /couchio/_design/app/_rewrite"
    

well, what the hell is local.ini?

who knows, who cares, go to your couchdbx app and navigate to the
"configuration" section.

go to the bottom of the page and "Add new section"

type into the 3 fields that popup:

  * `vhosts`
  * `your-pages-site-name:5984`
  * `/pages/_design/pages/_rewrite`

now go to the terminal and type:

    
    
    textmate /etc/hosts 
    

(notice i'm making assumptions here, basically, get to the hosts file and open
it...)

add:

    
    
    127.0.0.1 your-pages-site-name
    

save, go to a browser type:  
`your-pages-site-name:5984`

hopefully that worked out OK for you.

see ya!

