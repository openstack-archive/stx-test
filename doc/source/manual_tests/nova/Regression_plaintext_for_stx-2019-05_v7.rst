Regression STX-2019-05

Feature Overview
===================
Nova - Large Disk
Large disk lock and evacuation
Where the host storage type is local_image, there is a restriction on lock and evacuation operations where the instances was booted from glance image with all flavor disks local and disk size greater than 60GB. The instance that exceeds this disk size will 

fail to evacuate if the host that it resides on is rebooted. A lock host operation will be restricted as instance(s) with disk size greater than 60 GB exist on the host.

Test Requirements:
===================
System with 2 or more hosts with compute subfunction

Test Cases:
===============

Nova_Storage_1.0
test_nova_boot_from_glance_(image_backing),_volume_attach_and_try_lock_host_-_root_disk_exceeds_60GB
Tags: p2,regression, nova
Sub-domain = Storage


Testcase Objective:
--------------------------------
Nova - Boot from glance (image backing) with root disk exceeds 60GB, volume attached and confirm migration on host lock operation

Test Pre-Conditions:
--------------------------
2 or more workers available and have local_image backing and have sufficient room for the disk

Test Steps:
----------------
1. Boot instance from glance with the following flavor
eg. local_image storage type
4vcpu, 1024 RAM, >60GB disk
dedicated cpu policy
(optional swap disk, optionally prefer thread policy)

2. After the instance launches successfully, attached a volume.
3. Attempt to lock the host where the instance lands.

Expected Behavior:
-----------------------------
In STX, VIM should allow the lock operaton. This will perform a live migrate (instead of doing a system initiated cold migrate).
The host will lock and the instance migrates.


Nova_Storage_2.0
test_nova_Boot_from_glance_(image_backing),_volume_attach_and_try_evacuate_host_operations_-_root_disk_exceeds_60GB

Testcase Objective:
--------------------------------
Nova - Boot from glance (image backing) with root disk exceeds 60GB, volume attach and attempt an evacuate operation on the host

Test Pre-Conditions:
--------------------------
2 or more workers available and have local_image backing and have sufficient room for the disk

Test Steps:
----------------
1. Boot instance from glance with the following flavor
eg. local_image storage type
4vcpu, 1024 RAM, >60GB disk
dedicated cpu policy
(optionally prefer thread policy)

2. After the instance launches successfully, attached a volume.
3. Attempt evacuation of the host where this instance resides.


Expected Behavior:
-----------------------------
The instance does not evacuate on host lock.
Confirm nfv-vim.log will report the reason the instance could not evacuate.
"...can't be evacuated by the system, the disk is too large, max_disk_gb=60, disk_size_gb=XX."

Confirm user is given disk size threshold reason for not evacuating the instance (in fm-event.log) when the disk size of the instance exceed 60GB
Run 'system host-show <host>' to confirm reason for evacuation failure is reported.

Feature Overview:
=================
Nova - Flavor Access
As an admin user I can remove tenant user(s) access to flavors so that the flavor can not be used for creating or resizing instances.
A flavor in use is not allowed to be removed or modified.

Test Requirements:
===================

Test Cases:
===============

Nova_Flavor_1.0
test_nova_FlavorAccess_1.1a_Removing_access_to_the_flavor
Tags: p1,regression,nova

Testcase Objective:
--------------------------------
Admin user can remove tenant user(s) access to flavors so that the flavor can not be used for creating or resizing instances.

Test Pre-Conditions:
--------------------------

Test Steps:
----------------
1. As admin user, create a flavor.

2. Remove Nova flavor access:
$nova flavor-show <flavorname>

3. Remove user access from the flavor
$nova flavor-access-remove small.float 934cb48930ab4b6c819d0b1b191e795a
The corresponding tenant users removed do not have access to the flavor for creating and resizing instances.

Expected Behavior:
-----------------------------
Confirm the tenant user removed can not access the flavor for creating instance
Confirm the tenant user removed can not access the flavor for resizing instance
(Access permission for others in the access list should be retained).


Nova_Flavor_2.0
test_nova_FlavorInUse_2_Flavor_in_use_can_not_be_modified_(or_deleted)
Tags: regression,nova

Testcase Objective:
--------------------------------
Admin user should not be allowed to modify or remove a flavor that is in use.

Test Pre-Conditions:
--------------------------

Test Steps:
----------------
1. As admin user, create a flavor
2. Launch an instance using the new flavor.
3. Attempt to modify a setting in the flavor (that is in use by the instance.)

For example attempt to change the disk size, VCPU setting
For example attempt to change or remove any extra spec setting

5. Create a flavor
6. Launch an instance using the new flavor.
7. Attempt to delete the flavor (that is in use by the instance.)

Expected Behavior:
-----------------------------
The change to the flavor settings should be rejected if in use by the instance.
The deletion of the flavor should be rejected if in use by the instance.


Feature Overview:
=================
Nova - CPU Thread Policy
Resize interactions cpu thread policy and vcpu number under hyperthreaded conditions

Test Requirements:
===================
Hyperthread capable host

Test Cases:
===============
Nova_CPUThreadPolicy_1.0
test_nova_resize_to_CPU_Thread_Policy_“require”_As_a_user,_attempt_resize_operation_changing_the_VCPU_size_as_well
Tags: p2,regression,nova


Testcase Objective:
--------------------------------

Test Pre-Conditions:
--------------------------

Test Steps:
----------------
Resize an instance from isolate to CPU Thread Policy “require | prefer”
1. As a user, attempt instance resize operation changing the VCPU size as well
eg.
Istance with flavor with 2 VCPU and the extra spec resized.
hw:cpu_thread_policy=isolate

2. To a flavor with
extra spec hw:cpu_thread_policy=require
where the new flavor has 3 VCPU or 1 VCPU

3. To a flavor with
extra spec hw:cpu_thread_policy=prefer
where the new flavor has 3 VCPU or 1 VCPU


Expected Behavior:
-----------------------------
The resize should not fail (since Mitaka) regardless of whether the flavor vcpu is not evenly divisible
ie. where vcpu is not a multiple of 2 and the hw:cpu_threads_policy=require was specified in the flavor



Nova_CPUThreadPolicy_2.0
test_nova_resize_changing_VCPU_settings_and_setting_CPU_Thread_Policy_to_isolate,_prefer
Tags: p2,regression,nova
Sub-domain = Hyperthreading

Testcase Objective:
--------------------------------
Resize an instance changing vCPU settings and setting CPU Thread Policy to isolate {to prefer}

Test Pre-Conditions:
--------------------------

Test Steps:
----------------
1. Resize instance from flavor with 2 VCPU and the extra spec
eg. hw:cpu_thread_policy= require  to a flavor with extra spec:

hw:cpu_thread_policy=isolate where the new flavor has 3
VCPU (ie. or some number that is not a multiple of 2)


2. Repeat resize test but to a new target CPU Thread Policy prefer
ie. resize instance from 'require' flavor with 2 VCPU to a flavor with extra spec:

extra spec eg. hw:cpu_thread_policy=prefer where the new flavor has 3
VCPU (ie. or some number that is not a multiple of 2)


Expected Behavior:
-----------------------------
The resize should be successful  (if the host has specifient room)



Nova_CPUThreadPolicy_3.0
test_nova_imagemetadata_property_hw_cpu_threads_policy=isolate_override_test,_conflict_with_require_in_flavor
Tags: p2,regression,nova
Sub-domain = Hyperthreading

Testcase Objective:
--------------------------------
Attempts to launch instance with Image metadata hw_cpu_thread_policy=isolate and dedicated cpu policy
but flavor has conflicting thread policy setting hw:cpu_thread_policy=require

Test Pre-Conditions:
--------------------------

Test Steps:
----------------
1. Attempts to launch instance with Image metadata has hw_cpu_thread_policy=isolate and dedicated cpu policy
Flavor has hw:cpu_thread_policy=require

Expected Behavior:
-----------------------------
Conflict results
Error: Image property 'hw_cpu_thread_policy' is not permitted to override CPU thread
pinning policy set against the flavor (HTTP 403)



Nova_CPUThreadPolicy_4.0
test_nova_imagemetadata_property_hw_cpu_threads_policy=require_override_test,_prefer_thread_policy_in_flavor
Tags: p2,regression,nova
Sub-domain = Hyperthreading


Testcase Objective:
--------------------------------
image metadata hw_cpu_thread_policy=require; flavor has hw:cpu_thread_policy=prefer
The instance will have cpu thread policy require

Test Pre-Conditions:
--------------------------

Test Steps:
----------------
1. Set image metadata to the require cpu thread policy, hw_cpu_policy=dedicated
2. Set flavor extra spec to hw:cpu_thread_policy=prefer
3. Launch the instance with that image and flavor.

Expected Behavior:
-----------------------------
Instance expected to have CPU Thread Policy require


Feature Overview:
=================
Nova - Server Actions
(Tests shelving, resize, instance deletion, migration stress)

Test Requirements:
====================

Test Cases:
===============

Nova_ServerAction_1.0
test_nova_attempt_resize_on_a_shelved_instance
Tags: p2,regression,nova

Description
Nova Test Plan - Instance server actions

Testcase Objective:
--------------------------------
Instance shelving server action test

Test Pre-Conditions:
--------------------------

Test Steps:
----------------
Attempt resize on a shelved instance.

1. Perform nova server action ‘shelve’ on an instance and attempt resize of the instance
$ nova list --all-tenants
$nova shelve c918ba1b-462a-47ca-bada-4cdd22476170	nova resize <instance-id> small
ERROR (Conflict): Cannot 'resize' instance <instance-id> while it is in vm_state shelved_offloaded (HTTP 409)

2. Unshelve the instance.
3. Complete the resize.

Expected Behavior:
-----------------------------
Resize is allowed to complete if the instance is unshelved.

Nova_ServerAction_2.0
test_perform_nova_delete_action_on_instance_in_error_state
Tags: p2,regression,nova

Testcase Objective:
--------------------------------
This test confirms that an instance in error can be deleted.

Test Pre-Conditions:
--------------------------

Test Steps:
----------------
1. Create image that requires large disk to be specified in the flavor
eg. Ubuntu or centos image

2. Create a flavor that is insufficient in disk size for the image
$nova flavor-create tst-flavor-hatong-Dedicated auto 256 1 1
$nova flavor-key tst-flavor-hatong-Dedicated set
hw:cpu_model=Haswell hw:cpu_policy=dedicated

3. Attempt to boot the instance (this will error) with the image (Ubuntu or centos) and with the flavor that does not meet disk size requirement
$ nova boot --flavor tst-flavor-hatong-Dedicated --image ubutes_14 --nic net-id=250e54fe-9512-413b-8609-30481419c4d5 b

4. Run $nova show <instanceid> to see the instance in error

5. Attempt to delete the instance in error state
$ nova delete <instance name>

Expected Behavior:
-----------------------------
The instance launches in error eg. when flavor disk too small for the image choice

$nova show <instanceid>
| fault | {"message": "Build of instance
88f76943-9a4d-4e28-8995-79792fed559f aborted:
Flavor's disk is too small for requested image. Flavor disk
is 1073741824 bytes, image is 2361393152 bytes.",
"code": 500, "created": "2016-10-07T17:18:35Z"} |


Instance in error should delete successfully


Nova_StressMigrate_1.0
test_nova_stress_migration_operations_on_an_instance_launched_from_volume_(repeating_migration_tests_1000+_times)
Tags: p1,regression,nova,stress

Testcase Objective:
--------------------------------
This tests confirms live migration and cold migrations and system stability over a longer period

Test Pre-Conditions:
--------------------------
Instance is launched from cinder volume.

Test Steps:
----------------
1. Perform live-migrations 1000+ times.
Check the nova-compute.log for “processing ERROR” live_migration or “Exception during message handling”

2. Attempt live block migration (should fail) followed by live-migrations operation on the instance.
Repeat sequence 1000+ times.
Check the nova-compute.log for “processing ERROR” live_migration or “Exception during message handling”

3. Cold migrate the instance followed by live-migration operation on the instance.
Repeat sequence 1000+ times.
Check the nova-compute.log for “processing ERROR” live_migration or “Exception during message handling”

Expected Behavior:
-----------------------------
Migrations complete as expected. The system remains stable.
No processing ERROR in the nova-compute.log and also no "Exception during message handling"


Nova_StressMigrate_2.0
test_nova_stress_migration_operations_on_an_instance_launched_from_image_(repeating_migration_tests_1000+_times)
Tags: p1,regression,nova,stress

Testcase Objective:
--------------------------------
This tests confirms live block migration and cold migrations and system stability over a longer period

Test Pre-Conditions:
--------------------------

Test Steps:
----------------
Launch instance from image
a. Attempt live migration (should fail) followed by live block migration operation on the instance.
Repeat sequence 1000+ times	Check the nova-compute.log for “processoring ERROR”
live_migration or “Exception during message handling”


b.
Cold migrate the instance followed by live-block migration operation on the instance.
Repeat sequence 1000+ times	Check the nova-compute.log for “processoring ERROR”
live_migration or “Exception during message handling”


Expected Behavior:
-----------------------------
Migrations complete as expected. The system remains stable.
No processing ERROR in the nova-compute.log
Also no "Exception during message handling"



Feature Overview:
=================
Nova - Server Group
Attempt to boot instances in parallel exceeding workers count and with anti-affinity policy will not be allowed.
(Note: The Server group extensions are being changed upstream: Server group size is no longer used)

Test Requirements:
===================

Test Cases:
===============

Nova_ServerGroup_1.0
test_attempt_boot_instances_parallel_exceeding_workers_count_-_anti-affinity_policy
Tags: p2,regression,nova
Sub-domain = ServerGroup

Testcase Objective:
--------------------------------
Server Group anti-affinity test where number of instances exceed the number of available workers

Test Pre-Conditions:
--------------------------

Test Steps:
----------------
1. Tenant creates server group with anti-affinity setting

2. Attempt to launch X instances (where X is the number of worker nodes +1).
eg. minimum count specified is 3 (but there is only 2 workers available).

Expected Behavior:
-----------------------------
All 3 instances fail to launch (remain in error)
The following error should be reported.
Error: Failed to perform requested operation on instance "Name", the instance has an error status: No valid host was found. There are not enough hosts available. compute-1: (ServerGroupAntiAffinityFilter) Anti-affinity server group specified, but this host is 

already used by that group


Feature Overview:
=================
Nova - Migrations and PortStatus
The neutron port status should always be active after a successful migration (cold, live, block)

Test Requirements:
===================

Test Cases:
===============

Nova_Migrations_PortStatus_1.0
test_neutron_port_status_after_successful_migration_ACTIVE_(cold,_live,_block)
Tags: p2,regression,nova

Testcase Objective:
--------------------------------
Confirm neutron port status after successful migration, The "Port State" should be "ACTIVE" after successful migrations (cold, live, block)

Test Pre-Conditions:
--------------------------

Test Steps:
----------------
1. Test with cold migration, on a system with at least 2 workers
Launch instance
Once active, look at the status of the related neutron ports – they should be reported as ACTIVE

$ neutron port-list --device_id=e80a0560-42f7-4d81-bbed-68b52664566b -c id -c status -c binding:host_id
+--------------------------------------+--------+
| id                                   | status |
+--------------------------------------+--------+
| 3cf1a5eb-3eed-4596-b021-7e7051f98c5c | ACTIVE |
| ca9a0b07-219d-4b8e-9f0d-c3e64cf61024 | ACTIVE |
| d5c463ef-21fc-4a29-a5b6-c6daf4e7743c | ACTIVE |
+--------------------------------------+--------+


Cold migrate the instance to another worker
Check the related neutron ports to ensure they are reporting ACTIVE

Repeat

2. Test with live migration, on a system with at least 2 workers
Launch instance
Once active, look at the status of the related neutron ports – they should be reported as ACTIVE

$ neutron port-list --device_id=e80a0560-42f7-4d81-bbed-68b52664566b -c id -c status -c binding:host_id

| id                                   | status |
| 3cf1a5eb-3eed-4596-b021-7e7051f98c5c | ACTIVE |
| ca9a0b07-219d-4b8e-9f0d-c3e64cf61024 | ACTIVE |
| d5c463ef-21fc-4a29-a5b6-c6daf4e7743c | ACTIVE |

Live migrate an instance to another worker
Check the related neutron ports to ensure they are reporting ACTIVE

Repeat

3.
Test with live block migration, on a system with at least 2 workers
Launch instance
Once active, look at the status of the related neutron ports – they should be reported as ACTIVE

$ neutron port-list --device_id=e80a0560-42f7-4d81-bbed-68b52664566b -c id -c status -c binding:host_id

| id                                   | status |
| 3cf1a5eb-3eed-4596-b021-7e7051f98c5c | ACTIVE |
| ca9a0b07-219d-4b8e-9f0d-c3e64cf61024 | ACTIVE |
| d5c463ef-21fc-4a29-a5b6-c6daf4e7743c | ACTIVE |

Live block migrate an instance to another worker
Check the related neutron ports to ensure they are reporting ACTIVE

Repeat

Expected Behavior:
-----------------------------
1. Expect the neutron ports to be ACTIVE after cold migration.
2. Expect the neutron ports to be ACTIVE after live migration.
3. Expect them to be ACTIVE after live block migration.


Feature Overview:
=================
Nova - Snapshot
An instance can be booted from a volume snapshot under various conditions:
-with/without snapshot description
-with force lock flag set to false (should not allow creating snapshot from volume in-use)
-with force lock flag set to true (should allow creating snapshot from volume in-use)

Test Requirements:
===================

Test Cases:
===============

Nova_Snapshot_1.0
test_instance_boot_from_volume_snapshot_(using_cli),_storage_Local_CoW_Image,_without_snapshot_description
Tags: p1,regression,nova

Testcase Objective:
--------------------------------
Volume Snapshots created without description.
Local_CoW Image Backed storage and instantiation

Test Pre-Conditions:
--------------------------
Precondition: worker(s) with local_image backing

Test Steps:
----------------
1. Create a Volume
ie. create volume from an image (where image optionally has additional metadata such as hw_cpu_policy dedicated or shared)
and specifying Nova as the availability zone.

2. Run cinder list command to get the volumeid
$cinder list

Test Scenario a. Create Snapshot Without Display Description
3. Snapshot create without --display-description parameter
Create snapshot from volume without any display-description specified
$ cinder snapshot-create --display-name WendySnap1  <volumeid>

4. List and show the newly created snapshot
$ cinder snapshot-list

Run cinder snapshot-show <snapshotid> to view the properties and values of the snapshot
$cinder snapshot-show <snapshotid>
eg. $ cinder snapshot-show <id> displays property as follows:
created_at
display_description
display_name
id
metadata
os-extended-snapshot-attributes:progress
os-extended-snapshot-attributes:project_id
size
status
volume_id
wrs-snapshot:backup_status

