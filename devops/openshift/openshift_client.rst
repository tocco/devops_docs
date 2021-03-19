OpenShift Client / Basic Commands
=================================

Installation
------------

See :ref:`setup-openshift-client`.

Login
-----

.. code:: bash

    oc login https://console.appuio.ch
    Username: <USERNAME>
    Password: <PASSWORD>

.. hint::

   You can also specify the username using ``-u $USERNAME``.


Changing Current Project
------------------------

Just like ``cd $DIR`` can be used to change your current directory,
``oc project $PROJECT`` can be used to change the current project.


Switch to the Nice project of ${INSTALLATION}::

    oc project toco-nice-${INSTALLATION}

Show current project::

    oc project

Show all projects::

    oc projects


Resource Types
--------------

To get a list of all resources use ``oc get``.

Here are the resource types you'll need:

======= =================================================================================================================
 Type   Description
======= =================================================================================================================
 dc      **D**\eployment **c**\onfiguration: A template for a pod. If you need to change a pod, change this
         configuration. Changes are automatically deployed.

 pod     A pod is an instance of a Deployment Configuration (AKA dc).

         * Change the dc to change the configuration of a pod

 route   A DNS route

         .. code::

            oc get route
            NAME      HOST/PORT          PATH      SERVICES   PORT      TERMINATION     WILDCARD
            nice      abc.tocco.ch                 nice       80-tcp    edge/Redirect   None
            nice      www.abc.ch                   nice       80-tcp    edge/Redirect   None

 svc     Service (**svc**): Represents a set of pods that provide a service. Allows talking to Solr, for instance,
         without having to worry about in what pod it is running in or how many instances are running.

 is      **I**\mage **s**\tream: This is a docker image that has been pushed to OpenShift. In the world of docker, they
         are called repositories.
======= =================================================================================================================


.. _list-resources:

List Resources
--------------

Use ``oc get TYPE`` to get all list of the resource of a certain type or ``oc get all`` to show all of them.

Example
^^^^^^^

  .. code:: bash

    $ oc get pod
    NAME            READY     STATUS    RESTARTS   AGE
    nice-25-kchv3   2/2       Running   0          3h
    solr-2-gt5tg    1/1       Running   0          1h


Describe a Resource in Detail
-----------------------------

Use ``oc describe TYPE RESOURCE`` to show details about a specific pod/dc/is/….

.. hint:: :ref:`list-resources` shows how to obtain the RESOURCE name.

Example
^^^^^^^

     .. code::

        $ oc describe pod nice-25-kchv3
        Name:                   nice-25-kchv3
        Namespace:              toco-nice-test212
        Security Policy:        restricted
        Node:                   node19.prod.zrh.appuio.ch/172.17.176.161
        Start Time:             Wed, 18 Oct 2017 13:07:00 +0200
        Labels:                 deployment=nice-25
                                deploymentconfig=nice
                                run=nice
        …


Edit Resources
--------------

Use ``oc edit TYPE RESOURCE`` to edit a specific pod/dc/is/….

.. hint:: :ref:`list-resources` shows how to obtain the RESOURCE name.

Example
^^^^^^^

    #. Open config in editor: ``oc edit pod nice-25-kchv3``
    #. Make any changes you want to the configuration.
    #. Save changes and exit in order to trigger a deployment.

See document :doc:`../nice/configuration` for all the details.


Open Shell in Pod
-----------------

.. code::

    oc rsh -c nice PODNAME bash

``-c`` specifies the container name, use ``-c nginx`` to enter the nginx container or ``oc rsh PODNAME bash`` to enter
a Solr pod (has only one container).


Copy File from Pod
------------------

.. code::

    oc cp -c nice PODNAME:/path/to/file.txt ~/destination/folder/


Synchronize Folder with Pod
---------------------------

.. code::

    oc rsync -c nice PODNAME:/path/to/folder ~/destination/folder/


Manually Deploy
---------------

Deploy latest version of Nice:

.. code::

    oc rollout latest dc/nice


Retry Failed Deployment
-----------------------

Retry failed deployment of Nice:

.. code::

    oc rollout retry dc/nice


Open a Remote Shell in a Pod
----------------------------

To get a shell within a Nice pod use ``oc rsh -c nice POD``.

Example
^^^^^^^

.. code::

    $ oc rsh -c nice nice-25-kchv3
    nice-25-kchv3:/app $ …

Open a shell in the Nginx container using ``oc rsh -c nginx nice-25-kchv3`` or in the Solr Pod using
``oc rsh solr-2-gt5tg``.

Access Log Files in Nice Pod
----------------------------

.. code::

    oc exec -c nice PODNAME -- tail -n +0 var/log/nice.log |less


Scale Up/Down (Start/Stop instances)
------------------------------------

.. parsed-literal::

    $ oc get dc nice
    NAME      REVISION   **DESIRED**   **CURRENT**   TRIGGERED BY
    nice      49         **2**         **2**         config,image(nice:latest)

* ``DESIRED``: Number of replicas/instances configured
* ``CURRENT``: Number of replicas/instances currently online

Use this command to scale Nice instances:

.. code::

    oc scale dc/nice --replicas=${N}

This scales Nice to ``N`` replicas. Use 0 to stop all instances.

Restart a Nice Instance
-----------------------

List pods:

.. parsed-literal::

    $ oc get pod
    NAME            READY     STATUS    RESTARTS   AGE
    nice-61-6wlvx   2/2       Running   0          8d
    solr-6-c6t2t    1/1       Running   0          9d

To restart the nice instance, we have to delete the pod called ``nice-61-6wlvx`` in this example.

.. parsed-literal::

    $ oc delete pod nice-61-6wlvx
    pod "nice-61-6wlvx" deleted

If we list the pods again, we see that the deleted pod is terminating and a new one has been created (``nice-61-pkksw``):

.. parsed-literal::

    $ oc get pod
    NAME            READY     STATUS              RESTARTS   AGE
    nice-61-6wlvx   1/2       Terminating         0          8d
    nice-61-pkksw   0/2       ContainerCreating   0          4s
    solr-6-c6t2t    1/1       Running             0          9d

The new instance is not fully ready and, therefore, not reachable yet.

If we check again a few minutes later, the old instance has disappeared. Also, the new instance should be fully ready
(and we should be able to access it again via browser):

.. parsed-literal::

    $ oc get pod
    NAME            READY     STATUS    RESTARTS   AGE
    nice-61-pkksw   2/2       Running   0          7m
    solr-6-c6t2t    1/1       Running   0          9d


Start PSQL (SQL Console)
------------------------

See :ref:`connect-to-db-via-openshift`.
