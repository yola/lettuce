.. _recipes-django-lxml:

###########################################
Web development fun with Lettuce and Django
###########################################

Django_ is a awesome web framework, very mature, aims for simplicity
and the best of all: it's fun to use it.

To make it even more fun, lettuce has built-in support for Django.

***************
Getting started
***************

1. install the lettuce django app
=================================

Pick up any Django project, and add ``lettuce.django`` in its
``settings.py`` configuration file:

::

   INSTALLED_APPS = (
       'django.contrib.auth',
       'django.contrib.admin',

       # ... other apps here ...
       'my_app',
       'foobar',
       'another_app',
       'lettuce.django', # this guy will do the job :)
   )

Considering the configuration above, let's say we want to write tests
for the ``my_app`` django application.

2. create the feature directories
=================================

Lettuce will look for a ``features`` folder inside every installed app:

::

    /home/user/projects/djangoproject
         | settings.py
         | manage.py
         | urls.py
         | my_app
               | features
                    - index.feature
                    - index.py
         | foobar
               | features
                    - carrots.feature
                    - foobar-steps.py
         | another_app
               | features
                    - first.feature
                    - second.feature
                    - many_steps.py

3. write your first feature
===========================

``@index.feature``:

.. highlight:: ruby

::

    Feature: Rocking with lettuce and django

        Scenario: Simple Hello World
            Given I access the url "/"
            Then I see the header "Hello World"

        Scenario: Hello + capitalized name
            Given I access the url "/some-name"
            Then I see the header "Hello Some Name"


``@index-steps.py``:

.. highlight:: python

::

    from lettuce import *
    from lxml import html
    from django.test.client import Client
    from nose.tools import assert_equals

    @before.all
    def set_browser():
        world.browser = Client()

    @step(r'I access the url "(.*)"')
    def access_url(step, url):
        response = world.browser.get(url)
        world.dom = html.fromstring(response.content)

    @step(r'I see the header "(.*)"')
    def see_header(step, text):
        header = world.dom.cssselect('h1')[0]
        assert header.text == text

4. run the tests
================

Once you install the ``lettuce.django`` app, the command ``harvest`` will be available:

.. highlight:: bash

::

   user@machine:~projects/djangoproject $ python manage.py harvest


The ``harvest`` command executes the ``django.test.utils.setup_test_environment``
function before it starts up the Django server. Typically, invoking this function
would configure Django to use the locmem_ in-memory email backend. However,
Lettuce uses a custom Django email backend to support retrieving email from
Lettuce test scripts. See :ref:`lettuce-checking-email` for more details.

5. specifying feature files
===========================

The ``harvest`` command accepts a path to feature files, in order to run
only the features you want.

Example:

.. highlight:: bash

::

   user@machine:~projects/djangoproject $ python manage.py harvest path/to/my-test.feature

6. grab actual example code
===========================

In order to assure that lettuce integrate well with Django, it have a
set of integration tests, there are a actual Django project running
with lettuce.

You can grab the code at the alfaces_ folder of lettuce git repository

*****************
Technical details
*****************

If you want to write acceptance tests that run with web browsers, you
can user tools like twill_, selenium_, webdriver_ and windmill_

red-tape-less builtin server
============================

Lettuce cleverly runs an instance of the built-in Django HTTP server in
the background. It tries to bind the HTTP server at ``localhost:8000``
but if the port is busy, it keeps trying to run in higher ports: 8001,
8002 and so on until it reaches the maximum port number 65535.

.. note::

   You can override the default starting port from "8000" to any other
   port you want.

   To do so, refer to "running the HTTP server in other port than
   8000" below.

So that you can use browser-based tools such as those listed above to
access Django.

.. warning::

   When running the http server, lettuce sets the environment
   variables SERVER_NAME and SERVER_PORT. It was brought for a GAE_
   issue. If it can possibly bring any errors, be warned.

figure out django urls
======================

As the Django HTTP server can be running in any port within the range
8000 - 65535, it could be hard to figure out the correct URL for your
project, right?

Wrong!

Lettuce is here for you. Within your steps you can use the
``django_url`` utility function:

.. highlight:: python

::

    from lettuce import step, world
    from lettuce.django import django_url

    @step(r'Given I navigate to "(.*)"')
    def navigate_to_url(step, url):
        full_url = django_url(url)
        world.browser.get(full_url)


what does ``django_url`` do ?!?
-------------------------------

It prepends a Django-internal URL with the HTTP server address.

In other words, if lettuce binds the http server to localhost:9090 and
you call ``django_url`` with ``"/admin/login"``:

.. highlight:: python

::

    from lettuce.django import django_url
    django_url("/admin/login")

It returns:

.. highlight:: python

::

    "http://localhost:9090/admin/login"

terrain also available in django projects
=========================================

At this point you probably know how :ref:`terrain-py` works, and it
also works with Django projects.

You can setup environment and stuff like that within a ``terrain.py``
file located at the root of your Django project.

Taking the very first example of this documentation page, your Django
project layout would like like this:

::

    /home/user/projects/djangoproject
         | settings.py
         | manage.py
         | urls.py
         | terrain.py
         | my_app
               | features
                    - index.feature
                    - index.py
         | foobar
               | features
                    - carrots.feature
                    - foobar-steps.py
         | another_app
               | features
                    - first.feature
                    - second.feature
                    - many_steps.py

Notice the ``terrain.py`` file at the project root, there you can
populate the :ref:`lettuce-world` and organize your features and steps
with it :)

.. _lettuce-checking-email:

Checking email
==============

When you run your Django server under lettuce, emails sent by your server
do not get transmitted over the Internet. Instead, these emails are
added to a :class:`multiprocessing.Queue` object at
``lettuce.django.mail.queue``.

Example:

.. highlight:: python

::

  from lettuce import step
  from lettuce.django import mail
  from nose.tools import assert_equals


  @step(u'an email is sent to "([^"]*?)" with subject "([^"]*)"')
  def email_sent(step, to, subject):
      message = mail.queue.get(True, timeout=5)
      assert_equals(message.subject, subject)
      assert_equals(message.recipients(), [to])


Running without HTTP server
===========================

Sometimes you may just do not want to run Django's built-in HTTP server
running in background, in those cases all you need to do is run the
``harvest`` command with the ``--no-server`` or ``-S`` option.

Example:

.. highlight:: bash

::

   python manage.py harvest --no-server
   python manage.py harvest -S

running the HTTP server in other port than 8000
===============================================

If you face the problem of having lettuce running on port 8000, you
can change that behaviour.

Before running the server, lettuce will try to read the setting ``LETTUCE_SERVER_PORT`` which **must** be a **integer**

Example:

.. highlight:: python

::

   LETTUCE_SERVER_PORT = 7000

This can be really useful if 7000 is your default development port,
for example.


running the HTTP server with settings.DEBUG=True
================================================

In order to run tests against the nearest configuration of production,
lettuce sets up settings.DEBUG=False

However, for debug purposes one can face a misleading HTTP 500 error without traceback in Django.
For those cases lettuce provides the ``--debug-mode`` or ``-d`` option.

.. highlight:: bash

::

   python manage.py harvest --debug-mode
   python manage.py harvest -d

using the test database
=======================

If you want to use a test database by default, instead of a live database,
with your test server you can specify the ``-T`` flag or set the following
configuration variable in ``settings.py``.

.. highlight:: python

::

    LETTUCE_USE_TEST_DATABASE = True

running only the specified scenarios
====================================

You can also specify the index of the scenarios you want to run
through the command line, to do so, run with ``--scenarios`` or ``-s``
options followed by the scenario numbers separated by commas.

For example, let's say you want to run the scenarios 4, 7, 8 and 10:

.. highlight:: bash

::

   python manage.py harvest --scenarios=4,7,8,10
   python manage.py harvest -s 4,7,8,10

to run or not to run? That is the question!
===========================================

During your development workflow you may face two situations:

running tests from just certain apps
------------------------------------

Lettuce takes a comma-separated list of app names to run tests against.

For example, the command below would run ONLY the tests within the apps ``myapp`` and ``foobar``:

.. highlight:: bash

::

   python manage.py harvest --apps=myapp,foobar

   # or

   python manage.py harvest --a  myapp,foobar

You can also specify it at ``settings.py`` so that you won't need to type the same command-line parameters all the time:

.. highlight:: python

::

   LETTUCE_APPS = (
       'myapp',
       'foobar',
   )
   INSTALLED_APPS = (
       'django.contrib.auth',
       'django.contrib.admin',
       'my_app',
       'foobar',
       'another_app',
       'lettuce.django',
   )


running tests from all apps, except by some
-------------------------------------------

Lettuce takes a comma-separated list of app names which tests must NOT be ran.

For example, the command below would run ALL the tests BUT those within the apps ``another_app`` and ``foobar``:

.. highlight:: bash

::

   python manage.py harvest --avoid-apps=another_app,foobar

You can also specify it at ``settings.py`` so that you won't need to type the same command-line parameters all the time:

.. highlight:: python

::

   LETTUCE_AVOID_APPS = (
       'another_app',
       'foobar',
   )

   INSTALLED_APPS = (
       'django.contrib.auth',
       'django.contrib.admin',
       'my_app',
       'foobar',
       'another_app',
       'lettuce.django',
   )

.. _alfaces: http://github.com/gabrielfalcao/lettuce/tree/master/tests/integration/django/alfaces/
.. _Django: http://djangoproject.com
.. _twill: http://twill.idyll.org/python-api.html
.. _selenium: http://seleniumhq.org/docs/appendix_installing_python_driver_client.html
.. _windmill: http://www.getwindmill.com/
.. _webdriver: http://code.google.com/p/selenium/wiki/PythonBindings?redir=1
.. _GAE: http://code.google.com/appengine
.. _locmem: https://docs.djangoproject.com/en/dev/topics/email/#in-memory-backend
