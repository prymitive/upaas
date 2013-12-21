Metadata format
===============


Syntax
------

Distribution specific settings, currently only list of packages to install can be configured here.

< id > is the id of the distribution same as output of `lsb_release -si` command.

.. code-block:: yaml

    os:
      < id >:
        packages:
          - < package name >

Configuration for interpreter used in application.

```type``` can be any supported type (python, ruby, php).

You can provide list of interpreter versions supported by your application, highest supported version will be used.

```settings``` key allows to pass interpreter specific options, uPaaS administator should document what options are avilable here.

By default only module option for python interpreter can be set under settings key, uPaaS administrator might add support for more options.

.. code-block:: yaml

    interpreter:
      type: < type >
      versions:
        - < version >
        - < version >
      settings:
        < interpreter settings >

Configuration for repository life cycle management:

    # clone - how to clone repository, allowed variables: %destination% - path where application should be cloned to
    # update - how to fetch updates
    # info - how to obtain informations about this repository (latest commit), not used yet
    # changelog - how to generate list of changes since last commit, allowed variables - %old%, %new% - id of old and new commit id, not used yet

.. code-block:: yaml

    repository:
      clone: < command to clone application repository >
      update:
        - < command >
        - < command >
      info: < command >
      changelog: < command >

List of environment variables that should be set for this application, optional.

.. code-block:: yaml

    env:
      MYENV: value

After cloning repository multiple actions needs to be executed in order to deploy application, see :doc:`buildprocess` for details.

.. code-block:: yaml

    actions:
      setup:
        main:
          - < command >
          - < command >

List of files to create after cloning app repository.

.. code-block:: yaml

    files:
      < path>: < content >

Under ```uwsgi``` key you can pass additional settings to uWSGI.

uPaaS administrator might limit options that can be set here, unsupported options will be ignored.

.. code-block:: yaml

    uwsgi:
      settings:
        - "option = value"

Example
-------

phpmyadmin example metadata

.. code-block:: yaml

    os:
      Debian: &debian
        packages:
          - git-core
          - php5-mysql
          - php5-mcrypt
      Ubuntu: *debian

    interpreter:
      type: php
      versions:
        - "5.5"

    repository:
      clone: git clone --depth=10 --quiet --branch STABLE git://github.com/phpmyadmin/phpmyadmin.git %destination%
      update:
        - git reset --hard
        - git pull
      info: git log -n 1
      changelog: git log --no-merges %old%..%new%"

    files:
      config/config.inc.php: |
        <?php
        $cfg['blowfish_secret'] = 'changeme';
        $i = 0;
        $i++;
        $cfg['Servers'][$i]['auth_type'] = 'cookie';
        $cfg['Servers'][$i]['host'] = 'localhost';
        $cfg['Servers'][$i]['connect_type'] = 'tcp';
        $cfg['Servers'][$i]['compress'] = false;
        $cfg['Servers'][$i]['extension'] = 'mysqli';
        $cfg['Servers'][$i]['AllowNoPassword'] = false;
        $cfg['UploadDir'] = '';
        $cfg['SaveDir'] = '';
        ?>
