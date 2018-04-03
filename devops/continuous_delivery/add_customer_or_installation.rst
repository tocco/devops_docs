Add Customer / Installation
===========================

Create a new Customer
---------------------

.. figure:: add_customer_or_installation_static/new_customer.png

1. Go to the `Continuous Delivery project settings`_
2. **Create subproject** (screenshot above)

   .. _Continuous Delivery project settings: https://dev.tocco.ch/teamcity/admin/editProject.html?projectId=ContinuousDeliveryNg

.. figure:: add_customer_or_installation_static/new_customer2.png

   Project setting for a new customer

3. Fill in parameters as shown above


.. _create-installation-in-teamcity:

Create a new Installation
-------------------------

.. figure:: add_customer_or_installation_static/new_installation1.png

1. Go to the `Continuous Delivery project settings`_
2. Find the customer you want and click on **edit**. If doesn't exist, it needs to be
   `created <#create-a-new-customer>`_ first.

.. figure:: add_customer_or_installation_static/new_installation2.png

   Build configurations for customers

3. **Create build configuration** (screenshot above)

.. figure:: add_customer_or_installation_static/new_installation3.png

   Template parameters

4. Fill in parameters as shown above
5. Fill in these additional template parameters:

   ============================  =======================================================================================
   CUSTOMER                      Customer name (e.g. agogis or smc but never :strike:`agogistest` or :strike:`smctest`)
   DOCKER_PULL_URL [#f1]_        For prod. systems where a test system exists: [#f3]_

                                    ``registry.appuio.ch/toco-nice-%env.INSTALLATION%test/%env.DOCKER_IMAGE%``

                                 For test systems as well as production system with no test system:

                                    ``""`` (empty)
   DUMP_MODE                     ``dump`` for production systems and ``no_dump`` for test systems [#f2]_
   GIT_TREEISH                   Git branch (e.g. ``releases/2.13``). [#f1]_
   INSTALLATION                  Installation name (e.g. smc or smctest)
   ============================  =======================================================================================

   It shouldn't be necessary to touch any of the other parameters.

|

6. Check again what type of system your creating.

   For test systems or stand-alone production system you mustn't set the DOCKER_PULL_URL parameter!

   For production system where a test system exists you have to set the DOCKER_PULL_URL parameter!

.. important:: 

   If you add a test system for a customer who just has one production or pilot system you have to adjust the **DOCKER_PULL_URL**. A production system do always pull the image it uses from a test system!

.. note::

    The installation needs also to be :doc:`created in OpenShift <../openshift/create_nice_installation>`.


.. rubric:: Footnotes

.. [#f1] If **DOCKER_PULL_URL** is present, the Docker image for the installation is fetched from that URL. Otherwise,
         a new image is built from **GIT_TREEISH**.
.. [#f2] **DUMP_MODE** decides whether a dump is done by default. The user can still override this when deploying.
.. [#f3] This fetches the current image from the test system.
