Installing uPaaS
================

Pre-Requirements
----------------

MongoDB database accessible from backend nodes. MongoDB is used both for uPaaS
data and package files, so it might grow to several gigabytes or more, depending
on the number of registered application. Each package will use at least 200MB 
(in case of Ubuntu server).

Installing from sources
-----------------------

TODO

Ubuntu server 12.04 LTS installation
------------------------------------

uPaaS is packaged, developed and tested on Ubuntu 12.04 LTS release, but it
should be working on any recent Linux distribution.
PPA containing uPaaS packages requires some packages that are not available in
standard 12.04 release, so few other PPAs must be added to fulfil those
dependencies.

Router nodes
^^^^^^^^^^^^

Add required PPAs:

.. code:: bash

   sudo add-apt-repository ppa:nginx/stable
   sudo add-apt-repository ppa:chris-lea/zeromq
   sudo add-apt-repository ppa:upaas/stable

Install upaas-router meta package. It contains uWSGI config files needed to run
uWSGI FastRouter node.

.. code:: bash

   sudo apt-get update
   sudo apt-get dist-upgrade
   sudo apt-get install upaas-router

Backend nodes
^^^^^^^^^^^^^

Add required PPAs:

.. code:: bash

   sudo add-apt-repository ppa:brightbox/ruby-ng
   sudo add-apt-repository ppa:chris-lea/zeromq
   sudo add-apt-repository ppa:chris-lea/php5.5
   sudo add-apt-repository ppa:upaas/stable

Install python-upaas-admin package. It contains uPaaS API and web UI server.

.. code:: bash

   sudo apt-get update
   sudo apt-get dist-upgrade
   sudo apt-get install python-upaas-admin

After installing uPaaS admin package edit /etc/upaas/upaas.yml config, it must
be identical on all backend nodes.
