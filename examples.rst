Example metadata
================

Note
----

Current version of metadata examples can be found at `github <https://github.com/prymitive/upaas-admin/tree/master/tests/meta>`_.

Redmine
-------

.. code-block:: yaml

    # packages needed on Ubuntu and Debian
    os:
    Debian: &debian
      packages:
        - git-core
        - rake
        - libpq-dev
        - libmysqlclient-dev
        - libsqlite3-dev
        - imagemagick
        - libmagickcore-dev
        - libmagickwand-dev
    Ubuntu: *debian

    # current redmine will run with any recent ruby verison
    # all supported versions can be found on redmine wiki
    # uPaaS will use highest version supported by both redmine and uPaaS itself
    interpreter:
    type: ruby
    versions:
      - 2.0.0
      - 1.9.3

    # fetch redmine code from github repo
    repository:
    clone: git clone --depth=10 --quiet --branch 2.4-stable git://github.com/redmine/redmine.git %destination%
    update:
      - rm -f Gemfile.lock
      - git reset --hard
      - git pull
    info: git log -n 1
    changelog: git log --no-merges %old%..%new%"

    # REDMINE_LANG sets default language that redmine will use for UI and for seeding db with sample data
    env:
    REDMINE_LANG: en

    # after installing all gems run after.sh script
    actions:
    setup:
      after: /bin/bash /tmp/after.sh

    # create two files
    # after.sh - script that will execute all commands required to deploy redmine
    # config/database.yml - redmine database config file, modify it as needed
    files:
      /tmp/after.sh: |
        if [ -n "$UPAAS_FRESH_PACKAGE" ] ; then
          echo ">>> Generating secret token"
          rake generate_secret_token || exit 1
        fi
        echo ">>> Migrating database"
        rake db:migrate || exit 1
        if [ -n "$UPAAS_FRESH_PACKAGE" ] ; then
          echo ">>> Loading default data"
          rake redmine:load_default_data || exit 1
        fi
      config/database.yml: |
        production:
          adapter: postgresql
          database: redmine
          host: < db host >
          username: redmine
          password: "< password >"
