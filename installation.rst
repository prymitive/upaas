Installing uPaaS
================

Pre-Requirements
----------------

MongoDB database accessible from backend nodes. MongoDB is used both for uPaaS data and package files, so it might grow to several gigabytes or more, depending on the number of registered application. Each package will use at least 200MB (in case of Ubuntu server).

Domain name that will be pointing to router nodes. You don't need to buy one for testing purposes or intranet usage, if you have dnsmasq running as your dns server you might add those lines to dnsmasq configuration file:

.. code-block:: none

    address=/upaas.domain/<router node ip>

Ubuntu server 12.04 LTS installation
------------------------------------

uPaaS is packaged, developed and tested on Ubuntu 12.04 LTS release, but it should be working on any recent Linux distribution. PPA containing uPaaS packages requires some packages that are not available in standard 12.04 release, so few other PPAs must be added to fulfil those dependencies.

Router nodes
^^^^^^^^^^^^

Add required PPAs:

.. code-block:: none

    sudo add-apt-repository ppa:upaas/stable

Install upaas-router meta package. It contains uWSGI config files needed to run uWSGI FastRouter node.

.. code-block:: none

    sudo apt-get update
    sudo apt-get dist-upgrade
    sudo apt-get install upaas-router

Place custom SSL certificate and key in /etc/upaas/ssl (server.crt and server.key). This step is optional, by default self signed certificate is used.

Backend nodes
^^^^^^^^^^^^^

Add required PPAs:

.. code-block:: none

    sudo add-apt-repository ppa:brightbox/ruby-ng
    sudo add-apt-repository ppa:fkrull/deadsnakes
    sudo add-apt-repository ppa:ondrej/php5
    sudo add-apt-repository ppa:upaas/stable

Install python-upaas-admin package. It contains uPaaS API and web UI server.

.. code-block:: none

    sudo apt-get update
    sudo apt-get dist-upgrade
    sudo apt-get install python-upaas-admin

After installing uPaaS admin package edit ``/etc/upaas/upaas.yml`` config, it must be identical on all backend nodes. Be sure to set proper storage handler since 0.1.0 release defaults to local file system storage.

See See :doc:`configuration`.

Once configured uPaaS web UI will be avaiable under http://backend-node-ip/.
See ``/etc/upaas/upaas_admin_local.ini`` for exampled configuration needed to connect uPaaS web UI to upaas-router nodes for high availability.

Create database indexes
-----------------------

.. code-block:: none

    upaas_admin create_indexes

Add first user
--------------

Once installed and configured we need to create user with administrator rights:

.. code-block:: none

    upaas_admin create_user --login john --firstname John --lastname Doe --email john@doe.com --admin

Add router node(s)
------------------

Login as administrator, go to admin area and create router node(s). Backends will auto-register during task worker startup.
