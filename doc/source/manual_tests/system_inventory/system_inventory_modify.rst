=======================
System Inventory/Modify
=======================

.. contents::
   :local:
   :depth: 1

-----------------------
sysinv_mod_01
-----------------------

:Test ID: sysinv_mod_01
:Test Title: change the mtu value of the data interface using gui
:Tags: sysinv

~~~~~~~~~~~~~~~~~~
Testcase Objective
~~~~~~~~~~~~~~~~~~

Change the mtu value of the data interface using Horizon

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

N/A

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. Login to platform horizon with 'admin' user

2. Lock a worker node using gui

2.1. Go to Platform / Host Inventory / Hosts

2.2. from the edit menu for the worker-0, select lock host.

3. Change the MTU

3.1. click the name of the host, and then go to "Interfaces" tab. Click "Edit" on data0.

3.2. Modify the mtu field (use a value of 3000). Click save

4. unlock the node

5. repeat the above steps on each worker node

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

2. the node gets locked without any error

3. the MTU value changes to the value specified

4.
   - worker node is unlocked after boot

   - network works with no issues

5. the rest of the worker nodes are unlocked and enabled after boot

~~~~~~~~~~
References
~~~~~~~~~~

N/A


-----------------------
sysinv_mod_02
-----------------------

:Test ID: sysinv_mod_02
:Test Title: change the mtu value of the data interface using cli
:Tags: sysinv

~~~~~~~~~~~~~~~~~~
Testcase Objective
~~~~~~~~~~~~~~~~~~

Change the MTU value of the data interface using CLI

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

N/A

~~~~~~~~~~
Test Steps
~~~~~~~~~~


1. Authenticate with platform keystone

2. Lock worker-0

3. Change the MTU value usingsystem host-if-modify

.. code:: bash

  system host-if-modify -m 3000 worker-0 eth1000

4. unlock the node

5. repeat the above steps on each worker node

Note:

- For Duplex use the second controller

- MTU must be greater or equal to MTU of the underline provider network

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

2. the node gets locked without any error

3. the MTU value changes to the value specified

4.
   - worker node is unlocked after boot
   - network works with no issues

5. the rest of the worker nodes are unlocked and enabled after boot

~~~~~~~~~~
References
~~~~~~~~~~

N/A


-----------------------
sysinv_mod_03
-----------------------

:Test ID: sysinv_mod_03
:Test Title: change the size of the image storage pool on a ceph-based system using gui
:Tags: sysinv

~~~~~~~~~~~~~~~~~~
Testcase Objective
~~~~~~~~~~~~~~~~~~

Change the size of the image storage pool on a ceph-based system using Horizon

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

N/A

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. Login to platform horizon with 'admin' user

2. Go to Admin / Platform / System Configuration

3. Select the Ceph Storage Pools tab

4. Click on "Edit pool quotas" on the "Storage" tier

5. Modify "Glance image", "Cinder" and "Kubernetes" quotas to add up the total of the tier space. Click save

6. Verify "Ceph storage pools" tab displays the quotas as you defined them.

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

5. Pool sizes are saved.

   Check them with:

.. code:: bash

  system storage-backend-show ceph-store

6. Tab displays information correctly

~~~~~~~~~~
References
~~~~~~~~~~

N/A


-----------------------
sysinv_mod_04
-----------------------

:Test ID: sysinv_mod_04
:Test Title: modify number of hugepages using Horizon
:Tags: sysinv

~~~~~~~~~~~~~~~~~~
Testcase Objective
~~~~~~~~~~~~~~~~~~

Change the Application hugepages on a worker node

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

N/A

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. Login to platform horizon using 'admin'

2. Go to Admin / Platform / Host Inventory, "Hosts" tab

3. Lock worker-1 using the "Edit Host" button

4. Click on worker-1 to go to "host detail

5. Select "Memory" tab and click on "Update Memory

6. Update the Application hugepages to the maximum number allowed.

7. Unlock worker-1

8. Launch VMs on worker-1 using hugepage memory

.. code:: bash

  openstack flavor set m1.small --property hw:mem_page_size=1GB
  openstack server create --image cirros --flavor m1.small --nic net-id=net3 testvm
  openstack server show testvm


~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

3. worker-1 locked

6. 1g hugepages are in ‘pending’ status

7. the worker boots and is available

8. The VMs are consuming hugepage memory from the correct numa node in worker-1

~~~~~~~~~~
References
~~~~~~~~~~

N/A