5. Look for failures in cinder log on snapshot creation.
$grep <snapshotid> /var/log/cinder/<*>.log

6. Launch as instance from the newly created volume snapshot where the storage type specified in the flavor is local_image

Expected Behavior:
-----------------------------
No failures in the cinder log on snapshot creation
Instantiation from the snapshot is successful.


Nova_Snapshot_2.0
test_instance_boot_from_volume_snapshot_(using_cli),_storage_Local_CoW_Image,_with_snapshot_description

Testcase Objective:
--------------------------------
Volume Snapshots created with description.
Instantiation

Test Pre-Conditions:
--------------------------
workers(s) with local_image backing

Test Steps:
----------------
1. Create Volume
Create volume from an image (where image optionally has additional metadata such as hw_cpu_policy dedicated or shared) and specifying Nova as the availability zone.

2. Run cinder command to get the volumeid
$cinder list

Test scenario b. Snapshot create with Display Description
3. Create snapshot from volume using option  –display-description
$ cinder snapshot-create --display-name WendySnap1 --display-description somedescription <volumeid>

List the newly created snapshot
$cinder snapshot-list
Run cinder snapshot-show <snapshotid> to view the properties and values of the snapshot
$cinder snapshot-show <snapshotid>
eg. $ cinder snapshot-show <id> displays property as follows:
created_at
display_description
display_name
id
metadata
os-extended-snapshot-attributes:progress
os-extended-snapshot-attributes:project_id
size
status
volume_id
wrs-snapshot:backup_status

4. Look for failures in cinder log on snapshot creation.
$grep <snapshotid> /var/log/cinder/<*>.log

5. Launch as instance from the newly created volume snapshot where the storage type specified in the flavor is local_image

Expected Behavior:
-----------------------------
No failures in the cinder log on snapshot creation
Instantiation from the snapshot is successful.


Nova_Snapshot_3.0
test_instance_boot_from_volume_snapshot_(using_cli),_storage_Local_CoW_Image,_with_--force_false

Testcase Objective:
--------------------------------
Attempts to create a snapshot from volume that is in-use should be detected and result in an error where the --force flag is false

Test Pre-Conditions:
--------------------------

Test Steps:
----------------
1. Create new image as admin user.

2. As tenant user create volume from the image.

3. Launch instance from the volume to attach.
To get the volume-id run
$cinder list

4. Test Scenario c. Snapshot created with Force flag false
Attempt to create a snapshot from volume using the optional –force flag set to False (the default) where the instance has been attached to the volume already.
for eg. $ cinder snapshot-create --display-name WendySnapForceFalse --force False --display-description withforceflagFalse <volumeid>

Try also allcaps FALSE for the force flag
for eg. $ cinder snapshot-create --display-name WendySnapForceFalse --force FALSE --display-description withforceflagFalse <volumeid>

Expected Behavior:
-----------------------------
Ensure the volume snapshot can not be created and an appropriate error is returned
Where the force flag is set to false, attempts to create a snapshot from volume that is in-use should be detected and result in an error
ERROR: Invalid volume: Volume <volumeid> status must be available, but current status is: in-use. (HTTP 400)



Nova_Snapshot_4.0
test_instance_boot_from_volume_snapshot_(using_cli),_storage_Local_CoW_Image,_with_--force_true

Testcase Objective:
--------------------------------
Creating a snapshot from volume that is in-use should be detected and still be allowed if the --force flag is true

Test Pre-Conditions:
--------------------------

Test Steps:
----------------
1. Create new image as admin user.

2. As tenant user create volume from the image.

3. Launch instance from the volume to attach.
Run $cinder list to get the volumeid

4. Test Scenario d. Snapshot created with Force flag true
Create a snapshot from volume using the optional –force flag set to True (or eg. TRUE)
(The force flag True is used to snapshot a volume even if it is attached to an instance

$ cinder snapshot-create --display-name WendySnapForceTrue --force TRUE --display-description withforceflagTRUE <volumeid>

5. Ensure the volume snapshot can be listed and shown from the cli
$ cinder snapshot-list
Run cinder snapshot-show <snapshotid> to view the properties and values of the snapshot
$cinder snapshot-show <snapshotid>

eg. $ cinder snapshot-show <id> displays property as follows:
created_at
display_description
display_name
id
metadata
os-extended-snapshot-attributes:progress
os-extended-snapshot-attributes:project_id
size
status
volume_id
wrs-snapshot:backup_status

6. Look for failures in cinder log on snapshot creation.
$grep <snapshotid> /var/log/cinder/<*>.log

7. Launch as instance from the newly created volume snapshot where the storage type specified in the flavor is local_image.



Expected Behavior:
-----------------------------
Cinder snapshot lists and shows appropriate values and no errors in cinder log on creation
The volume snapshot is created successfully.

| Property | Value
| display_description | withforceflagTRUE
| display_name | WendySnapForceTrue
| id | <id>
| metadata | {}
| size | 1
| status | creating
| volume_id | <volumeid>

An instance successfully launches from the new volume snapshot


Feature Overview:
=================
Nova - Prioritized VM Recovery
The evacuation order from a failed host can be prioritized using the instance metadata setting. This will ensure that instances with higher value priority setting can be recovered first.

Instance metadata will be able to specify the recovery priority in the valid range 1-10 (where 1 is the highest priority).
The metadata setting will be available in Horizon from the Available Metadata list for the instance as well as from the cli
$nova meta <instance> set sw:wrs:recovery_priority=<value>
sw:wrs:recovery_priority=1 {2,3,4,5,6,7,8,9,10}

Where all instance priorities differ, instances recover (by recovery audit) are in the specified order in a multi-host failure scenario where there is no suitable host remaining.
Disk size and recovery priority combinations are used in recovery decisions in a host failure (reboot) scenario where priorities, vcpu and memory are the same but disk sizes differ.

Refer to the start event in the nfv-vim-event.log. Ensure it is in the relative order expected (as set by the recovery priority instance metadata setting.

Test Requirements:
===================

Test Cases:
===============
Nova_RecoveryPriority_1.0
test_instance_metadata_sw:wrs:recovery_priority_respect_on_host_reboot_-_priority_same,_disk_GB_differs
Tags: p1,regression,nova


Testcase Objective:
--------------------------------
Disk size and recovery priority combination is used in recovery decision in a host failure (reboot) scenario where priorities are the same, vcpu and memory are the same but disk sizes differ.
Instance metadata sw:wrs:recovery_priority respected on host reboot where all priorities are the same but disk GB size differs

Test Pre-Conditions:
--------------------------

Test Steps:
----------------
1.
3 instances created with recovery priority all set to eg. 2
VCPUs for each 4 vcpu
Memory setting for each 1024

Flavor disk size differs as follows
(1) - 10 GB (no swap)
(2) - 15 GB (5 swap)
(3*) - 20 GB (no swap)

2. Test on host reboot that the priority is based on disk size if all instances have the same priority setting for instance metadata
sw:wrs:recovery_priority.

Expected Behavior:
-----------------------------
The instance (3*) has the largest disk size so will recover first
ie. relative recovery will be largest to smallest when all priorities are equal
Refer to the start event in the nfv-vim-event.log. Ensure it is in the relative order expected (as set by the recovery priority instance metadata setting.


Nova_RecoveryPriority_2.0
test_instancemetadata_sw:wrs:recovery_priority_respected_in_multi_host_failure,_all_priorities_differ
Tags: p2,regression,nova

Testcase Objective:
--------------------------------
Where all instance priorities differ, instances recovery (by recovery audit) in the specified order in a multi-host failure scenario where there is no suitable host remaining.
Instance metadata sw:wrs:recovery_priority respected in multi host failure, all priorities differ

Test Pre-Conditions:
--------------------------


Test Steps:
----------------
1. More than 4 instances are created on 2 or more hosts.
Most of the instances on each host have sw:wrs:recovery_priority set in the instance metadata. Some have the same priority set but their flavor size differs.
(Some are not set and are expected to have the lowest priority 10)

2. Perferm a simultaneous operation on multi-hosts to cause them to fail, ensuring each instance, on a per host basis, will be sorted by the recovery priority setting (in the instance metadata) when rebuilt (or evacuated to a new host).


Expected Behavior:
-----------------------------
Note: A maximum of 4 instances will recover in parallel (by rebuild or evacuate)
On a multi-host failure, each instance, on a per host basis, will be sorted by the recovery priority setting (in the instance metadata) when rebuilding/evacuating the instance.

The instance recovery order is respected.
(If there were some priorities the same, then the instance order are sorted by size (largest to smallest)

Ensure each instance (per source host) is in the relative order expected (as set by the recovery priority instance metadata setting. (Refer to the start event in the nfv-vim-event.log.)
Check the nova.compute.log to confirm that the instances migrate in the set order on multi-host failures based on the sw:wrs:recovery_priority setting
cat /var/log/nova/nova-compute.log | grep nova.compute.resource_tracker | grep migrat | grep change=
or grep "Rebuilding instance"


Feature Overview:
=================
Nova - DataPort and Hypervisor failures
Allows VIM to continue migrations from a host on data port failure.
Allow VIM to continue migrations from a host if a migration is rolled back.

The host is expected to degrade and remain online when the data port fails.
On data port failure, live migrations will be triggered. Instances where live migration fails (rollsback), will go to error state as long as the host remains online. Other instances will still continue to migrate. The host is expected to degrade and remain 

online when the data port fails. (If the host is eg. rebooted, the host will go offline and instances will be expected to evacuate)

VIM disables the hypervisor on the host and any instances that could not be migrated need to be set to the error state so they can be recovered either by evacuation or by rebooting/rebuilding them if the host is re-enabled.
There is a new operation state and VIM changes to determine the overall success of the operation.
Nova reports live migration rollback to VIM, and VIM will continue to attempt migrations for other instances (and not stop migrations from the host altogether).

Multiple instances can be migrated from the host for several reasons. The behavior described below is what is expected in each scenario:

1)Host Lock and Orchestration
a) Host lock
Migration failure/rollback will still cause the host lock to fail immediately with the appropriate error provided to the user. The instance does not fail.
b) Orchestration
The migration failure/rollback will still cause the orchestration to fail immediately with the appropriate error provided to the user.
The instance does not fail.

2)Host Force Lock
Reboots the host and evacuates the instances, some instances may go to error while the host is being rebooting if they could not be evacuated.

3)Host (hypervisor) Disabled or Failed - VIM disables hypervisor and instances that could not be migrated are set to error so they can be recovered by evac or rebuild

4)Data port failure
Data port disconnects will attempt to live migrate instances from that host.
If the live migration fails for any of the instances, subsequent instances should still attempt recovery

VIM disables the hypervisor and degrades the host in this case and any instances that could not be migrated need to be set to the error state so they can be recovered either by evacuation or by reboot/rebuild when the host is re-enabled.  Migration of other 

instances continue despite the migration failures or rollbacks.

For example, in the case of:
a) Scheduling filter or policy restrictions - One or more instance cannot be evacuated due to some scheduling restriction eg thread policy, shared cpu assignment, huge page requirement etc.  Subsequent instances should attempt to evacuate from the host
b) Dirty memory - Multiple instances are live migrating, but 1 or more is under stress (dirtying memory too fast).
Other instances should still attempt to evacuate from the host.

Note: Instances that could not be live migrated will NOT be evacuated unless the host goes offline (eg. rebooted or powered down).


Test Requirements:
===================
standard
storage
Duplex


Test Cases:
===============

Nova_Failure_DataPort_1.0
test_on_dataport_failure_some_migrations_rolled_back_-_instance_will_error,_other_instances_continue_migrating_(std._no_infra)
Tags: p1,regression,nova,no_infra
Sub-domain = Networking


Testcase Objective:
--------------------------------
On data port failure, VIM disables the hypervisor on the host
Data port failure could be:
a. Cable pull
b. interface disabled on the switch

This test verified that instances continue to migrate on a standard system with *no infra*
It finds the data port and disables it on the switch ie. from LLDP Neighbour info (or pulls the data cabling)
Alarms on the data port will be raised and hypervisor is expected to become disabled

Any instance that rollsback to the source host, will go to error state as the hypervisor is disabled
(It must recover by evacuation if a valid target host becomes available or reboots when the source host has become enabled again)


Test Pre-Conditions:
--------------------------
On a standard controller/compute system (without infra)


Test Steps:
----------------
1. Boot multiple instances on the same worker node where these instance settings may vary.
Some instances will expect to be able to live migrate.
+Some instances will not be expected to be able to migrate since, for example, they can not be scheduled.
affinity server group policy setting
thread policy require setting
1G Hugepages setting

Instance with server group policies that can not be migrated go to Error state with the appropriate error reason provided)

2. Perform an action which will result in data port failure and subsequent hypervisor being disabled on the worker host.
ie. Disable the port on the switch:
Check the LLDP Neighbour (see LLDP - Name, Neighbour and System Name that corresponds to the hosts provider network)
sudo vshell -H compute-0 lldp-neighbour-list
| port-id | remote-port | remote-chassis | mac-address | management-address | rx-ttl | system-name | system-description | system-capabilities |

Expected Behavior:
-----------------------------
In step 2, confirm the data port alarm on the worker node.
eg. data port failure alarm
set 	300.001	'Data' Port failed.

Confirms instances that can be scheduled, migrate from the worker.

Confirm hypervisor status becomes 'disabled'
$nova hypervisor-show <id>
state changes from up, status enabled ---> to down, disabled

Host availability state changes to Degraded
Run the following system command to confirm the host availability state is 'degraded' due to the data interface alarm.
$system host-list

Confirm instances that can not schedule, roll back and error.
Confirm details of the error reason are reported in instance details.
Note: The instance(s) that fail in this case will not be evacuated unless the host goes offline (eg. reboots or powered down)

