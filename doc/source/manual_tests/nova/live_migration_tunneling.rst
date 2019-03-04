=========================
Live Migration Tunnelling
=========================

The nova.conf setting has:

- VIR_MIGRATE_UNDEFINE_SOURCE
- VIR_MIGRATE_PEER2PEER
- VIR_MIGRATE_LIVE
- VIR_MIGRATE_TUNNELLED

in the flag "live_migration_flag".

VIR_MIGRATE_TUNNELLED: With this flag the actual data for migration
will be tunnelled over the libvirtd RPC channel. This requires that
VIR_MIGRATE_PEER2PEER also be set.

Reference:

http://libvirt.org/git/?p=libvirt.git;a=patch;h=fae0da5c1331dc9e5efb6eab345eba1412139136

Note: When the tunneling is enabled, run 'top' on the host during migration to
see heightened usage in libvirtd. (It will be lower if not tunnelling)
Alternatively, tcpdump could be used to capture the live migrations in pcap
file to confirm that migrations are using port 16509 when tunneling is
enabled. Analyze TCP port 16509 (Filter on the port or follow TCP stream).

Logs will also report when the live migration has the tunnel enabled
"libvirt_tunnel enabled for this migration".

Test Requirements
-----------------

2 node system minimum.

.. contents::
   :local:
   :depth: 1

----------------------
Nova_LiveMigration_4.0
----------------------

:Test ID: test_LiveMigratonTunnelling_Cinder_boot,no_ephem_or_swap_-_live_migration_tunnelling_enabled_in_flavor
:Tags: p2, regression, nova

~~~~~~~~~~~~~~
Test Objective
~~~~~~~~~~~~~~

Test live migration tunnelled where instance booted from cinder - no ephem or
swap. Live migration tunnel enabled in flavor. Flavor extra spec:

- hw:wrs:live_migration_tunnel=True

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. Where storage backing is remote: Create instance with tunnelling enabled
   in the flavor extra spec (where the instance is booted from cinder volume,
   no ephem or swap):

   - hw:wrs:live_migration_tunnel=True

   Test live migration with the tunnelling enabled (locking host) - while
   running top on the source worker. Live migration should succeed. Review
   top output from libvirtd to confirm tunneling is enabled.

2. Where storage backing is local_image: Create instance with tunnelling
   enabled in the flavor extra spec (where the instance is booted from cinder
   volume, no ephem or swap)

   - hw:wrs:live_migration_tunnel=True

   Test live migration with the tunnelling enabled (locking host) - while
   running top on the source worker.

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

For both 1 and 2, live migration should succeed. Review top output from libvirtd
to confirm tunneling is enabled (will see higher %CPU when tunnelling has been enabled).

----------------------
Nova_LiveMigration_4.1
----------------------

:Test ID: test_LiveMigratonTunnelling_Cinder_boot,_ephem,swap_disk_-attempt_live_migration,tunnelling_enabled_instance_metadata
:Tags: p2, regression, nova
:Sub-Domain: Storage

~~~~~~~~~~~~~~
Test Objective
~~~~~~~~~~~~~~

- hw:wrs:live_migration_tunnel=True

Where instance is booted from cinder volume, with ephem and swap disk. Cinder
boot instance with ephem, swap disk. Attempt live migration,tunnelling enabled
instance metadata.

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

Storage backing is remote for this test.

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. Create instance where tunnelling is enabled using the instance metadata
   where the instance is booted from cinder volume, with both ephem and
   swap disk settings) and the flavor has:

   - hw:wrs:live_migration_tunnel=True

2. Test live migration with the tunnelling enabled (locking host).

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

Should succeed live migration with tunneling  (all the disks are remote,
worker node is configured for remote storage)).

----------------------
Nova_LiveMigration_4.2
----------------------

:Test ID: test_LiveMigratonTunnelling_Cinder_boot,_ephem_and_swap_disk_-_live_migration_with_tunnelling_disabled_in_flavor
:Tags: p2, regression, nova
:Sub-domain: Storage

~~~~~~~~~~~~~~
Test Objective
~~~~~~~~~~~~~~

Test instantiation from cinder volume, with ephem and swap disk.
Cinder boot instance with ephem and swap disk and perform live migration with
tunnelling disabled in flavor. Flavor extra spec:

- hw:wrs:live_migration_tunnel=False

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

Storage backing Remote.

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. reate instance where tunnelling is disabled using the flavor extra spec
   (where the instance is booted from cinder volume, with both ephemn and swap
   disk).

   - hw:wrs:live_migration_tunnel=False

2. Test live migration with the tunnelling disabled (locking host).

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

The live migrations should succeed with tunnelling disabled.

----------------------
Nova_LiveMigration_4.3
----------------------

