==========================================
Live Migration Auto Converge and Tunneling
==========================================

Migrations timeouts where an application memory is dirty faster than it can be
copied to the destination the migration can not complete before timeout.
The test will verify that the instance is slowed (cannot dirty memory as fast)
in order that live migrations can converged and complete.

Auto converge interact with post copy. Auto-converge will only be used if
live_migration_permit_post_copy=False (or not available) on source/target
host.

Live migration post copy will not be enabled (see Reference as follows for
potential performance impacts:

https://specs.openstack.org/openstack/nova-specs/specs/newton/implemented/auto-live-migration-completion.html)

The compute flag for auto converge in the /etc/nova/nova.conf file must also
be set to True.

Autoconverge is controlled only by flavor extra-specs and instance metadata,
not image properties. The live migration auto converge setting can be enabledo
or disabled in the flavor extra spec or the instance metadata:

   .. code:: sh

      nova flavor-key <flavorid> set
      hw:wrs:live_migration_auto_converge=True {False}

Auto converge can also be enabled or disabled using the instance metadata.

   .. code:: sh

      hw:wrs:live_migration_auto_converge=True {False}

Live Migration Condition: Dirty pages. The auto converge option is enabled
by default. Hypervisor throttles down CPU of an instance during live migration
in case of slow progress due to dirty pages. Tests will confirm that it is
throttled down when autoconverge has been enabled (and not when it is
disabled).

Conflicting settings. If both flavor and instance metadata behavior are
specified, the instance metadata setting is used:

   .. code:: sh

      hw:wrs:live_migration_permit_auto_converge=True {False}

Live Migration Maximum Downtime: Downtime for live migration can now also
be configured in the instance metadata (at boot time or updated after the
instance is running), in addition to the image metadata and flavor. The
default is 500 ms flavor or instance metadata syntax:

   .. code:: sh

      hw:wrs:live_migration_max_downtime=ms
      instance metadata
      hw_wrs_live_migration_max_downtime=ms

The extra spec setting overrides the instance metadata. If both extra spec
and instance metadata are set, the flavor setting takes priority.

Live Migration Timeout: Default timeout value is 180 sec.

   .. code:: sh

      nova flavor-key <id> set hw:wrs:live_migration_timeout=<seconds>

Horizon must also change to reflect new range and context help in eg. "Extra
Spec". New Support for Live Migration Timeout in the Instance Metadata
It is now supported in the image metadata, flavor extra spec and instance
metadata. If set in multiple locations, the *lowest* of the values set is:
used:

   .. code:: sh

      In flavor extra spec           hw:live_migration_timeout=X (seconds)
      In image metadata              hw_live_migration_timeout=Y (seconds)
      In instance metadata           hw:live_migration_timeout=Z (seconds)

Live Migraton Tunnelling: Considerations, live migration tunneling enabled
and disabled (success and failure paths), where tunneling is set in the
flavor or instance metadata. (Confirm tunnelling by examining the CPU usage
of libvirtd (or alternatively checking the TCP port that it uses in live
migraton via tcpdump)):

   .. code:: sh

      hw:wrs:live_migration_tunnel=True {or False}

Autoconverge is controlled only by flavor extra-specs and instance metadata,
not image properties.

Conflicting settings: Where the setting is in both flavor and instance
metadata, the instance metadata setting takes priority:

- i.e. if tunnelling is disabled in the flavor but enabled in the instance
  metadata, the instance will tunnel.

- i.e. if tunnelling is enabled in the flavor, disabled in the instance metadata,
  the instance will not tunnel.

Live migration throughput is observed under multiple system (such as CPE,
Infra only, LACP bonded and a system with only Management interface)
With tunneling enabled in live migration, throughput is reduced
The nova-compute.log may report a value much lower than it would with
tunnelling disable:

- e.g. 11000MB/sec with tunelling vs 600-700 MB/sec with tunnelling enabled. (see
  nova-compute.log)

Live migration stress (bandwidth) test cases use stress-ng tool to stress the
guest. For example, on the Instance based off of TIS-Centos, install stress-ng
packages:

.. code:: sh

   $ yum install util-linux (for taskset)
   $ yum install stress
   $ yum install stress-ng

Using Stress-ng on each CPU,
e.g: for isntance with 4 VCPUs, run a stress-ng for each vcpu

.. code:: sh

   $ taskset -c 1 stress-ng --vm 1 --vm-bytes 512M --vm-keep --vm-method swap &
   $ taskset -c 2 stress-ng --vm 1 --vm-bytes 512M --vm-keep --vm-method swap &
   $ taskset -c 3 stress-ng --vm 1 --vm-bytes 512M --vm-keep --vm-method swap &
   $ taskset -c 4 stress-ng --vm 1 --vm-bytes 512M --vm-keep --vm-method swap &

Launch Bogus Operations as well with Stress-ng or Stress:

   .. code:: sh

      sudo stress-ng --cpu 8 --io 3 --vm 2 --vm-bytes 512M &
      stress-ng --fork 4 --fork-ops 100000 &


- The stress tests were run with stress-ng with taskset as well as the bogus
- operations using stress side by side.

For creating a Larger Dirty Pages Footprint, launch several instances of dd as
follows:

.. code:: sh

   dd if=/dev/zero of=todel bs=1048576 count=8192 &

You do not need to always employ the dd tool as Stress-ng puts a lot more load
on the VMs then dd.

The "auto-converge" feature in qemu is enabled by default. Autoconverge is
controlled by flavor or instance metadata (not image metadata). This test
enables it also in the instance metadata:

.. code:: sh

   hw:wrs:live_migration_auto_converge=True

~~~~~~~~~~~~~~~~~
Test Requirements
~~~~~~~~~~~~~~~~~

2 node system minimum.

.. contents::
   :local:
   :depth: 1

----------------------
Nova_LiveMigration_1.0
----------------------

:Test ID: test_autoconverge_Instance_launch_with_live_migration_auto_converge_enabled_from_instance_metadata
:Tags: p2, regression, nova

~~~~~~~~~~~~~~
Test Objective
~~~~~~~~~~~~~~

Autoconverge is controlled by flavor or instance metadata (not image
metadata). Instance launch with live migration auto converge enabled
from instance metadata:

.. code:: sh

   hw_wrs_live_migration_permit_auto_converge=True

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. Launch instance from cinder volume, setting the auto converge setting to
   True in the instance metadata:

   .. code:: sh

      hw:wrs:live_migration_auto_converge=True

2. Test live migrate under a high workload which will prevented the live
   migration from completing. For example, run a stress tool on the guest
   VM to dirty memory faster than the transfer rate. E.g. run stress-ng tool
   (or perform read/write operations on a large file with dd).

   Monitor with top to confirm 100% usage and perform the live migration.

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

Confirm the auto converge setting is enabled. With auto converge enabled,
the usage should visibly drop down from 100% to allow the migration to
complete, then return back up when the live migration has completed.
The live migration will use auto-converge when it is enabled from the
instance metadata.

The live migration will not be as likely to time out as the guest VM usage is
slowed to prevent dirty rate from exceeding transfer rate.


----------------------
Nova_LiveMigration_2.0
----------------------

:Test ID: test_LiveMigrateMaxDowntime_Live_Migration,_Maximum_Downtime_not_set_defaults_to_500_ms
:Tags: p2, regression, nova

~~~~~~~~~~~~~~
Test Objective
~~~~~~~~~~~~~~

Where the maximum live migration downtime has not been specified anywhere,
the default will be 500 ms. The system default Live Migration Maximum Downtime
is 500 ms if it is not set in the flavor or image meta or instance metadata.

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. Boot an instance where the live migration maximum downtime has not been
   specified anywhere in the flavor or image metadata or instance metadata.

2. Perform live migration timeout test to confirm the default maximum downtime
   of 500 ms.

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

The nova-compute.log will report the following on live migration expected
downtime (ms): 

.. code:: sh

   INFO nova.virt.libvirt.migration [instance: <id>] Increasing downtime
   to X ms after 90 sec elapsed time.

Confirm X does not exceed the expected downtime (on lengthy migrations)
ie. Confirm 500ms is used for the maximum downtime on lengthy live migrations.


----------------------
Nova_LiveMigration_2.1
----------------------

:Test ID: test_LiveMigrateMaxDowntime_Live_Migration,_Maximum_Downtime_set_from_image_metadata
:Tags: p2, regression, nova

~~~~~~~~~~~~~~
Test Objective
~~~~~~~~~~~~~~

Live migration maximum downtime can be configured in the image metadata.
This is the maximum downtime allowed for live migration. The minimum setting
that can be set is 100 ms:

.. code:: sh

   hw_wrs_live_migration_max_downtime=<value in ms>

Live Migration, Maximum Downtime set from image metadata.

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. Set the Live Migration Maximum downtime in the image metadata.

2.As tenant user, create a cinder volume from the image.

3.Run the following to confirm the value is set in the volume_image_metadata.

.. code:: sh

   $ cinder show <volumeid>
   'hw_wrs_live_migration_max_downtime': u'100'
   'hw_wrs_live_migration_permit_auto_converge': u'True'

4. Launch instance from the volume that contains the volume_image_metadata
   setting for live migration Maximum Downtime.
   E.g. 'hw_wrs_live_migration_max_downtime': u'100'.
   Note: The higher he value for allowable downtime, the more likely it will
   converge.

5. Run live migration test under stress to confirm the setting specified from
   the volume_image_metadata is used for the downtime.


~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

nova-compute.log reports the following on live migration
expected downtime (ms):

.. code:: sh

   INFO nova.virt.libvirt.migration [instance: <id>] Increasing downtime
   to X ms after 90 sec elapsed time.

Confirm X does not exceed the downtime set on lengthy migration.

----------------------
Nova_LiveMigration_3.0
----------------------

:Test ID: test_Live_Migration_Timeout_set_in_the_flavor_extra_spec,_cli_validation
:Tags: p2, regression, nova

~~~~~~~~~~~~~~
Test Objective
~~~~~~~~~~~~~~

Live migration timeout valid range is 120 to 800 and 0 to disable. Some parameters
that can effect timeout are as follows:

- Dirtying rate (pages/sec),size of a page (KB), number of memory pages on the
  VM network bandwidth available KB/sec (between source and target), network
  utilization during migration.


~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. Set the following valid live migration timeout values from the cli:

   .. code:: sh

      nova flavor-key <id> set hw:wrs:live_migration_timeout=<time in seconds>
      nova flavor-key <id> set hw:wrs:live_migration_timeout=120
      nova flavor-key <id> set hw:wrs:live_migration_timeout=150
      nova flavor-key <id> set hw:wrs:live_migration_timeout=800
      nova flavor-key <id> set hw:wrs:live_migration_timeout=0

2. Attempt setting the following invalid live migration timeout values from
   the cli:

   .. code:: sh

      nova flavor-key <id> set hw:wrs:live_migration_timeout=<time in seconds>
      nova flavor-key <id> set hw:wrs:live_migration_timeout=-1
      nova flavor-key <id> set hw:wrs:live_migration_timeout=1
      nova flavor-key <id> set hw:wrs:live_migration_timeout=119
      nova flavor-key <id> set hw:wrs:live_migration_timeout=801

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

Confirm error response indicates the correct valid range if the user attempts
setting beyond the valid range from cli:

.. code:: sh

   ERROR (BadRequest): hw:wrs:live_migration_timeout must be 0 or in the range
   120 to 800 (HTTP 400)

----------------------
Nova_LiveMigration_3.1
----------------------

:Test ID: test_Live_MigrateTimeout_set_in_the_flavor_extra_spec,cli/Horizon_validation
:Tags: p2, regression, nova

~~~~~~~~~~~~~~
Test Objective
~~~~~~~~~~~~~~

The live migration timeout range remains as follows:
120 seconds - 800 seconds, and 0

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. Confirm the default value when the Live Migration Timeout extra spec is
   selected in Horizon is 180.

2. Set or edit the following valid values for live migration timeout using
   Horizon:

   .. code:: sh

      nova flavor-key <id> set hw:wrs:live_migration_timeout=<time in seconds>
      nova flavor-key <id> set hw:wrs:live_migration_timeout=120
      nova flavor-key <id> set hw:wrs:live_migration_timeout=150
      nova flavor-key <id> set hw:wrs:live_migration_timeout=800
      nova flavor-key <id> set hw:wrs:live_migration_timeout=0

3. Attempt editing/setting the following invalid live migration timeout values
   from Horizon:

   .. code:: sh

      nova flavor-key <id> set hw:wrs:live_migration_timeout=<time in seconds>
      nova flavor-key <id> set hw:wrs:live_migration_timeout=-1
      nova flavor-key <id> set hw:wrs:live_migration_timeout=1
      nova flavor-key <id> set hw:wrs:live_migration_timeout=119
      nova flavor-key <id> set hw:wrs:live_migration_timeout=801

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

1. Confirm the default value for live migration timeout is 180.
2. Confirm an error response indicates the correct valid range if the user
   attempts setting beyond the valid range.
3. Confirm the context help and field labels in Horizon reflect the default
   (180 seconds).

----------------------
Nova_LiveMigration_3.2
----------------------

:Test ID: test_LiveMigrationTimeout_Defaults_to_180_sec
:Tags: p2, regression, nova

~~~~~~~~~~~~~~
Test Objective
~~~~~~~~~~~~~~

The default value if unspecified is actually 180 sec by default (if it has not
been specified in the flavor, instance meta or image meta).

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. Launch an instance without specifying live migration timeout values in
   either the flavor, image or instance metadata.

2. Test that the default value for live migration timeout is now 180 sec (if
   it has not been specified in the flavor, instance meta or image meta)
   by running live migration under memory stress conditions that would cause live
   migration to timeout.

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

Confirm in nova-compute.log on timeout:
"Live migration not completed after 180 sec"

----------------------
Nova_LiveMigration_3.2
----------------------

:Test ID: test_LiveMigrationTimeout_set_in_only_the_instance_metadata_and_confirm_timeout_value_respected
:Tags: p2, regression, nova, stress

~~~~~~~~~~~~~~
Test Objective
~~~~~~~~~~~~~~

Live Migration Timeout set in only the instance metadata and confirm timeout
value respected.

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. Boot instance from volume live migration timeout set in the instance
   metadata.

   .. code:: sh

      nova meta <id> set hw:wrs:live_migration_timeout=<value>

2. Run stress on guest VM to dirty memory.
3. Attempt live migration.

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

Confirm live migration timeout value set in only the instance metadata is
respected.
