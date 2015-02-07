# Notes on Pages Application

* * *

# Pages Wiki Application

* * *

Disclaimer: This is my current (3 day) understanding of the way it works. I
could be totally/probably wrong but I am sure someone will help and point that
out. Apologies in advance for the non-javascripty language, it is mostly a
brain-dump so reflects my biases.

The order of the document is based on the layout of the files in TextMate
while examining the Pages project.

* * *

# Evently Widgets

* * *

  1. The profile widget extends the core couchapp profile widget. It is used to create dynamic options in the navigation depending on login status. A duplicate folder with the same name is created in the higher level evently folder. Then the mustache templates (all.html and mustache.html in both loggedOut and profileReady folders) override their core counterparts and display a modified appearance compared to the default.
  2. The tools widget provides a history popup and upload facility to the page.
  3. The wiki widget reacts to editing and version requests 
  4. The comments widget displays the comments below the wiki page and provides a method for adding new ones.

* * *

# File Structure and Individual Notes

* * *

* * *

## _attachments (folder)

* * *

This folder holds all the files that will be uploaded as binary attachments
when the app is pushed to the server. They are then available directly from a
url built as follows

http://127.0.0.1:5984/databasename/_design/designdocname/uriencodedfilepathandname.xyz

So files nested in folders are requested by replacing the / with %2f

    
    
    http://127.0.0.1:5984/databasename/_design/designdocname/style%2fmain.css
    

Typically you will put anything in here that you don't want json encoded and
escaped such as html, css, images etc.

> Note: The whole vendor subdirectory is also stored this way in the couchdb.

* * *

### index.html

* * *

This file can represent the scaffolding that holds the structures that will be
replaced dynamically when individual divs are replaced with evently mustache
content. In the case of "pages" it just has a body because the pages of the
wiki are dynamically generated and it uses templates/page.html more often, but
the default couchapp generates with account,profile and sidebar divs.

It has a reference to the main.css file for styling but no javascript code at
all.

* * *

### script (folder)

* * *

This looks like it should hold any non-core javascript libraries that are used
in the application including in this case myloader.js.

* * *

#### myloader.js

* * *

myloader.js is a small function to add javascript library references to the
index.html page so they are available for use.

The core jQuery library, json library, couch library and the secure hash
library are loaded directly from the filesystem (on OSX _utils maps to a
directory located at /Applications/CouchDBX-
R13B04-trunk_983493.app/Contents/Resources/couchdbx-
core/couchdb/share/couchdb/www/)

    
    
    couchapp_load([
      "/_utils/script/sha1.js",
      "/_utils/script/json2.js",
      "/_utils/script/jquery.js",
      "/_utils/script/jquery.couch.js",
      "../vendor/couchapp/jquery.couch.app.js",
      "../vendor/couchapp/jquery.couch.app.util.js",
      "../vendor/couchapp/jquery.mustache.js",
      "../vendor/couchapp/jquery.pathbinder.js",
      "../vendor/couchapp/jquery.evently.js"
    ]);
    

The other jQuery plugins do the heavy lifting in communicating with the
couchdb, creating final html with mustache, triggering and reacting to events
in the browser with evently, and binding events to a url while making them
bookmarkable.

Pathbinder is used in the logic to select the proper evently reaction and pass
parameters.

* * *

### style (folder)

* * *

Holds all the css files for styling your static and dynamic html.

* * *

#### base-min.css

* * *

The usual css reset

* * *

#### main.css

* * *

Where all the bits and pieces get their look and feel

* * *

### _id

* * *

This just names the design document in the database. It is created
automatically on generation from the name you supplied.

* * *

### couchapp.json

* * *

Single json global object that holds metadata about the application for use by
other parts of the application. Typically it includes the human name of the
app, a description and with rewrites turned on, the default start page.

* * *

### evently (folder)

* * *

At this level, the user made widgets are contained in this folder and any with
similar names to the core evently widgets override their counterparts.

* * *

#### comments (folder)

* * *

Container for the Comments widget.

* * *

##### _init (folder)

* * *

The _init action is fired automatically upon the widget being called. It can
either be a single javascript file, or alternatively a folder with collected
javascript, templates and json to acheive what is required. In Pages it is a
folder.

* * *

###### after.js

* * *