Instances with server group policy that can not be migrated go to Error state with the appropriate error reason provided
For example.  (Code 501) Message  No valid host was found. There are not enough hosts available. compute-1: (ServerGroupAffinityFilter) not found in: compute-0, hint={u'node': u'compute-0', u'host': u'compute-0', u'task_state': u'migrating', u'group': 

u'13fd8fbe-a6a2-4efd-a88b-e3faa477c8f


Nova_Failure_DataPort_2.0
test_dataport_failure,_if_some_migrations_fail/error_(stress),_other_instances_will_continue_to_migrate_(Std.System)
Tags: p1,regression,nova,stress
Sub-domain = Networking


Testcase Objective:
--------------------------------
This test finds causes a port failure (eg. pull data port or disable on the switch)
It confirms alarms on the data port are raised and hypervisor becomes disabled
Any instance that could not be migrated will go to error state as the hypervisor on the source is disabled
(If a valid target host becomes available the instance recovers or reboots/rebuilds if the source host becomes enabled again)
When the host data port fails and an instance(s) fail during the migrate attempt (and change to error state), others that can migrate will continue their migration.


Test Pre-Conditions:
--------------------------
Standard system
Instances exist on the host where the data port will be failed
Only some instances dirtying memory (such that they can't be migrated)

Test Steps:
----------------
1. Trigger a data port failure so that VIM disables the hypervisor on the host
Data port failure could be a. Cable pull  b. interface disabled on the switch
(Note: If vshell is used to lock the data port (as follows) that is not enough to cause the hypervisor to become disabled.)

2. Check for alarms and hypervisor disable state.
Alarms on the data port will be raised and hypervisor is expected to become disabled
Confirm hypervisor status becomes 'disabled' and host degraded.

3. Any instance that could not be migrated will go to error state as the hypervisor on the source is disabled
(It must be recovered by evacuation if a valid target host becomes available or reboot/rebuild if the source host becomes enabled again)
Others that can migrate will continue their migration.

Some instances dirtying memory (and in theory should be unable to complete a migration to the new host ie. auto-converge disabled, live migration max downtime is low eg. 100 - 120, hw:wrs:live_migration_autocoverge disabled ie. false. Confirm instance(s) that 

could not be live migrated (rollback) go to error.

Others instances are not dirtying memory and should continue to be able to schedule and complete migration.
Confirm instances that can be scheduled, live migrate to another available host.

Run ping test from the guest VM. The ping will stop temporarily during the live migration.
Run stress in the guest to dirty pages but not so much that the instance will not migrate. The instance is paused to complete the live migration.

4. Confirm the instances that do not migrate recover when the hypervisor is enabled again.


Expected Behavior:
-----------------------------
1. When the host data port fails, the hypervisor becomes disabled.

2. Alarms raised as expected and other instances continue to be migrated to the other available worker host
275.001 Host<worker> hypervisor is now locked-disabled
300.001	'Data' Port failed.
300.002	'Data' Interface failed.
700.151	Live-Migrate issued by the system against instance <name> owned by tenant1 from host compute-0, reason = host component failure
700.151	Live-Migrate issued by the system against instance <name> owned by
tenant2 from host compute-0, reason = host component failure

3. When the host data port fails and instances fail during the migrate attempt (and error), others that can migrate will continue their migration.
Some instances were unable to be migrated (and error)
Note: Under guest stress condition (dirtying memory too quickly) where autocoverge is disabled, the instance under stress will be issued live migration request but will eventually timeout and fail (3 minutes later for example). Then VIM will issue the next 

live migration request. So a delay is expected here.

Others that can migrate will continue their migration.

4. When the data port is restored, confirm evacuation issued on the instances that are in error state (ie. to recover when the hypervisor is enabled again)
700.175 Evacuate issued by the system against instance <name> owned by tenant2 on host <worker>, reason = host component failure
700.175 Evacuate issued by the system against instance <name> owned by tenant2 on host <worker>, reason = host component failure
...
700.181	Reboot (hard-reboot) issued by the system against instance <name> owned by tenant2 on host <worker>


Nova_Failure_Hypervisor_3.0
test_where_some_instance(s)_may_rollback_while_others_continue_to_be_migrated_on_hypervisor_failure/disabled_(Duplex)
Tags: p2,regression,nova,DX

Testcase Objective:
--------------------------------
Test where some instance(s) may rollback while others continue to be migrated on eg. on a hypervisor failure (eg. where hypervisor state down; status goes to disabled)

Test Pre-Conditions:
--------------------------
Duplex system

Test Steps:
----------------
1. Launch multiple instances on one worker host where the instance has particular requirements such as cpu thread policy, shared cpu assignment, 1G hugepages
2. Disable the hypervisor of the host where these instances reside (to trigger the migration)

For example  <worker>:~$  ps ax | grep nova
72507 ?        Ssl   12:24 /usr/bin/python2 /usr/bin/nova-compute
72647 ?        S      0:00 /usr/bin/python2 /bin/privsep-helper --config-file /usr/share/nova/nova-dist.conf --config-file /etc/nova/nova.conf --config-file /etc nova/nova-compute.conf --privsep_context os_brick.privileged.default --privsep_sock_path 

/tmp/tmpArG3Fj/privsep.sock

<worker>:~$ sudo systemctl stop libvirtd.service openstack-nova-compute.service


Expected Behavior:
-----------------------------
In step 2, the hypervisor changes to ‘locked-disabled’,  the host remains in  “Unlocked, Enabled, Available” state.
Confirm hypervisor becomes disabled

$ nova  hypervisor-list
ID | Hypervisor hostname | State | Status   |
<id>  | <host            | down  | disabled

nova hypervisor-show <host>
state                     | down                                     |
| status                    | disabled

Instance(s) that can be migrated, migrate.
Instances can not live migrate to the new host target for valid reason(s) where the hypervisor has been disabled:
For example, instance can not be scheduled on the new target host for any of the reasons below

a. the target host is not hyperthreaded but instance has thread policy require

b. the target host does not have shared cpu assignment and the instance requires it.
(NUMATopologyFilter) Shared not enabled for cell 0

c. the target host does not have available 1G Hugepages
(NUMATopologyFilter) Not enough memory:

Confirm instances that can not, rollback. The instance attempts evacuation (but fail as there is no other host suitable) then eventually reboots to recover when the hypervisor recovers.

 	 275.001	Host controller-1 hypervisor is now locked-disabled

700.001	Instance <instancename> owned by <tenantX> has failed on host <host>

700.152	Live-Migrate inprogress for instance <instancenameA> from host controller-1
700.175	Evacuate issued by the system against instance <instancenameB> owned by <tenantX> on host <host>, reason = host disable action
700.179 Evacuate failed for instance <instancenameB> on host <host>
275.001	Host <host> hypervisor is now unlocked-enabled
..
700.186 Reboot complete for instance <instancenameB> now enabled on host controller-1
270.102	Host <host> compute services enabled



Feature Overview:
=================
Nova - Auto-convergence Threshold

Test Requirements:
===================
Minimum 2 node

Test Cases:
===============

Nova_Convergence_1.0
test_nova_livemigration_autoconverge-_4VCPU,_4GB_RAM,_stress-ng_with_1_per_vcpu_and_2048M,1024M_vm-bytes

Live migration auto-converge threshold (92% throttling max)
Live migration with autoconverge enabled (multiple supported interfaces)
While running stress to observe CPU throttling in increments to attempt to allow live migration to complete.
Tags: p2,regression,nova,stress


Testcase Objective:
--------------------------------
Live migrate instance under stress where instance has multiple supported interfaces to ensure auto-converge is throttling but not starving CPU (max CPU throttling 92%)
stress-ng (running up to 4 in parallel)
2048, 1024M, 512 vm-bytes

Test Pre-Conditions:
--------------------------

Test Steps:
----------------
1. Boot instances with multiple supported interfaces (from cinder)
Flavor has 4 VCPU, 4GB RAM, local_image, dedicated cpu policy
4VCPU, 4GB/6GB RAM
'hw:cpu_policy': 'dedicated'
'hw:mem_page_size': '2048'

As well, the following instance metadata are set on the instance and auto converge settings are enabled (enabled by default).
'hw:wrs:live_migration_max_downtime': 'X'
'hw:wrs:live_migration_timeout': 'X'

2. Run live migrations, monitoring CPU using top (on the source host) to see CPU% for each pCPU in the host.  It should throttle each vCPU by (maybe 20% initially) then by 10% each step (starting from 99%). CPU should never throttle below 8% (ie. max 92 

percent throttling)

$ top -H | grep CPU  (on the host)

Run stress (not in parallel)
stress-ng --vm 1 --vm-bytes 2048M --vm-keep --vm-method swap &

The instance-00000000#.log reports the following:
initiating migration
shutting down

The libvirt.log reports:
Migration running for X secs; memory progress XX% (X MiB processed, XX MiB remaining, XXX MiB total); data progress X% (XXX MiB processed, XXX MiB remaining, XXX MiB total); memory bandwidth XXX MB/s;expected downtime XXX ms;since X seconds ago
..
Increasing downtime to X ms after X sec elapsed time
...
VM Paused (Lifecycle Event)
Migration operation has completed
_post_live_migration() is started

CPU increments down roughly 10% each step

Run stress-ng (only 1 vm ie. not running in parallel) with virtual memory stress (paging and memory) --vm-bytes 1024M
$ top -H | grep CPU  (on the host)


3. Repeat the migration stress test with instance tht has the following flavor. Expect in live migration, the CPU will throttle down in increments of approx. 10 starting from 99 Migrate completes/converges.

4 VCPU, 4 GB RAM, local_image, dedicated cpu policy
auto converge enabled by default
stress-ng --vm 1 --vm-bytes 1024M --vm-keep --vm-method swap &	CPU increments down roughly 10% each step

Run stress-ng (run 1 for 2 of 4 vCPU in parallel) with virtual memory stress (paging and memory) --vm-bytes 1024M  to busy multple pCPU on the host.
Flavor has 4 VCPU, 4 GB RAM, local_image, dedicated cpu policy
auto converge enabled by default

Live migrate the instance while watching CPU% in the host
$ top -H | grep CPU  (on the host)

stress-ng --vm 1 --vm-bytes 1024M  --vm-keep --vm-method swap &
stress-ng --vm 1 --vm-bytes 1024M --vm-keep --vm-method swap &

CPU increments down roughly 10% each step (but does not go below 8%)


Run stress-ng (1 to 3 out of 4 vCPU in parallel) with virtual memory stress (paging and memory) --vm-bytes 1024M


4. Repeat the migration stress test with instance tht has the following flavor. Expect in live migration, the CPU will throttle down in increments of approx. 10 starting from 99 Migrate completes/converges.

Flavor has 4 VCPU, 4 GB RAM, 25 root disk, local_image, dedicated cpu policy, 2048 mem page size,
auto converge enabled by default

Live migrate the instance while watching CPU% in the host
$ top -H | grep CPU  (on the host)

stress-ng --vm 1 --vm-bytes 1024M --vm-keep --vm-method swap &
stress-ng --vm 1 --vm-bytes 1024M --vm-keep --vm-method swap &
stress-ng --vm 1 --vm-bytes 1024M --vm-keep --vm-method swap &


CPU increments down roughly 10% each step (but does not go below 8%)
eg. CPU% decrements eg. 99 to 80, 70, 60, 50, 40, 30, 20, 12.2, 9.9, ~7.9/8.3

hw:wrs:live_migration_max_downtime=600
hw:wrs:live_migration_timeout=600

timeout and downtime set in instance metadata (increased to 600 to see the throttling)

700.154 Live-Migrate cancelled for instance test now on host compute-2, reason = Live migration timeout after 600 sec
...
13355 root      20   0 7030552  29588  11092 S  8.3  0.0  17:25.57 CPU 3/KVM
13354 root      20   0 7030552  29588  11092 S  8.3  0.0  10:31.23 CPU 2/KVM
13355 root      20   0 7030552  29588  11092 S  8.3  0.0  17:25.82 CPU 3/KVM
13353 root      20   0 7030552  29588  11092 S  7.9  0.0  19:49.28 CPU 1/KVM

Expected Behavior:
-----------------------------
When live migration autoconverge is enabled, the CPU is throttled (increments down roughly 10% each step) but does not go below 8%.
This increases the likelihood of migration successfully completing. The live migration settings are respected.


Nova_Convergence_2.0
test_nova_livemigration_autoconverge-_4VCPU,_6GB_RAM,_stress-ng_with_1_per_vcpu_and_512M_vm-bytes

Testcase Objective:
--------------------------------
Live migrate instance under stress where instance has multiple supported interfaces to ensure auto-converge is throttling but not starving CPU (max CPU throttling 92%)
stress-ng (running up to 4 in parallel)

Test Pre-Conditions:
--------------------------

Test Steps:
----------------
1. Boot instances with multiple supported interfaces (from cinder)
Flavor has 4 VCPU, 6 GB RAM, local_image,
dedicated cpu policy
auto converge enabled by default

2. Instance metadata optionally set to hw:wrs:live_migration_downtime 180, and live migration timeout 180

3. Run live migrations
Monitor CPU using top (on the source host) to see CPU% for each pCPU in the host. It should throttle each vCPU by (maybe 20% initially) then by 10% each step (starting from 99%). CPU should never throttle below 8% (ie. max 92 percent throttling)

$ top -H | grep CPU (on the host)

(stress run 4 in parallel)
stress-ng --vm 1 --vm-bytes 512M --vm-keep --vm-method swap & stress-ng --vm 1 --vm-bytes 512M --vm-keep --vm-method swap & stress-ng --vm 1 --vm-bytes 512M --vm-keep --vm-method swap & stress-ng --vm 1 --vm-bytes 512M --vm-keep --vm-method swap &

CPU increments down roughly 10% each step to allow converging.
Confirm CPU is not throttled to 0 eg. only throttles down then converges (8% min)
97046 root      20   0 9160524  30356  11092 S 11.9  0.0   1:05.36 CPU 0/KVM
97047 root      20   0 9160524  30356  11092 S 11.9  0.0   1:03.47 CPU 1/KVM
97048 root      20   0 9160524  30356  11092 S 11.9  0.0   1:05.16 CPU 2/KVM
97049 root      20   0 9160524  30356  11092 S 11.9  0.0   1:05.72 CPU 3/KVM


Expected Behavior:
-----------------------------
When live migration autoconverge is enabled, the CPU is throttled (increments down roughly 10% each step) but does not go below 8%. This increases the likelihood of migration successfully completing.


Feature Overview:
=================
Nova - Log nova-api-proxy events (nova server actions)
An admin user using the remote cli client can issue nova commands and the events are recorded in the nova-api-proxy.log. Confirm /var/log/nova_api_proxy.log output for each of the following where the NFV action can be extracted from the payload of the message
The log will indicate the NFV action as extracted from the payload of the message. The ip address, the POST request issued on the server, the <instanceid> of the server and by whom the request was issued (user authenticated and their project user admin tenant 

admin)


Test Requirements:
===================

Test Cases:
===============

Nova_APIProxyLog_1.0
test_proxy_logs_cli_admin_novaactions_payload_remote_1.0
Tags: p2,regression,nova,DX,admin

Testcase Objective:
--------------------------------
As admin user, I want to see nova actions performed via remote cli from off-box, logged to the nova-api-proxy.log
The nova-api-proxy.log will indicate the NFV action as extracted from the payload of the message for these actions.

Test Pre-Conditions:
--------------------------

Test Steps:
----------------
1. Connect to the controller user the remote cli client (off box) as an admin user.
2. Perform the following nova actions:

$ nova stop <instanceid>
$ nova start <instanceid>
$ nova suspend <instanceid>
$ nova resume <instanceid>
$ nova pause <instanceid>
$ nova unpause <instanceid>

2. Perform nova Resize operations:
$ nova resize <instanceid> --poll <instanceid> <newflavor>
$ nova resize-revert <instanceid> (or resize-confirm)

3. Perform instance Soft reboot actions
$nova reboot <instanceid>

4. Perform nova Migration operations on the Instance
Cold migrate and Confirm
$nova migrate <instanceid>
$nova resize-confirm <instanceid>

$nova live-migration <instanceid>

5. Perform instance Rebuild operatons:
$nova rebuild --poll --name <name> <instanceid>

Expected Behavior:
-----------------------------
Confirm /var/log/nova_api_proxy.log output for each of the following where the NFV action can be extracted from the payload of the message
The log will indicate the NFV action as extracted from the payload of the message, the ip address, the POST request issued on the server, the <instanceid> of the server and by whom the request was issued. In this test it will be the admin user authenticating 

(project user admin tenant admin)


Nova_APIProxyLog_2.0
test_proxy_logs_cli_admin_novaactions_remote_2.0
Tags: p2,regression,nova,DX,admin

Testcase Objective:
--------------------------------
As admin user, I want to see nova actions performed via remote cli from off-box, logged to the nova-api-proxy.log
The nova-api-proxy.log will not indicate these NFV actions that are not included in the payload of the message. It will include the POST request issued by the project admin

Test Pre-Conditions:
--------------------------
Connect to the controller user the remote cli client (off box) as an admin user.

Test Steps:
----------------
1. Connected to the controller user the remote cli client (off box) as an admin user, perform the following nova actions:

nova shelve <instanceid>
nova unshelve <instanceid>
nova lock <instanceid>
nova unlock <instanceid>
nova reset-state <instanceid>
nova rescue <instanceid>

2. Abort an on-going live migration or Force on-going live migration to complete.
$nova migration-list
$live-migration-abort <instanceid> <migrationid> (to abort the on-going live migration)
$live-migration-force-complete <instanceid> <migrationid> (to force the on-going live migration to complete)
$ nova scale 1f8a9a0d-e33a-4d2e-9609-be976ce5a513 cpu down

Expected Behavior:
-----------------------------
Confirm /var/log/nova_api_proxy.log output for each of the following POST operations where the NFV action is *NOT* included in the payload of the message
The log will include the ip address, the POST request issued on the server, the <instanceid> of the server and by whom the request was issued (user authenticated and their project user admin tenant admin). The log will not indicate the NFV action (as it is not 

included in the payload of the message).


Nova_APIProxyLog_3.0
test_proxy_logs_cli_tenant_novaactions_remote
Tags: p2,regression,nova,tenant

Testcase Objective:
--------------------------------
As a tenant user using the remote cli client (off box), run nova actions to confirm nova_api_proxy.log output for each. Perform Nova commands with NFV action (listed below) as the tenant user
Attempt Nova actions even though not allowed by policy: NFV action should be logged in thenova-api-proxy.log

As a tenant user using the remote cli client (off box), run nova actions to confirm nova_api_proxy.log output for each.
Perform Nova commands with NFV actions (listed below) as the remote tenant user (off box). Perform nova actions ensuring that the nova-api-proxy.log includes the POST request and the NFV action as extracted from the payload of the message.

Attempt Nova actions even though not allowed by policy: NFV action should be logged in thenova-api-proxy.log
Log also nova actions attempted by tenant user where the user is not allowed by policy.

Test Pre-Conditions:
--------------------------
Tenant user connects using the remote cli (offbox)

Test Steps:
----------------
1.
Perform the following nova actions (sourced as tenant user) confirming the nova_api_proxy.log output for each:
ie. nova commands with NFV action (listed below) as the tenant user eg. tenant2

Nova commands with NFV action
Pause instance, Unpause instance
Suspend instance, Resume instance
$nova suspend <instanceid>
$nova resume a6a1e06d-afbf-4fa7-99b0-84a67218292c

Stop instance, Start instance
$ nova stop 026f427e-10c8-4981-a10b-415efe4d2211
$nova start 026f427e-10c8-4981-a10b-415efe4d2211

Resize Instance (or stop instance,resize and confirm)
Revert | confirm
~(keystone_tenant2)]$ nova resize-confirm a6a1e06d-afbf-4fa7-99b0-84a67218292c
Resize revert
$ nova resize-revert a6a1e06d-afbf-4fa7-99b0-84a67218292c

Stop/shutdown – start then reboot the instance (soft and hard reboots)
~(keystone_tenant2)]$nova stop <instanceid>
Stop/shutdown – start then reboot the instance (soft and hard reboots)
~(keystone_tenant2)]$nova stop <instanceid>
~(keystone_tenant2)]$nova start <instanceid>
~(keystone_tenant2)]$nova reboot <instanceid>
~(keystone_tenant2)]$ nova reboot --poll --hard <instanceid>
Rebuild instance as tenant user
(keystone_tenant2)]$ nova rebuild –poll <instancename> <imageid>



2. Perform the following nova actions (sourced as tenant user) even though not allowed by policy.

cold migrate (even though not allowed by policy)
$ nova migrate <instanceid>
live block migration (even though not allowed by policy)
$ nova live-migration --block-migrate <instanceid>
live migration (even though not allowed by policy)
$ nova live-migration <instanceid>



3.
Perform the following nova actions (sourced as tenant user).
- For these Post operations the NFV action is not included in the payload of the message.

(keystone_tenant2)]$nova lock <instanceid>
(keystone_tenant2)]$ nova unlock <instanceid>

