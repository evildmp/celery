=================================
 celery - Distributed Task Queue
=================================

.. image:: http://cloud.github.com/downloads/celery/celery/celery_128.png

:Version: 2.6.0rc5
:Web: http://celeryproject.org/
:Download: http://pypi.python.org/pypi/celery/
:Source: http://github.com/celery/celery/
:Keywords: task queue, job queue, asynchronous, rabbitmq, amqp, redis,
  python, webhooks, queue, distributed

--

.. contents::
    :local:

.. _celery-synopsis:

Synopsis
========

Celery is an open source asynchronous task queue/job queue based on
distributed message passing. It is focused on real-time operation,
but supports scheduling as well.

The execution units, called tasks, are executed concurrently on one or
more worker nodes using multiprocessing, `Eventlet`_ or `gevent`_.  Tasks can
execute asynchronously (in the background) or synchronously
(wait until ready).

Celery is used in production systems to process millions of tasks every hour.

Celery is written in Python, but the protocol can be implemented in any
language.  It can also `operate with other languages using webhooks`_.
There's also `RCelery` for the Ruby programming language, and a `PHP client`.

The recommended message broker is `RabbitMQ`_, but support for
`Redis`_, `MongoDB`_, `Beanstalk`_, `Amazon SQS`_, `CouchDB`_ and
databases (using `SQLAlchemy`_ or the `Django ORM`_) is also available.

Celery is easy to integrate with web frameworks, some of which even have
integration packages:

    +--------------------+------------------------+
    | `Django`_          | `django-celery`_       |
    +--------------------+------------------------+
    | `Pyramid`_         | `pyramid_celery`_      |
    +--------------------+------------------------+
    | `Pylons`_          | `celery-pylons`_       |
    +--------------------+------------------------+
    | `Flask`_           | `flask-celery`_        |
    +--------------------+------------------------+
    | `web2py`_          | `web2py-celery`_       |
    +--------------------+------------------------+
    | `Tornado`_         | `tornado-celery`_      |
    +--------------------+------------------------+

.. _`RCelery`: http://leapfrogdevelopment.github.com/rcelery/
.. _`PHP client`: https://github.com/gjedeer/celery-php
.. _`RabbitMQ`: http://www.rabbitmq.com/
.. _`Redis`: http://code.google.com/p/redis/
.. _`SQLAlchemy`: http://www.sqlalchemy.org/
.. _`Django`: http://djangoproject.com/
.. _`Django ORM`: http://djangoproject.com/
.. _`Memcached`: http://memcached.org/
.. _`Eventlet`: http://eventlet.net/
.. _`gevent`: http://gevent.org/
.. _`Beanstalk`: http://kr.github.com/beanstalkd/
.. _`MongoDB`: http://mongodb.org/
.. _`CouchDB`: http://couchdb.apache.org/
.. _`Amazon SQS`: http://aws.amazon.com/sqs/
.. _`Pylons`: http://pylonshq.com/
.. _`Flask`: http://flask.pocoo.org/
.. _`web2py`: http://web2py.com/
.. _`Bottle`: http://bottlepy.org/
.. _`Pyramid`: http://docs.pylonsproject.org/en/latest/docs/pyramid.html
.. _`pyramid_celery`: http://pypi.python.org/pypi/pyramid_celery/
.. _`django-celery`: http://pypi.python.org/pypi/django-celery
.. _`celery-pylons`: http://pypi.python.org/pypi/celery-pylons
.. _`flask-celery`: http://github.com/ask/flask-celery/
.. _`web2py-celery`: http://code.google.com/p/web2py-celery/
.. _`Tornado`: http://www.tornadoweb.org/
.. _`tornado-celery`: http://github.com/mher/tornado-celery/
.. _`operate with other languages using webhooks`:
    http://celery.github.com/celery/userguide/remote-tasks.html
.. _`limited support`:
    http://kombu.readthedocs.org/en/latest/introduction.html#transport-comparison

.. _celery-overview:

Overview
========

This is a high level overview of the architecture.

.. image:: http://cloud.github.com/downloads/celery/celery/Celery-Overview-v4.jpg

The broker delivers tasks to the worker instances.
A worker instance is started by running the `celery worker` program.
You can have many networked machines running worker instances, forming a
cluster, or you can run everything on a single machine.

