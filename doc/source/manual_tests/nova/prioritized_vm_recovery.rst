======================
Prioritized VM Recover
======================

The evacuation order from a failed host can be prioritized using the instance
metadata setting. This will ensure that instances with higher value priority
setting can be recovered first.

Instance metadata will be able to specify the recovery priority in the valid
range 1-10 (where 1 is the highest priority).

The metadata setting will be available in Horizon from the Available Metadata
list for the instance as well as from the cli:

$ nova meta <instance> set sw:wrs:recovery_priority=<value>
  sw:wrs:recovery_priority=1 {2,3,4,5,6,7,8,9,10}

Where all instance priorities differ, instances recover (by recovery audit)
are in the specified order in a multi-host failure scenario where there is no
suitable host remaining.

Disk size and recovery priority combinations are used in recovery decisions in
a host failure (reboot) scenario where priorities, vcpu and memory are the
same but disk sizes differ.

Refer to the start event in the nfv-vim-event.log. Ensure it is in the
relative order expected (as set by the recovery priority instance metadata
setting.

-----------------
Test Requirements
-----------------

Tbd

.. contents::
   :local:
   :depth: 1

-------------------------
Nova_RecoveryPriority_1.0
-------------------------

:Test ID: test_instance_metadata_sw,wrs,recovery_priority_respect_on_host_reboot_-_priority_same,_disk_GB_differs
:Test Title:
:Tags: p1, regression, nova


~~~~~~~~~~~~~~
Test Objective
~~~~~~~~~~~~~~

Disk size and recovery priority combination is used in recovery decision in a
host failure (reboot) scenario where priorities are the same, vcpu and memory
are the same but disk sizes differ.

Instance metadata sw:wrs:recovery_priority respected on host reboot where all
priorities are the same but disk GB size differs.

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. 3 instances created with recovery priority all set to e.g.
   2 VCPUs for each 4 vcpu, memory setting for each 1024. Flavor disk size
   differs as follows:

   - (1) - 10 GB (no swap)
   - (2) - 15 GB (5 swap)
   - (3*) - 20 GB (no swap)

2. Test on host reboot that the priority is based on disk size if all
   instances have the same priority setting for instance metadata
   sw:wrs:recovery_priority.

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

The instance (3*) has the largest disk size so will recover first
i.e. relative recovery will be largest to smallest when all priorities are
equal.

Refer to the start event in the nfv-vim-event.log. Ensure it is in the
relative order expected (as set by the recovery priority instance metadata
setting.

-------------------------
Nova_RecoveryPriority_2.0
-------------------------

:Test ID: test_instancemetadata_sw:wrs:recovery_priority_respected_in_multi_host_failure,_all_priorities_differ
:Test Title:
:Tags: p2,regression,nova

~~~~~~~~~~~~~~~~~~
Testcase Objective
~~~~~~~~~~~~~~~~~~

Where all instance priorities differ, instances recovery (by recovery audit)
in the specified order in a multi-host failure scenario where there is no
suitable host remaining.

Instance metadata sw:wrs:recovery_priority respected in multi host failure,
all priorities differ.


~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. More than 4 instances are created on 2 or more hosts.
   Most of the instances on each host have sw:wrs:recovery_priority set in the
   instance metadata. Some have the same priority set but their flavor size
   differs. (Some are not set and are expected to have the lowest priority
   10).

2. Perform a simultaneous operation on multi-hosts to cause them to fail,
   ensuring each instance, on a per host basis, will be sorted by the recovery
   priority setting (in the instance metadata) when rebuilt (or evacuated to a
   new host).

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

Note: A maximum of 4 instances will recover in parallel (by rebuild or
evacuate). On a multi-host failure, each instance, on a per host basis,
will be sorted by the recovery priority setting (in the instance metadata)
when rebuilding/evacuating the instance.

The instance recovery order is respected. (If there were some priorities the
same, then the instance order are sorted by size (largest to smallest).

Ensure each instance (per source host) is in the relative order expected (as
set by the recovery priority instance metadata setting. (Refer to the start
event in the nfv-vim-event.log.)

Check the nova.compute.log to confirm that the instances migrate in the set
order on multi-host failures based on the sw:wrs:recovery_priority setting
cat /var/log/nova/nova-compute.log | grep nova.compute.resource_tracker | grep
migrat | grep change= or grep "Rebuilding instance".
