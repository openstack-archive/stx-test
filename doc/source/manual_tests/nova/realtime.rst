--------
Realtime
--------

Openstack documentation for hw:cpu_realtime is here:
https://specs.openstack.org/openstack/nova-specs/specs/mitaka/implemented/libvirt-real-time.html

Extra-spec validation enforces that hw:cpu_realtime requires:

- hw:cpu_realtime_mask and
- hw:cpu_policy=dedicated

Specification of hw:cpu_realtime_mask implies that all vCPUs are first
included. The user specifies further vCPU exclusion or inclusions on top of
that:

- Must have at least 1 RT vCPU.
- Must have at least 1 normal vCPU.
- vCPUs must be in valid range (0 - <n-1>).
- if hw:wrs:shared_vcpu=X is specified, then X must be a subset of the normal
  vCPUs.

The resultant Realtime vCPUS have scheduling policy FIFO with priority 1. This
cannot be modified by end-user since it has no bearing on performance. The
resultant "emulatorpin cpuset" should be the pCPUs of the normal
(non-realtime) vCPUs.

The linux scheduler policy shows up in field "PO" (of ps-shed.sh):

- "TS" policy means SCHED_OTHER (normal)
- "FF" policy means SCHED_FIFO (fifo)
-  The vCPU TIDs of qemu-kvm process will have the COMM field (thread name) as
   "x/KVM".

Examples:

- 4 vCPU flavor, implied mask "0-3". Specifying "^0-1" means "0-3,^0-1",
  resulting in RT vCPUs "2-3", normal VCPUS "0-1".
- 4 vCPU flavor, implied mask "0-3". Specifying "^0,1" means "0-3,^0,1", means
  "1-3,1", resulting in RT vCPUs "1-3", normal VCPUS "0".

Example fail due to no normal VCPUs:

- 4 vCPU flavor, implied mask "0-3". Specifying "2-3" means "0-3,2-3", means
  "0-3", resulting in RT vCPUs "0-3", normal VCPUS None.

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

To verify the hw:cpu_realtime feature, you can look at <cputune>,
<emulatorpin>, <vcpusched> XML sections. The vcpusched tag is optional.

.. code:: sh

   <cputune>
     <vcpupin vcpu='0' cpuset='4'/>
     <vcpupin vcpu='1' cpuset='5'/>
     <vcpupin vcpu='2' cpuset='6'/>
     <vcpupin vcpu='3' cpuset='7'/>
     <emulatorpin cpuset='4'/>
     <vcpusched vcpus='1-3' scheduler='fifo' priority='1'/>
   </cputune>

Can verify actual threads linux scheduling and affinity. For example:

.. code:: sh

   compute-0:~# ps-sched.sh | grep -e COMM -e qemu-| cut -c 1-120

   PID    TID       PPID S PO NICE RTPRIO PR AFFINITY    P COMM
   COMMAND
   24913  24913      1 S TS    0      -   20 0x10        4 qemu-kvm
   /usr/libexec/qemu-kvm -c 0x00000
   24913  24918      1 S TS    0      -   20 0x10        4 qemu-kvm
   /usr/libexec/qemu-kvm -c 0x00000
   24913  24919      1 S TS    0      -   20 0x10        4 eal-intr-thread
   /usr/libexec/qemu-kvm -c 0x00000
   24913  24922      1 S TS    0      -   20 0x10        4 CPU              0/KVM
   /usr/libexec/qemu-kv
   24913  24923      1 S FF    -      1   -2 0x20        5 CPU              1/KVM
   /usr/libexec/qemu-kv
   24913  24924      1 S FF    -      1   -2 0x40        6 CPU              2/KVM
   /usr/libexec/qemu-kv
   24913  24925      1 S FF    -      1   -2 0x80        7 CPU              3/KVM
   /usr/libexec/qemu-kv

----------------
Nova_CPURealtime
----------------

:Test ID: test_realtime_CPU_realtime_mask_in_and_outside_valid_range._Error_feedback_when_cpu_realtime_mask_is_not_valid
:Tags: p2,regression,nova

~~~~~~~~~~~~~~
Test Objective
~~~~~~~~~~~~~~

Reason for failure reported if instance fails to schedule (on launch) due to
invalid cpu realtime mask setting. Nova_Realtime_3.0.1 CPU realtime mask in
and outside valid range. Error feedback when cpu realtime mask is not valid.

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. Create flavor that is 6 VCPU and attempt to set extra specs for cpu
   realtime and mask.

   .. code:: sh

      hw:cpu_policy   dedicated
      hw:cpu_realtime  yes
      hw:cpu_realtime_mask  test partial ranges eg. 3-4, 4-5

   or

   .. code:: sh

      hw:cpu_policy   dedicated
      hw:cpu_realtime  yes
      hw:cpu_realtime_mask  3,4,5


   .. code:: sh

      Error: Invalid hw:cpu_realtime_mask '4-5', reason: hw:cpu_realtime_mask (4-5)
      does not have normal vCPUS defined.
      Error: Invalid hw:cpu_realtime_mask '3,4,5', reason: hw:cpu_realtime_mask
      (3,4,5) does not have normal vCPUS defined.

2. Create flavor that is 3 VCPU and attempt tp set extra specs for cpu
   realtime and mask:

   .. code:: sh

      hw:cpu_policy   dedicated
      hw:cpu_realtime_mask  test individual value eg. 1

   .. code:: sh

      Error: Invalid hw:cpu_realtime_mask '1', reason: hw:cpu_realtime_mask (1) does
      not have normal vCPUS defined.

3. Create flavor that is 6 VCPU and attempt to set extra specs for cpu
   realtime and mask:

   .. code:: sh

      hw:cpu_policy   dedicated
      hw:cpu_realtime  yes
      hw:cpu_realtime_mask  outside the range eg. 3-6

   Attempt to change the range for the mask to a subset e.g.

   .. code:: sh

      hw:cpu_realtime_mask 1-5

   .. code:: sh

      Error: Invalid hw:cpu_realtime_mask '3-6', reason: hw:cpu_realtime_mask (3-6)
      must be a subset of vCPUs (0-5).
      Error: Invalid hw:cpu_realtime_mask '1-5', reason: hw:cpu_realtime_mask (1-5)
      does not have normal vCPUS defined.

4. Create flavor that is 3 VCPU and attemp to set extra specs for cpu
   realtime and mask to all 3.

   .. code:: sh

      hw:cpu_policy   dedicated
      hw:cpu_realtime  yes
      hw:cpu_realtime_mask  0,1,2 Verify that the user cannot set this from the cli
      as well.

   .. code:: sh

      Error: Invalid hw:cpu_realtime_mask '0,1,2', reason: hw:cpu_realtime_mask
      (0,1,2) does not have normal vCPUS defined.

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

Error Invalid hw:cpu_realtime_mask (error reason as indicated in the steps).


