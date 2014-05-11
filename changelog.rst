Changelog
=========

0.3.1
-----

Released: under development

* improved instance placement - uPaaS is now aware of each backend resources and can pick the best backends for each application instance
* improved backend and router configuration
* improved self healing - uPaaS will now try to detect and fix more instance issues

0.3.0
-----

Released: 04.05.2014

* task queue replaced with new task framework
* backbone and tastypie used for ajax
* uWSGI 2.0 packages in PPA
* many improvements under the hood

0.2.1
-----

Released: 06.01.2014

* Added test suite
* Fixed many small bugs found by new tests
* Django 1.6.1 is now used
* Refactored custom domains (db migration needed with ``upaas_admin migrate_db`` command)

0.2
---

Released: 22.12.2013

First usable release with most basic functionality implemented.

0.1
---

Released: 27.10.2013

Technical preview release.