after.js is run last in the widget :) In this case, it is a simply uses jQuery
to select all items with the class "date" and applies the prettyDate function
from jquery.couch.app.util.js to it to make friendly dates like "yesterday"
and "2 months ago".

* * *

###### data.js

* * *

data.js provides the dynamic data from couchdb to be used in displaying the
mustache template. It takes a view from couchdb which is defined in the
sibling query.js file (in this case recent-comments) and returns a json object
with fields that match the variables in the mustache.html template in the same
director.

The variables at the top of the file are used to make it easy to refer to the
page element and store data in a related object.

The $$ notation allows you to grab a jQuery object that is related to the
#wiki div where the current couchdb document _id has been stored in the
property docid.

$$ also allows you to access the application easily. The third line uses this
to create a variable "linkup" and makes it into a mini-library with a single
function (encode) which takes the body text, runs it through the mustache
library to escape any nasties, then (with a regex) turns any text urls into
working links that open in a new window.

The view returns a json object where the key is an array of the page name and
comment datetime. The value is the document itself (eg the comment). This is
then converted into the returned json object for use in the template.

  * "topic" is set from the property docid that was stored on the #wiki div
  * "title" is also set from the property title that was stored on the #wiki div
  * "comments" is an array of comment objects (one for each row returned from the view)

The map of rows has the anonymous function applied to each item and turns the
data into another simple json object holding the:

  * "gravatar_url" from the comment authors profile in the nested "by" object to display the avatar
  * "by" the nickname from "by"
  * "at" the second part of the array from the key (eg the datatime)
  * "comment" the body text of the comment converted back from markdown to html with links linkupped

So now the mustache template can display the first level parts of the variable
directly and itereate over the nested objects for repeated sections.

* * *

###### mustache.html

* * *

The template for displaying the comments. It is simple html with double curly
braces to surround variable replacements.

The object returned from data.js above supplies the information for the
replacement. A title is placed at the top of the section, then each comment
becomes a list item which is styled from the main.css. Finally the comment
form is rendered at the end with a textarea for new comments and a hidden
field "topic" which has a reference to the document id.

* * *

###### query.js

* * *

This file is a function that returns a json object with the name of the view
to run and the start and end key set to the document id. In effect, the view
doesn't have a time limit defined in views/recent-comments/map.js so it will
list _all_ the comments, then the start and end key will narrow it down to the
comments related to the single document we are looking at.

startkey is the docid by itself because the natural sorting of keys by the
database will make this the first possible row returned. The endkey includes
an empty object because that would be the last possible row returned and will
include comments from all possible times.

* * *

###### selectors (folder)

* * *

In this case it is a container folder. Items listed inside it relate to dom
elements and the files inside are named as the events that happen to those
elements.

Alternatively this could be replaced by a selectors.json file that contains an
object with keys of the dom element and another object keyed to the event and
containing an array of functions to call.

    
    
    {
          "a[href=#signup]" : {"click" : ["signupForm"]},
           "a[href=#login]" : {"click" : ["loginForm"]}
    }
    

Essentially, couchapp will create a similar json structure from the directory
structure on push.

* * *

###### # form

* * *

As described above, this folder identifies the dom element (form) that the
event file inside relates to.

* * *

###### ## submit.js

* * *

This is the submit event for the form dom element in the comments widget. So
it is called when someone submits a comment on the page.

A private variable called "form" is created to refer to "this" without
confusing context. $$ notation allows us to retrieve the application by
picking it from the dom element "app". It was placed there by line 33 of
templates/page.html where evently associates the comments widget with the
comments div and adds a property of app to it.

"f" then serializes the form putting the submitted fields/values from the
mustache template (comment, topic and submit) in as properties.

Other record metadata is added to the object to allow for searching in views
such as "type" which specifies the record type, "at" which is the current
datetime, "by" is a copy of the authors profile object.

Then using the "app" the database saveDoc method is called submitting new
comment document and then triggering the _init event on the #comments div
again to redraw the view with the updated information.

The function then returns false to prevent the browser trying to submit
normally.

* * *

#### profile (folder)

* * *

This upper folder works in conjunction with and overrides some aspects of the
evently profile widget in vendor/couchapp/evently/profile

* * *

##### loggedOut (folder)

* * *