Expected Behavior:
-----------------------------
1. The nova-api-proxy.log will indicate the NFV action as extracted from the payload of the message
ip address
the POST request issued on the server
The <instanceid> of the server
By whom the request was issued (user authenticated and their project (tenant <user>)


Note:
The requests come from haproxy in this case and as such the haproxy forwarded address ie. the <frontendip> as listed in the haproxy.cfg, should be shown in the nova-api-proxy.log (as the remote address)

The following are the frontend and backend internal settings in the haproxy.cfg (/etc/haproxy/haproxy.cfg).

frontend nova-ec2-restapi
bind <frontendip>:port

default_backend nova-ec2-restapi-internal
reqadd X-Forwarded-Proto:\ http

backend nova-ec2-restapi-internal
server s-nova-ec2 <backendinternalip>:port


2. The NFV action should be logged in the nova-api-proxy.log
The nova-api-proxy.log will indicate the NFV action as extracted from the payload of the message including:
ip address, the POST request issued on the server, the <instanceid> of the server, by whom the request was issued (user authenticated and their project (tenant <user>)

3. The nova-api-proxy.log will include the ip address, the POST request issued on the server.
The <instanceid> of the server
By whom the request was issued (user authenticated and their project (tenant <user>)




Nova_APIProxyLog_4.0
test_new_proxy_logs_exists_in_log_collection
Tags: p2,regression,nova

Testcase Objective:
--------------------------------
New nova-api-proxy.log exists after installation and log collection includes this log.
A new log "nova-api-proxy.log" will be created in the /var/log folder and the log collector will include this log.


Test Pre-Conditions:
--------------------------

Test Steps:
----------------
1. On a lab that has been installed, navigate to the /var/log location and confirm the new log file nova-api-proxy.log exists
The new log file exists in this location

2. Perform log collection
$collect  all

Expected Behavior:
-----------------------------
Confirm collection completes successfully and that the new file nova-api-proxy.log is included in the collected logs (for both controllers)



Feature Overview:
=================
Nova - Host CPU Passthrough     ---- Refactoring in STX?

https://www.berrange.com/posts/2018/06/29/cpu-model-configuration-for-qemu-kvm-on-x86-hosts/

Host CPU Passthrough enabled for a guest (ie., -cpu host).
This feature adds a new VCPU model (“Host-Passthrough”) that will enable the “host-passthrough” libvirt cpu_mode.

Openstack documentation describes it as follows
"host-passthrough" causes libvirt to tell KVM to passthrough the host CPU with no modifications. The difference to host-model, instead of just matching feature flags, every last detail of the host CPU is matched. This gives absolutely best performance, and can 

be important to some apps which check low level CPU details, but it comes at a cost on migrations. The guest can only be migrated to an exactly matching host CPU.


For this feature, the vcpu model 'host-passthrough' can be selected and host cpu features are exposed to the guest.
	hw:cpu_model 	Passthrough

Confirm the xml for the instance contains
<cpu mode='host-passthrough'>
....
</cpu>

Cold migrate, live-migrate and evacuate will be supported.
The VCPU filter will only select hosts with the exact same CPU model of the source host. Scheduler checks VCpuModelFilter to determine if it can migrate to the destination.

Test Requirements:
===================

Test Cases:
===============

Nova_CPUPassthrough_1.0
test_evacuate_instances_with_hw:cpu_model_passthrough_to_new_compute_host  --- Refactoring
Tags: p2,regression,nova

Testcase Objective:
--------------------------------
Instances with VCPU model passthrough can evacuate to a new host
Instances can evacuate to a new host where VCPU Model has been set to 'passthrough'

Test Pre-Conditions:
--------------------------


Test Steps:
----------------
1. Instances with hw:cpu_model passthrough exist on the host

2. Run evacuation test and verify hosts land on another available host

3. Log into the guest that has migrated to the new host and confirm the cpu model in the guest:
$cat /proc/cpuinfo

Expected Behavior:
-----------------------------
Confirm the expected cpu model in the guest reported when cat /proc/cpuinfo
cpu model is eg. Intel(R) Xeon(R) CPU E5-2699 V3 @ 2.30ghZ



Feature Overview:
=================
Nova - Live Migration Auto Converge and Tunnelling

Migrations timeouts where an application memory is dirty faster than it can be copied to the destination the migration can not complete before timeout.
The test will verify that the instance is slowed (cannot dirty memory as fast) in order that live migrations can converged and complete.

Auto converge interact with post copy. Auto-converge will only be used if live_migration_permit_post_copy=False (or not available) on source/target host.
Live migration post copy will not be enabled (see Reference as follows for potential performance impacts https://specs.openstack.org/openstack/nova-specs/specs/newton/implemented/auto-live-migration-completion.html)
The compute flag for auto converge in the /etc/nova/nova.conf file must also be set to True.

Autoconverge is controlled only by flavor extra-specs and instance metadata, not image properties.
The live migration auto converge setting can be enabled or disabled in the flavor extra spec or the instance metadata.
nova flavor-key <flavorid> set
hw:wrs:live_migration_auto_converge=True {False}

Auto converge can also be enabled or disabled using the instance metadata.
hw:wrs:live_migration_auto_converge=True {False}

Live Migration Condition: Dirty pages
The auto converge option is enabled by default.
Hypervisor throttles down CPU of an instance during live migration in case of slow progress due to dirty pages.
Tests will confirm that it is throttled down when autoconverge has been enabled (and not when it is disabled)

Conflicting settings.
If both flavor and instance metadata behavior are specified, the instance metadata setting is used.
hw:wrs:live_migration_permit_auto_converge=True {False}

Live Migration Maximum Downtime
Downtime for live migration can now also be configured in the instance metadata (at boot time or updated after the instance is running), in addition to the image metadata and flavor. The default is 500 ms

flavor or instance metadata syntax
hw:wrs:live_migration_max_downtime=ms
instance metadata
hw_wrs_live_migration_max_downtime=ms

The extra spec setting overrides the instance metadata.
If both extra spec and instance metadata are set, the flavor setting takes priority.

Live Migration Timeout
Default timeout value is 180 sec

nova flavor-key <id> set hw:wrs:live_migration_timeout=<seconds>
Horizon must also change to reflect new range and context help in eg. "Extra Spec"

New Support for Live Migration Timeout in the Instance Metadata
It is now supported in the image metadata, flavor extra spec and instance metadata

If set in multiple locations, the *lowest* of the values set is used.
In flavor extra spec           hw:live_migration_timeout=X (seconds)
In image metadata              hw_live_migration_timeout=Y (seconds)
In instance metadata           hw:live_migration_timeout=Z (seconds)

Live Migraton Tunnelling
Considerations: Live migration tunneling enabled and disabled (success and failure paths), where tunneling is set in the flavor or instance metadata.
(Confirm tunnelling by examining the CPU usage of libvirtd (or alternatively checking the TCP port that it uses in live migraton via tcpdump))

hw:wrs:live_migration_tunnel=True {or False}

Autoconverge is controlled only by flavor extra-specs and instance metadata, not image properties.
Conflicting settings
Where the setting is in both flavor and instance metadata, the instance metadata setting takes priority
ie. if tunnelling is disabled in the flavor but enabled in the instance metadata, the instance will tunnel
ie. if tunnelling is enabled in the flavor, disabled in the instance metadata, the instance will not tunnel

Live migration throughput is observed under multiple system (such as CPE, Infra only, LACP bonded and a system with only Management interface)
With tunneling enabled in live migration, throughput is reduced
The nova-compute.log may report a value much lower than it would with tunnelling disable
eg. 11000MB/sec with tunelling vs 600-700 MB/sec with tunnelling enabled. (see nova-compute.log)

Live migration stress (bandwidth) test cases use stress-ng tool to stress the guest
For example.
On the Instance based off of TIS-Centos, install stress-ng packages
1 – yum install util-linux (for taskset)
2 – yum install stress
3 – yum install stress-ng

Using Stress-ng on each CPU
e.g: for isntance with 4 VCPUs, run a stress-ng for each vcpu
taskset -c 1 stress-ng --vm 1 --vm-bytes 512M --vm-keep --vm-method swap &
taskset -c 2 stress-ng --vm 1 --vm-bytes 512M --vm-keep --vm-method swap &
taskset -c 3 stress-ng --vm 1 --vm-bytes 512M --vm-keep --vm-method swap &
taskset -c 4 stress-ng --vm 1 --vm-bytes 512M --vm-keep --vm-method swap &

Launch Bogus Operations as well with Stress-ng or Stress
sudo stress-ng --cpu 8 --io 3 --vm 2 --vm-bytes 512M&
stress-ng --fork 4 --fork-ops 100000&

* The stress tests were run with stress-ng with taskset as well as the bogus operations using stress side by side.

For creating a Larger Dirty Pages Footprint, launch several instances of dd as follows:
dd if=/dev/zero of=todel bs=1048576 count=8192&
(You don't need to always employ the dd tool as Stress-ng puts a lot more load on the VMs then dd)

The “auto-converge” feature in qemu is enabled by default
Autoconverge is controlled by flavor or instance metadata (not image metadata)
This test enables it also in the instance metadata
hw:wrs:live_migration_auto_converge=True

Test Requirements:
===================
2 node system minimum

Test Cases:
===============

Nova_LiveMigration_1.0
test_autoconverge_Instance_launch_with_live_migration_auto_converge_enabled_from_instance_metadata
Tags: p2,regression,nova


Testcase Objective:
--------------------------------
Autoconverge is controlled by flavor or instance metadata (not image metadata)
Instance launch with live migration auto converge enabled from instance metadata
hw_wrs_live_migration_permit_auto_converge=True

Test Pre-Conditions:
--------------------------

Test Steps:
----------------
1. Launch instance from cinder volume, setting the auto converge setting to True in the instance metadata.
hw:wrs:live_migration_auto_converge=True

2. Test live migrate under a high workload which will prevented the live migration from completing.

For example.
Run a stress tool on the guest VM to dirty memory faster than the transfer rate.
eg. run stress-ng tool   (or perform read/write operations on a large file with dd)

Monitor with top to confirm 100% usage and perform the live migration.

Expected Behavior:
-----------------------------
Confirm the auto converge setting is enabled
With auto converge enabled, the usage should visibly drop down from 100% to allow the migration to complete, then return back up when the live migration has completed.	The live migration will use auto-converge when it is enabled from the instance metadata.

The live migration will not be as likely to time out as the guest VM usage is slowed to prevent dirty rate from exceeding transfer rate.



Nova_LiveMigration_2.0
test_LiveMigrateMaxDowntime_Live_Migration,_Maximum_Downtime_not_set_defaults_to_500_ms
Tags: p2,regression,nova

Testcase Objective:
--------------------------------
Where the maximum live migration downtime has not been specified anywhere, the default will be 500 ms
The system default Live Migration Maximum Downtime is 500 ms if it is not set in the flavor or image meta or instance metadata.

Test Pre-Conditions:
--------------------------

Test Steps:
----------------
1. Boot an instance where the live migration maximum downtime has not been specified anywhere in the flavor or image metadata or instance metadata.
2. Perform live migration timeout test to confirm the default maximum downtime of 500 ms

Expected Behavior:
-----------------------------
The nova-compute.log will report the following on live migration
expected downtime (ms)
INFO nova.virt.libvirt.migration [instance: <id>] Increasing downtime to X ms after 90 sec elapsed time

Confirm X does not exceed the expected downtime (on lengthy migrations)
ie. Confirm 500ms is used for the maximum downtime on lengthy live migrations.


Nova_LiveMigration_2.1
test_LiveMigrateMaxDowntime_Live_Migration,_Maximum_Downtime_set_from_image_metadata
Tags: p2,regression,nova

Testcase Objective:
--------------------------------
Live migration maximum downtime can be configured in the image metadata.
This is the maximum downtime allowed for live migration
The minimum setting that can be set is 100 ms
hw_wrs_live_migration_max_downtime=<value in ms>

Live Migration, Maximum Downtime set from image metadata

Test Pre-Conditions:
--------------------------

Test Steps:
----------------
1. Set the Live Migration Maximum downtime in the image metadata

2.As tenant user, create a cinder volume from the image
3.Run the following to confirm the value is set in the volume_image_metadata

$cinder show <volumeid>
'hw_wrs_live_migration_max_downtime': u'100'
'hw_wrs_live_migration_permit_auto_converge': u'True'

4. Launch instance from the volume that contains the volume_image_metadata setting for live migration Maximum Downtime

For eg. 'hw_wrs_live_migration_max_downtime': u'100'
Note: The higher he value for allowable downtime, the more likely it will converge.

5. Run live migration test under stress to confirm the setting specified from the volume_image_metadata is used for the downtime.

Expected Behavior:
-----------------------------
nova-compute.log reports the following on live migration
expected downtime (ms)
INFO nova.virt.libvirt.migration [instance: <id>] Increasing downtime to X ms after 90 sec elapsed time

Confirm X does not exceed the downtime set on lengthy migration.



Nova_LiveMigration_3.0
test_Live_Migration_Timeout_set_in_the_flavor_extra_spec,_cli_validation
Tags: p2,regression,nova

Testcase Objective:
--------------------------------
Live migration timeout valid range is 120 to 800 and 0 to disable.

Some parameters that can effect timeout are as follows:
dirtying rate (pages/sec),size of a page (KB), number of memory pages on the VM
network bandwidth available KB/sec (between source and target), network utilization during migration

Test Pre-Conditions:
--------------------------


Test Steps:
----------------
1. Set the following valid live migration timeout values from the cli

nova flavor-key <id> set hw:wrs:live_migration_timeout=<time in seconds>
nova flavor-key <id> set hw:wrs:live_migration_timeout=120
nova flavor-key <id> set hw:wrs:live_migration_timeout=150
nova flavor-key <id> set hw:wrs:live_migration_timeout=800
nova flavor-key <id> set hw:wrs:live_migration_timeout=0

2. Attempt setting the following invalid live migration timeout values from the cli

nova flavor-key <id> set hw:wrs:live_migration_timeout=<time in seconds>
nova flavor-key <id> set hw:wrs:live_migration_timeout=-1
nova flavor-key <id> set hw:wrs:live_migration_timeout=1
nova flavor-key <id> set hw:wrs:live_migration_timeout=119
nova flavor-key <id> set hw:wrs:live_migration_timeout=801

Expected Behavior:
-----------------------------
Confirm error response indicates the correct valid range if the user attempts setting beyond the valid range from cli
ERROR (BadRequest): hw:wrs:live_migration_timeout must be 0 or in the range 120 to 800 (HTTP 400)


Nova_LiveMigration_3.1
test_Live_MigrateTimeout_set_in_the_flavor_extra_spec,cli/Horizon_validation
Tags: p2,regression,nova

Testcase Objective:
--------------------------------
The live migration timeout range remains as follows:
120 seconds - 800 seconds, and 0

Test Pre-Conditions:
--------------------------


Test Steps:
----------------
1. Confirm the default value when the Live Migration Timeout extra spec is selected in Horizon is 180.

2. Set or edit the following valid values for live migration timeout using Horizon
nova flavor-key <id> set hw:wrs:live_migration_timeout=<time in seconds>
nova flavor-key <id> set hw:wrs:live_migration_timeout=120
nova flavor-key <id> set hw:wrs:live_migration_timeout=150
nova flavor-key <id> set hw:wrs:live_migration_timeout=800
nova flavor-key <id> set hw:wrs:live_migration_timeout=0

3. Attempt editing/setting the following invalid live migration timeout values from Horizon
nova flavor-key <id> set hw:wrs:live_migration_timeout=<time in seconds>
nova flavor-key <id> set hw:wrs:live_migration_timeout=-1
nova flavor-key <id> set hw:wrs:live_migration_timeout=1
nova flavor-key <id> set hw:wrs:live_migration_timeout=119
nova flavor-key <id> set hw:wrs:live_migration_timeout=801

Expected Behavior:
-----------------------------
1. Confirm the default value for live migration timeout is 180.
2. Confirm an error response indicates the correct valid range if the user attempts setting beyond the valid range
3. Confirm the context help and field labels in Horizon reflect the default (180 seconds)


Nova_LiveMigration_3.2
test_LiveMigrationTimeout_Defaults_to_180_sec
Tags: p2,regression,nova

Testcase Objective:
--------------------------------
The default value if unspecified is actually 180 sec by default (if it has not been specified in the flavor, instance meta or image meta)

Test Pre-Conditions:
--------------------------

Test Steps:
----------------
1. Launch an instance without specifying live migration timeout values in either the flavor, image or instance metadata.

2. Test that the default value for live migration timeout is now 180 sec (if it has not been specified in the flavor, instance meta or image meta)
by running live migration under memory stress conditions that would cause live migration to timeout

Expected Behavior:
-----------------------------
Confirm in nova-compute.log on timeout:
"Live migration not completed after 180 sec"


Nova_LiveMigration_3.2
test_LiveMigrationTimeout_set_in_only_the_instance_metadata_and_confirm_timeout_value_respected
Tags: p2,regression,nova,stress

Testcase Objective:
--------------------------------
Live Migration Timeout set in only the instance metadata and confirm timeout value respected

Test Pre-Conditions:
--------------------------

Test Steps:
----------------
1. Boot instance from volume live migration timeout set in the instance metadata.
nova meta <id> set hw:wrs:live_migration_timeout=<value>

2. Run stress on guest VM to dirty memory
3. Attempt live migration

Expected Behavior:
-----------------------------
Confirm live migration timeout value set in only the instance metadata is respected.



Feature Overview:
=================
Nova - Live Migration-Tunnelling

The nova.conf setting has:
VIR_MIGRATE_UNDEFINE_SOURCE, VIR_MIGRATE_PEER2PEER, VIR_MIGRATE_LIVE, VIR_MIGRATE_TUNNELLED
in the flag "live_migration_flag"

VIR_MIGRATE_TUNNELLED:
With this flag the actual data for migration will be tunnelled over the libvirtd RPC
channel. This requires that VIR_MIGRATE_PEER2PEER also be set.

Reference:
http://libvirt.org/git/?p=libvirt.git;a=patch;h=fae0da5c1331dc9e5efb6eab345eba1412139136

Note: When the tunneling is enabled, run 'top' on the host during migration to see heightened usage in libvirtd. (It will be lower if not tunnelling)
Alternatively, tcpdump could be used to capture the live migrations in pcap file to confirm that migrations are using port 16509 when tunneling is enabled.
Analyze TCP port 16509 (Filter on the port or follow TCP stream)

Logs will also report when the live migration has the tunnel enabled
"libvirt_tunnel enabled for this migration"

Test Requirements:
===================
2 node system minimum

Test Cases:
===============

Nova_LiveMigration_4.0
test_LiveMigratonTunnelling_Cinder_boot,no_ephem_or_swap_-_live_migration_tunnelling_enabled_in_flavor
Tags: p2,regression,nova

Testcase Objective:
--------------------------------
Test live migration tunnelled where instance booted from cinder - no ephem or swap
Live migration tunnel enabled in flavor
Flavor extra spec hw:wrs:live_migration_tunnel=True

Test Pre-Conditions:
--------------------------

Test Steps:
----------------
1. Where storage backing is remote:
Create instance with tunnelling enabled in the flavor extra spec (where the instance is booted from cinder volume, no ephem or swap)
hw:wrs:live_migration_tunnel=True

Test live migration with the tunnelling enabled (locking host) - while running top on the source worker.
Live migration should succeed.
Review top output from libvirtd to confirm tunneling is enabled.


2. Where storage backing is local_image:
Create instance with tunnelling enabled in the flavor extra spec (where the instance is booted from cinder volume, no ephem or swap)
hw:wrs:live_migration_tunnel=True
Test live migration with the tunnelling enabled (locking host) - while running top on the source worker.

Expected Behavior:
-----------------------------
For both 1 and 2, live migration should succeed.
Review top output from libvirtd to confirm tunneling is enabled (will see higher %CPU when tunnelling has been enabled).



Nova_LiveMigration_4.1
test_LiveMigratonTunnelling_Cinder_boot,_ephem,swap_disk_-attempt_live_migration,tunnelling_enabled_instance_metadata
Tags: p2,regression,nova
Sub-domain = Storage

Testcase Objective:
--------------------------------
hw:wrs:live_migration_tunnel=True
Where instance is booted from cinder volume, with ephem and swap disk
Cinder boot instance with ephem,swap disk
Attempt live migration,tunnelling enabled instance metadata

Test Pre-Conditions:
--------------------------
Storage backing is remote for this test

Test Steps:
----------------
1. Create instance where tunnelling is enabled using the instance metadata where the instance is booted from cinder volume,
with both ephem and swap disk settings) and the flavor has hw:wrs:live_migration_tunnel=True

2. Test live migration with the tunnelling enabled (locking host)

Expected Behavior:
-----------------------------
Should succeed live migration with tunneling  (all the disks are remote, worker node is configured for remote storage))

Nova_LiveMigration_4.2
test_LiveMigratonTunnelling_Cinder_boot,_ephem_and_swap_disk_-_live_migration_with_tunnelling_disabled_in_flavor
Tags: p2,regression,nova
Sub-domain = Storage

Testcase Objective:
--------------------------------
Test instantiation from cinder volume, with ephem and swap disk.
Cinder boot instance with ephem and swap disk and perform live migration with tunnelling disabled in flavor
Flavor extra spec hw:wrs:live_migration_tunnel=False

Test Pre-Conditions:
--------------------------
Storage backing Remote

Test Steps:
----------------
1. reate instance where tunnelling is disabled using the flavor extra spec (where the instance is booted from cinder volume, with both ephemn and swap disk)
hw:wrs:live_migration_tunnel=False

2. Test live migration with the tunnelling disabled (locking host)

Expected Behavior:
-----------------------------
The live migrations should succeed with tunnelling disabled

Nova_LiveMigration_4.3
test_LiveMigratonTunnelling_Live_migration_tunnelling_setting_conflict,_instance_metadata_takes_priority_to_disable
Tags: p2,regression,nova
Sub-domain = Storage

Testcase Objective:
--------------------------------
Tunnelling is controlled only by flavor extra-specs and instance metadata, not image properties.
Test live migration tunnelling setting conflicts (instance metadata takes priority on boot)
ie. test conflict scenario where encryption tunnelling is enabled in the flavor extra spec but disabled in the instance metadata on boot.
Instance metadata takes priority to disable tunnelling.

flavor key <flavorid> set hw:wrs:live_migration_tunnel=True
AND
instance metadata hw:wrs:live_migration_tunnel=False

Test Pre-Conditions:
--------------------------


Test Steps:
----------------
1. On instantiation, select a flavor that has tunneling enabled and instance metadata that has it disabled.
flavor key <flavorid> set hw:wrs:live_migration_tunnel=True
AND
instance metadata hw:wrs:live_migration_tunnel=False

Expected Behavior:
-----------------------------
The instance metadata will take effect to disable tunnelling.
Review top output from libvirtd to confirm tunneling is disabled (should be lower %CPU on migration when not tunnelling).


Nova_LiveMigration_4.4
test_LiveMigratonTunnelling_Live_migration_tunnelling_setting_conflict,_instance_metadata_takes_priority_to_enable

Testcase Objective:
--------------------------------
Tunnelling is controlled only by flavor extra-specs and instance metadata, not image properties.

Test Conflict case where encryption tunnelling is disabled in the flavor extra spec but enabled in the instance metadata on boot. Instance metadata takes priority (enables).

flavor key <flavorid> set hw:wrs:live_migration_tunnel=False
AND instance metadata hw:wrs:live_migration_tunnel=True

Test Pre-Conditions:
--------------------------

Test Steps:
----------------
1. On boot of an instance, select a flavor that has tunneling disabled and instance metadata that has it enabled
flavor key <flavorid> set hw:wrs:live_migration_tunnel=False
AND
instance metadata hw:wrs:live_migration_tunnel=True

Expected Behavior:
-----------------------------
The instance metadata will take effect to enable tunnelling.
Review top output from libvirtd to confirm tunneling is enabled (higher %CPU when tunnelling).


Nova_LiveMigration_4.5
test_Live_migration_stress_test_(bandwidth)_-_system_with_mgmt_only_(no_Infra),_storage_CoW_backing
Tags: p2,regression,nova,stress

Testcase Objective:
--------------------------------
Live migration stress test (bandwidth) on a system with mgmt interface only (no Infra), storage CoW backing
Instance booted from volume (no local disks) on host with CoW Backing.
Perform live migration test under memory stress, tunnelling enable

Test Pre-Conditions:
--------------------------
System with Management interface (no infra configured)
Storage local_image backing

Test Steps:
----------------
1. Boot an instance from volume (no local disk).
2. Run live migrations with stress (migrating the instance under memory stress)

Using Stress-ng on each CPU
For example where the instance has 4 VCPUs, run a stress-ng for each vcpu
taskset -c 1 stress-ng --vm 1 --vm-bytes 512M --vm-keep --vm-method swap &
taskset -c 2 stress-ng --vm 1 --vm-bytes 512M --vm-keep --vm-method swap &
taskset -c 3 stress-ng --vm 1 --vm-bytes 512M --vm-keep --vm-method swap &
taskset -c 4 stress-ng --vm 1 --vm-bytes 512M --vm-keep --vm-method swap &

Launch Bogus Operations as well with Stress-ng or Stress
sudo stress-ng --cpu 8 --io 3 --vm 2 --vm-bytes 512M&
stress-ng --fork 4 --fork-ops 100000&

* The stress tests were run with stress-ng with taskset as well as the bogus operations using stress side by side.


Run the following to watch the output of each class, ensuring that the live migration class eg. 1:30 is borrowing
$ watch -n 1 tc -s class show dev <interface>	 Validate the traffic control filters for bandwidth.

Expected Behavior:
-----------------------------
Confirm traffic control filters for bandwidth are in effect. Ensure by watching the class output to ensure the packets are transferred over the migration class eg. 1:30 once live migraiton is triggered and borrowing is observed if under stress condition.
Confirm throughput reporting for eg. nova-compute.log
Migration running for ##.# secs; memory progress <percent>% (<#####MiB> MiB processed, <###MIB> MiB remaining, <####TotalMib> MiB total); data progress <percent>% (<#####MiB> MiB processed, <###MIB> MiB remaining, <####TotalMib> MiB total); memory bandwidth 

#### MB/s;expected downtime ### ms;since ##. seconds ago

Feature Overview:
=================
Nova - Realtime hw:cpu_realtime

Openstack documentation for hw:cpu_realtime is here:
https://specs.openstack.org/openstack/nova-specs/specs/mitaka/implemented/libvirt-real-time.html

Extra-spec validation enforces that hw:cpu_realtime requires hw:cpu_realtime_mask and hw:cpu_policy=dedicated.
Specification of hw:cpu_realtime_mask implies that all vCPUs are first included. The user specifies further vCPU exclusion or inclusions on top of that.
• must have at least 1 RT vCPU
• must have at least 1 normal vCPU
• vCPUs must be in valid range (0 - <n-1>)
• if hw:wrs:shared_vcpu=X is specified, then X must be a subset of the normal vCPUs

The resultant Realtime vCPUS have scheduling policy FIFO with priority 1. This cannot be modified by end-user since it has no bearing on performance.
The resultant "emulatorpin cpuset" should be the pCPUs of the normal (non-realtime) vCPUs.

The linux scheduler policy shows up in field "PO" (of ps-shed.sh)
- "TS" policy means SCHED_OTHER (normal)
- "FF" policy means SCHED_FIFO (fifo)
-  The vCPU TIDs of qemu-kvm process will have the COMM field (thread name) as "x/KVM".

Examples:
• 4 vCPU flavor, implied mask "0-3". Specifying "^0-1" means "0-3,^0-1", resulting in RT vCPUs "2-3", normal VCPUS "0-1".
• 4 vCPU flavor, implied mask "0-3". Specifying "^0,1" means "0-3,^0,1", means "1-3,1", resulting in RT vCPUs "1-3", normal VCPUS "0".

Example fail due to no normal VCPUs
• 4 vCPU flavor, implied mask "0-3". Specifying "2-3" means "0-3,2-3", means "0-3", resulting in RT vCPUs "0-3", normal VCPUS None.


Expected Behavior:
-----------------------------
To verify the hw:cpu_realtime feature, you can look at <cputune>, <emulatorpin>, <vcpusched> XML sections.
The vcpusched tag is optional.

<cputune>
<vcpupin vcpu='0' cpuset='4'/>
<vcpupin vcpu='1' cpuset='5'/>
<vcpupin vcpu='2' cpuset='6'/>
<vcpupin vcpu='3' cpuset='7'/>
<emulatorpin cpuset='4'/>
<vcpusched vcpus='1-3' scheduler='fifo' priority='1'/>
</cputune>

Can verify actual threads linux scheduling and affinity. For example:
compute-0:~# ps-sched.sh | grep -e COMM -e qemu-| cut -c 1-120

PID    TID       PPID S PO NICE RTPRIO PR AFFINITY    P COMM             COMMAND
24913  24913      1 S TS    0      -   20 0x10        4 qemu-kvm         /usr/libexec/qemu-kvm -c 0x00000
24913  24918      1 S TS    0      -   20 0x10        4 qemu-kvm         /usr/libexec/qemu-kvm -c 0x00000
24913  24919      1 S TS    0      -   20 0x10        4 eal-intr-thread  /usr/libexec/qemu-kvm -c 0x00000
24913  24922      1 S TS    0      -   20 0x10        4 CPU              0/KVM       /usr/libexec/qemu-kv
24913  24923      1 S FF    -      1   -2 0x20        5 CPU              1/KVM       /usr/libexec/qemu-kv
24913  24924      1 S FF    -      1   -2 0x40        6 CPU              2/KVM       /usr/libexec/qemu-kv
24913  24925      1 S FF    -      1   -2 0x80        7 CPU              3/KVM       /usr/libexec/qemu-kv


Test Requirements:
===================

Test Cases:
===============

Nova_CPURealtime
test_realtime_CPU_realtime_mask_in_and_outside_valid_range._Error_feedback_when_cpu_realtime_mask_is_not_valid
Tags: p2,regression,nova

Testcase Objective:
--------------------------------
Reason for failure reported if instance fails to schedule (on launch) due to invalid cpu realtime mask setting
Nova_Realtime_3.0.1 CPU realtime mask in and outside valid range. Error feedback when cpu realtime mask is not valid

Test Pre-Conditions:
--------------------------


Test Steps:
----------------
1. Create flavor that is 6 VCPU
and attempt to set extra specs for cpu realtime and mask

hw:cpu_policy 	dedicated
hw:cpu_realtime  yes
hw:cpu_realtime_mask  test partial ranges eg. 3-4, 4-5
or
hw:cpu_policy 	dedicated
hw:cpu_realtime  yes
hw:cpu_realtime_mask  3,4,5

Error: Invalid hw:cpu_realtime_mask '4-5', reason: hw:cpu_realtime_mask (4-5) does not have normal vCPUS defined.
Error: Invalid hw:cpu_realtime_mask '3,4,5', reason: hw:cpu_realtime_mask (3,4,5) does not have normal vCPUS defined.


2. Create flavor that is 3 VCPU
and attempt tp set extra specs for cpu realtime and mask

hw:cpu_policy 	dedicated
hw:cpu_realtime_mask  test individual value eg. 1

Error: Invalid hw:cpu_realtime_mask '1', reason: hw:cpu_realtime_mask (1) does not have normal vCPUS defined.

3. Create flavor that is 6 VCPU
and attempt to set extra specs for cpu realtime and mask

hw:cpu_policy 	dedicated
hw:cpu_realtime  yes
hw:cpu_realtime_mask  outside the range eg. 3-6

Attempt to change the range for the mask to a subset eg.
hw:cpu_realtime_mask 1-5

Error: Invalid hw:cpu_realtime_mask '3-6', reason: hw:cpu_realtime_mask (3-6) must be a subset of vCPUs (0-5).
Error: Invalid hw:cpu_realtime_mask '1-5', reason: hw:cpu_realtime_mask (1-5) does not have normal vCPUS defined.


4. Create flavor that is 3 VCPU
and attemp to set extra specs for cpu realtime and mask to all 3.

hw:cpu_policy 	dedicated
hw:cpu_realtime  yes
hw:cpu_realtime_mask  0,1,2	Verify that the user cannot set this from the cli as well.

Error: Invalid hw:cpu_realtime_mask '0,1,2', reason: hw:cpu_realtime_mask (0,1,2) does not have normal vCPUS defined.

Expected Behavior:
-----------------------------
Error Invalid hw:cpu_realtime_mask (error reason as indicated in the steps)



Feature Overview:
=================
Nova - Guest meta.js
meta.js file exists in guest if instantiation with '--meta' option

Test Requirements:
===================

Test Cases:
===============

Nova_MetaFile
test_meta.js_file_in_the_guest_if_instance_booted_with_--meta_option
Tags: p1,regression,nova

Testcase Objective:
--------------------------------
Ensure guest has meta.js file if instantiation with '--meta' option

Test Pre-Conditions:
--------------------------

Test Steps:
----------------
1.
Instantiate a vm with the option ‘--meta’ and ensure /meta.js file in the guest
eg. Launch instance from centos guest with option --meta

$ source ./openrc.tenant1
$nova boot --flavor <flavor> --image <eg.centos> --nic net-id=<netid> --meta role=webservers <instance_name>

For example:
$ nova boot --flavor medium.dpdk --image tis-centos-guest --nic net-id=ae1d9039-bdf4-4118-9282-b1ea0b03fb74 --meta role=webservers test

2.
Log into the guest and check if there is a file /meta.js in the guest
<instance_name>:~# cd /
<instance_name>:~# ls | grep meta.js
meta/js

Expected Behavior:
-----------------------------
Ensure guest has meta.js file if instantiation with '--meta' option
(note: some customer applications are unable to start without the required meta.js file)


Feature Overview:
=================
Nova - NTFS filesystem and large Windows images

NTFS tools will be available directly on all nodes with this feature.
The device under test will only support USB (and not partition the server disk).

USB formatted with NTFS filesystem must be recognized.
This is required for > 4GB window images (ie. where file size limit exceeds the FAT32 limit of 4GiB)

The tests will confirm that the required rpms are installed in TiS by default so that the ntfs tools (both ntfs-3g and ntfsprogs) are available directly on the nodes.

This HLTP will also include tests as follows:

1) Formatting a newly created partition or USB device with NTFS format
This will be tested on nodes with different personalities ie. worker, controller
- The device eg. USB can be formatted (from a windows system) in NTFS format

2) The NTFS device can be mounted and a large file created/deleted
- Confirm Large file(s) >4GB can be written or deleted from the mount point

3) Label update on the NTFS formatted device
4) Large windows test image file (win2016_cygwin_compressed.qcow2 7.48GB) can be copied to the NTFS mount point
5) Unmount of the NTFS device can be performed (if not busy)
6) Windows image can be imported (glance-image create) and instance can be successfully launched using the glance image created from the file at the NTFS mount point


