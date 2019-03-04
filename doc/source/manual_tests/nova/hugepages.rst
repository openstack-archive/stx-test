=========
Hugepages
=========

HugePage configuration is driven through sysinv and apply changes via puppet
(reboot and runtime) manifests.

Hugepage replacement will require test coverage to the following areas:

- Memory
- CPU Assignment Updates
- Topology Breakdown

System inventory will recalculate per numa resource allocation (and update the
DB):

- Check on audits in CPU and memory inventory updates to the system.
- Mounts the hugetlbfs for each memory pages size.
- Generation and population of various conf file (including compute_extend.conf
  etc..).
- Reboot test (where host cpu or memory changes have already been applied).
- Mounts the resctrl for CAT support.
- Latency constraints for CPU power management.

Default memory reservations are unchanged (initial memory allocation is 100%
2M pages):

- Sysinv-agent reads from Linux and reports  the memory inventory to conductor.
- Upon unlock sysinv recalculated 4K pages based on the remaining available
  memory.
- When the manifest is applied, it allocates per numa node huge page values and
  compute_extend.conf file is generated, (new changes also include that Nova
  and vswitch manifest are applied without reloading compute-huge).

Sysinv Memory Audit:

- Sysinv-agent audits  the current huge pages memory allocations, The interval
  is still 5 minute as in prevous releases.
- Sysinv-agent reads from Linux and reports the memory inventory to conductor.

Config files:

- Compute_extend.conf  - contains extended nova memory options (managed by
  puppet), e.g.:

  - compute_vswitch_2M_pages=0,0
  - compute_vswitch_1G_pages=1,1
  - compute_vm_4K_pages=<4Knode0,4Knode1>
  - compute_vm_2M_pages=<2M_node0,2M_node1>
  - compute_vm_1G_pages=<1G_node0,1G_node1>

Compute_reserved.conf and vswitch.conf updated by puppet only, i.e.:

- /etc/nova/compute_reserved.conf
- /etc/vswitch/vswitch.conf
- /etc/vswitch/pmon/vswitch.conf
- /etc/nova/compute_hugepages_total.conf removed/deprecated

It contained maximum huge pages of each type, total memory. Sysinv will now
calculate maximums for each).

New manifest:

 - Audits CPU and grub config.
 - Updates the grub.
 - Sets power management QoS resume latency constraints for CPUs
   (c-states).
 - Mounts hugetlbfs for vswitch, libvirt (after booting).
 - Allocates huge pages values per numa node.

-----------------
Test Requirements
-----------------

Configuration Lab Requirements: Standard, Duplex, Simplex (lowlatency hosts,
Broadwell/Skylake hosts).

.. contents::
   :local:
   :depth: 1

-----------------
Nova_HugePage_1.0
-----------------

:Test ID: test_total_Hugepages_and_Free_Hugepages_using_cat\_/sys/devices
:Tags: p1, regression, nova
:Sub-domain: Sysinv

~~~~~~~~~~~~~~
Test Objective
~~~~~~~~~~~~~~

This test confirm 2M and 1G Huge page total and Huge page Free values
prelaunch, post launch and confirms values after instance deletion.
The test checks the following pre-instance launch, post-instance launch and
post-instance deletion to confirm accounting:

- 1G huge pages Total and free
- 2M huge pages Total and free
- Memory accounting

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. On the controller host where 2M hugepages are configured, for eg. X (24147) on
   node0, Y (30467) on node 1, ssh to the controller.

   .. code:: sh

      $ cat /sys/devices/system/node/node*/hugepages/hugepages-2048kB/nr_hugepages
      24147
      30467
      Total 2M huge pages sum = 54614

      $ cat /sys/devices/system/node/node*/hugepages/hugepages-2048kB/free_hugepages
      24147
      30467
      Available (free) 2M huge pages sum = 54614

      $ cat /proc/meminfo | grep HugePage

   HugePages_Total (check against Huge pages Total sum in horizon), and
   HugePages_Free: (check against hugepages Available sums in horizon)