loggedOut is one of the 3 states in the account widget (adminParty, loggedIn,
loggedOut) and the profile widget is connected to the account widget on line
29 of templates/page.html, so whenever the sister event fires in the account
widget, it is also fired in the profile widget. This allows you to present
different content depending on whether the user is logged in and has a
profile.

* * *

###### all.html

* * *

In the case of loggedOut the all.html and the mustache.html template both show
the same thing. It is a snippet of html used to render the menu links for
Home, Changes and Comments pages. Note the urls are not standard couchdb lists
or shows. This is because there is a json file called rewrites.json at the top
level that converts these urls to the standard.

So ../page/index becomes _show/page/index or run the "show" called "page" with
the document called "index" whereas the recent on runs a "list" called "pages"
with the "recent-changes" view providing the data

In effect it is mostly about removing the editing links etc from the
navigation bar when the user logs out.

* * *

###### mustache.html

* * *

Identical to above all.html

* * *

##### profileReady (folder)

* * *

The core profile widget in vendor/couchapp/evently has a loggedIn.js file
which fires when the user inputs their username and password. That file then
checks to see if the user has an existing profile object and fires
profileReady if it does, or noProfile if it doesn't.

The higher level profileReady and the files it contains overrides the default
ones with identical names and add additional content that is activated by ???
(eg all.html)

In the wiki application the users profile is used for two main purposes

  1. Stamping documents with author details
  2. Displaying different navigation options to logged in users verses the public

* * *

###### all.html

* * *

??

* * *

###### mustache.html

* * *

This version of the html template shows the additional links for logged in
users that allow editing, history and uploading.

The links use the hash in the url to pass a path and and a parameter to
application. Note: This may seem like an authentication system, but the urls
are still directly accessible. Security if needed must be approached in a
different manner.

The link Edit makes evently check each widget look for a function that has a
path.txt file which contains /edit only and triggers that event. In this case
it is the "wiki" widget in evently/wiki/edit/path.txt

Similarly, the #/history is in tools/history/path.txt and #/version
tools/history/path.txt

* * *

#### tools (folder)

* * *

The tools widget container

* * *

##### history (folder)

* * *

The history event reactor.

* * *

###### async.js

* * *
    
    
    ???  Needs some clarification here
    

Typically callbacks are used for event handlers and this one is making an Ajax
call which are asynchronous so you don't know when (or if) a response will
come. I guess async.js is to specify that it happens when it happens and
doesn't hold anything else up. Perhaps this is because the page may have to
wait for the database to provide the page and it will load it when it is
ready.

As previously, the variables wiki, app and docid are populated from the dom,
the attached app and the attached document id respectively.

Then the app uses the function openDoc from jquery.couch.js to retrieve the
document with a success continuation passes it to "cb" which makes async = the
current document.

* * *

###### data.js

* * *

Because we are dealing with the history of the documents we need to understand
how the older versions are stored within the current document. They are
serialized, then stored as an attachment to the current document so you can
restore them exactly at any time. So the variables in data.js are a "versions"
array to hold them all,

First a "topVersionNum" function is defined that takes the document retrieved
by async.js and checks for attachments. Then it walks through the attachments
scanning each for a string starting "rev" which it then splits off the
revision number (revNum) and checks to see if it is the latest and sets the
topVer. Next it checks the document for a property called "log" and walks
through the array until it finds the property rev_num which it sets topLog to
the revision number if it is greater. Finally it returns the highest version
number whether it is the log version or the revision.

This result is then stored in the topVer variable.

Now a loop from 1 to the highest revision number is created and each is
compared to the log entries on the document. If the log entry has a document
rev_num then the old document attachment is pushed into the "versions" array
along with some metadata about who edited it and any notes. If it no longer
exists (say after database compaction) it is marked as missing.

Finally it returns an object with the document id, the title and the versions
in an array that is sorted in descending order.

Thanks to this post for helping understanding
http://blog.couch.io/post/632718824/simple-document-versioning-with-couchdb

* * *

###### mustache.html

* * *

This template creates a new div and populates it with an unordered list of
items. Here mustache will display the title line, then iterate for each
version in the array creating a list item. Then depending upon whether the
"available" data is set or the "missing" the content of the list item will
either be a link to the version, or simply list the revision metadata.

* * *

###### path.txt

* * *

/history

