============
Server Group
============

Attempt to boot instances in parallel exceeding workers count and with
anti-affinity policy will not be allowed. (Note: The Server group extensions
are being changed upstream: Server group size is no longer used).

-----------------
Test Requirements
-----------------

Tbd

.. contents::
   :local:
   :depth: 1

--------------------
Nova_ServerGroup_1.0
--------------------

:Test ID: test_attempt_boot_instances_parallel_exceeding_workers_count_anti-affinity_policy
:Test Title:
:Tags: p2,regression,nova
:Sub-domain: ServerGroup

~~~~~~~~~~~~~~
Test Objective
~~~~~~~~~~~~~~

Server Group anti-affinity test where number of instances exceed the number of
available workers.

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. Tenant creates server group with anti-affinity setting.
2. Attempt to launch X instances (where X is the number of worker nodes +1),
   (e.g. minimum count specified is 3 but there are only 2 workers available).

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

All 3 instances fail to launch (remain in error). The following error should be reported:

   .. code:: sh

     Error: Failed to perform requested operation on instance "Name", the
     instance has an error status: No valid host was found. There are not enough
     hosts available. compute-1: (ServerGroupAntiAffinityFilter) Anti-affinity
     server group specified, but this host is already used by that group.