NTFS rpm tools
In order to support NTFS format the following rpms will be required on the node(s)
ntfs-3g-2017.3.23-1.el7.x86_64.rpm and ntfsprogs-2017.3.23-1.el7.x86_64.rpm
The following file should exist on the node /usr/sbin/mkfs.ntfs

Useful Commands
sudo lsblk -o NAME,FSTYPE,SIZE,MOUNTPOINT,LABEL

$ sudo mount -t ntfs /dev/<device> /media/ntfs
or alternatively
$sudo ntfs-3g /dev/<device> /media/ntfs

$sudo parted -l /dev/<device>
$ df -h /media/ntfs

$sudo umount /media/ntfs

Natively, you cannot store files larger than 4 GB on a FAT file system. The 4 GB barrier is a hard limit of FAT you cannot copy a file that is larger than 4 GiB to any plain-FAT volume.
NTFS is able to support this though

Test creation of large file(s) >4GB to usb drive formatted in NTFS
/create large file(s)
$ dd if=/dev/zero of=testfile bs=1024 count=5120000

Test copy (scp) of large img file eg. windiows file, to the NTFS mount point

<host>:/media/ntfs$ df -h /media/ntfs
Filesystem      Size  Used Avail Use% Mounted on
/dev/sXX#        <size>G   <used>G   <avail>G  <usedperc.>% /media/ntfs

Test Requirements:
===================
USB formatted with NTFS filesystem

Test Cases:
===============

Nova_NTFS_Windows_1.0
test_copy_of_large_image_file_>4GB_file_to/from_NTFS_mount_point_(eg_windows_2016_image)
Tags: p1,regression,nova
Sub-domain = Storage


Testcase Objective:
--------------------------------
Copy large iso (one that would exceed a FAT32 limit)
to/from NTFS mount point

Test Pre-Conditions:
-------------------------

Test Steps:
----------------
1.
mkdir -p /media/NTFS
<host>:/$ sudo mount -t ntfs /dev/sdx# /media/ntfs
Confirm ntfs filesystem type
<host>:/media/ntfs$ sudo  parted -l
...
Number  Start   End     Size    File system  Name                 Flags
.
.
#        29.9GB  75.0GB  45.1GB  ntfs         LVM Physical Volume

Use dd to create a large file (>4GB) or copy large windows image file to the ntfs mount point
eg. large file(s)copied to NTFS mount point
eg. create large file (1.0 GB)
dd if=/dev/zero of=partition bs=1024 count=1024000
eg. create larger file (5.0 GB) with dd
<host>:/media/ntfs$ dd if=/dev/zero of=testfile bs=1024 count=5120000
5120000+0 records in
5120000+0 records out
5242880000 bytes (5.2 GB) copied, 90.0053 s, 58.3 MB/s

