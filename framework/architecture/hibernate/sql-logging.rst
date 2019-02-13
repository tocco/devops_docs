SQL Statement Logging
=====================

The DevCon contains an SQL Logging tab which (if enabled) displays all executed SQL statements and their execution time.
Hibernate provides the option ``hibernate.show_sql`` that logs all statements to the console, however the parameters
are missing and only the ``?`` placeholder is visible.
Therefore we use `p6spy`_ to log the complete SQL statements.

.. _p6spy: https://github.com/p6spy/p6spy

Configuration
-------------

First a ``spy.properties`` file has to be created which configures an appender for the SQL statements:

.. code::

    appender=ch.tocco.nice2.persist.hibernate.spy.NiceLogger

A reference to this file needs to be passed as a command line argument when Nice2 is started:
``-Dspy.properties=../../persist/core/module/spy.properties``

In addition a custom JDBC Driver (:java:ref:`P6SpyDriver<com.p6spy.engine.spy.P6SpyDriver>`) and JDBC url (``jdbc:p6spy:postgresql://...``) needs to be used.
This is configured in the :java:ref:`HibernatePropertiesBuilder<ch.tocco.nice2.persist.hibernate.HibernatePropertiesBuilder>`.
The p6spy driver is currently only configured for the ``development`` environment.

Logging
-------

All statements are passed to the configured :java:ref:`NiceLogger<ch.tocco.nice2.persist.hibernate.spy.NiceLogger>` by the
p6spy driver.

To decouple the SQL statement consumers from p6spy the :java:ref:`SpyLoggerService<ch.tocco.nice2.persist.hibernate.spy.SpyLoggerService>`
acts as a bridge between the logger and the consumers.
Consumers can register with the service and receive the SQL statement events. Currently the only consumer is the
:java:ref:`SqlLogViewModel<ch.tocco.nice2.devcon.views.persist.sqllogger.SqlLogViewModel>` which publishes the statements
to the DevCon.

The :java:ref:`NiceLogger<ch.tocco.nice2.persist.hibernate.spy.NiceLogger>` publishes the SQL statements to the
:java:ref:`SpyLoggerService<ch.tocco.nice2.persist.hibernate.spy.SpyLoggerService>`.

