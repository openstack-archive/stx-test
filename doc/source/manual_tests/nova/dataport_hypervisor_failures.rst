================================
DataPort and Hypervisor Failures
================================

Allows VIM to continue migrations from a host on data port failure.
Allow VIM to continue migrations from a host if a migration is rolled back.

The host is expected to degrade and remain online when the data port fails.
On data port failure, live migrations will be triggered. Instances where live
migration fails (rollsback), will go to error state as long as the host
remains online. Other instances will still continue to migrate. The host is
expected to degrade and remain online when the data port fails. (If the host is
e.g. rebooted, the host will go offline and instances will be expected to
evacuate).

VIM disables the hypervisor on the host and any instances that could not be
migrated need to be set to the error state so they can be recovered either by
evacuation or by rebooting/rebuilding them if the host is re-enabled. There is
a new operation state and VIM changes to determine the overall success of the
operation. Nova reports live migration rollback to VIM, and VIM will continue
to attempt migrations for other instances (and not stop migrations from the
host altogether).

Multiple instances can be migrated from the host for several reasons. The
behavior described below is what is expected in each scenario:

1. Host Lock and Orchestration

   a) Host lock: Migration failure/rollback will still cause the host lock
      to fail immediately with the appropriate error provided to the user.
      The instance does not fail.

   b) Orchestration: The migration failure/rollback will still cause the
      orchestration to fail immediately with the appropriate error provided
      to the user. The instance does not fail.

2. Host Force Lock: Reboots the host and evacuates the instances, some
   instances may go to error while the host is being rebooting if they
   could not be evacuated.

3. Host (hypervisor) Disabled or Failed:  VIM disables hypervisor and
   instances that could not be migrated are set to error so they can be
   recovered by evac or rebuild.

4. Data port failure: Data port disconnects will attempt to live migrate
   instances from that host. If the live migration fails for any of the
   instances, subsequent instances should still attempt recovery.

VIM disables the hypervisor and degrades the host in this case and any
instances that could not be migrated need to be set to the error state
so they can be recovered either by evacuation or by reboot/rebuild when
the host is re-enabled. Migration of other instances continue despite
the migration failures or rollbacks.

For example, in the case of:

1. Scheduling filter or policy restrictions: One or more instance cannot be
   evacuated due to some scheduling restriction eg thread policy, shared cpu
   assignment, huge page requirement etc.  Subsequent instances should
   attempt to evacuate from the host
2. Dirty memory: Multiple instances are live migrating, but 1 or more is
   under stress (dirtying memory too fast). Other instances should still
   attempt to evacuate from the host.

Note: Instances that could not be live migrated will NOT be evacuated unless
the host goes offline (eg. rebooted or powered down).

-----------------
Test Requirements
-----------------

- Standard
- Storage
- Duplex

-------------------------
Nova_Failure_DataPort_1.0
-------------------------

:Test ID: test_on_dataport_failure_some_migrations_rolled_back_instance_will_error,_other_instances_continue_migrating_(std._no_infra)
:Tags: p1, regression, nova, no_infra
:Sub-domain: Networking

~~~~~~~~~~~~~~
Test Objective
~~~~~~~~~~~~~~

On data port failure, VIM disables the hypervisor on the host. Data port
failure could be:

A. Cable pull
B. interface disabled on the switch

This test verified that instances continue to migrate on a standard system
with *no infra*. It finds the data port and disables it on the switch,
i.e. from LLDP Neighbour info (or pulls the data cabling). Alarms on the
data port will be raised and hypervisor is expected to become disabled.

Any instance that rollsback to the source host, will go to error state as the
hypervisor is disabled. (It must recover by evacuation if a valid target host
becomes available or reboots when the source host has become enabled again).

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

On a standard controller/compute system (without infra).

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. Boot multiple instances on the same worker node where these instance
   settings may vary. Some instances will expect to be able to live migrate.
   Some instances will not be expected to be able to migrate since
   for example, they can not be scheduled:

   a) affinity server group policy setting.
   b) thread policy require setting.
   c) 1G Hugepages setting.

   Instance with server group policies that can not be migrated go to
   Error state with the appropriate error reason provided).

