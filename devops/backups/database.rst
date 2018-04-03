.. include:: /global.rst

Database Backups
================

Get the Latest Backup
---------------------

Databases are dumped to disk, at ``/var/lib/postgresql-backup/``, before they are archived. If you need the most
current backups you can get it from there.

:ref:`restore-database` explains how to restore a dump.


Get Backup Created by :term:`CD`
--------------------------------

Dumps created by picking the option "dump database" in TeamCity are created in ``/var/postgres/dumps/``. Dumps in that
directory are removed after a few days. If a dump has been removed already, you'll have to extract it out of the
archive, see next section.

:ref:`restore-database` explains how to restore a dump.


Get Backup from Archive
-----------------------

The directories ``/var/lib/postgresql-backup/`` and ``/var/postgres/dumps/``, which contain dumps made during backups
and deployments respectively, are archived daily using :term:`BURP`. If you need backups/dumps no longer in these
directories, you can extract them from the archive.

.. warning::

      You need root access to access the archive.


List Available Archives
^^^^^^^^^^^^^^^^^^^^^^^

.. parsed-literal::

      $ sudo burp -C db2.tocco.cust.vshn.net -a list
      Backup: 0000155 2018-01-31 03:03:22 +0100 (deletable)
      Backup: 0000162 2018-02-09 01:14:05 +0100 (deletable)
      Backup: :green:`0000169` 2018-02-16 01:10:54 +0100 (deletable)
      Backup: 0000176 2018-02-23 01:06:28 +0100 (deletable)
      …


List Files in Archive
^^^^^^^^^^^^^^^^^^^^^

Show the content of directory ``/var/lib/postgresql-backup/`` in archive :green:`0000169`.

.. parsed-literal::

      $ sudo burp -C db2.tocco.cust.vshn.net -a list -b :green:`0000169` -r '^/var/lib/postgresql-backup/'
      Backup: 0000169 2018-02-16 01:10:54 +0100 (deletable)
      With regex: ^/var/lib/postgresql-backup/
      /var/lib/postgresql-backup/postgres-nice_awpf.dump.gz
      /var/lib/postgresql-backup/postgres-nice_awpftest.dump.gz
      :red:`/var/lib/postgresql-backup/postgres-nice_bnftest.dump.gz`
      /var/lib/postgresql-backup/postgres-nice_create_installation.dump.gz
      /var/lib/postgresql-backup/postgres-nice_dghtest.dump.gz
      /var/lib/postgresql-backup/postgres-nice_esrplus.dump.gz


Extract File from Archive
^^^^^^^^^^^^^^^^^^^^^^^^^

Restore **postgres-nice_bnftest.dump.gz** from backup :green:`0000169` to directory **~/restores/**.

.. parsed-literal::

      $ mkdir -p :blue:`~/restores/`
      $ sudo burp -C db2.tocco.cust.vshn.net -a restore -b :green:`0000169` -d :blue:`~/restores/` -r '^\ :red:`/var/lib/postgresql-backup/postgres-nice_bnftest.dump.gz`'
      …
      2018-03-09 16:01:30 +0100: burp[23156] restore finished
      $ ls -lh :blue:`~/restores/`:red:`var/lib/postgresql-backup/postgres-nice_bnftest.dump.gz`
      -rw-rw-r-- 1 postgres postgres 4.1G Feb 16 01:26 /home/peter.gerber/restores/var/lib/postgresql-backup/postgres-nice_bnftest.dump.gz

:ref:`restore-database` explains how to restore a dump.

.. include:: /global.rst