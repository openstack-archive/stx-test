==============
Server Actions
==============

Tests shelving, resize, instance deletion, migration stress.

-----------------
Test Requirements
-----------------

Tbd

.. contents::
   :local:
   :depth: 1

---------------------
Nova_ServerAction_1.0
---------------------

Tbd

:Test ID: test_nova_attempt_resize_on_a_shelved_instance
:Test Title:
:Tags: p2, regression, nova

Nova Test Plan - Instance server actions

~~~~~~~~~~~~~~
Test Objective
~~~~~~~~~~~~~~

Instance shelving server action test.

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~


~~~~~~~~~~
Test Steps
~~~~~~~~~~

Attempt resize on a shelved instance.

1. Perform nova server action ‘shelve’ on an instance and attempt resize of
   the instance.

   .. code:: sh

     $ nova list --all-tenants
     $ nova shelve c918ba1b-462a-47ca-bada-4cdd22476170	nova resize <instance-id> small
       ERROR (Conflict): Cannot 'resize' instance <instance-id> while it is in vm_state shelved_offloaded (HTTP 409)

2. Unshelve the instance.
3. Complete the resize.

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

Resize is allowed to complete if the instance is unshelved.

---------------------
Nova_ServerAction_2.0
---------------------

Tbd

:Test ID: test_perform_nova_delete_action_on_instance_in_error_state
:Test Title:
:Tags: p2,regression,nova

~~~~~~~~~~~~~~~~~~
Test Objective
~~~~~~~~~~~~~~~~~~

This test confirms that an instance in error can be deleted.

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

Tbd

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. Create image that requires large disk to be specified in the flavor, (e.g.
   Ubuntu or centos image).
2. Create a flavor that is insufficient in disk size for the image:

   .. code:: sh

    $ nova flavor-create tst-flavor-hatong-Dedicated auto 256 1 1
    $ nova flavor-key tst-flavor-hatong-Dedicated set
      hw:cpu_model=Haswell hw:cpu_policy=dedicated

3. Attempt to boot the instance (this will error) with the image (Ubuntu or
   centos) and with the flavor that does not meet disk size requirement:

   .. code:: sh

    $ nova boot --flavor tst-flavor-hatong-Dedicated --image ubutes_14 --nic net-id=250e54fe-9512-413b-8609-30481419c4d5 b

4. Run `nova show <instanceid>` to see the instance in error:

   .. code:: sh

    $ nova show <instanceid>

5. Attempt to delete the instance in error state

   .. code:: sh

    $ nova delete <instance name>

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

The instance launches in error (e.g. when flavor disk too small for the image
choice):

   .. code:: sh

    $ nova show <instanceid>
    | fault | {"message": "Build of instance
    88f76943-9a4d-4e28-8995-79792fed559f aborted:
    Flavor's disk is too small for requested image. Flavor disk
    is 1073741824 bytes, image is 2361393152 bytes.",
    "code": 500, "created": "2016-10-07T17:18:35Z"} |

Instance in error should delete successfully.

----------------------
Nova_StressMigrate_1.0
----------------------

Tbd

:Test ID: test_nova_stress_migration_operations_on_an_instance_launched_from_volume_(repeating_migration_tests_1000+_times)
:Test Title:
:Tags: p1,regression,nova,stress

~~~~~~~~~~~~~~~~~~
Testcase Objective
~~~~~~~~~~~~~~~~~~

This tests confirms live migration and cold migrations and system stability
over a longer period.

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

Instance is launched from cinder volume.

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. Perform live-migrations 1000+ times. Check the nova-compute.log for
   “processing ERROR” live_migration or “Exception during message handling”.

2. Attempt live block migration (should fail) followed by live-migrations
   operation on the instance. Repeat sequence 1000+ times. Check the
   nova-compute.log for “processing ERROR” live_migration or “Exception during
   message handling”.

3. Cold migrate the instance followed by live-migration operation on the
   instance. Repeat sequence 1000+ times. Check the nova-compute.log for
   “processing ERROR” live_migration or “Exception during message handling”.

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

Migrations complete as expected. The system remains stable. No processing
ERROR in the nova-compute.log and also no "Exception during message
handling".

----------------------
Nova_StressMigrate_2.0
----------------------

Tbd

:Test ID: test_nova_stress_migration_operations_on_an_instance_launched_from_image_(repeating_migration_tests_1000+_times)
:Test Title:
:Tags: p1, regression, nova, stress

~~~~~~~~~~~~~~
Test Objective
~~~~~~~~~~~~~~

This tests confirms live block migration and cold migrations and system
stability over a longer period.

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

Tbd

~~~~~~~~~~
Test Steps
~~~~~~~~~~

Launch instance from image:

- Attempt live migration (should fail) followed by live block migration
  operation on the instance. Repeat sequence 1000+ times	Check the
  nova-compute.log for “processoring ERROR” live_migration or “Exception during
  message handling”.
- Cold migrate the instance followed by live-block migration operation on the
  instance. Repeat sequence 1000+ times	Check the nova-compute.log for
  “processoring ERROR” live_migration or “Exception during message handling”.

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

Migrations complete as expected. The system remains stable. No processing
ERROR in the nova-compute.log. Also no "Exception during message handling".

