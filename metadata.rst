Metadata format
===============


Syntax
------

Configuration file uses YAML syntax.

| Distribution specific settings, currently only list of packages to install can be configured here.
| < id > is the id of the distribution same as output of `lsb_release -si` command.

.. code-block:: yaml

    os:
      < id >:
        packages:
          - < package name >

| Configuration for interpreter used in application.
| ``type`` can be any supported type (python, ruby, php).
| You can provide list of interpreter versions supported by your application, highest supported version will be used.
| ``settings`` key allows to pass interpreter specific options, uPaaS administrator should document what options are available here.
| By default only module option for python interpreter can be set under settings key, uPaaS administrator might add support for more options.

.. code-block:: yaml

    interpreter:
      type: < type >
      versions:
        - < version >
        - < version >
      settings:
        < interpreter settings >

Configuration for repository life cycle management:

    * clone - how to clone repository, allowed variables: %destination% - path where application should be cloned to
    * update - how to fetch updates

.. code-block:: yaml

    repository:
      clone: < command to clone application repository >
      update:
        - < command >
        - < command >

After cloning or updating repository uPaaS will try to extract some information about current revision, this should work out of the box for:

    * git
    * bazaar
    * mercurial
    * subversion

For any other repository type user can set custom commands used for revision information extraction:

.. code-block:: yaml

    repository:
      revision:
        id: <command>
        author: <command>
        date: <command>
        description: <command>
        changelog: <command>

Command description:

    * id - must return current revision id
    * author - must return current revision author
    * date - must return current revision date, date must be in any format parsable using `timestring module <https://pypi.python.org/pypi/timestring>`_
    * description - must return current revision description
    * changelog - must return list of changes between two revisions, string ``%old%`` will  be substituted with previous revision id, string ``%new%`` will be substituted with current revision id

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

| Under ``uwsgi`` key you can pass additional settings to uWSGI.
| uPaaS administrator might limit options that can be set here, unsupported options will be ignored.

.. code-block:: yaml

    uwsgi:
      settings:
        - "option = value"

List of enabled features, optional.

.. code-block:: yaml

    features:
      < name>: < value >
      < name>: < value >

| ``name`` is the name of the feature.
| ``value`` can be boolean (``True`` or ``False``) for simple features that do not require application level config, or any YAML formatted options for advanced features. See specific feature documentation.

See :doc:`features` for list of available features.


Examples
--------

See :doc:`examples`
