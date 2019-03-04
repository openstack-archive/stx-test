=============
Flavor Access
=============

As an admin user I can remove tenant user(s) access to flavors so that the
flavor can not be used for creating or resizing instances.
A flavor in use is not allowed to be removed or modified.

-----------------
Test Requirements
-----------------

System with 2 or more hosts with compute subfunction.

.. contents::
   :local:
   :depth: 1

---------------
Nova_Flavor_1.0
---------------

:Test ID: test_nova_FlavorAccess_1.1a_Removing_access_to_the_flavor
:Test Title:
:Tags: p1,regression,nova

~~~~~~~~~~~~~~
Test Objective
~~~~~~~~~~~~~~

Admin user can remove tenant user(s) access to flavors so that the flavor can
not be used for creating or resizing instances.

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

Tbd

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. As admin user, create a flavor.
2. Remove Nova flavor access:

   .. code:: bash

     $ nova flavor-show <flavorname>

3. Remove user access from the flavor

   .. code:: bash

     $ nova flavor-access-remove small.float 934cb48930ab4b6c819d0b1b191e795a

The corresponding tenant users removed do not have access to the flavor for
creating and resizing instances.

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

- Confirm the tenant user removed can not access the flavor for creating
  instance.
- Confirm the tenant user removed can not access the flavor for resizing
  instance (Access permission for others in the access list should be
  retained).

---------------
Nova_Flavor_2.0
---------------

:Test ID: test_nova_FlavorInUse_2_Flavor_in_use_can_not_be_modified_(or_deleted)
:Test Title:
:Tags: regression,nova

~~~~~~~~~~~~~~~~~~
Testcase Objective
~~~~~~~~~~~~~~~~~~

Admin user should not be allowed to modify or remove a flavor that is in use.

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

Tbd

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. As admin user, create a flavor
2. Launch an instance using the new flavor.
3. Attempt to modify a setting in the flavor (that is in use by the instance.)

- For example attempt to change the disk size, VCPU setting
- For example attempt to change or remove any extra spec setting

5. Create a flavor
6. Launch an instance using the new flavor.
7. Attempt to delete the flavor (that is in use by the instance.)

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

- The change to the flavor settings should be rejected if in use by the
  instance.
- The deletion of the flavor should be rejected if in use by the instance.