:Test ID: test_LiveMigratonTunnelling_Live_migration_tunnelling_setting_conflict,_instance_metadata_takes_priority_to_disable
:Tags: p2, regression, nova
:Sub-domain: Storage

~~~~~~~~~~~~~~
Test Objective
~~~~~~~~~~~~~~

Tunnelling is controlled only by flavor extra-specs and instance metadata, not
image properties. Test live migration tunnelling setting conflicts
(instance metadata takes priority on boot), i.e. test conflict scenario
where encryption tunnelling is enabled in the flavor extra spec but disabled
in the instance metadata on boot. Instance metadata takes priority to disable
tunnelling:

- flavor key <flavorid> set hw:wrs:live_migration_tunnel=True AND
- instance metadata hw:wrs:live_migration_tunnel=False

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. On instantiation, select a flavor that has tunneling enabled and instance
   metadata that has it disabled.

   - flavor key <flavorid> set hw:wrs:live_migration_tunnel=True AND
   - instance metadata hw:wrs:live_migration_tunnel=False

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

The instance metadata will take effect to disable tunnelling.
Review top output from libvirtd to confirm tunneling is disabled (should be
lower %CPU on migration when not tunnelling).

----------------------
Nova_LiveMigration_4.4
----------------------

:Test ID: test_LiveMigratonTunnelling_Live_migration_tunnelling_setting_conflict,_instance_metadata_takes_priority_to_enable

~~~~~~~~~~~~~~
Test Objective
~~~~~~~~~~~~~~

Tunnelling is controlled only by flavor extra-specs and instance metadata, not
image properties.

Test Conflict case where encryption tunnelling is disabled in the flavor extra
spec but enabled in the instance metadata on boot. Instance metadata takes
priority (enables).

- flavor key <flavorid> set hw:wrs:live_migration_tunnel=False AND
- instance metadata hw:wrs:live_migration_tunnel=True

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. On boot of an instance, select a flavor that has tunneling disabled and
   instance metadata that has it enabled:

   - flavor key <flavorid> set hw:wrs:live_migration_tunnel=False AND
   - instance metadata hw:wrs:live_migration_tunnel=True

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

The instance metadata will take effect to enable tunnelling. Review top output
from libvirtd to confirm tunneling is enabled (higher %CPU when tunnelling).

----------------------
Nova_LiveMigration_4.5
----------------------

:Test ID: test_Live_migration_stress_test_(bandwidth)_-_system_with_mgmt_only_(no_Infra),_storage_CoW_backing
:Tags: p2, regression, nova, stress

~~~~~~~~~~~~~~
Test Objective
~~~~~~~~~~~~~~

Live migration stress test (bandwidth) on a system with mgmt interface only
(no Infra), storage CoW backing. Instance booted from volume (no local disks)
on host with CoW Backing. Perform live migration test under memory stress,
tunnelling enable.

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

System with Management interface (no infra configured). Storage local_image
backing.

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. Boot an instance from volume (no local disk).

2. Run live migrations with stress (migrating the instance under memory
   stress). Using Stress-ng on each CPU. For example where the instance
   has 4 VCPUs, run a stress-ng for each vcpu:

   .. code:: sh

      taskset -c 1 stress-ng --vm 1 --vm-bytes 512M --vm-keep --vm-method swap &
      taskset -c 2 stress-ng --vm 1 --vm-bytes 512M --vm-keep --vm-method swap &
      taskset -c 3 stress-ng --vm 1 --vm-bytes 512M --vm-keep --vm-method swap &
      taskset -c 4 stress-ng --vm 1 --vm-bytes 512M --vm-keep --vm-method swap &

   Launch Bogus Operations as well with Stress-ng or Stress:

   .. code:: sh

      sudo stress-ng --cpu 8 --io 3 --vm 2 --vm-bytes 512M&
      stress-ng --fork 4 --fork-ops 100000&

   The stress tests were run with stress-ng with taskset as well as the bogus
   operations using stress side by side.

   Run the following to watch the output of each class, ensuring that the live
   migration class eg. 1:30 is borrowing.

   .. code:: sh

      $ watch -n 1 tc -s class show dev <interface>

   Validate the traffic control filters for bandwidth.

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

Confirm traffic control filters for bandwidth are in effect. Ensure by
watching the class output to ensure the packets are transferred over the
migration class eg. 1:30 once live migraiton is triggered and borrowing is
observed if under stress condition. Confirm throughput reporting for
e.g. nova-compute.log

.. code:: sh

   Migration running for ##.# secs; memory progress <percent>% 
   (<#####MiB> MiB processed, <###MIB> MiB remaining, 
    <####TotalMib> MiB total); data progress <percent>%
   (<#####MiB> MiB processed, <###MIB> MiB remaining,
    <####TotalMib> MiB total); memory bandwidth.
     #### MB/s;expected downtime ### ms;since ##. seconds ago

