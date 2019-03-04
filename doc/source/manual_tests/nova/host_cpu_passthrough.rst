====================
Host CPU Passthrough
====================

Host CPU Passthrough enabled for a guest (ie., -cpu host).
This feature adds a new VCPU model (“Host-Passthrough”) that will enable the
“host-passthrough” libvirt cpu_mode.

Openstack documentation describes it as follows
"host-passthrough" causes libvirt to tell KVM to passthrough the host CPU with
no modifications. The difference to host-model, instead of just matching
feature flags, every last detail of the host CPU is matched. This gives
absolutely best performance, and can 

be important to some apps which check low level CPU details, but it comes at a
cost on migrations. The guest can only be migrated to an exactly matching host
CPU.


For this feature, the vcpu model 'host-passthrough' can be selected and host
cpu features are exposed to the guest: hw:cpu_model Passthrough.

Confirm the xml for the instance contains:

   .. code:: xml

      <cpu mode='host-passthrough'>
       ....
      </cpu>

Cold migrate, live-migrate and evacuate will be supported.
The VCPU filter will only select hosts with the exact same CPU model of the
source host. Scheduler checks VCpuModelFilter to determine if it can migrate
to the destination.

https://www.berrange.com/posts/2018/06/29/cpu-model-configuration-for-qemu-kvm-on-x86-hosts/

-----------------
Test Requirements
-----------------

Tbd

.. contents::
   :local:
   :depth: 1

-----------------------
Nova_CPUPassthrough_1.0
-----------------------

:Test ID: test_evacuate_instances_with_hw:cpu_model_passthrough_to_new_compute_host, Refactoring
:Tags: p2, regression, nova

~~~~~~~~~~~~~~
Test Objective
~~~~~~~~~~~~~~

Instances with VCPU model passthrough can evacuate to a new host.
Instances can evacuate to a new host where VCPU Model has been set to
'passthrough'.

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. Instances with hw:cpu_model passthrough exist on the host.
2. Run evacuation test and verify hosts land on another available host.
3. Log into the guest that has migrated to the new host and confirm the cpu
   model in the guest:

   .. code:: sh

      $ cat /proc/cpuinfo

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

Confirm the expected cpu model in the guest reported when cat /proc/cpuinfo
cpu model is eg. Intel(R) Xeon(R) CPU E5-2699 V3 @ 2.30GHz.
