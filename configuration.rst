uPaaS configuration
===================

Configuration file uses YAML syntax.
All options are stored in /etc/upaas/upaas.yml configuration file.
For convenience settings are split into multiple files that are included in main config (upaas.yml).
Splitting uses ``!include <path>`` statement. If path is not absolute it will be interpreted as relative to the parent file (the one with include statement).

MongoDB settings
----------------

Basic settings:

.. code-block:: yaml

    mongodb:
      host: localhost
      port: 27017
      database: upaas

If MongoDB requires authorization for database access (recommended) add ``username`` and ``password`` options.

.. code-block:: yaml

    mongodb:
      [...]
      username: username
      password: password

To provide high availability to MongoDB installation it is recommended to use MongoDB cluster (replica set or sharding). To pass replica set address to uPaaS use uri option instead of host and port pair, example:

.. code-block:: yaml

    mongodb:
      [...]
      uri: mongodb://db1,db2/?replicaSet=upaas&readPreference=primaryPreferred

.. note:: Remeber to set read preference when using HA MongoDB setup.

Admin UI settings
-----------------

``secretkey``
.............

  For every installation secret key must be set, it must be unique (for each cluster) and unpredictable string. Every node in uPaaS cluster must have identical value secret key.

  .. code-block:: yaml

      admin:
        secretkey: very-very-secret

``loglevel``
............

  Verbosity level for all logged messages, possible values: debug, info, warning, error.

``debug``
.........

  Enable django debug mode, see `django docs <https://docs.djangoproject.com/en/dev/ref/settings/#debug>`_ for details.

``domains``
...........

  List of domains uPaaS web UI can be served for. Any domain will be allowed if this option is not specified. Details can be found in `django docs <https://docs.djangoproject.com/en/1.5/ref/settings/#allowed-hosts>`_.

``smtp``
........

  Contains options for sending email notifications. Avaiable settings:
    * ``host`` - hostname of SMTP server used for sending emails, default is localhost
    * ``port`` - remote SMTP port to use, defult is 25
    * ``tls`` - whenever to use TLS or not, default is false
    * ``username`` - username if SMTP authentication is going to be used, default is not set - not authentication
    * ``password`` - password if SMTP authentication is going to be used, default is not set - not authentication
    * ``sender`` - email address used as sender address, default is no-reply@localhost

Full example:

.. code-block:: yaml

    admin:
      secretkey: very-very-secret
      loglevel: info
      debug: false
      domains:
        - "admin.upaas.domain"
        - "admin.upaas.com"
        - "*.upaas-admin.io"
      smtp:
        host: localhost
        port: 25
        tls: true
        username: upaas@localhost
        password: smtppass


Paths settings
--------------

uPaaS stores files in few location, they can be customized with those settings:

.. code-block:: yaml

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

  Directory where applications uWSGI config files are placed. This directory must be the path that uWSGI emperor will be monitoring.


Storage
-------

Package files are stored by default in MongoDB database but custom storage handlers can be created. To use local storage (only useful with single node installations) use those settings:

.. code-block:: yaml

    storage:
      handler: upaas.storage.local.LocalStorage
      settings:
        dir: /var/upaas/storage

This way uPaaS will store all packages as plain files in /var/upaas/storage directory.

To use dedicated MongoDB database for packages use:

.. code-block:: yaml

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

All application packages are built using empty os system image, so first such empty image must be generated. Example config for Ubuntu server:

.. code-block:: yaml

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

  List of commands used to create system image files. ``%workdir%`` macro will be expanded into directory path where image is being created.

``maxage``
..........

  Images older than this value (in days) will be ignored and new image will be generated. This is intended to keep system images current, with all updates applied.

``packages``
............

  List of packages to install in system image once it is generated.


System commands
---------------

This settings are used to tell uPaaS what commands should be used to interact with system images. Mostly how to (un)install packages using system package manager.