2. Perform an action which will result in data port failure and subsequent
   hypervisor being disabled on the worker host. i.e. Disable the port on
   the switch:

   a) Check the LLDP Neighbour (see LLDP - Name, Neighbour and System Name that
      corresponds to the hosts provider network):

     .. code:: sh

        sudo vshell -H compute-0 lldp-neighbour-list | port-id | remote-port
        | remote-chassis | mac-address | management-address | rx-ttl
        | system-name | system-description | system-capabilities |

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

In step 2, confirm the data port alarm on the worker node. E.g. data port
failure alarm.

.. code:: sh

   set     300.001 'Data' Port failed.

Confirms instances that can be scheduled, migrate from the worker.

Confirm hypervisor status becomes 'disabled'

.. code:: sh

   $ nova hypervisor-show <id>

state changes from up, status enabled ---> to down, disabled

Host availability state changes to Degraded. Run the following system
command to confirm the host availability state is 'degraded' due to
the data interface alarm:

.. code:: sh

   $ system host-list

Confirm instances that can not schedule, roll back and error. Confirm
details of the error reason are reported in instance details. Note: The
instance(s) that fail in this case will not be evacuated unless the host
goes offline (e.g. reboots or powered down).

Instances with server group policy that can not be migrated go to Error
state with the appropriate error reason provided. For example:

.. code:: sh

   (Code 501) Message No valid host was found. There are not
   enough hosts available. compute-1 (ServerGroupAffinityFilter) not found
   in ...

-------------------------
Nova_Failure_DataPort_2.0
-------------------------

:Test ID: test_dataport_failure,_if_some_migrations_fail/error_(stress),_other_instances_will_continue_to_migrate_(Std.System)
:Tags: p1, regression, nova, stress
:Sub-domain: Networking

~~~~~~~~~~~~~~
Test Objective
~~~~~~~~~~~~~~

This test finds causes a port failure (eg. pull data port or disable on the
switch). It confirms alarms on the data port are raised and hypervisor
becomes disabled. Any instance that could not be migrated will go to error
state as the hypervisor on the source is disabled. (If a valid target host
becomes available the instance recovers or reboots/rebuilds if the source
host becomes enabled again). When the host data port fails and an instance(s)
fail during the migrate attempt (and change to error state), others that can
migrate will continue their migration.

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

Standard system. Instances exist on the host where the data port will be
failed. Only some instances dirtying memory (such that they can't be migrated)

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. Trigger a data port failure so that VIM disables the hypervisor on the
   host. Data port failure could be:

   a) Cable pulled.
   b) Interface disabled on the switch.

  Note: If vshell is used to lock the data port (as follows) that is not
  enough to cause the hypervisor to become disabled.

2. Check for alarms and hypervisor disable state. Alarms on the data port
   will be raised and hypervisor is expected to become disabled. Confirm
   hypervisor status becomes 'disabled' and host degraded.

