Web development fun with Lettuce and Django
===========================================

[Django](http://djangoproject.com) is a awesome web framework, very
mature, aims for simplicity and the best of all: it's fun to use it.

To make it even more fun, lettuce has built-in support for Django.

Getting started
---------------

### 1. install the lettuce django app

Pick up any Django project, and add `lettuce.django` in its
`settings.py` configuration file:

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
for the `my_app` django application.

### 2. create the feature directories

Lettuce will look for a `features` folder inside every installed app:

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

### 3. write your first feature

`@index.feature`:

    Feature: Rocking with lettuce and django

        Scenario: Simple Hello World
            Given I access the url "/"
            Then I see the header "Hello World"

        Scenario: Hello + capitalized name
            Given I access the url "/some-name"
            Then I see the header "Hello Some Name"

`@index-steps.py`:

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

### 4. run the tests

Once you install the `lettuce.django` app, the command `harvest` will be
available:

    user@machine:~projects/djangoproject $ python manage.py harvest

The `harvest` command executes the
`django.test.utils.setup_test_environment` function before it starts up
the Django server. One side-effect is that the server will not send out
any emails, because it configures Django to use the
[locmem](https://docs.djangoproject.com/en/dev/topics/email/#in-memory-backend)
in-memory email backend.

### 5. specifying feature files

The `harvest` command accepts a path to feature files, in order to run
only the features you want.

Example:

    user@machine:~projects/djangoproject $ python manage.py harvest path/to/my-test.feature

### 6. grab actual example code

In order to assure that lettuce integrate well with Django, it have a
set of integration tests, there are a actual Django project running with
lettuce.

You can grab the code at the
[alfaces](http://github.com/gabrielfalcao/lettuce/tree/master/tests/integration/django/alfaces/)
folder of lettuce git repository

Technical details
-----------------

If you want to write acceptance tests that run with web browsers, you
can user tools like [twill](http://twill.idyll.org/python-api.html),
[selenium](http://seleniumhq.org/docs/appendix_installing_python_driver_client.html),
[webdriver](http://code.google.com/p/selenium/wiki/PythonBindings?redir=1)
and [windmill](http://www.getwindmill.com/)

### red-tape-less builtin server

Lettuce cleverly runs an instance of the built-in Django HTTP server in
the background. It tries to bind the HTTP server at `localhost:8000` but
if the port is busy, it keeps trying to run in higher ports: 8001, 8002
and so on until it reaches the maximum port number 65535.

So that you can use browser-based tools such as those listed above to
access Django.

### figure out django urls

As the Django HTTP server can be running in any port within the range
8000 - 65535, it could be hard to figure out the correct URL for your
project, right?

Wrong!

Lettuce is here for you. Within your steps you can use the `django_url`
utility function:

    from lettuce import step, world
    from lettuce.django import django_url

    @step(r'Given I navigate to "(.*)"')
    def navigate_to_url(step, url):
        full_url = django_url(url)
        world.browser.get(full_url)

#### what does `django_url` do ?!?

It prepends a Django-internal URL with the HTTP server address.

In other words, if lettuce binds the http server to localhost:9090 and
you call `django_url` with `"/admin/login"`:

    from lettuce.django import django_url
    django_url("/admin/login")

It returns:

    "http://localhost:9090/admin/login"

### terrain also available in django projects

At this point you probably know how :ref:\`terrain-py\` works, and it
also works with Django projects.

You can setup environment and stuff like that within a `terrain.py` file
located at the root of your Django project.

Taking the very first example of this documentation page, your Django
project layout would like like this:

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

Notice the `terrain.py` file at the project root, there you can populate
the :ref:\`lettuce-world\` and organize your features and steps with it
:)

### Running without HTTP server

Sometimes you may just do not want to run Django's built-in HTTP server
running in background, in those cases all you need to do is run the
`harvest` command with the `--no-server` or `-S` option.

Example:

    python manage.py harvest --no-server
    python manage.py harvest -S

Notice that if you have tests with some email checks they wont pass because
email queue should be in the same process with django server. To bypass this
limitation you can use --smtp-queue option.

Example:

Define lettuce smtp server host & port in `settings.py` :

    LETTUCE_SMTP_QUEUE_HOST
    LETTUCE_SMTP_QUEUE_PORT

This is where lettuce smtp server will be listen to for incoming email messages.

Configure django application smtp settings accordingly for sending emails
to lettuce smtp server:

    EMAIL_HOST  # = LETTUCE_SMTP_QUEUE_HOST
    EMAIL_PORT  # = LETTUCE_SMTP_QUEUE_PORT
    EMAIL_HOST_USER = None
    EMAIL_HOST_PASSWORD = None

Configured this way django application will send email messages to lettuce email queue.

Run django application

Run lettuce

    ./manage.py harvest --no-server --smtp-queue

Notice that this plugin use `lettuce.django.email.queue` , but as it parse incoming
smtp emails to django `EmailMessage`, there can be some differences
in emails (emails usually won't be fully equal in all headers, attachments, etc,
but it works ok for simple cases like checking body and common headers).

### running the HTTP server in other port than 8000

If you face the problem of having lettuce running on port 8000, you can
change that behaviour.

Before running the server, lettuce will try to read the setting
`LETTUCE_SERVER_PORT` which **must** be a **integer**

Example:

    LETTUCE_SERVER_PORT = 7000

This can be really useful if 7000 is your default development port, for
example.

### running the HTTP server with settings.DEBUG=True

In order to run tests against the nearest configuration of production,
lettuce sets up settings.DEBUG=False

However, for debug purposes one can face a misleading HTTP 500 error
without traceback in Django. For those cases lettuce provides the
`--debug-mode` or `-d` option.

    python manage.py harvest --debug-mode
    python manage.py harvest -d

### running only the specified scenarios

You can also specify the index of the scenarios you want to run through
the command line, to do so, run with `--scenarios` or `-s` options
followed by the scenario numbers separated by commas.

For example, let's say you want to run the scenarios 4, 7, 8 and 10:

    python manage.py harvest --scenarios=4,7,8,10
    python manage.py harvest -s 4,7,8,10

### to run or not to run? That is the question!

During your development workflow you may face two situations:

#### running tests from just certain apps

Lettuce takes a comma-separated list of app names to run tests against.

For example, the command below would run ONLY the tests within the apps
`myapp` and `foobar`:

    python manage.py harvest --apps=myapp,foobar

    # or

    python manage.py harvest --a  myapp,foobar

You can also specify it at `settings.py` so that you won't need to type
the same command-line parameters all the time:

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

#### running tests from all apps, except by some

Lettuce takes a comma-separated list of app names which tests must NOT
be ran.

For example, the command below would run ALL the tests BUT those within
the apps `another_app` and `foobar`:

    python manage.py harvest --avoid-apps=another_app,foobar

You can also specify it at `settings.py` so that you won't need to type
the same command-line parameters all the time:

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