.. code-block:: yaml

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

  Describes how to install package. ``cmd`` option contains command that needs to be executed, ``%package%`` macro will be expanded into package name. ``env`` and ``timelimit`` options have the same meaning as in bootstrap section.

``uninstall``
.............

  Same as ``install`` but describes how to uninstall package.


Application deployment settings
-------------------------------

``uid``
.......

  Uid of user application will be running as, for example www-data.

``gid``
.......

  Name of group that will be used to run application.

``home``
........

  Path where application directory will be placed inside system image.

``domains``
..........

  Every application will be accessible using:
    * automatic system domain (use ``app.domains.system`` key to set it)
    * custom domain assigned by the user (user must own this domain or at least be able to modify it)
  All domains used in application must point to uPaaS router nodes, user will be notified during custom domain assigment.
  To protect from domain hijacking every custom domain that user want to assign to his application must be verified.
  This is done by checking if domain contains TXT record with application key.
  This can be disabled if only trusted apps are deployed in uPaaS cluster, set ``apps.domains.validation = False`` to disable it.


``tcp``
.......

  Contains two options ``port_min`` and ``port_max`` used to specify port range used for application sockets.

Example:

.. code-block:: yaml

    apps:
      uid: www-data
      gid: www-data
      home: /home/app
      domain: upaas.domain
      tcp:
        port_min: 2001
        port_max: 7999

``uwsgi``
.........

  Contains uWSGI specific options.
  Currently only ``safe_options`` section is available with list of safe uWSGI options that user can set in application metadata.
  Safe options should can be written as python compatible regular expressions.

Example:

.. code-block:: yaml

  uwsgi:
    safe_options:
      - "^check-static"
      - "^static-"
      - "^harakiri"
      - "^enable-threads$"
      - "^(worker-|)reload-mercy$"
      - "^max-requests$"
      - "^(min|max)-worker-lifetime$"
      - "^upload-progress$"
      - "^lazy"
      - "^route"
      - "^(response|final|error)-route$"

``graphite``
............

  Configuration for carbon/graphite statistics integration.
  This is optional feature and it requires working carbon server and graphite frontend applications.
  Available options:
  # ``carbon`` list of carbon servers for pushing statistics from uWSGI backends
  # ``render_url`` graphite frontend url, it will be used for rendering statistics, must be accessible by uPaaS users
  # ``timeout`` timeout for pushing statistics from uWSGI backend, default is 3 seconds
  # ``frequency`` push statistics from uWSGI to carbon every N seconds, default is 60
  # ``max_retry`` how many times uWSGI should retry pushing stats in case of errors, default is 1
  # ``retry_delay`` how many seconds should uWSGI wait before retry, default is 7
  # ``root`` root node for all statistics, default is "uwsgi"

  Only ``carbon`` and ``render_url`` options are required to integrate carbon/graphite with uPaaS.

Defaults
--------

Default settings.
Currently only ``limits`` section is available.
Those limits will be used for all users that do not have custom limits set by uPaaS administrator.

``running_apps``
................

  Numer of applications user is allowed to run simultaneously, 0 means no limit. Default is 0.

``workers``
...........

  Number of workers user is allowed to allocate to running applications, 0 means no limit. Default is 0.

``memory_per_worker``
.....................

  Memory limit for application workers, this limit is applied to each worker process. Default is 128.

``packages_per_app``
....................

  Number of package files that are kept for every applications, allowing to rollback application to previous package. Default is 5.

``max_log_size``
................

  Maximum application instance log size (in MB). Each instance logs to file in application home directory (upaas.log), once size limit is reached log is rotated, one rotated log is kept.

Example:

.. code-block:: yaml

  limits:
    running_apps: 0
    workers: 16
    memory_per_worker: 128
    packages_per_app: 5
    max_log_size: 50

Interpreter settings
--------------------

Every available interpreter must be configured before app can use it. See :doc:`interpreters`.