3. Any instance that could not be migrated will go to error state as the
   hypervisor on the source is disabled. It must be recovered by evacuation
   if a valid target host becomes available or reboot/rebuild if the source
   host becomes enabled again. Others that can migrate will continue their
   migration.

   Some instances dirtying memory (and in theory should be unable to complete
   a migration to the new host ie. auto-converge disabled, live migration max
   downtime is low e.g. 100 - 120, hw:wrs:live_migration_autocoverge disabled
   i.e. false. Confirm instance(s) that could not be live migrated (rollback)
   go to error.

   Others instances are not dirtying memory and should continue to be able to
   schedule and complete migration. Confirm instances that can be scheduled,
   live migrate to another available host.

   Run ping test from the guest VM. The ping will stop temporarily during the
   live migration. Run stress in the guest to dirty pages but not so much
   that the instance will not migrate. The instance is paused to complete the live migration.

4. Confirm the instances that do not migrate recover when the hypervisor is
   enabled again.

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

1. When the host data port fails, the hypervisor becomes disabled.

2. Alarms raised as expected and other instances continue to be migrated to
   the other available worker host:

   .. code:: sh

      275.001 Host<worker> hypervisor is now locked-disabled
      300.001 'Data' Port failed.
      300.002 'Data' Interface failed.
      700.151 Live-Migrate issued by the system against instance <name> owned by
      tenant1 from host compute-0, reason = host component failure
      700.151 Live-Migrate issued by the system against instance <name> owned by
      tenant2 from host compute-0, reason = host component failure

3. When the host data port fails and instances fail during the migrate attempt
   (and error), others that can migrate will continue their migration.
   Some instances were unable to be migrated (and error)

   Note: Under guest stress condition (dirtying memory too quickly) where
   autocoverge is disabled, the instance under stress will be issued live
   migration request but will eventually timeout and fail (3 minutes later for
   example). Then VIM will issue the nextlive migration request. So a delay
   is expected here.

   Others that can migrate will continue their migration.

4. When the data port is restored, confirm evacuation issued on the instances
   that are in error state (ie. to recover when the hypervisor is enabled again):

   .. code:: sh

      700.175 Evacuate issued by the system against instance <name> owned by tenant2
      on host <worker>, reason = host component failure
      700.175 Evacuate issued by the system against instance <name> owned by tenant2
      on host <worker>, reason = host component failure
      ...
      700.181 Reboot (hard-reboot) issued by the system against instance <name>
      owned by tenant2 on host <worker>

---------------------------
Nova_Failure_Hypervisor_3.0
---------------------------

:Test ID: test_where_some_instance(s)_may_rollback_while_others_continue_to_be_migrated_on_hypervisor_failure/disabled_(Duplex)
:Tags: p2, regression, nova, DX

~~~~~~~~~~~~~~~~~~
Testcase Objective
~~~~~~~~~~~~~~~~~~

Test where some instance(s) may rollback while others continue to be migrated
on e.g. on a hypervisor failure (eg. where hypervisor state down; status goes
to disabled).

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

Duplex system.

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. Launch multiple instances on one worker host where the instance has
   particular requirements such as cpu thread policy, shared cpu assignment,
   1G hugepages.
2. Disable the hypervisor of the host where these instances reside (to trigger
   the migration). For example:

   .. code:: sh

      <worker>:~$  ps ax | grep nova
      72507 ?        Ssl   12:24 /usr/bin/python2 /usr/bin/nova-compute
      72647 ?        S      0:00 /usr/bin/python2 /bin/privsep-helper --config-file
      /usr/share/nova/nova-dist.conf --config-file /etc/nova/nova.conf --config-file
      /etc nova/nova-compute.conf --privsep_context os_brick.privileged.default
      --privsep_sock_path 

      /tmp/tmpArG3Fj/privsep.sock

      <worker>:~$ sudo systemctl stop libvirtd.service
      openstack-nova-compute.service

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

In step 2, the hypervisor changes to ‘locked-disabled’,  the host remains in
“Unlocked, Enabled, Available” state. Confirm hypervisor becomes disabled:

   .. code:: sh

      $ nova  hypervisor-list
      ID | Hypervisor hostname | State | Status   |
      <id>  | <host            | down  | disabled

      nova hypervisor-show <host>
      | status                    | down
      | status                    | disabled


Instance(s) that can be migrated, migrate. Instances can not live migrate to
the new host target for valid reason(s) where the hypervisor has been
disabled. For example, instance can not be scheduled on the new target host
for any of the reasons below:

a. The target host is not hyperthreaded but instance has thread policy require

b. The target host does not have shared cpu assignment and the instance
   requires it:
   (NUMATopologyFilter) Shared not enabled for cell 0.

c. the target host does not have available 1G Hugepages:
   (NUMATopologyFilter) Not enough memory:

Confirm instances that can not, rollback. The instance attempts evacuation
(but fail as there is no other host suitable) then eventually reboots to
recover when the hypervisor recovers.

   .. code:: sh

      275.001    Host controller-1 hypervisor is now locked-disabled

      700.001 Instance <instancename> owned by <tenantX> has failed on host <host>

      700.152 Live-Migrate inprogress for instance <instancenameA> from host
      controller-1
      700.175 Evacuate issued by the system against instance <instancenameB> owned
      by <tenantX> on host <host>, reason = host disable action
      700.179 Evacuate failed for instance <instancenameB> on host <host>
      275.001 Host <host> hypervisor is now unlocked-enabled
      ..
      700.186 Reboot complete for instance <instancenameB> now enabled on host
      controller-1
      270.102 Host <host> compute services enabled
