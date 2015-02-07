# CouchApp Configuration

NOTE: In this document, "`couchapp`" refers to the CouchApp command-line tool
while "CouchApp" refers to the application you are developing and pushing to a
CouchDB server.

## `.couchapprc`

Every CouchApp must have a `.couchapprc` file in the application directory.
This file is a JSON object which contains configuration parameters that the
command-line app uses to build and push your CouchApp. The `couchapp generate`
and `couchapp init` commands create a default version of this file for you.

The most common use for the `.couchapprc` file is to specify one or more
CouchDB databases to use as the destination for the `couchapp push` command.
Destination databases are listed under the `env` key of the `.couchapprc` file
as follows:

    
    
    {
      "env" : {
        "default" : {
          "db" : "http://localhost:5984/mydb"
        },
        "prod" : {
          "db" : "http://admin:password@myhost.com/mydb"
        }
      }
    }
    

In this example, two environments are specified: "default", which pushes to a
local CouchDB instance without any authentication, and "prod", which pushes to
a remote CouchDB that requires authentication. Once these sections are defined
in `.couchapprc`, you can push to your local CouchDB by running `couchapp
push` (the environment name "default" is used when no environment is
specified) and push to the remote machine using `couchapp push prod`. For a
more complete discussion of the `env` section of the `.couchapprc` file, see
the [Managing Design Documents](http://guide.couchdb.org/draft/managing.html#configuring) chapter of **CouchDB: The Definitive Guide**.

The `.couchapprc` file is also used to configure extensions to the `couchapp`
tool. See the [Extend couchapp](couchapp-extend.md) page on this wiki for more details.

## `~/.couchapp.conf`

One drawback to declaring environments in the `.couchapprc` file is that any
usernames and passwords required to push documents are stored in that file. If
you are using source control for your CouchApp, then those authentication
credentials are checked in to your (possibly public) source control server. To
avoid this problem, the `couchapp` tool can also read environment
configurations from a file stored in your home directory named
`.couchapp.conf`. This file has the same syntax as `.couchapprc` but has the
advantage of being outside of the source tree, so sensitive login information
can be protected. If you already have a working `.couchapprc` file, simply
move it to `~/.couchapp.conf` and run `couchapp init` to generate a new, empty
`.couchapprc` file inside your CouchApp directory. If you don't have a
`.couchapprc` file, `couchapp` will display the dreaded `couchapp error: You
aren't in a couchapp` message.

## `~/.couchapp`

Most of the time, you will use `couchapp generate` to create a new CouchApp
with the default directory layout, example functions, and vendor directories.
If you find yourself creating multiple CouchApps that always contain the same
third-party or in-house files and libraries, you might consider creating a
custom app template containing these files and using the `\--template` option
of the generate command to create your customized CouchApps.

Custom templates are stored as subdirectories under the
`~/.couchapp/templates` directory. The name of the subdirectory is used in the
`\--template` option to specify which template files are to be used in the
`couchapp generate` command. The default template name is "app", so by
creating `~/.couchapp/templates/app` and placing files and directories under
that path, you can replace almost all of the default files created by
`couchapp generate`.

## `.couchappignore`

A .couchappignore file specifies intentionally untracked files that couchapp
should ignore. It's a simple json file containing an array of regexps that
will be use to ignore file.

For example:

    
    
    [
      ".*\\.swp$",
      ".*~$"
    ]
    

will ignore all files ending in .swp and ~ . Be sure to leave out the final ,
in the list.

You can check if couchapp really ignores the files by specifying the -v
option: couchapp -v push ...

Note: windows doesn't like files that only have an extension, so creating the
.couchappignore file will be a challenge in windows. Possible solutions to
creating this file are :

Using cygwin, type: touch .couchappignore cd /to/couchappand then notepad
.couchappignore

TODO: more information about other templates like vendor, view, etc.