The return value of the task can be stored for later retrieval,
and the progress of the task can be tracked (called the task's *state*).

.. _celery-example:

Example
=======

You probably want to see some code by now, so here's an example task
which adds two numbers:
::

    from celery import task

    @task()
    def add(x, y):
        return x + y

You can execute the task in the background, or wait for it to finish::

    >>> result = add.delay(4, 4)
    >>> result.wait() # wait for and return the result
    8

Simple!

.. _celery-features:

Features
========

    +-----------------+----------------------------------------------------+
    | Messaging       | Supported brokers include `RabbitMQ`_, `Redis`_,   |
    |                 | `MongoDB`_, `Beanstalk`_, SQL databases,           |
    |                 | Amazon SQS and more.                               |
    +-----------------+----------------------------------------------------+
    | Fault-tolerant  | Excellent configurable error recovery when using   |
    |                 | `RabbitMQ`, ensures your tasks are never lost.     |
    +-----------------+----------------------------------------------------+
    | Distributed     | Runs on one or more machines. Supports             |
    |                 | broker `clustering`_ and `HA`_ when used in        |
    |                 | combination with `RabbitMQ`_.  You can set up new  |
    |                 | workers without central configuration (e.g. use    |
    |                 | your grandma's laptop to help if the queue is      |
    |                 | temporarily congested).                            |
    +-----------------+----------------------------------------------------+
    | Concurrency     | Concurrency is achieved by using multiprocessing,  |
    |                 | `Eventlet`_, `gevent` or a mix of these.           |
    +-----------------+----------------------------------------------------+
    | Scheduling      | Supports recurring tasks like cron, or specifying  |
    |                 | an exact date or countdown for when after the task |
    |                 | should be executed.                                |
    +-----------------+----------------------------------------------------+
    | Latency         | Low latency means you are able to execute tasks    |
    |                 | *while the user is waiting*.                       |
    +-----------------+----------------------------------------------------+
    | Return Values   | Task return values can be saved to the selected    |
    |                 | result store backend. You can wait for the result, |
    |                 | retrieve it later, or ignore it.                   |
    +-----------------+----------------------------------------------------+
    | Result Stores   | Database, `MongoDB`_, `Redis`_, `Memcached`_,      |
    |                 | `Cassandra`, or `AMQP`_ (message notification).    |
    +-----------------+----------------------------------------------------+
    | Webhooks        | Your tasks can also be HTTP callbacks, enabling    |
    |                 | cross-language communication.                      |
    +-----------------+----------------------------------------------------+
    | Rate limiting   | Supports rate limiting by using the token bucket   |
    |                 | algorithm, which accounts for bursts of traffic.   |
    |                 | Rate limits can be set for each task type, or      |
    |                 | globally for all.                                  |
    +-----------------+----------------------------------------------------+
    | Routing         | Using AMQP's flexible routing model you can route  |
    |                 | tasks to different workers, or select different    |
    |                 | message topologies, by configuration or even at    |
    |                 | runtime.                                           |
    +-----------------+----------------------------------------------------+
    | Remote-control  | Worker nodes can be controlled from remote by      |
    |                 | using broadcast messaging.  A range of built-in    |
    |                 | commands exist in addition to the ability to       |
    |                 | easily define your own. (AMQP/Redis only)          |
    +-----------------+----------------------------------------------------+
    | Monitoring      | You can capture everything happening with the      |
    |                 | workers in real-time by subscribing to events.     |
    |                 | A real-time web monitor is in development.         |
    +-----------------+----------------------------------------------------+
    | Serialization   | Supports Pickle, JSON, YAML, or easily defined     |
    |                 | custom schemes. One task invocation can have a     |
    |                 | different scheme than another.                     |
    +-----------------+----------------------------------------------------+
    | Tracebacks      | Errors and tracebacks are stored and can be        |
    |                 | investigated after the fact.                       |
    +-----------------+----------------------------------------------------+
    | UUID            | Every task has an UUID (Universally Unique         |
    |                 | Identifier), which is the task id used to query    |
    |                 | task status and return value.                      |
    +-----------------+----------------------------------------------------+
    | Retries         | Tasks can be retried if they fail, with            |
    |                 | configurable maximum number of retries, and delays |
    |                 | between each retry.                                |
    +-----------------+----------------------------------------------------+
    | Task Sets       | A Task set is a task consisting of several         |
    |                 | sub-tasks. You can find out how many, or if all    |
    |                 | of the sub-tasks has been executed, and even       |
    |                 | retrieve the results in order. Progress bars,      |
    |                 | anyone?                                            |
    +-----------------+----------------------------------------------------+
    | Made for Web    | You can query status and results via URLs,         |
    |                 | enabling the ability to poll task status using     |
    |                 | Ajax.                                              |
    +-----------------+----------------------------------------------------+
    | Error Emails    | Can be configured to send emails to the            |
    |                 | administrators when tasks fails.                   |
    +-----------------+----------------------------------------------------+
    | Message signing | Supports message signing. Messages are signed      |
    |                 | using public-key cryptography.                     |
    +-----------------+----------------------------------------------------+


.. _`clustering`: http://www.rabbitmq.com/clustering.html
.. _`HA`: http://www.rabbitmq.com/pacemaker.html
.. _`AMQP`: http://www.amqp.org/
.. _`Stomp`: http://stomp.codehaus.org/
.. _`Tokyo Tyrant`: http://tokyocabinet.sourceforge.net/

.. _celery-documentation:

Documentation
=============

The `latest documentation`_ with user guides, tutorials and API reference
is hosted at Github.

.. _`latest documentation`: http://celery.github.com/celery/

.. _celery-installation:

Installation
============

You can install Celery either via the Python Package Index (PyPI)
or from source.

To install using `pip`,::

    $ pip install -U Celery

To install using `easy_install`,::

    $ easy_install -U Celery

.. _bundles:

Bundles
-------

Celery also defines a group of bundles that can be used
to install Celery and the dependencies for a given feature.

The following bundles are available:

:`celery-with-redis`_:
    for using Redis as a broker.

:`celery-with-mongodb`_:
    for using MongoDB as a broker.

:`django-celery-with-redis`_:
    for Django, and using Redis as a broker.

:`django-celery-with-mongodb`_:
    for Django, and using MongoDB as a broker.

.. _`celery-with-redis`:
    http://pypi.python.org/pypi/celery-with-redis/
.. _`celery-with-mongodb`:
    http://pypi.python.org/pypi/celery-with-mongdb/
.. _`django-celery-with-redis`:
    http://pypi.python.org/pypi/django-celery-with-redis/
.. _`django-celery-with-mongodb`:
    http://pypi.python.org/pypi/django-celery-with-mongdb/

.. _celery-installing-from-source:

Downloading and installing from source
--------------------------------------

Download the latest version of Celery from
http://pypi.python.org/pypi/celery/

You can install it by doing the following,::

    $ tar xvfz celery-0.0.0.tar.gz
    $ cd celery-0.0.0
    $ python setup.py build
    # python setup.py install # as root

.. _celery-installing-from-git:

Using the development version
-----------------------------

You can clone the repository by doing the following::

    $ git clone https://github.com/celery/celery
    $ cd celery
    $ python setup.py develop

The development version will usually also depend on the development
version of `kombu`_, the messaging framework Celery uses
to send and receive messages, so you should also install that from git::

    $ git clone https://github.com/celery/kombu
    $ cd kombu
    $ python setup.py develop

.. _`kombu`: http://kombu.readthedocs.org/en/latest/

.. _getting-help:

Getting Help
============

.. _mailing-list:

Mailing list
------------

For discussions about the usage, development, and future of celery,
please join the `celery-users`_ mailing list.

.. _`celery-users`: http://groups.google.com/group/celery-users/

.. _irc-channel:

IRC
---

Come chat with us on IRC. The **#celery** channel is located at the `Freenode`_
network.

.. _`Freenode`: http://freenode.net

.. _bug-tracker:

Bug tracker
===========

If you have any suggestions, bug reports or annoyances please report them
to our issue tracker at http://github.com/celery/celery/issues/

.. _wiki:

Wiki
====

http://wiki.github.com/celery/celery/

.. _contributing-short:

Contributing
============

Development of `celery` happens at Github: http://github.com/celery/celery

You are highly encouraged to participate in the development
of `celery`. If you don't like Github (for some reason) you're welcome
to send regular patches.

Be sure to also read the `Contributing to Celery`_ section in the
documentation.

.. _`Contributing to Celery`: http://celery.github.com/celery/contributing.html

.. _license:

License
=======

This software is licensed under the `New BSD License`. See the ``LICENSE``
file in the top distribution directory for the full license text.

.. # vim: syntax=rst expandtab tabstop=4 shiftwidth=4 shiftround