The target for the url rewrite.json

* * *

##### upload (folder)

* * *

The container folder for the "upload" widget.

* * *

###### async.js

* * *

Identical to the previous async.js file except in one respect. The
tools/history/async.js file specified the #wiki div to get the document id
from. This file uses "this" so I guess will work with whatever document or div
is calling it. It makes sense I guess as you will be attaching the uploaded
file to whatever document provided the content for the div.

* * *

###### data.js

* * *

In this case data.js take the current document and simply returns the _id and
_rev number which are needed when you attach a file to a document. They will
be part of the form which is submitted.

* * *

###### mustache.html

* * *

This template renders the upload form. It provides a link to display the
document that the file will be attached to if needed, and only has 3 fields.

  1. "_attachments" which is the compulsory name for the file input
  2. "_rev" as a hidden value
  3. "_id" as a hidden value

* * *

###### path.txt

* * *

/upload is the target for this rewrite.

* * *

###### selectors (folder)

* * *

The dom element container folder.

* * *

###### # form (folder)

* * *

Selects the form and places the event below onto it.

* * *

###### ## submit.js

* * *

The anonymous function that reacts to the submit event on the form in the
widget. A jquery variable "form" is created to represent the selected form.
"app" is made available again and the form is serialised into the "f"
variable.

A jquery ajax submission is made to the database with the url of the document.
There must be some magic happening somewhere due to the special _attachment
field because I originally thought it had to be a PUT with content-type
specified.

Anyway, then it returns false to stop the normal form submission and on
success sets the browser to the base page again (essentially a refresh that
triggers all the usual init stuff) to display the new attachment listed at the
bottom.

[More detail on attachments](http://wiki.apache.org/couchdb/HTTP_Document_API#Attachments)

* * *

#### wiki (folder)

* * *

Contains the "wiki" widget.

* * *

##### _init.js

* * *

Does the usual initialisation then creates additional variables to hold the
links in the wiki, the pages and the unique links (keys).

A jquery iterator walks through all the "a" anchors in the #wiki div. A
regular expression looks at the href that and puts everything after the "page"
part of the url into a variable called "m". It then checks to see if m is set
and that there is a part url captured from the regex and uses it as a key in a
"pages" array while setting the value to true. This weeds out duplicate
entries. Then the unique keys are pushed into the "keys" array.

If the keys array has some entries the view called "all-pages" is run to
retrieve a set of rows from those links that provides the document ids for
those internal linked pages. Upon success, the rows ("resp") are looped
through and if the key exists in the view response it is removed from the
"pages" link array. At the end you are left with an array of internal document
links that can't be found in the database, so this array is then used to add a
"missing" class (via jquery) to the links so they can be styled with css
appropriately.

An error just returns an empty object.

* * *

##### edit (folder)

* * *

The "edit" event on the wiki widget.

* * *

###### async.js

* * *

Similar to previous async.js files where the document is attempted to be
retrieved and made accessible, but in this case it also has an error handler
defined. This is so that when the edit event is called and a document does not
exist (eg the server returned a 404 code) then the document is created on the
fly.

* * *

###### data.js

* * *

Editing the document only requires 4 bits of information from the existing
document, the _id, the _rev, the title and the current markdown content, so
these are retrieved from the document and returned in an object to be rendered
in the template.

* * *

###### mustache.html

* * *

The mustache template renders an editing form showing which document is being
edited. It puts the current values into the input fields and provides an extra
field for a note about the changes.

Below the submit button a partial mustache template is rendered called "help"
which in this case is a cheatsheet for markdown syntax.

Additionally, two placeholder divs with classes of "preview" and "clear".

* * *

###### partials (folder)

* * *

The partials folder contains additional mustache templates for inclusion.
Typically they are used for repetitive or static data such as headers and
footers. Included partial sections have a greater than sign in the mustaches
{{>help}}

* * *

###### # help.html

* * *

This file contains a markdown cheatsheet for display in the editing form.

* * *

###### path.txt

* * *

/edit is the target for this event.

* * *

###### selectors (folder)

* * *

Container for the selectors

* * *

###### # form (folder)

* * *

Links the event below (submit.js) to the form created in the mustache.html
file

* * *

###### ## submit.js

* * *

All the heavy lifting of the edit happens here. The form is serialized in the
variable "f".

A function called "saveDoc" is defined which adds the metadata about the edit,
the log entry, the note and saves the document using the usual method and a
refresh.

Then the serialised form is checked for a revision number setting the _rev to
the one from the document that was edited. If one exists the document is
retrieved and the update saved. If the _rev is missing then save it as a new
document with the id from the serialised form and set the "attachPrevRev" to
true so next time this version will be saved to the new document.

Again, return false to stop the browser submit.

* * *

###### # textarea[name=markdown] (folder)

* * *

Selecting the textarea with the name markdown. (I loved writing that :) )

