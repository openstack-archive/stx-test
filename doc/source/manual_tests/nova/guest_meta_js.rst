=============
Guest meta.js
=============

meta.js file exists in guest if instantiation with '--meta' option.

-----------------
Test Requirements
-----------------

Tbd

.. contents::
   :local:
   :depth: 1

-------------
Nova_MetaFile
-------------

:Test ID: test_meta js_file_in_the_guest_if_instance_booted_with meta_option
:Tags: p1,regression,nova

~~~~~~~~~~~~~~
Test Objective
~~~~~~~~~~~~~~

Ensure guest has meta.js file if instantiation with '--meta' option.

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. Instantiate a vm with the option ‘--meta’ and ensure /meta.js file in the
   guest. E.g. Launch instance from centos guest with option --meta:

   .. code:: sh

      $ source ./openrc.tenant1
      $ nova boot --flavor <flavor> --image <eg.centos> --nic net-id=<netid> --meta
        role=webservers <instance_name>

  For example:

      $ nova boot --flavor medium.dpdk --image tis-centos-guest --nic
        net-id=ae1d9039-bdf4-4118-9282-b1ea0b03fb74 --meta role=webservers test

2. Log into the guest and check if there is a file /meta.js in the guest:

   .. code:: sh

      <instance_name>:~# cd /
      <instance_name>:~# ls | grep meta.js
       meta/js

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

Ensure guest has meta.js file if instantiation with '--meta' option,
(note: some customer applications are unable to start without the required
meta.js file).

