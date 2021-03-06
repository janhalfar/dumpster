# Dumpster

_Dump project data and restore them_

![Image dumpster from wikipedia](https://upload.wikimedia.org/wikipedia/commons/thumb/5/55/Dumpsters_in_Anaktuvuk_Pass%2C_Alaska.jpg/440px-Dumpsters_in_Anaktuvuk_Pass%2C_Alaska.jpg)

<small>image &copy; [Paxson Woelber](https://en.wikipedia.org/wiki/Dumpster#/media/File:Dumpsters_in_Anaktuvuk_Pass,_Alaska.jpg)</small>

Dumpster allows you to define dumps. **A dump is genrated by a program**, that
will write the dump into a file and another program that will be able to
**restore** the previously created dump from that file.

A simple **RESTful service** allows you to create, access, restore and
delete those dumps.

Things become very interesting with remotes. **Remotes allow you to restore remote
dumps locally**. Common use cases are backups and development.

## Security

Dumpster has **no security mechanisms built in**! You have to be very careful on
how to make it accessible and with which privileges you will let it run.

## Dump and restore programs

Since we do not (want to) know how to dump or restore your data let us define
a very simple contract for dumping and restoring:

```bash
# DUMPING
#   program [as many args as you like] but the last arg is the file to dump to
/path/to/dump-program arg1 arg2 ... /path/to/dump/file

# RESTORING
#   same rules as with the dump - last arg is the dumpfile
/path/to/restore-program arg1 arg2 ... /path/to/dump/file

```

You are cordially invited to use stdout and stderr to provide information on
your dump or restore. As long as you exit with code 0 and write the data
synchronously to the filename passed as last arg dumpster will be happy.

Your output to stdout and stderr will be recorded and presented to the user.

Typically your dump programs will be shell scripts on a nix system.

## How to configure it

Dumpsters config is a yaml file:

```yaml
# this is where I listen and provide a REST interface
address: 127.0.0.1:8080
# a directory to put my data, those are plain files in a simple data structure
datadir: /var/lib/dumpster
# these are my dumps, which is what I am all about
dumps:
  # this is a dump config and it defines a dump called dumpster
  dumpster:
    # dump defines what program should be called to create a dump
    dump:
      # this example program will simply tar the directory passed as $1
      program: /home/jan/go/src/github.com/foomo/dumpster/examples/files/dump.sh
      # with some args
      args:
        - /home/jan/go/src/github.com/foomo/dumpster
    # restore defines a program, which has to restore a previously created dump
    restore:
      program: tar
      args:
        - -tzvf
  # this is another dump
  projectx:
    # only the dump part is defined, thus restore can not be called
    dump:
      program: /home/jan/go/src/github.com/foomo/dumpster/examples/files/dump.sh
      args:
        - /home/jan/go/src/github.com/foomo/projectx
# remotes are pointing to other dumpster instances
remotes:
    myself:
        endpoint: http://127.0.0.1:8080

```

## REST interface

There is a fairly extensive documentation [http://docs.dumpster.apiary.io/](http://docs.dumpster.apiary.io/)

A little curl guide

```bash
# list dumps
curl 127.0.0.1:8080/dump

# list dumps of type "dumpster"
curl 127.0.0.1:8080/dump/dumpster

# create a dump of type "dumpster" with id "my-first-dump"
curl -X POST -d '{ "id" : "my-first-dump", "comment" : "what a great day ..." }' 127.0.0.1:8080/dump/dumpster

# restore a dump of type "dumpster" with id "my-first-dump"
curl -X POST 127.0.0.1:8080/restore/dumpster/my-first-dump

# list all remote dumps
curl 127.0.0.1:8080/dumpremote

# list remote dumps on server "myself"
curl 127.0.0.1:8080/dumpremote/myself

# restore a remote dump from server "myself" of type "dumpster" and id "my-first-dump"
curl -X POST 127.0.0.1:8080/restoreremote/myself/dumpster/my-first-dump

```

## TODO

- add tests
- daemonize
- make some debian... packages