2. Instance launch test: Launch an instance with 1GB RAM using 2M huge page
   to the host. E.g. vm-topology "U:memory" on node 1 reports 1024 as
   expected. Check sys device to confirm the free hugepages has reduced
   accordingly, for total:

   .. code:: sh

      $ cat /sys/devices/system/node/node*/hugepages/hugepages-2048kB/nr_hugepages

   For free:

   .. code:: sh

      cat /sys/devices/system/node/node*/hugepages/hugepages-2048kB/free_hugepages

   Alternatively, after instance launch check HugePages_Free has reduced using
   meminfo:

   .. code:: sh

      $ cat /proc/meminfo | grep HugePage
      HugePages_Total:   54614
      HugePages_Free:    54102

   Check using the system host command from the cli:

   .. code:: sh

      $ system host-memory-list <host>

   2M Hugepages (sum of both nodes eg. using vm_hp_total_2M | vm_hp_avail_2M
   values).

   .. code:: sh

      vm_hp_total_2M nodes (24147 +  30467) = 54614
      vm_hp_avail_2M nodes (23635 +  30467) = 54102

   The Huge page total and free values can be obtained per numa node using the
   following commands, total 2M Huge pages where nmode* is node0 or node1:

   .. code:: sh

      $ cat /sys/devices/system/node/node*/hugepages/hugepages-2048kB/nr_hugepages

   Free 2M Huge pages:


   .. code:: sh

      $ cat /sys/devices/system/node/node*/hugepages/hugepages-2048kB/free_hugepages

   Total 1G huge pages, Reduce by 1G for AVS total huge pages:

   .. code:: sh

      $ cat /sys/devices/system/node/node*/hugepages/hugepages-1048576kB/nr_hugepages

   Free 1G huge pages

   .. code:: sh

      $ cat /sys/devices/system/node/node*/hugepages/hugepages-1048576kB/free_hugepages

   Alternatively, for 2M hugepages you can change total and free, sum
   HugePages 'Total' using cat /proc/meminfo, i.e. HugePages_Total X +
   HugePages_Total Y:

   .. code:: sh

      $ cat /proc/meminfo | grep Huge

   Confirm HugePages "Available" sum using cat /proc/meminfo,
   e.g. HugePages_Free.

3. Instance deletion test: Delete the instance and confirm the 2M
   HugePage_Free returns to pre-launch values, for total:

   .. code:: sh

      $ cat /sys/devices/system/node/node*/hugepages/hugepages-2048kB/nr_hugepages

   For free:

   .. code:: sh

      $ cat /sys/devices/system/node/node*/hugepages/hugepages-2048kB/free_hugepages

   Aternatively:

   .. code:: sh

      $ cat /proc/meminfo | grep Huge
      HugePages_Total:   54614
      HugePages_Free:    54614

   Run vm-topology after the instance deletion to confirm returns A:mem_2M,
   A:mem_1G to prelaunch values. Confirm system memory auditing:

   .. code:: sh

      $ system-host-memory-list <host>

   - Audit will return vm_hp_avail_2M back to prelaunch value when the instance is
     deleted.
   - Audit will return vm_hp_avail_1G back to prelaunch value when the instance is
     deleted.

   Note: The audit runs every 5 minutes so the values may take some time for an
   update to appear here!.

   1G huge pages Total (per numa): Check the 1G Huge pages available (per numa)
   using cat /sys/devices:

   .. code:: sh

      cat /sys/devices/system/node/node*/hugepages/hugepages-1048576kB/nr_hugepages

   Note: Less 1G (by default) for AVS on each node.

   1G huge pages Free (per numa): Check the 1G Huge pages available (per numa)
   using `cat /sys/devices`, where node* is node0 and node1.:

   .. code:: sh

      $ cat /sys/devices/system/node/node*/hugepages/hugepages-1048576kB/free_hugepages

   2M huge pages Total (per numa): Check the 2M Huge pages available (per numa)
   using cat /sys/devices:

   .. code:: sh

      $ cat /sys/devices/system/node/node*/hugepages/hugepages-2048kB/nr_hugepages

   2M huge pages Free (per numa): Check the 2M Huge pages available (per numa)
   using cat /sys/devices:

   .. code:: sh

      $ cat /sys/devices/system/node/node*/hugepages/hugepages-2048kB/free_hugepages


~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

The accounting is confirmed accurately pre-instantiation, post-instantiation
and post-deletion. The cli/horizon output updates for VM Pages huge pages total
and Available accordingly per numa. Confirm audit runs update at the expected
interval and vm-topology accounting updates accordingly.

1. Pre-instance launch:

   - Less 1G (by default) for AVS.
   - 1G huge pages Total (per numa): Check the 1G Huge pages available
     (per numa) using cat /sys/devices:

   .. code:: sh

      cat /sys/devices/system/node/node*/hugepages/hugepages-1048576kB/nr_hugepages

   - 1G huge pages Free (per numa): Check the 1G Huge pages available
     (per numa) using cat /sys/devices, where node* is node0 and node1:

   .. code:: sh

      $ cat /sys/devices/system/node/node*/hugepages/hugepages-1048576kB/free_hugepages
      vm_hp_total_1G =  2 + 1 = 3
      vm_hp_avail_1G = 2 + 1 = 3

2. Post instantiation:

   - Check /sys/device output to confirm reducing as expected for
     free_hugepages.
   - 1G huge pages Total (per numa): Check the 1G Huge pages available
     (per numa) using cat /sys/devices:

   .. code:: sh

      $ cat /sys/devices/system/node/node*/hugepages/hugepages-1048576kB/nr_hugepages

     Note: Less 1G (by default) for AVS

   - 1G huge pages Free (per numa): Check the 1G Huge pages available
     (per numa) using cat /sys/devices, where node* is node0 and node1:

   .. code:: sh

      $ cat /sys/devices/system/node/node*/hugepages/hugepages-1048576kB/free_hugepages

   - Check also vm-topology output: A:mem_1G reduces by 1024:

   .. code:: sh

      $ system-host-memory-list <host>

     vm_hp_avail_1G reduced to 1 (from 2) on the respective node.
     Note: The audit runs every 5 minutes so the values may take some time for an
     update to appear here!.

