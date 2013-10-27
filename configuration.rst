uPaaS configuration
===================

All options are stored in /etc/upaas/upaas.yml configuration file.
Config file uses YAML syntax.

MongoDB settings
----------------

Basic settings:

.. code:: yaml

   mongodb:
     host: localhost
     port: 27017
     database: upaas

If MongoDB requires authorization for database access (recommended) add 
username and password options.

.. code:: yaml

   mongodb:
     [...]
     username: username
     password: password

To provide high availability to MongoDB installation it is recommended to 
use MongoDB cluster (replica set or sharding). To pass replica set address 
to uPaaS use uri option instead of host and port pair, example:

.. code:: yaml

   mongodb:
     [...]
     uri: mongodb://db1,db2/?replicaSet=upaas


Admin UI settings
-----------------

For every installation secret key must be set, it must be unique (for each 
cluster) and unpredictable string. Every node in uPaaS cluster must 
have identical value secret key.

.. code:: yaml

   admin:
     secretkey: very-very-secret

Other options:

``debug``
.........

  Enabled debug messages in logs.

``domains``
...........

  List of domains uPaaS web UI can be served for. Any domain will be allowed if 
this option is not specified. Details can be found in `django docs 
<https://docs.djangoproject.com/en/1.5/ref/settings/#allowed-hosts>`_.

Full example:

.. code:: yaml

   admin:
     secretkey: very-very-secret
     debug: false
     domains:
       - "admin.upaas.org"
       - "admin.upaas.com"
       - "*.upaas-admin.io"


Paths settings
--------------

uPaaS stores files in few location, they can be customized with those settings:

.. code:: yaml

   paths:
     workdir: /var/upaas/workdir
     apps: /var/upaas/apps
     vassals: /etc/uwsgi-emperor/vassals

``workdir``
...........

  Directory for temporary files.

``apps``
........

  Directory where packages for running applications are stored.

``vassals``
...........

  Directory where applications uWSGI config files are placed. This directory
  must be the path that uWSGI emperor will be monitoring.


Storage
-------

Package files are stored by default in MongoDB database but custom 
storage handlers can be created. To use local storage (only useful with single 
node installations) use those settings:

.. code:: yaml

   storage:
     handler: upaas.storage.local.LocalStorage
     settings:
       dir: /var/upaas/storage

This way uPaaS will store all packages as plain files in 
/var/upaas/storage directory.

To use dedicated MongoDB database for packages use:

.. code:: yaml

   storage:
     handler: upaas.storage.mongodb.MongoDBStorage
     settings:
       host: mongo-db-packages-host
       port: 27017
       database: upaas-packages
       username: username
       password: password


OS bootstrap
------------

All application packages are built using empty os system image, so first such 
empty image must be generated. Example config for Ubuntu server:

.. code:: yaml

   bootstrap:
     timelimit: 600
     env:
       LC_ALL: C
       LANG: C
     commands:
       - debootstrap --components=main,universe,multiverse,restricted `lsb_release -sc` %workdir%
     maxage: 7
     packages:
       - python-software-properties
       - build-essential

``timelimit``
.............

  How long single command can take before it is killed (in seconds).

``env``
.......

  List of environment variables passed to each command (optional).

``commands``
............

  List of commands used to create system image files. %workdir% makro will be 
expanded into directory path where image is being created.

``maxage``
..........

  Images older than this value (in days) will be ignored and new image will be 
generated. This is intended to keep system images current, with all updates 
applied.

``packages``
............

  List of packages to install in system image once it is generated.


System commands
---------------

This settings are used to tell uPaaS what commands should be used to interact 
with system images. Mostly how to (un)install packages using system package 
manager.

.. code:: yaml

   commands:
     timelimit: 600
     install:
       env:
         DEBIAN_FRONTEND: noninteractive
         LC_ALL: C
         LANG: C
       cmd: apt-get install --no-install-recommends -y %package%
     uninstall:
       env:
         DEBIAN_FRONTEND: noninteractive
         LC_ALL: C
         LANG: C
       cmd: apt-get remove -y %package%

``install``
...........

  Describes how to install package. ``cmd`` option contains command that needs 
to be executed, %package% macro will be expanded into package name.
  ``env`` and ``timelimit`` options have the same meaning as in bootstrap 
section.

``uninstall``
.............

  Same as ``install`` but describes how to uninstall package.
