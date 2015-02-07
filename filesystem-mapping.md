# The Couchapp Filesystem Mapping

The `couchapp` script has a cool way of pushing files to CouchDB's design
documents. The [filesystem mapping](http://couchdbkit.org/docs/storing_docs_and_designdocs_on_filesystem.html)
is done via the [couchdbkit](http://couchdbkit.org/) Python library.

If you have folders like:

    
    
    myapp/
      views/
        foobar/
          map.js
          reduce.js
    

It will create a design document like this:

    
    
    {
      "_id" : "_design/myapp",
      "views" : {
        "foobar" : {
          "map" : "contents of map.js",
          "reduce" : "contents of reduce.js"
        }
      }
    }
    

This is designed to make it so you get proper syntax highlighting in your text
editor.

### Complete Filesystem-to-Design Doc Mapping Example

    
    
    myapp/
      _attachments/
        images/
          logo.png
      _docs/
        sample.json
        doc_needing_encoding/
          _id (the ID for the document as text on the first line of this file)
          title (same as ID, just for the title field. Repeat pattern as needed)
          content.html (HTML content that will be encoded when it's added to the JSON doc)
      lists/
        xml.js
      rewrites.js
      shows/
        preview.js
        xml.js
      updates/
        in-place.js
      views/
        foobar/
          map.js
          reduce.js
      validate_doc_update.js
    

The `_attachments` folder will turn each file into an attachment on the
resulting Design Document. The attachments will be named based on their file
path (ex: "image/logo.png").

The contents of the `_docs` folder are turned into actual JSON documents in
CouchDB. The contents of the .json files will be input exactly as they are in
the file. The name of the document with be either the file name or the `_id`
field from the JSON object in that file.

Folders under `_docs` will be turned into documents with each file in the
folder being a key/value pair in the resulting JSON document. HTML and XML
files (and maybe others?) will be JSON encoded before being added to the JSON
document. An `_id` file will be used (if present) as the ID of the new
document. Otherwise the folder name will become the ID.

The rest of the folder structure above will become this JSON Design Document

    
    
    {
      "_id" : "_design/myapp",
      "_attachments": {
        "images/logo.png": {
          "content_type": "image/png","revpos":1,"digest":"md5-GDPL+eLwE7kzEDWY7X4KdQ==","length":886,"stub":true
        }
      },
      "lists": {
        "xml": "function..."
      },
      "rewrites": "function...",
      "shows": {
        "preview": "function...",
        "xml": "function..."
      }
      "updates": {
        "in-place": "function..."
      },
      "views": {
        "foobar": {
          "map": "function...",
          "reduce": "function..."
        }
      },
      "validate_doc_update": "function...",
    }
    

#### Evently

It's not a very big leap to see that an Evently widget could be editing in a
filesystem tree like this: (Picking the most complex -- `nestedEvently` from
the [Evently Primer](evently-primer) document.)

    
    
    myapp/
      evently/
        mustache.html
        data.json
        selectors/
          span.word/
            click/
              mustache.html
            mouseenter/
              mustache.html
            congrats/
              mustache.html
              data.js
          a[href=#win]/
            click.js
    