* * *

###### ## _init.js

* * *

This function just triggers the "keyup" function after 50 milliseconds. It is
in a loop with the keyup function to constantly update the markdown/html
preview window.

* * *

###### ## keyup.js

* * *

This function sets "form" to be the parent form that triggered the function.
"prev" is a div that has a class "preview" and the context is the previously
created form jquery object. "app" is available as usual.

This time however, the variable "wiki" becomes a plugin to encode the markdown
links in the content of the textarea and make them active. The referenced file
is ilb/wiki.js and it in turn utilised vendor/couchapp/lib/markdown.

The logic of the preview goes like this. If the preview content doesn't match
the form content then set the preview content to the form. Then replace the
preview content with the rendered content. Check to see if there is a
"timeout" property on the preview div, if not set that property to trigger the
_init function again in 1.1 seconds and delete the timeout property from the
preview div so it is ready to loop again.

* * *

##### version (folder)

* * *

The version event is called from pathbinder looking at the url after the # and
finding the matching event. Additonal parameters are passed in the url after
the initial hash name and thes are passed to the version event below this
folder.

This was all set up in the evently/tools/history/mustache.html template that
created the links

    
    
    <a href="#/version/{{rev}}">
    

* * *

###### async.js

* * *

A more interesting async.js this time. Apart from the callback we have the
event and parameters being passed in. The dom, app and docid are set as usual
then a simple ajax GET call is made to retrieve that version of the document.
On completion of the request the json is put into the variable "resp". The
server request status is checked to make sure it was ok (code 200) and if so
the "resp" calls back. Alternatively it puts up an alert that the version
could not be retrieved.

* * *

###### data.js

* * *

Take the returned document and fix the wikilinks and return the rendered html
in a variable called "body"

* * *

###### mustache.html

* * *

Replace the inners with the html from data.js

* * *

###### path.txt

* * *

/version/:num indicating the target event and also that the parameters will be
stored in a variable called "num"

* * *

### language

* * *

Just javascript. I am guessing that this is pluggable and other languages are
possible

* * *

### lib (folder)

* * *

As far as I can see an arbitrary place to put library code.

* * *

##### wiki.js

* * *

The previously mentioned library code to turn internal wiki links and markdown
into real html

* * *

### lists (folder)

* * *

Special folder that holds all the _list types.

* * *

##### comments.js

* * *

The first list function. I'm not sure why it is here when the widget seems to
take care of displaying comments. ??? Perhaps some light.

Anyway ... variables "row" to hold each iterated returned doc, "ddoc" to hold
a reference to the design document it self so we can find the templates,
"mustache" is an object that can apply the library functions and "markdown" is
similar.

A "data" object is created with metadata "title", "site-title" (from the
couchapp.json file), "path", and an empty array of "comments".

The return type is specified as html with "provides" and an anonymous function
works on each row to create a log entry and push the retrieved comment object
into the comments array.

Finally the whole lot is sent back to the requestor after rendering the data
through markdown and the mustache templates from templates/comments.html and
templates/partials/header.html

* * *

##### pages.js

* * *

The pages.js list is functionally almost the same as comments.js. An identical
process is followed with a few changes of names of fields etc. However, the
mustache template applied from templates/pages.html is more ambitious. The
same header.html file is used.

The document links are all list items in an unordered list. At the bottom the
standard connection of account and profile divs happens, but a new variable
"pr" (short for profileReady) is created and the mustache property is set to
the all.html template in the profileReady widget. A note to modify that
template probably explains why all.html and mustache.html are currently the
same.

* * *

### README.md

* * *

Installation documentation.

* * *

### rewrites.json

* * *

