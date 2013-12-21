Example metadata
================

Note
----

Current version of metadata examples can be found at `github <https://github.com/prymitive/upaas-admin/tree/master/tests/meta>`_.

Ruby On Rails app
-----------------

In this example `redmine <http://redmine.org>`_ will be deployed.

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


Django app
----------

In this example `Django jQuery File Upload <https://github.com/sigurdga/django-jquery-file-upload>`_ demo application will be deployed.

.. code-block:: yaml

    # only git is needed to clone application repository from github
    os:
      Debian: &debian
        packages:
          - git-core
      Ubuntu: *debian

    # both 2.7 and 3.x is supported (needs django == 1.5)
    # application will be deployed using Django built in WSGI handler module
    interpreter:
      type: python
      versions:
        - "3.2"
        - "2.7"
      settings:
        module: django.core.handlers.wsgi:WSGIHandler()

    # clone repository from github
    repository:
      clone: git clone --depth=10 --quiet git://github.com/sigurdga/django-jquery-file-upload.git %destination%
      update:
        - git reset --hard
        - git pull
      info: git log -n 1
      changelog: git log --no-merges %old%..%new%"

    # Django needs to be told how to load settings module
    env:
      DJANGO_SETTINGS_MODULE: django-jquery-file-upload.settings

    # django 1.5 is required, 1.6 is not yet supported
    # we also symlink app directory since /home will be added to python modules path
    # so with this symlink python can load our app as django-jquery-file-upload module
    actions:
      setup:
        main:
          - ln -sf /home/app /home/django-jquery-file-upload
          - pip install "django<1.6"
          - pip install pillow
          - python manage.py syncdb --noinput

    # pass additional settings to uWSGI so that all it will handle all requests for static files
    uwsgi:
      settings:
        - "static-map = /static=fileupload/static"


PHP app
-------

In this example phpmyadmin will be deployed.

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