3. Instance deletion

   - 1G huge pages Total (per numa): Check the 1G Huge pages available
     (per numa) using cat /sys/devices, less 1G (by default) for AVS:

   .. code:: sh

      $ cat /sys/devices/system/node/node*/hugepages/hugepages-1048576kB/nr_hugepages

     Confirm free_hugepages total returns to pre-launch values.

   - 1G huge pages Free (per numa): Check the 1G Huge pages available
     (per numa) using cat /sys/devices, where node* is node0 and node1:

   .. code:: sh

      $ cat /sys/devices/system/node/node*/hugepages/hugepages-1048576kB/free_hugepages

   nova hypervisor-show for memory_mb_used_node will reduce 1G hugepages value
   accordingly "1G": 0}. vm-topology output will show a return of A:mem_1G
   to prelaunch value.

   .. code:: sh

      $ system-host-memory-list <host>
      vm_hp_avail_1G will return vm_hp_avail_1G back to prelaunch value
      vm_hp_total_2M (sum each node)  24147 + 30467
      vm_hp_avail_2M (sum each node)  24147 + 30467

   Note: The audit runs every 5 minutes so the values may take some time for an
   update to appear here! Refer to /sys/devices output and vm_hp_total_1G and
   vm_hp_avail_1G values in:

   .. code:: sh

      $ system-host-memory-list <host>

------------------
Nova_HugePages_2.0
------------------

:Test ID: test_increase_1G_huge_pages_on_numa_node_0_only_and_unlock_to_apply
:Tags: p2, regression, nova
:Sub-domain: Sysinv

~~~~~~~~~~~~~~
Test Objective
~~~~~~~~~~~~~~

On controller and worker nodes, test the increase of 1G huge pages on numa
node 0 only. Unlock to apply.

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

- 2+2
- Duplex
- Simplex

~~~~~~~~~~
Test Steps
~~~~~~~~~~

On controller node (with compute subfunction) or worker node

1. Attempt to set 1G huge page of numa 0 to invalid value -1.
   Value that would exceed available space.

2. Test the increase of 1G huge pages on numa node 0 only.
   Set 1G huge page of numa 1 to eg
   0, 1 or 4.

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

- Where value is invalid eg.-1:

  .. code:: sh

     Error: Processor 0:VM huge pages 1G must be greater than or equal to zero

- Where exceed the max value (without changing 2M):

  .. code:: sh

     Error feedback should be reported if exceed the limit
     Error: Processor 0:No available space for 1G huge page allocation, max 1G
     pages: 53
     Error: Processor 0:No available space for new settings. Max 1G pages is 21
     when 2M is 16137, or Max 2M pages is 15095 when 1G is 24.

------------------
Nova_HugePages_2.1
------------------

:Test ID: test_decrease_both_memory_AND_update_platform_cpu_assignment_before_unlock
:Tags: p2, regression, nova
:Sub-domain: Sysinv

~~~~~~~~~~~~~~
Test Objective
~~~~~~~~~~~~~~

Decrease both memory AND update platform cpu assignment before unlock.

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

2+2, Duplex and Simplex systems.


~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. Lock the host under test: Using Horizon/cli on a host with compute
   subfunction reduce 1G hugepages on one (or more) nodes and change
   the platform cpu assignment, e.g. reduce 1G hugepages from 4 to 3.

   .. code:: sh

      $ system host-memory-modify -1G 3 <host> <processor#>
      $ system host-memory-list <host>
      vm_hp_total_1G 4
      vm_hp_pending_1G 3

2. Unlock the host: Confirm updates to the system list after unlock
   and check out the system inventory log.

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

.. code:: sh

   $ system host-memory-list <host>
   $ system host-cpu-list <host> | grep Platform
   $ system host-cpu-modify -f platform ... -p# 1 <host>

Logs:

.. code:: sh

   $ cat /var/log/sysinv.log | grep host_cpus_modify

This config file populates with logical Cores for each processor for the
Platform function (also shown in Horizon "Processor" tab),
/etc/nova/compute_reserved.conf, for e.g.:

.. code:: sh

   PLATFORM_CPU_LIST="0,..."
   COMPUTE_PLATFORM_CORES=("node0:0..")

The compute_reserved.conf file also populates with the total cpu list
eg. number of logical CPU instances available in the system,
(this can also be determined from the Processor tab in horizon):

.. code:: sh

   COMPUTE_CPU_LIST="0-X"
   $ system host-memory-list <host>
   $ system host-cpu-list <host> | grep platform
