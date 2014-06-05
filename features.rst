Available features
==================


Features are additional functionality that you can enable in application metadata.
To be able to use given feature it must be enabled in uPaaS configuration.


storage
-------

Mount external storage in application namespace.

.. code-block:: yaml

    features:
      storage: true


cron
----

Cron tasks. Commands will be executed on every running instance.
Syntax same as for standard crontab, no value means ``*``.

.. code-block:: yaml

    features:
      cron:
        - command: <command>
          minute: <0-59>
          hour: <0-24>
          day: <1-31>
          month: <1-12>
          weekday: <0-7>
        - command <command 2>
          [...]


mongodb
-------

Creates mongodb database for application and puts connection URI into application ENV.

.. code-block:: yaml

    features:
      mongodb: true
