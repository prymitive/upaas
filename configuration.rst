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

To provide high availability to MongoDB installation it is recommended to use
MongoDB cluster (replica set or sharding). To pass replica set address to uPaaS
use uri option instead of host and port pair, example:

.. code:: yaml

   mongodb:
     [...]
     uri: mongodb://db1,db2/?replicaSet=upaas


Admin UI settings
-----------------

For every installation secret key must be set, it must be unique (for each 
cluster) and unpredictable string. Every node in uPaaS cluster must have 
identical value secret key.

.. code:: yaml

   admin:
     secretkey: very-very-secret

Other options:

debug
~~~~~

Enabled debug messages in logs.

domains
~~~~~~~

List of domains uPaaS web UI can be served for. Any domain will be allowed if 
this option is not specified. Details can be found in 
`django docs <https://docs.djangoproject.com/en/1.5/ref/settings/#allowed-hosts>`_.

Full example:

.. code:: yaml

   admin:
     secretkey: very-very-secret
     debug: false
     domains:
       - admin.upaas.org
       - admin.upaas.com
       - *.upaas-admin.io


Paths settings
--------------

uPaaS stores files in few location, they can be customized with those settings:

.. code:: yaml

   paths:
     workdir: /var/upaas/workdir
     apps: /var/upaas/apps
     vassals: /etc/uwsgi-emperor/vassals

workdir
~~~~~~~

Directory for temporary files.

apps
~~~~

Directory where packages for running applications are stored.

vassals
~~~~~~~

Directory where applications uWSGI config files are placed. This directory
must be the path that uWSGI emperor will be monitoring.


Storage
-------

Package files are stored by default in MongoDB database but custom storage
handlers can be created. To use local storage (only useful with single node
installations) use those settings:

.. code:: yaml

   storage:
     handler: upaas.storage.local.LocalStorage
     settings:
       dir: /var/upaas/storage

This way uPaaS will store all packages as plain files in /var/upaas/storage 
directory.

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
