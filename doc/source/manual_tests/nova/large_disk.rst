==========
Large Disk
==========

Large disk lock and evacuation. Where the host storage type is local_image,
there is a restriction on lock and evacuation operations where the instances
was booted from glance image with all flavor disks local and disk size greater
than 60GB. The instance that exceeds this disk size will fail to evacuate if
the host that it resides on is rebooted. A lock host operation will be
restricted as instance(s) with disk size greater than 60 GB exist on the host.

-----------------
Test Requirements
-----------------

System with 2 or more hosts with compute subfunction.

.. contents::
   :local:
   :depth: 1

----------------
Nova_Storage_1.0
----------------

:Test ID: test_nova_boot_from_glance_(image_backing),_volume_attach_and_try_lock_host_-_root_disk_exceeds_60GB
:Test Title:
:Tags: p2, regression, nova
:Sub-domain: Storage

~~~~~~~~~~~~~~
Test Objective
~~~~~~~~~~~~~~

Nova - Boot from glance (image backing) with root disk exceeds 60GB, volume
attached and confirm migration on host lock operation.

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

2 or more workers available and have local_image backing and have sufficient
room for the disk.

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. Boot instance from glance with the following flavor e.g.:

- local_image storage type
- 4vcpu, 1024 RAM, >60GB disk
- dedicated cpu policy (optional swap disk, optionally prefer thread policy)

2. After the instance launches successfully, attached a volume.
3. Attempt to lock the host where the instance lands.

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

In STX, VIM should allow the lock operaton. This will perform a live migrate
(instead of doing a system initiated cold migrate).
The host will lock and the instance migrates.

----------------
Nova_Storage_2.0
----------------

:Test ID: test_nova_Boot_from_glance_(image_backing),_volume_attach_and_try_evacuate_host_operations_-_root_disk_exceeds_60GB
:Test Title:
:Tags: p2, regression, nova
:Sub-domain: Storage

~~~~~~~~~~~~~~
Test Objective
~~~~~~~~~~~~~~

Nova - Boot from glance (image backing) with root disk exceeds 60GB, volume
attach and attempt an evacuate operation on the host.

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

2 or more workers available and have local_image backing and have sufficient
room for the disk.

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. Boot instance from glance with the following flavor, e.g.:

- local_image storage type
- 4vcpu, 1024 RAM, >60GB disk
- dedicated cpu policy (optionally prefer thread policy)

2. After the instance launches successfully, attached a volume.
3. Attempt evacuation of the host where this instance resides.

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

The instance does not evacuate on host lock.
Confirm nfv-vim.log will report the reason the instance could not evacuate.
"...can't be evacuated by the system, the disk is too large, max_disk_gb=60,
disk_size_gb=XX."

Confirm user is given disk size threshold reason for not evacuating the
instance (in fm-event.log) when the disk size of the instance exceed 60GB
Run `system host-show <host>` to confirm reason for evacuation failure is
reported.
