# Description

redux is top down software build tool similar to make but simpler and more reliable.

It implements and extends the redo concept somewhat sparsely described by [DJ Bernstein](http://cr.yp.to/redo.html).

I implemented a minimal set of redo tools some years ago and used them enough to become convinced
that the idea was worthwhile. However, they needed to be better, sharper, and faster before they could
replace Make in my toolbox, thus, this tool.

# Installation

redux is written in Go and requires the Go compiler to be installed.
Go can be fetched from the [Go site](http://www.golang.com) or your favorite distribution channel.

Assuming Go is installed (and your GOPATH is a single directory), the commands:

    $ go get github.com/gyepisam/redux/redux
    $ sudo $(echo $GOPATH | cut -f1 -d:)/src/github.com/gyepisam/redux/install

will fetch, build and install redux and associated links to the default bin directory in $GOPATH, with man
pages in /usr/local/man/man1.

Change the MANDIR environment variables to install to other locations like so:

    $ sudo env MANDIR=/some/other/path/man/man1 \
           $(echo $GOPATH | cut -f1 -d:)/src/github.com/gyepisam/redux/install

If you prefer to install the binary in a different directory, you need to run go build:

    $ go build -o /some/other/path/bin/redux github.com/gyepisam/redux/redux
    $ sudo $(echo $GOPATH | cut -f1 -d:)/src/github.com/gyepisam/redux/install /some/other/path/bin/redux

The installation is overly complicated because the go tool only installs a single package. At some point,
I will add an `install` command to redux...

redux supports the following commands:

       init -- Creates or reinitializes one or more redo root directories.

   ifchange -- Creates dependency on targets and ensure that targets are up to date.

   ifcreate -- Creates dependency on non-existence of targets.

       redo -- Builds files atomically.


The installer creates symbolic links for each of these commands to they can be invoked as:

  [redo-init](/doc/redo-init.html)
  [redo-ifchange](/doc/redo-ifchange.html)
  [redo-ifcreate](/doc/redo-ifcreate.html)
  [redo](/doc/redo.html)

# Overview

A redo based system must have a directory named .redo at the top level project
directory. 

It is created with the `redo-init` command, whose effects are idempotent.

The database contains file status and metadata. Its format is not specified,
but it must be capable of supporting multiple readers and multiple writers
(though not necessarily writing to the same sections). Redux implements a file based
database and a null database. The current file based database is probably about the
slowest. Fortunately, new databases can be easily pluggled in.

In the redo system, there are three kinds of files.

* `do` files are scripts that build targets. In addition, they establish
file dependencies by calling `redo-ifcreate` and `redo-ifchange`. `do` files
are invoked by the `redo` commands directly and indirectly by `redo-ifchange`.

* `target` files are generated or otherwise composed, likely from other files, by `do` scripts

* `static` or `source` files are manually created/edited. They are tracked
for changes, but are not generated. As such, they have no do files.

There are two kinds of dependencies

* ifchange denotes a relationship where a change in one file marks the other as out of date.
  A ifchange B implies that a change in B invalidates A. Change includes creation, modification or deletion.

* ifcreate denotes a dependency on the creation or deletion of a file.
  A ifcreate B implies that when B comes into existence or ceases to exist, A will be outdated.
