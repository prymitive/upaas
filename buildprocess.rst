Package building internals
==========================

This page describe all steps during package building.

Step 1 - System image
---------------------

Before we can start building package uPaaS needs to generate bare system image. Under Ubuntu server this is done by using debootstrap command. Once system image is created it is unpacked in temporary directory.

This step can be only customized by uPaaS administrator as it requires tweaking uPaaS configuration file. See :doc:`configuration` for details.

All commands executed later are run chrooted in this system image.

Empty system image is used only when building first package or if user requests to create fresh package, in other cases current application package is used to build next packages. This way we don't need to reinstall everything all the time.

Step 2 - Install interpreter
----------------------------

This step is executed only for first or fresh package.

uPaaS installs all packages required for interpreter used by our application. This step can be customized only be uPaaS administrator.

Step 3 - Install application dependencies
----------------------------------------

uPaaS installs all packages requested to install in application metadata file. This step can be customized only using application metadata.

Step 4 - Cloning application
----------------------------

If this is first or fresh package uPassS will try to clone application, otherwise it will only try to update it to the latest revision. This step can be customized by uPaaS administrator, application metadata might override it with custom commands.

Step 5 - Executing setup actions
--------------------------------

Every type of application requires different commands to be deployed properly, for example:

  * ruby needs to run ``bundle install``
  * python needs ``pip install -r requirements.txt``

uPaaS handles that by executing few customizable actions:

  * before - this action is intended for application, it allows metadata author to inject any commands needed to be executed before main action, but uPaaS admin provide default commands for this action if needed
  * main - this action should be used to run all main actions for given interpreter (like bundler or pip commands), application can override it in metadata file if really needed
  * after - this action is intended for application, they should call database migration commands here, uPaaS administrator can provide default actions in configuration file
  * finalize - this action can only be customized by uPaaS administrator, it is intended to run cleanup commands

Step 6 - Creating package archive
---------------------------------

Compressed tar archive will be created and uploaded to configured storage handler. This can be customized only by uPaaS administrator.