2. Use scp to copy large windows 2016 iso file to the NTFS mount point
<host>:/media/ntfs$ scp wrsroot@<ip>:/home/wrsroot/images/[win2016_cygwin_compressed.qcow2 /media/ntfs/win2016_cygwin_compressed.qcow2


Note: host cpu usage alarm appears during scp operation
Per info from Jack "There is no more development on ntfs-3g. It's been bought by Tuxera. Any future performance fixes would only available in commercial version."

set/clear 	100.101	Platform CPU Usage threshold exceeded; threshold: 95%, actual: 99.00%.

3. Remove the large file from the ntfs mount point and confirm the use/Avail space returns.
<host>:/media/ntfs$ df -h /media/ntfs

Expected Behavior:
-----------------------------
1. sudo  parted -l output confirms the ntfs as the file system type.

2. Confirm the filesystem used/available when the large file has been created
compute-0:/media/ntfs$ df -h /media/ntfs
Filesystem      Size  Used Avail Use% Mounted on
/dev/sdX#        42G  7.6G   35G  18% /media/ntfs

3. Confirm the use/Avail space returns when the large file is removed
<host>:/media/ntfs$ df -h /media/ntfs
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda6        42G   66M   42G   1% /media/ntfs


Nova_NTFS_Windows_2.0
test_glance_image-create_the_large_windows_image_on_the_ntfs_mount_point
Tags: p1,regression,nova
Sub-domain = Storage

Testcase Objective:
--------------------------------
Glance image-create the large windows image on the ntfs mount point

Test Pre-Conditions:
--------------------------

Test Steps:
----------------
1. As admin user, use the $glance image-create command to create a windows image
ie. create the glance image using a large windows file copied to an ntfs mount point

$ glance image-create --property os_type=windows --name win_2016wendy --disk-format qcow2 --file /media/NTFS/win2016_cygwin_compressed.qcow2 --container-format bare --visibility public


Expected Behavior:
-----------------------------
The image is created from the large windows file at the mount point
| Property         | Value                                |
| checksum         | c1fb0a12c3dc11c056976fedcd26e6dd     |
| container_format | bare                                 |
| created_at       | 2018-05-07T20:49:03Z                 |
| disk_format      | qcow2                                |
| id               | 332cf194-ad31-4c06-984f-320a512ab7d2 |
| min_disk         | 0                                    |
| min_ram          | 0                                    |
| name             | win_2016wendy                        |
| os_type          | windows                              |
| owner            | ed347b35d6384b8eab70dad27c74c2cf     |
| protected        | False                                |
| size             | 8039104512                           |
| status           | active                               |
| store            | file                                 |
| tags             | []                                   |
| updated_at       | 2018-05-07T20:49:29Z                 |
| virtual_size     | None                                 |
| visibility       | publiC


Nova_NTFS_Windows_3.0
test_nova_boot_windows_image_(ie._boot_from_glance_image_created_from_file_on_ntfs_mount_point)
Tags: p2,regression,nova
Sub-domain = Storage

Testcase Objective:
--------------------------------
Nova instantiates from the windows glance image (ie. can boot from the glance image that is on the ntfs mount point)

Test Pre-Conditions:
--------------------------

Test Steps:
----------------
1. As tenant user (eg. tenant2), use the $nova boot command to boot an instance from the windows image; creating volume eg. size 32 GiB, 4 VCPU, 4GB RAM, Disk 10GB
(ie. Booting from the windows glance image created from the file on the ntfs mount point)
nova boot --flavor <flavorname> <instancename> --nic net-id=<net-id> --image <glanceimagename>

As tenant user, using Horizon, launch an instance directly from a large win2016 glance image (that was created from the NTFS mount point).
Specify a flavor with eg. RAM 4GB, VCPUs 4GB and disk  30GB

2.Attempt to launch an instance directly from the win2016 glance image but with a flavor that has a disk that is too small
eg. using flavor with RAM 2GB, VCPUS 2, Disk 8GB


3.Attempt to launch and instance from a cinder volume created from the windows 2016 image, but the volume size is too small for the image

eg. specify cinder volume size 8GB or 28GB
	Confirm error feedback on instance launch if the volume size specified is too small to launch from the windows image specified.

4.Launch the instance from a cinder volume (created from the windows 2016 glance image) where the volume size is sufficient
for eg.
cinder volume size 32GiB
cinder volume size 29GiB

Expected Behavior:
-----------------------------
1. Confirm the windows instance launches from the glance image if the disk size specified is large enough.


2. Confirm error feedback on instance launch if the disk size specified is too small to launch with the windows image specified.
The feedback indicates that disk is too small
Message     Build of instance <id> aborted: Flavor's disk is too small for requested image. Flavor disk is 8589934592 bytes, image is 31138512896 bytes.
Code    500

3. Horizon reports error 500 error instance is in error state.
Build of instance <id> aborted: Volume <vol_id> creation did not finish after 46 seconds or 16 attempts. Status is error.
Error creating volume. Message from driver: Image 332cf194-ad31-4c06-984

The cinder-volume.log reports the following error.
ERROR oslo_messaging.rpc.server ImageUnacceptable: Image 332cf194-ad31-4c06-984f-320a512ab7d2 is unacceptable: Image virtual size is 29GB and doesn't fit in a volume of size 8GB.

4. Confirm the windows instance launches with cinder if the volume size is large enough to launch.


Feature Overview:
=================
Nova - Hugepages
HugePage configuration is driven through sysinv and apply changes via puppet (reboot and runtime) manifests.

Hugepage replacement will require test coverage to the following areas:
Memory, CPU assignment updates, topology breakdown

System inventory will recalculate per numa resource allocation (and update the DB)
Check on audits in CPU and memory inventory updates to the system
Mounts the hugetlbfs for each memory pages size
Generation and population of various conf file (including compute_extend.conf etc..)
Reboot test (where host cpu or memory changes have already been applied)
Mounts the resctrl for CAT support
Latency constraints for CPU power management

Default memory reservations are unchanged (initial memory allocation is 100% 2M pages)
Sysinv-agent reads from Linux and reports  the memory inventory to conductor.
Upon unlock sysinv recalculated 4K pages based on the remaining available memory
When the manifest is applied, it allocates per numa node huge page values and compute_extend.conf file is generated.
(New changes also include that Nova and vswitch manifest are applied without reloading compute-huge)

Sysinv Memory Audit
Sysinv-agent audits  the current huge pages memory allocations, The interval is still 5 minute as in prevous releases.
Sysinv-agent reads from Linux and reports the memory inventory to conductor.

Config files
Compute_extend.conf  - contains extended nova memory options (managed by puppet)
eg.
compute_vswitch_2M_pages=0,0
compute_vswitch_1G_pages=1,1
compute_vm_4K_pages=<4Knode0,4Knode1>
compute_vm_2M_pages=<2M_node0,2M_node1>
compute_vm_1G_pages=<1G_node0,1G_node1>

Compute_reserved.conf and vswitch.conf updated by puppet only
ie
/etc/nova/compute_reserved.conf
/etc/vswitch/vswitch.conf
/etc/vswitch/pmon/vswitch.conf

(/etc/nova/compute_hugepages_total.conf removed/deprecated
It contained maximum huge pages of each type, total memory. Sysinv will now calculate maximums for each)

New manifest
       - audits CPU and grub config
       - updates the grub
       - sets power management QoS resume latency constraints for CPUs (c-states)
       - mounts hugetlbfs for vswitch, libvirt (after booting)
       - allocates huge pages values per numa node

Test Requirements:
===================
Configuration Lab Requirements
Standard, Duplex, Simplex
(lowlatency hosts, Broadwell/Skylake hosts)


Test Cases:
===============

Nova_HugePage_1.0
test_total_Hugepages_and_Free_Hugepages_using_cat_/sys/devices
Tags: p1,regression,nova
Sub-domain = Sysinv

Testcase Objective:
--------------------------------
This test confirm 2M and 1G Huge page total and Huge page Free values prelaunch, post launch and confirms values after instance deletion.
The test checks the following pre-instance launch, post-instance launch and post-instance deletion to confirm accounting.
1G huge pages Total and free
2M huge pages Total and free
Memory accounting

Test Pre-Conditions:
--------------------------

Test Steps:
----------------
1.
On the controller host where 2M hugepages are configured, for eg. X (24147) on node0, Y (30467) on node 1
ssh to the controller

$ cat /sys/devices/system/node/node*/hugepages/hugepages-2048kB/nr_hugepages
24147
30467
Total 2M huge pages sum = 54614

$ cat /sys/devices/system/node/node*/hugepages/hugepages-2048kB/free_hugepages
24147
30467
Available (free) 2M huge pages sum = 54614

($ cat /proc/meminfo | grep HugePage
HugePages_Total (check against Huge pages Total sum in horizon), and HugePages_Free: (check against hugepages Available sums in horizon)

2. Instance launch test
Launch an instance with 1GB RAM using 2M huge page to the host.
eg. vm-topology "U:memory" on node 1 reports 1024 as expected

Check sys device to confirm the free hugepages has reduced accordingly.
cat /sys/devices/system/node/node*/hugepages/hugepages-2048kB/nr_hugepages (for total)
cat /sys/devices/system/node/node*/hugepages/hugepages-2048kB/free_hugepages (for free)

+Alternatively, after instance launch check HugePages_Free has reduced using meminfo
eg. $ cat /proc/meminfo | grep HugePage
HugePages_Total:   54614
HugePages_Free:    54102

Check using the system host command from the cli.
$system host-memory-list <host>
2M Hugepages (sum of both nodes eg. using vm_hp_total_2M | vm_hp_avail_2M  values)

vm_hp_total_2M nodes (24147 +  30467) = 54614
vm_hp_avail_2M nodes (23635 +  30467) = 54102


The Huge page total and free values can be obtained per numa node using the following command:
Total 2M Huge pages
$cat /sys/devices/system/node/node*/hugepages/hugepages-2048kB/nr_hugepages  (where node* is node0 or node1)
Free 2M Huge pages
$cat /sys/devices/system/node/node*/hugepages/hugepages-2048kB/free_hugepages
Total 1G huge pages
$cat /sys/devices/system/node/node*/hugepages/hugepages-1048576kB/nr_hugepages
(Reduce by 1G for AVS total huge pages)

Free 1G huge pages
$cat /sys/devices/system/node/node*/hugepages/hugepages-1048576kB/free_hugepages


+Alternatively, for 2M hugepages you can change total and free:
sum  HugePages 'Total' using cat /proc/meminfo

$ cat /proc/meminfo | grep Huge
ie. HugePages_Total X +  HugePages_Total  Y

Confirm HugePages "Available"
sum using cat /proc/meminfo  eg. HugePages_Free


3. Instance deletion test
Delete the instance and confirm the 2M HugePage_Free returns to pre-launch values.

cat /sys/devices/system/node/node*/hugepages/hugepages-2048kB/nr_hugepages (for total)
cat /sys/devices/system/node/node*/hugepages/hugepages-2048kB/free_hugepages (for free)

+Aternatively,
$cat /proc/meminfo | grep Huge
HugePages_Total:   54614
HugePages_Free:    54614

Run vm-topology after the instance deletion to confirm returns A:mem_2M, A:mem_1G to prelaunch values

Confirm system memory auditing
$system-host-memory-list <host>
Audit will return vm_hp_avail_2M back to prelaunch value when the instance is deleted.
Audit will return vm_hp_avail_1G back to prelaunch value when the instance is deleted.

Note: The audit runs every 5 minutes so the values may take some time for an update to appear here!

1G huge pages Total (per numa)
Check the 1G Huge pages available (per numa) using cat /sys/devices
cat /sys/devices/system/node/node*/hugepages/hugepages-1048576kB/nr_hugepages
Note: Less 1G (by default) for AVS on each node

1G huge pages Free (per numa)
Check the 1G Huge pages available (per numa) using cat /sys/devices
cat /sys/devices/system/node/node*/hugepages/hugepages-1048576kB/free_hugepages (where node* is node0 and node1)

2M huge pages Total (per numa)
Check the 2M Huge pages available (per numa) using cat /sys/devices
$cat /sys/devices/system/node/node*/hugepages/hugepages-2048kB/nr_hugepages

2M huge pages Free (per numa)
Check the 2M Huge pages available (per numa) using cat /sys/devices
$cat /sys/devices/system/node/node*/hugepages/hugepages-2048kB/free_hugepages


Expected Behavior:
-----------------------------
The accounting is confirmed accurately pre-instantiation, post-instantiation and post-deletion.
The cli/horizon output updates for VM Pages huge pages total and Available accordingly per numa.
Confirm audit runs update at the expected interval and vm-topology accounting updates accordingly.


1. Pre-instance launch
Note: Less 1G (by default) for AVS
1G huge pages Total (per numa)
Check the 1G Huge pages available (per numa) using cat /sys/devices
cat /sys/devices/system/node/node*/hugepages/hugepages-1048576kB/nr_hugepages

1G huge pages Free (per numa)
Check the 1G Huge pages available (per numa) using cat /sys/devices
cat /sys/devices/system/node/node*/hugepages/hugepages-1048576kB/free_hugepages (where node* is node0 and node1)
vm_hp_total_1G =  2 + 1 = 3
vm_hp_avail_1G = 2 + 1 = 3


2. Post instantiation
Check /sys/device output to confirm reducing as expected for free_hugepages
1G huge pages Total (per numa)
Check the 1G Huge pages available (per numa) using cat /sys/devices
cat /sys/devices/system/node/node*/hugepages/hugepages-1048576kB/nr_hugepages
Note: Less 1G (by default) for AVS

1G huge pages Free (per numa)
Check the 1G Huge pages available (per numa) using cat /sys/devices
cat /sys/devices/system/node/node*/hugepages/hugepages-1048576kB/free_hugepages (where node* is node0 and node1)

Check also vm-topology output.
A:mem_1G reduces by 1024
$system-host-memory-list <host>
vm_hp_avail_1G reduced to 1 (from 2) on the respective node.

Note: The audit runs every 5 minutes so the values may take some time for an update to appear here!


3. Instance deletion

1G huge pages Total (per numa)
Check the 1G Huge pages available (per numa) using cat /sys/devices
cat /sys/devices/system/node/node*/hugepages/hugepages-1048576kB/nr_hugepages
Note: Less 1G (by default) for AVS

#. Confirm free_hugepages total returns to pre-launch values.
1G huge pages Free (per numa)
Check the 1G Huge pages available (per numa) using cat /sys/devices
cat /sys/devices/system/node/node*/hugepages/hugepages-1048576kB/free_hugepages (where node* is node0 and node1)

nova hypervisor-show for memory_mb_used_node will reduce 1G hugepages value accordingly "1G": 0}
vm-topology output will show a return of A:mem_1G to prelaunch value.

$system-host-memory-list <host>
vm_hp_avail_1G will return vm_hp_avail_1G back to prelaunch value
vm_hp_total_2M (sum each node)  24147 + 30467
vm_hp_avail_2M (sum each node)  24147 + 30467

Note: The audit runs every 5 minutes so the values may take some time for an update to appear here!
Refer to /sys/devices output and vm_hp_total_1G and vm_hp_avail_1G values in $system-host-memory-list <host>



Nova_HugePages_2.0
test_increase_1G_huge_pages_on_numa_node_0_only_and_unlock_to_apply
Tags: p2,regression,nova
Sub-domain = Sysinv

Testcase Objective:
--------------------------------
On controller and worker nodes, test the increase of 1G huge pages on numa node 0 only. Unlock to apply

Test Pre-Conditions:
--------------------------
2+2
Duplex
Simplex

Test Steps:
----------------
On controller node (with compute subfunction) or worker node

1, Attempt to set 1G huge page of numa 0 to invalid value -1
Value that would exceed available space

2. Test the increase of 1G huge pages on numa node 0 only.
Set 1G huge page of numa 1 to eg
0, 1 or 4



Expected Behavior:
-----------------------------
Where value is invalid eg. -1
Error: Processor 0:VM huge pages 1G must be greater than or equal to zero

Where exceed the max value (without changing 2M)
Error feedback should be reported if exceed the limit
Error: Processor 0:No available space for 1G huge page allocation, max 1G pages: 53
Error: Processor 0:No available space for new settings. Max 1G pages is 21 when 2M is 16137, or Max 2M pages is 15095 when 1G is 24.


Nova_HugePages_2.1
test_decrease_both_memory_AND_update_platform_cpu_assignment_before_unlock
Tags: p2,regression,nova
Sub-domain = Sysinv


Testcase Objective:
--------------------------------
Decrease both memory AND update platform cpu assignment before unlock

Test Pre-Conditions:
--------------------------
2+2, Duplex and Simplex systems

Test Steps:
----------------
1. Lock the host under test
Using Horizon/cli on a host with compute subfunction
reduce 1G hugepages on one (or more) nodes and change the platform cpu assignment
eg. reduce 1G hugepages from 4 to 3

$ system host-memory-modify -1G 3 <host> <processor#>
$ system host-memory-list <host>
vm_hp_total_1G 4
vm_hp_pending_1G 3

2. Unlock the host
Confirm updates to the system list after unlock and check out the system inventory log.


Expected Behavior:
-----------------------------
$ system host-memory-list <host>
$ system host-cpu-list <host> | grep Platform
$ system host-cpu-modify -f platform ... -p# 1 <host>

Logs
$ cat /var/log/sysinv.log | grep host_cpus_modify

This config file populates with logical Cores for each processor for the Platform function (also shown in Horizon "Processor" tab).
/etc/nova/compute_reserved.conf

for eg.
PLATFORM_CPU_LIST="0,..."
COMPUTE_PLATFORM_CORES=("node0:0..")

The compute_reserved.conf file also populates with the total cpu list
eg. number of logical CPU instances available in the system
(this can also be determined from the Processor tab in horizon)

COMPUTE_CPU_LIST="0-X"

$system host-memory-list <host>
$system host-cpu-list <host> | grep platform


Feature Overview:
=================
Nova - Memcached enabled
This feature ensures that memcached has been enabled using systemd and bonds to the management interface (on either IPv4 or IPv6 interfaces). Memcached has been enabled on controllers for AIO (Simplex and Duplex systems) as well as the for standard system. 

Memcached will be able to recover after controller operations such as lock/unlock, swact, reboot and handles degrade conditions. In addition memcached will be able to recover from a process kill.

Test coverage for this feature will verify the basics to ensure the memcached process is running and test the following scenarios:
a.Confirm bond, memached is listening on the mgmt address (IPv4 )

b.Confirm bond, memached is listening on the mgmt address (IPv6 )

c.Confirm bond on multiple systems; memached is listening on the mgmt address on AIO-SX, AIO-DX and standard systems.
Verifies memcached is bound to the tcp interface specified and listens only for TCP.
Memcached is refused connection to the UDP port

d.Confirm sysconfig/memcached memcached configuration with UDP option explicitly set to 0, with parameters as follows:
**PORT**: The default port used by Memcached to run.
**USER**: The user Memcached runs as.
**MAXCONN**: The maximum number of allowed connections to Memcached.
**CACHESIZE**: The cache size for memory.
**OPTIONS**: Set mgmt address (with which memcached will listen with tcp)

e.Checks the memcached version installed

f.Tests some basic functionality with memcached
- Memcached operations such as key add, get, replace, flush etc..
- Memcached Statistics returned - memcached can monitor stats
- view slab classes

g.Test permissions for restart

h.Test recovery on process kill

i.Test memcache bond and running service over various controller operations eg. lock, force lock, unlock, reboot, swact, degrade conditions

Tools
netstat
netcat
...or alternatively you could use telnet
# yum install telnet-server telnet
#rpm -qa | grep telnet

systemctl status
Memcached-tool stats

Test Requirements:
===================
Standard, Duplex, Simplex labs
Mgmt interface that is IPv6, IPv4

Test Cases:
===============

Nova_Memcached_1.0
test_confirm_memcached_listening_address_and_service_is_running_(Simplex)
On an Simplex controller node, confirm memcached configuration has restricted the services listening address and the service is running
Tags: p2,regression,nova,SX
Domain = Networking

Testcase Objective:
--------------------------------
Confirm memcached listening address and service is running on Simplex system

Test Pre-Conditions:
--------------------------


Test Steps:
----------------
1. Confirm sysconfig memcached configured on Simplex system
controller-0:/usr/lib/systemd/system/memcached.service
-rw-r--r--. 1 root root 3276 May 16 02:17 memcached.service

$systemctl status memcached
controller-0:/etc/sysconfig/memcached
-rw-r--r--  1 root root  130 May 16 03:34 memcached

$ cat memcached
PORT="11211"
USER="memcached"
MAXCONN="8192"
CACHESIZE="782"
OPTIONS="-l <mgmt ip> -U 0 -t 16  /var/log/memcached.log 2>&1"


2. Confirm memcached service is active and running on Simplex system port 11211 (-U is 0)

controller-0:/etc/sysconfig$ systemctl status memcached.service
$  systemctl status memcached.service
* memcached.service - memcached daemon
Loaded: loaded (/usr/lib/systemd/system/memcached.service; enabled; vendor preset: disabled)
Active: active (running) since Wed 2018-06-06 14:01:43 UTC; 1h 12min ago
Main PID: 15206 (memcached)
Tasks: 19
CGroup: /system.slice/memcached.service
└─15206 /usr/bin/memcached -p 11211 -u memcached -m 782 -c 8192 -l <mgmtip> -U 0 -t 16  /var/log/memcached.log 2>

3. Confirm bond to Management address of the specific controller matches what is running memcached.service
$systemctl status memcached.service

controller-X$ system host-show controller-X | grep mgmt_ip
| mgmt_ip             | <mgmt ip addr>
(or check the host "interface" in Horizon for the mgmt address )

Expected Behavior:
-----------------------------
memcached is configured, active and running on port 11211 and bonding to the management ip address


Nova_Memcached_2.0
Tags: p2,regression,nova,SX
Domain = Networking

Description
Reboot the AIO-SX controller and confirm that the memcache server is started


Nova_Memcached_3.0
test_lock/unlock_AIO-SX_controller_node_and_confirm_memcached_service_starts
Tags: p2,regression,nova,SX
Domain = Networking

Testcase Objective:
--------------------------------
When the controller node of the Simplex system unlocks the memcached service starts

Test Pre-Conditions:
--------------------------

Test Steps:
----------------
1. Lock or force lock the server
Confirm the service is running when the host is in Locked Disabled Online state

2. Unlock the host
$system host-unlock controller-X
controller_config[8067]: /etc/init.d/controller_config: Running puppet manifest apply
[   53.851487] controller_config[8067]: Applying puppet controller manifest...
...[[32m  OK  [0m] Started memcached daemon.
Starting memcached daemon...	Pass

Expected Behavior:
-----------------------------
After host is unlock the service is active and running and listening on the port (tcp only)

controller-X:$systemctl status memcached
memcached.service - memcached daemon
Loaded: loaded (/usr/lib/systemd/system/memcached.service; disabled; vendor preset: disabled)
Active: active (running) since Thu 2018-05-17 19:35:49 UTC; 8min ago
Main PID: 155920 (memcached)
Tasks: 91
CGroup: /system.slice/memcached.service
└─155920 /usr/bin/memcached ....

$ sudo netstat -plunt | grep  11211


Nova_Memcached_4.0
test_lock/unlock_AIO-DX_SB_controller_node_and_confirm_memcached_service_starts
Tags: p2,regression,nova
Domain = Networking

Description

Testcase Objective:
--------------------------------
When Duplex standby controller node unlocks the memcached service is started

Test Pre-Conditions:
--------------------------

Test Steps:
----------------
1. Lock or force lock the server
Confirm the service is running when the host is in
Locked Disabled Online state

2. Unlock the host
$system host-unlock controller-X
controller_config[8067]: /etc/init.d/controller_config: Running puppet manifest apply
[   53.851487] controller_config[8067]: Applying puppet controller manifest...

...[[32m  OK  [0m] Started memcached daemon.
Starting memcached daemon...

Check that the service is active and running and listening on the port  (tcp only)

Expected Behavior:
-----------------------------
The service is active and running and listening on the port (tcp only)
controller-X:$systemctl status memcached
memcached.service - memcached daemon
Loaded: loaded (/usr/lib/systemd/system/memcached.service; disabled; vendor preset: disabled)
Active: active (running) since Thu 2018-05-17 19:35:49 UTC; 8min ago
Main PID: 155920 (memcached)
Tasks: 91
CGroup: /system.slice/memcached.service
└─155920 /usr/bin/memcached ....

$ sudo netstat -plunt | grep  11211
Password:
tcp        0      0 <mgmt_address>:11211     0.0.0.0:*    LISTEN      150825/memcached


Nova_Memcached_3.0
test_lock/Unlock_standard_system_SB_controller_node_and_confirm_memcached_service_is_started
Tags: p2,regression,nova
Domain = Networking

Testcase Objective:
--------------------------------
When standard system standby controller node unlocks the memcached service is started

Test Pre-Conditions:
--------------------------

Test Steps:
----------------
1. Lock or force lock the server
Confirm the service is running when the host is in
Locked Disabled Online state

2. Unlock the host
$system host-unlock controller-X
controller_config[8067]: /etc/init.d/controller_config: Running puppet manifest apply
[   53.851487] controller_config[8067]: Applying puppet controller manifest...
...[[32m  OK  [0m] Started memcached daemon.
Starting memcached daemon...	Pass

tested also with force lock operation

3. Check that the service is active and running and listening on the port (tcp only)

Expected Behavior:
-----------------------------
The service is active and running and listening on the port (tcp only) on the standard system standby controller.

controller-X:$systemctl status memcached
memcached.service - memcached daemon
Loaded: loaded (/usr/lib/systemd/system/memcached.service; disabled; vendor preset: disabled)
Active: active (running) since Thu 2018-05-17 19:35:49 UTC; 8min ago
Main PID: 155920 (memcached)
Tasks: 91
CGroup: /system.slice/memcached.service
└─155920 /usr/bin/memcached ....

$ sudo netstat -plunt | grep 11211
Password:


Nova_Memcached_4.0
test_lock/unlock_AIO-DX_active_controller_node_and_confirm_memcached_service_is_started


Testcase Objective:
--------------------------------
When Duplex active controller node unlocks the memcached service is started

Test Pre-Conditions:
--------------------------

Test Steps:
----------------
1. Lock or force lock the server
Confirm the service is running when the host is in
Locked Disabled Online state

2. Unlock the controller host
$system host-unlock controller-X
controller_config[8067]: /etc/init.d/controller_config: Running puppet manifest apply
[   53.851487] controller_config[8067]: Applying puppet controller manifest...

...[[32m  OK  [0m] Started memcached daemon.
Starting memcached daemon...	Pass

Check that the service is active and running and listening on the port (tcp only)

Expected Behavior:
-----------------------------
The service is active and running and listening on the port (tcp only) on the duplex system active controller.

controller-X:$systemctl status memcached
memcached.service - memcached daemon
Loaded: loaded (/usr/lib/systemd/system/memcached.service; disabled; vendor preset: disabled)
Active: active (running) since Thu 2018-05-17 19:35:49 UTC; 8min ago
Main PID: 155920 (memcached)
Tasks: 91
CGroup: /system.slice/memcached.service
└─155920 /usr/bin/memcached ....

$ sudo netstat -plunt | grep 11211
Password:


Nova_Memcached_5.0
test_swact_confirm_memcached_service_listens_on_tcp_(port_11211)_on_each_controller
Tags: p2,regression,nova
Domain = Networking

Testcase Objective:
--------------------------------
Perform Swact from the active controller node and confirm memcached is listening and service is running after the swact operation.
On a system with more than one controller node, perform a swact and confirm that the memcached is listening on tcp on each controller

$sudo netstat -tulpn | grep 11211 (or  $ sudo netstat -pan | grep 11211)

Test Pre-Conditions:
--------------------------


Test Steps:
----------------
1. On standard system or duplex system (ie. a system with more than one controller node) perform a swact operation.

2. Confirm that the memcached is listening once again on the tcp (port 11211 or whatever was configured)
ie. on both controllers run the following:
$ sudo netstat -tulpn | grep :11211
Password:
tcp        0      0 <mgmt_ip>:11211     0.0.0.0:*               LISTEN      <PID>/memcache

ps -ef | grep mem

memcach+ <PID>      1  0 08:59 ?        00:00:02 /usr/bin/memcached -p 11211 -u memcached -m 782 -c 8192 -l <mgmt_ip> -U 0 -t 20 ...

where
-p is the port
-u is the user
-m is the cachesize
-c is the maxconn

Expected Behavior:
-----------------------------
The service is active and running and listening on the port (tcp only) on the duplex system (on both controllers) after swact operation.
controller-X:$systemctl status memcached



Feature Overview:
=================
Nova - Gnocchi Storage Backend for Ceilometer

Gnocchi storage backend has been integrated, for ceilometer.
Gnocchi will be used for both Standard Configurations and AIO Configurations. Changes to puppet, new logs have also been included. Ceilometer storage backend cleanup (ie. removal of the deprecated ceilometer database) as part of this feature.

Test will include the following areas:
Ceilometer cleanup
Deprecation - of ceilometer
- cli command 'ceilometer meter-list'
- ceilometer service, and the system command for modifying the metering_time_to_live value (for service ceilometer)

Gnocchi integration
Test new files and various configuration updates
New Log File
+ Logs will report on the actual time spent to process X metrics
+ Collect all includes new logs

Test new vswitch metrics (for vswitch engine, interface and port)
Tests new instance metrics for vcpu_util
+tests existing instance metrics
+tests existing compute.node metrics


Resource and Metrics
Openstack commands for getting and showing resources and metrics
Openstack command for getting metric measures (for instances, images, volumes,
snapshots and workers)
+Metric measures for vcpu, ram, boot time and instance cpu related measures
+Metric measures for CPU related meters from the worker host
Openstack command to get instance metric resource history

Aggregation
Openstack command for getting aggregation of more than one metric measure

Benchmark Measures
Openstack Benchmarking measure commands
+openstack metric benchmark measures add, show
+openstack metric benchmark metric create, show (with optional --verbose mode)

Metric measures for vswitch engine resource under traffic
Metric measures for instance metrics while instance under stress, resize
Metric resource for deleted items (instance or image)

Metric processing status

Archive Policy and Rules
Openstack commands for archive policy, archive policy rules

Post swact gnocchi functional


Test Requirements:
===================
Configuration Lab Considerations
Controllers on Standard, AIO Duplex and Simplex, Regions configuration

Distributed cloud subcloud
 +metrics/measures can be obtained if on subcloud
 +metric can not be obtained if on system controller

Storage backends
+ File only currently supported by this feature

Test Cases:
===============

Nova_Metrics_1.0
test_get_openstack_metric_list_and_measures_show_on_lab_configured_with_https
Tags: p1,regression,nova

Testcase Objective:
--------------------------------
Confirm metrics are listed and shown for the following on a system that has been configured with https
cpu, cpu_util and vcpu_util

Test Pre-Conditions:
--------------------------

Test Steps:
----------------
1. List metric and show metric resource (https system)
On a system that is configured with https, ensure that the instance lists and shows metrics vcpu_util, cpu and cpu_util

$openstack metric list | grep cpu_util
$openstack metric list | grep cpu
$openstack metric list | grep cpu_util

Alternatively confirm that the metrics for the following are listed with resource show.

$ openstack metric resource show <resource_id>

metrics
compute.instance.booting.time: 3e216260-5bb9-41e6-a2ab-5e21f9ff6833
disk.ephemeral.size: 308bde8f-bfd1-41c0-bc1b-f5addf7d5ad4
disk.root.size: 0b31fa62-c7c6-473a-9909-1cabdac3b8e8
memory: ce92c2f3-a8ed-4caf-847e-20485d996f0e
vcpus: e480a616-e065-4d89-9a0d-f8c907a257fa
etc..
plus... cpu, cpu_util, cpu_util


2. Show metric measure
With the respective instance metric id for cpu, cpu_util, cpu_util, show the metric measure (on a system with https configured )

As admin user on controller, get the cpu metric measure by metric id
eg. $ openstack metric measures show <metric_id>
$ openstack metric measures show <metric _id vcpu_util>
$ openstack metric measures show <metric _id cpu_util>
$ openstack metric measures show <metric _id cpu>


Expected Behavior:
-----------------------------
Metrics can be listed, shown and metric measures can be shown on a sytem that has https configuration


Nova_Metrics_2.0
test_confirm_metric_measure_aggregation_values_obtained-swact
Tags: p2,regression,nova


Testcase Objective:
--------------------------------
Aggregate 2 or more instance metric measures of interest
eg. vcpu_util and vcpus and memory, image.size
Aggregate instance metrics over a specified time period.
Aggregate metrics pre and post-swact (ie. where other controller has become the active controller)

Confirm that metric measure aggregation values can be obtained for
eg. more than one volume.size or memory (ram) value

Test Pre-Conditions:
--------------------------

Test Steps:
----------------
1 .Get the metric id for 2 or more metrics of interest related to the instance
$ openstack metric resource show <resource_id for the instance>

2. Get the aggregate of 2 or more instance metric measures of interest
eg. vcpu_util and vcpus and memory

3. Measure the aggregation metric
$ openstack metric measures aggregation -m 70d54ddd-4646-4cd5-b641-5deaf7a930d0 83ffb98a-1582-4cf9-8cbd-69a80d278f8e
+---------------------------+-------------+---------------+
| timestamp                 | granularity |         value |
+---------------------------+-------------+---------------+
| 2018-08-20T20:40:00+00:00 |       300.0 | <value> |


Alternatively,
1. Create 2 instances as tenant1 (one with ram 512, one with 1024)
2. Create a 3rd instance as tenant2 with different flavor RAM settings eg. 8192 MB

3. Use aggregration to get the mean memory usage of each of the instances of interest (across different project ids).
openstack metric measures aggregation --resource-type instance -m <memory_metric_id1> ... <memory_metric_id#> --start <dateTtime>  (where the memory_metric_ids are all tenant1 instances)
eg. $ openstack metric measures aggregation --resource-type instance -m af84727f-2add-4061-9ffe-609783d471a2 b16ae5a7-d40d-4b22-9f12-0805025157f0
+---------------------------+-------------+-------+
| timestamp                 | granularity | value |
+---------------------------+-------------+-------+
| 2018-08-20T22:00:00+00:00 |       300.0 | 768.0 |
| 2018-08-20T23:00:00+00:00 |       300.0 | 768.0 |
...
| 2018-08-21T11:00:00+00:00 |       300.0 | 768.0 |
| 2018-08-21T12:00:00+00:00 |       300.0 | 768.0 |
| 2018-08-21T13:00:00+00:00 |       300.0 | 768.0 |

Or

get the aggregation over a specified time period
$ openstack metric measures aggregation --resource-type instance -m af84727f-2add-4061-9ffe-609783d471a2 b16ae5a7-d40d-4b22-9f12-0805025157f0 9d5c9bc6-955b-4792-8abc-2d6da75922c5 --start 2018-08-21T09:00:00 --stop 2018-08-21T12:00:00
+---------------------------+-------------+---------------+
| timestamp                 | granularity |         value |
+---------------------------+-------------+---------------+
| 2018-08-21T09:00:00+00:00 |       300.0 | 3242.66666667 |
| 2018-08-21T10:00:00+00:00 |       300.0 | 3242.66666667 |
| 2018-08-21T11:00:00+00:00 |       300.0 | 3242.66666667 |
+---------------------------+-------------+---------------+

Get the aggregration measure of the vcpu_util metric for several  instances over time.
$ openstack metric measures aggregation --resource-type instance -m <instance vcpuid1> <instance vcpuid2> <instance vcpuid3>
---------------------------+-------------+-----------------+
| timestamp                 | granularity |           value |
+---------------------------+-------------+-----------------+
| 2018-08-20T21:25:00+00:00 |       300.0 | 0.0235137888048 |
| 2018-08-20T21:30:00+00:00 |       300.0 | 0.0200195062736 |
| 2018-08-20T21:35:00+00:00 |       300.0 | 0.0784611890975 |
| 2018-08-20T21:40:00+00:00 |       300.0 |  0.286563917783 |
....
| 2018-08-20T22:30:00+00:00 |       300.0 | 0.0168518238167 |
| 2018-08-20T22:35:00+00:00 |       300.0 |    0.1068479263 |
| 2018-08-20T22:40:00+00:00 |       300.0 |   0.24803859963 |
| 2018-08-20T22:45:00+00:00 |       300.0 |   0.03853666133 |
| 2018-08-20T22:50:00+00:00 |       300.0 | 0.0357943918267 |
..

Attempt to get metric measures aggregates for volume.sizes related to separate instance volumes
eg. Instances have been booted, volume of size 10 (launched at 20:40) and volume of size 2 (launched at 20:23)
$ openstack metric measures aggregation -m 37245995-ef9c-4de2-82c0-f53ad6337e1e  6724d1bd-0cc4-42c8-af0a-898418c20d72

Their aggregation value is reported as 6
$ openstack metric measures aggregation -m 37245995-ef9c-4de2-82c0-f53ad6337e1e  6724d1bd-0cc4-42c8-af0a-898418c20d72
+---------------------------+-------------+-------+
| timestamp                 | granularity | value |
+---------------------------+-------------+-------+
| 2018-08-20T20:45:00+00:00 |       300.0 |   6.0 |
| 2018-08-20T20:55:00+00:00 |       300.0 |   6.0 |
...
| 2018-08-20T21:35:00+00:00 |       300.0 |   6.0 |


Expected Behavior:
-----------------------------
Aggregation measures are returned using the openstack metric measures aggregation command.

[Note: Sometimes, an attempt to get the aggregation without allowing sufficient time, will respond with  "No overlap"
$ openstack metric measures aggregation --resource-type instance -m af84727f-2add-4061-9ffe-609783d471a2 b16ae5a7-d40d-4b22-9f12-0805025157f0
{u'reason': u'No overlap', u'cause': u"Metrics can't being aggregated", u'detail': [[u'af84727f-2add-4061-9ffe-609783d471a2', u'mean'], [u'b16ae5a7-d40d-4b22-9f12-0805025157f0', u'mean']]} (HTTP 400)  ]



Nova_Metrics_3.0
test_confirm_metric_measure_aggregation_values_can_be_obtained_after_a_swact_operation
Tags: p2,regression,nova

Testcase Objective:
--------------------------------
Swact controller and confirm the metric measure aggregation values can be retrieved eg. for resource type image

Test Pre-Conditions:
--------------------------

Test Steps:
----------------
1. Run the metric measures aggregation command to obtain the aggregate prior to swact operation

[wrsroot@controller-1 ~(keystone_admin)]$ openstack metric measures aggregation --resource-type image --metric image.size --query "name='tis-centos-guest'" --needed-overlap 0
+---------------------------+-------------+--------------+
| timestamp                 | granularity |        value |
+---------------------------+-------------+--------------+
| 2018-08-17T13:55:00+00:00 |       300.0 | 2519258752.0 |
+---------------------------+-------------+--------------

2. Swact and repeat running the metric measures aggregation command to obtain the aggregate prior to swact operation

Expected Behavior:
-----------------------------
After swact of the active controller, the metric measure aggregation values can still be retrieved

eg. for resource type image
[wrsroot@controller-1 ~(keystone_admin)]$ openstack metric measures aggregation --resource-type image --metric 57427dff-32b0-4dbb-b366-3c1c1fc1f3dd 8c10afea-912d-4190-bb17-0bc353e56d8a
+---------------------------+-------------+--------------+
| timestamp                 | granularity |        value |
+---------------------------+-------------+--------------+
...
| 2018-08-23T14:15:00+00:00 |       300.0 | 1064379392.0 |


Feature Overview:
=================
Nova - Per cpu low latency power management  --- Refactoring in STX?

An important part of cpu low latency is power management.
Two types of power management states and C-states (idle) and  P-states (runtime).
CPU idle latencies are controlled by processors C-states.  C-states are divided into 2 categories - core and package.
For this feature we will be looking at the C-states (the idle states) that shutdown parts of the processor (individual cores/CPUs) when the cores are unused. The net effect is to reduce power and energy usage.

C0 is the non-idle power state (meaning it is the fully functional state ie. the core is actually executing and not idle).
Two idle C-states -  C1 and C6 - are the per core idle power states. Core-C6 is the deep power conservation state
Once a transition has been made to C1 (Core-C1), power management comes into play.
From C1 state, decisions are made whether to drop into core-C6 state ( based on whether there would be adverse performance effects from restart latency.)

The latency for each C-state can be seen by examining the /sys/devices/system/cpu/cpu*/cpuidle/state*/latency file.

Test considerations
Tests will monitor CPU transitions (to C1 idle_state) on launch on low-latency as well as transition to C1 idle_state on migrations (cold/revert cold/live/block migrations) and return to C6 state on instance deletion.
Tests will confirm cpu C1 idle_state in cpu scaling up/down operations, evacuations (host reboot), resize/revert resize operations.
Numa topology tests will be included and tests will also attempt migration to a host that does not have sufficient vcpus (target host is already full).
In addition, tests will include cpu monitoring on hyperthread enabled host and thread policy interactions.

The tests will using the tools listed below to monitor the cpu power states.
Tools
cpupower monitor - linux tool to report processor frequency and idle statistics
https://lwn.net/Articles/433002/
Using this tool is way easier than crawling /proc/ to see info about your processor and its frequency governor

If the output is too much and you only want to compare specific stats, you can use cpupower monitor -m
eg. sudo cpupower monitor -m "Mperf,Idle_Stats"

turbostat (or turbostat --debug)
Check idling with the turbostat tool to see what is really happening with your CPU. 'turbostat' looks  directly into processor’s MSRs and fetches Power, CPU Frequency, and core/package Idle State information.


Description
On a 2 node system, monitor the idle_stats of the various states of a low latency host, after dedicated instance(s) are booted
cpupower monitor and turbostat are used in monitoring

Instance with dedicated cpus is launched on low latency host and the Idle_Stats (Busy and Mperf) expect to have CPU%c1 roughly 100%


Test Requirements:
===================
Assumptions:
This feature assumes that the host with compute subfunction is also low latency, the CPU assignment has been configured with function VMs and pmqos-static is not running.



Test Cases:
===============

Nova_CPUPowerStates_1.0 -  Refactoring within STX?
test_cpupower_monitor_and_turbostat_monitoring_-_dedicated_instance_booted_on_low_latency_host_(2_node_system)
Tags: regression,nova,DX

Testcase Objective:
--------------------------------
cpupower monitor and turbostat monitoring - dedicated instance booted on low latency host (2 node system)
Where the BIOS is tuned for low latency,the CPU%c1 should be roughly 100%

https://access.redhat.com/sites/default/files/attachments/201501-perf-brief-low-latency-tuning-rhel7-v1.1.pdf

Test Pre-Conditions:
--------------------------
a. hypervisors with eg. Broadwell CPU model
b. 2 node system with Subfunctions: Controller, Compute, Lowlatency
c. standard system - where workers have Subfunctions: Compute, Lowlatency
d. low latency host with Hyperthreading enabled and disabled.

Test Steps:
----------------
1.On 2 node system, low latency host:
Before launching an instance on the low-latency host, run the following to determine the value of CPU%c1 and CPU%c6

$sudo turbotstat

VMs
Processor 0 : 4-17,40-53
Processor 1 : 18-35,54-71

compute-1:~$ sudo turbostat sleep 5
CPU Avg_MHz   %Busy Bzy_MHz TSC_MHz   SMI  CPU%c1  CPU%c3  CPU%c6  CPU%c7 CoreTmp
X       0      0.00   3162    2295     0    0.01    0.00   99.99    0.00      38

2.Launch instance(s) with dedicate cpu, monitoring the cpu (with turbostat)
ie. launch dedicated instance pinned to CPUX, confirming CPU transition C-state after launch.

Get turbotstat output again
Ensure that turbostat output indicates roughly 100% CPU%C1  for cores.

By default, the kernel settles all idle cores in the deepest supported c-state to improve power efficiency.
Ensure that turbostat output indicates CPU%C1 at roughly 100% for the dedicated cpus of the VM

eg. instance with 3 VCPU
CPU Avg_MHz   %Busy Bzy_MHz TSC_MHz SMI CPU%c1 CPU%c3  CPU%c6 CPU%c7 CoreTmp
7  0  0.00  3102  2295 0  100.00 0.00 0.00 0.00 46
43 0  0.01  3237  2295 0   99.99
16 0  0.00  3077  2295 0  100.00    0.00    0.00   0.00      38

Use cpupower-monitor to get a report of the processor topology, frequency and idle power state statistics on the cpu that the instances are dedicated to.

On the low latency controller host where the instance (with dedicated cpu) has been launched, monitor cpu power with
$sudo cpupower monitor	
Nehalem SandyBridge Mperf Idle_Stats
PKG | CORE | CPU |	  C3 | C6 | PC3 | PC6	   || C7 |  PC2  | PC7 || C0   | Cx   | Freq || POLL | C1-H |   C1E- | C3-H |  C6-H
0| 10|   7|  0.00|  0.00|  0.00|  0.00||  0.00|  0.00|  0.00||  0.01| 99.99|  3017||  0.00| 99.99|  0.00|  0.00|  0.00
0| 10|  43|  0.00|  0.00|  0.00|  0.00||  0.00|  0.00|  0.00||  0.07| 99.93|  3194||  0.00| 100.0|  0.00|  0.00|  0.00
0| 26|  16|  0.00|  0.00|  0.00|  0.00||  0.00|  0.00|  0.00||  0.01| 99.99|  2940||  0.00| 100.1|  0.00|  0.00|  0.00

3.Delete the instance from the low latency host, confirming CPU transition C-state on delete.

Expected Behavior:
-----------------------------
C-states change on low-latency host on launch and delete of instance.


Feature Overview:
=================
Nova - PCI Interrupt affinity     Refactoring??

Prior to this user story, the system would blindly affine all external interrupts to the platform cores.
For guests that are using devices (PCI-Passthrough, PCI-SRIOV or Coleto Creek/crypto) this is not ideal as it limits guest throughput and can have latency impacts and the platform is not isolated (not controlling how much interrupt overhead charged to the 

platform core(s))

This feature implements a mechanism to affine these interrupts to the pcpus of the VM's vcpus.
It modifies Nova to pin the interrupts when the instance launches (or other actions are performed that pinning).
This feature will be tested in the context of PCI devices (pci-pt, sriov or coleto creek/crypto) with Nova actions such as instance launch/termination, incoming live migration/outgoing live/block migration, rollback or revert operations etc. Testing will also 

confirm the extension of the resource audit that runs at intervals to determine which PCI devices are available to guests. PCI devices remaining will be affined to the platform pCPUs; interrupts are re-affined back to the platform core on instance termination.

This feature will test the newly introduced flavor extra spec "hw:pci_irq_affinity_mask" that is intended to specify a set of vCPUs from the full set of guest vCPUs (ie. within the standard libvirt CPU range specification syntax).

The test of this feature will look at the sequence of events for affining interrupts in Nova:
From the full set of guest vCPUs, filter based on the set specified in the extra spec and then map the vCPUs to pCPUs, select only pCPUs on the same host as PCI devices, then map the requested host PCI device to the interrupts (setting the smp_affinity)

Testcases will use logs and command to confirm the affinity.
The nova-compute.log will now include an INFO log for requests on the particular instance that reports the PCI IRQ affinity (including the PCI vif_model, pci_address, irqs, numa_node and cpulist)

A new script command is also included to allow querying the pci irq affinity.
sudo nova-pci-interrupts

Note, the IRQ affinity can also be monitored from /proc/interrupts and /proc/irq/64/smp_affinity_list

When hw:pci_irq_affinity_mask is not specified, the default behavior is to select the pcpus matching the PCI's numa node. (This will exclude the shared_vcpu if the extra-spec hw:shared_vcpu=<vcpu> has also been specified.)

In addition, a new field "wrs-res: Pci Devices" can be queried with nova command (such as nova list and nova show) and will output the node(s), address(es) type (eg. VF or PF), vendor and the product code.

nova list --all --fields=wrs-res:pci_devices

Test Requirements:
===================
PCI support

Test Cases:
===============
Nova_PCI_IRQMask_1.0  - Refactoring?
test_nova_cli_output_for_PCI_(SRIOV,_Passthru,_Alias)_includes_new_wrs-res:pci_devices
Tags: p2,regression,nova,pci


Testcase Objective:
--------------------------------
This test confirms that wrs-res:pci_devices property/value output from nova list and nova show commands for instances with SRIOV, Passthrough or PCI Alias configurations.
It also confirms the new script command that allows admin to query the pci irq affinity.
$sudo nova-pci-interrupts


Test Pre-Conditions:
--------------------------

Test Steps:
----------------
1. Test nova list and show with PCI SRIOV/PassThru/Alias
For instance(s) with PCI SRIOV interface, verify the output of the nova show command now includes wrs-res:pci_devices
$nova show <instanceid>

Run the nova list command for all tenants to confirm the new column for pci devices.
$ nova list --all-tenants --fields wrs-res:pci_devices,wrs-res:topology

The nova show command now includes new output wrs-res:pci_devices with the respective property values
This includes the type VF (in the case of SRIOV), address, node in wrs-res:pci_devices Property value.

wrs-if:nics  {"nic2": {"vif_model": "pci-sriov", "network": ....
wrs-res:pci_devices  node:0, addr:0000:06:10.7, type:VF, vendor:8086, product:10ed

2. Test nova list and show with PCI Passthrough
For instance(s) with PCI Passthrough interface, verify the output of the nova show command now includes wrs-res:pci_devices
$nova show <instanceid>

Run the nova list command for all tenants to confirm the new column for pci devices.
$ nova list --all-tenants --fields wrs-res:pci_devices,wrs-res:topology

The nova show command now includes new output wrs-res:pci_devices with the respective property values - including the type PF (for PCI Passthrough), address, node in wrs-res:pci_devices Property value.
The Nova list output will include PCI Devices and type PF in use and the corresponding Topology (per node) for each instance

$nova show <id>
"nic2":
{"vif_model": "pci-passthrough", "network":"internal0-net0", "port_id": "30079f2a-435b-4424-bce7-1291ffadba53", "mtu": 9000, "mac_address": "90:e2:ba:60:c8:bc", "vif_pci_address": ""}} |
| wrs-res:pci_devices                  | node:0, addr:0000:06:00.0, type:PF, vendor:8086, product:10fb


3. Test nova list and show PCI Alias
Determine crypto device name id and vendor id using the following command and configure an instance with PCI Alias
$nova device-list

For instance(s) with PCI Alias interface, verify the output of the nova show command now includes wrs-res:pci_devices
$nova show <instanceid>

Run the nova list command for all tenants to confirm the new wrs-res:pci_devices field and output
$ nova list --all-tenants --fields wrs-res:pci_devices,wrs-res:topology

The coleto creek device is listed and eg. pci_vfs_used has incremented when the instance is configured
$ nova device-list
Device Name                    | Device Id | Vendor Id
Coleto Creek PCIe Co-processor | 0443      | 8086
pci_vfs_configured | pci_vfs_used
64                 | 1

$ nova device-show 0443
..
Host compute-0, compute-1
pci_vfs_configured 32, 32
pci_vfs_used 1, 16

Nova show command includes new output wrs-res:pci_devices with the respective property values:
The appropriate node (eg. node:1), addresses (eg. 12 of them if the extra spec value is qat-vf:12) , type eg. VF, (vendor 8086) and product (0443) is reported in the wrs-res:pci_devices 'Property' value in the nova show output.

Expected Behavior:
-----------------------------
1. The Nova list command output will include PCI Devices and type VF in use and the corresponding Topology (per node) for each instance
For example
wrs-res: Pci Devices
<instanceid>
node:0,
addr:0000:06:10.7,
type:VF,
vendor:8086,
product:10ed

2. For example, for and instance with PCI Passthrough
$nova list --all-tenants --fields wrs-res:pci_devices,wrs-res:topology

wrs-res: Pci Devices
node:0, addr:0000:06:00.0, type:PF, vendor:8086, product:10fb

3. For example, for and instance with PCI Alias
$nova list --all-tenants --fields wrs-res:pci_devices,wrs-res:topology

<instanceid> |
node:1, addr:0000:83:03.1, type:VF, vendor:8086, product:0443
...
node:1, node:1, addr:0000:83:04.0, type:VF, vendor:8086, product:0443

wrs-res: Topology
 node:1,  1024MB, pgsize:2M, 1s,2c,2t, vcpus:0-3, pcpus:16,36,11,31, siblings:{0,1},{2,3}, pol:ded, thr:pre

Nova_PCI_IRQMask_2.0  --- Refactoring?
test_shutdown_and_monitoring_IRQs_affinity_where_instance_configured_with_PCI
Tags: p2,regression,nova,pci,DX


Testcase Objective:
--------------------------------
Test shutdown of instance(s) with PCI SRIOV, Passthrough or Alias, monitoring IRQs affinity and affinity mask.

Test Pre-Conditions:
--------------------------

Test Steps:
----------------
1. As tenant user, shutdown instance (where instance has PCI SRIOV and AVS interface and was booted from glance or cinder)
2. As tenant user, shutdown instance (where instance has PCI Passthrough and AVS interface and was booted from glance or cinder)
3. As tenant user, shutdown instance (where instance has PCI Alias and AVS interface and was booted from glance or cinder)

irq cpulist is not listed when run
sudo nova-pci-interrupts

4. Start each of the instances again and run sudo nova-pci-interrupts
Confirm irq cpu list returned and correct affinity.

Expected Behavior:
-----------------------------
The irq cpu list is returned and affinity appears correct.
eg....
irq:71 cpulist:10,15,30,35
INFO addr:0000:83:04.4 vendor:8086 product:0443 numa_node:1 irq:160 msi_irqs:160 ; 83:04.4 Co-processor: Intel Corporation DH895XCC Series QAT Virtual Function

Nova_PCI_IRQMask_3.0 --- Refactoring?
test_delete_instance_monitoring_IRQs_affinity_decrement_where_instance_had_been_configured_with_PCI

Testcase Objective:
--------------------------------
Delete instances, with PCI SRIOV, Passthrough or Alias and monitor IRQs affinity removal/decrement.

Test Pre-Conditions:
--------------------------
System that support PCI SRIOV interface, Passthrough interface
System that support Crypto/Coleto Creek(PCI Alias)

Test Steps:
----------------
1. Test with SRIOV interface
Delete instance(s) that have been launched with the hw:pci_irq_affinity_mask (where the instance has PCI SRIOV interface and some other interface and was booted from glance or cinder)

Monitor functionality of IRQs affinity removed - looking at decrements in /proc/interrupts (CPUx)
eg. cat /proc/interrupts and view vfio-msix entries:
compute-#:~$ grep -e CPU -e vfio-msix /proc/interrupts | awk '/CPU/ {printf "%4s %5s %5s %5s %s\n", "irq:", $1, $4, $11, "name";} /:/ {printf "%4s %5d %5d %5d %s\n", $1, $2, $5, $12, $23}'

Confirm pci_vfs_used value decrements when the instance (with PCI) has been removed when run $nova providernet-show2
eg. $ nova providernet-show e0fa3ccc-607f-4a9d-9250-8972bfc63193
| Property           | Value
...
| pci_vfs_used       | 0

The nova-compute log reports the IRQ affinity (decrement)
nova-compute.log grep IRQs
"IRQs ..."

2. Test with Passsthrough interface
Delete instance(s) that had been launched with the hw:pci_irq_affinity_mask
(where instance has PCI Passthrough interface and some other interface and was booted from glance or cinder)
The nova-compute log reports the IRQ affinity decrement.

nova-compute.log grep IRQs
"IRQs ..."

Monitor functionality of IRQs affinity removing - looking at where they decrement in /proc/interrupts (CPUx)
Confirm pci_pfs_used decrements when the instance has been removed with pci.
pci_pfs_used       | X-1

3 Test with PCI Alias interface
Delete instance(s) that had been launched with the hw:pci_irq_affinity_mask  (where instance also has Crypto/Coleto Creek(PCI Alias) and was booted from glance or cinder)
The nova-compute log reports the IRQ affinity (decrement)

nova-compute.log grep IRQs
"IRQs ..."

Expected Behavior:
-----------------------------
Monitor functionality of IRQs affinity removing, looking at where they decrement in /proc/interrupts (CPUx) for each scenario.


Nova_PCI_IRQMask_4.0  --- Refactoring?
test_rebuild_instance_and_monitor_IRQs_affinity_where_instance_has_PCI_(SRIOV/PassThru/Alias)

Testcase Objective:
--------------------------------
Test nova "Rebuild" action for instances with PCI.
Rebuild instances and monitor IRQs affinity, irq affinity mask where instances have PCI (SRIOV/PassThru/Alias)

Test Pre-Conditions:
--------------------------

Test Steps:
----------------
1. Rebuild and monitoring IRQs affinity where instance (with dedicated cpu policy) has been configured with PCI SRIOV, and a hw:pci_irq_affinity_mask
Run the following on the host to confirm irq affinity as specified.
<worker>:~$ sudo nova-pci-interrupts

2. Rebuild and monitoring IRQs affinity where instance (with dedicated cpu policy) has been configured with PCI Passthrough , and a hw:pci_irq_affinity_mask
Run the following on the host to confirm irq affinity as specified.
<worker>:~$ sudo nova-pci-interrupts

3. Rebuild and monitor IRQs affinity where instance (with dedicated cpu policy) has been configured with PCI Alias and hw:pci_irq_affinity_mask
<worker>:~$ sudo nova-pci-interrupts

Expected Behavior:
-----------------------------
IRQs affinity and the irq affinity mask for each instance is as expected for each instance with PCI (SRIOV/PassThru/Alias) after a successful rebuild.


Nova_PCI_IRQMask_5.0  --- Refactoring?
test_resume_instance_monitoring_IRQ_affinity_where_instance_has_PCI_(SRIOV/PassThru/Alias)

Testcase Objective:
--------------------------------
Test potential impact of resume operations (following pause/suspend) on PCI IRQ affinity where instance has PCI (SRIOV/PassThru/Alias)

Test Pre-Conditions:
--------------------------

Test Steps:
----------------
1. Suspend (or Pause), resume and monitoring IRQs affinity where an instance has been configured with PCI SRIOV
Verify the guest is resumed successfully.
Monitor functionality of IRQs affining - look at where they increment in /proc/interrupts (CPUx)


2. Suspend (or Pause), resume and monitoring IRQs affinity where instance configured with PCI Passthrough
Verify the guest is resumed successfully.
Monitor functionality of IRQs affining - look at where they increment in /proc/interrupts (CPUx)


3. Suspend (or Pause), resume and monitoring IRQs affinity where instance configured with PCI Alias.
Verify the guest is resumed successfully.
Monitor functionality of IRQs affining - look at where they increment in /proc/interrupts (CPUx)

Expected Behavior:
-----------------------------
IRQ affinity does not change on pause/resume or suspend/resume operations in each case.

