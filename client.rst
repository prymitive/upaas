CLI client
==========

Installation
------------

.. code-block:: none

    sudo add-apt-repository ppa:upaas/stable
    sudo apt-get update
    sudo apt-get install python-upaas-client

Configuration file
------------------

Before we can start using uPaaS client we must create configuration file:

.. code-block:: yaml

    server:
      url: http://admin.upaas.org
      login: john
      apikey: 98f2ec25ffc9d57e7f168d7c1798c51da1c6a7e3

``url``
.......

URL under which we can access our uPaaS admin web UI.

``login``
.........

Your user login.

``apikey``
..........

API key generated for your user account.

Example usage
.............

Deploying redmine:

.. code-block:: none

    upaas register redmine path/to/redmine.yml
    upaas build redmine
    [wait until package is build]
    upaas start redmine -H 4 512

We will have redmine instance with HA enabled (multiple backends), 4 processes and 512MB memory limit (per backend).