The json object which defines url rewrites to make them a bit prettier. From
and to are pretty self-explanatory, but you can see further down there are
other parameters being passed as well.

* * *

### shows

* * *

The special folder containing the _show definitions.

* * *

##### page.js

* * *

The "page" show displays document if it exists, or an option to create it if
it doesn't. In conjunction with the page.html template this is the main page
of the wiki application displaying the documents.

"ddoc" becomes the design document, "mustache" the mustache object, "wiki" the
encoding object and "data" the returned document data with additional
metadata.

If a document exists, the content is converted back into html via mustache and
wiki en/decoding then the attachments are iterated and references to their
direct access urls added to the data object.

If a document doesn't exist, and empty data object is created and passed to
the template at templates/page.html + partials/header.html

* * *

##### redirect.js

* * *

No idea how this fits into the picture

* * *

### templates

* * *

A folder containing the html mustache templates used in the shows and lists.

* * *

##### comments.html

* * *

Comments template listing all comments from whichever view thrown at it.

* * *

##### page.html

* * *

This is the main page in the wiki that displays the individual page documents.
It has initial placeholder divs for "tools", "wiki", "comments" and "files".
The "account" and "profile" divs are included in the partial "header".

Evently connects up the divs to their events and some properties are set using
the $$ notation. Lastly, pathbinder sets to # element of the url to whatever
was appropriate from the data object supplied from shows/page.js (eg / or
/edit)

* * *

##### pages.html

* * *

Display template for the lists of documents.

* * *

##### partials (folder)

* * *

Folder to hold the generic inclusion templates.

* * *

###### header.html

* * *

Generic header template adding common divs and stylesheet etc.

* * *

### validate_doc_update.js

* * *

A special file used to validate documents before a save. Returning true will
allow the save, anything else will fail.

Not sure what the third line does.

* * *

### vendor (folder)

* * *

I will leave this section to greater minds.

##### couchapp (folder)

###### _attachments

###### # jquery.couch.app.js

###### # jquery.couch.app.util.js

###### # jquery.evently.js

###### # jquery.mustache.js

###### # jquery.pathbinder.js

###### # loader.js

###### evently

###### # account (folder)

###### ## _init.j

###### ## adminParty (folder)

###### ### mustache.html

###### ## doLogin.js

###### ## doLogout.js

###### ## doSignup.js

###### ## loggedIn (folder)

###### ### after.js

###### ### data.js

###### ### mustache.html

###### ### selectors.json

###### ## loggedOut (folder)

###### ### mustache.html

###### ### selectors.json

###### ## loginForm (folder)

###### ### after.js

###### ### mustache.html

###### ### selectors (folder)

###### #### a[href=#signup].json

###### #### form (folder)

###### ##### submit.js

###### ## signupForm (folder)

###### # profile (folder)

###### ## loggedIn.js

###### ## loggedOut (folder)

###### ### after.js

###### ### mustache.html

###### ## noProfile (folder)

###### ### data.js

###### ### mustache.html

###### ### selectors (folder)

###### #### form (folder)

###### ##### submit.js

###### ## profileReady (folder)

###### ### after.js

###### ### data.js

###### ### mustache.html

###### # README.md

###### lib (folder)

###### metadata.json

###### README.md

* * *

### views (folder)

* * *

A special folder containing all the definitions of the views of the data.

* * *

##### all-comments (folder)

* * *

The name of the view.

* * *

###### map.js

* * *

Checks for the type comment. If it exists and has a comment property add it to
the collection using "at" as a key. "at" is the timestamp of the comment so it
is sorted by time.

* * *

##### all-pages (folder)

* * *

The name of the view.

* * *

###### map.js

* * *

Checks for a title and markdown field. If they exist, then emits the document
with the id as key, and a 1 for the value so you can count how many are in a
particular result I suppose.

* * *

##### recent-changes (folder)

* * *

The name of the view.

* * *

###### map.js

* * *

Checks for the doc.edit_at, doc.edit_by, doc.title, doc.log, doc.log.length
fields existence, then pops the last log off the array to get the last change
made to the document. It then emits the editing time as key and a small object
with title, last edit note and editors details.

* * *

##### recent-comments (folder)

* * *

The name of the view

* * *

###### map.js

* * *

Checks if it is a comment the emits the document with the topic and timestamp
as a key in an array.


