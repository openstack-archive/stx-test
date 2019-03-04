==========================
Auto-Convergence Threshold
==========================

-----------------
Test Requirements
-----------------

Minimum 2 node.

--------------------
Nova_Convergence_1.0
--------------------

:Test ID: test_nova_livemigration_autoconverge-_4VCPU,_4GB_RAM,_stress-ng_with_1_per_vcpu_and_2048M,1024M_vm-bytes
:Tags: p2, regression, nova, stress

- Live migration auto-converge threshold (92% throttling max)
- Live migration with autoconverge enabled (multiple supported interfaces)
- While running stress to observe CPU throttling in increments to attempt to
  allow live migration to complete.

~~~~~~~~~~~~~~
Test Objective
~~~~~~~~~~~~~~

Live migrate instance under stress where instance has multiple supported
interfaces to ensure auto-converge is throttling but not starving CPU (max CPU
throttling 92%), stress-ng (running up to 4 in parallel) 2048, 1024M, 512
vm-bytes.


~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. Boot instances with multiple supported interfaces (from cinder):

   - Flavor has 4 VCPU, 4GB RAM, local_image, dedicated cpu policy
     4VCPU, 4GB/6GB RAM
   - 'hw:cpu_policy': 'dedicated'
   - 'hw:mem_page_size': '2048'

   As well, the following instance metadata are set on the instance and auto
   converge settings are enabled (enabled by default):

   - 'hw:wrs:live_migration_max_downtime': 'X'
   - 'hw:wrs:live_migration_timeout': 'X'

2. Run live migrations, monitoring CPU using top (on the source host) to see
   CPU% for each pCPU in the host.  It should throttle each vCPU by (maybe 20%
   initially) then by 10% each step (starting from 99%). CPU should never
   throttle below 8% (ie. max 92 percent throttling).

   .. code:: sh

      $ top -H | grep CPU  (on the host)

   Run stress (not in parallel)

   .. code:: sh

      $ stress-ng --vm 1 --vm-bytes 2048M --vm-keep --vm-method swap &

   The instance-00000000#.log reports the following:

   - initiating migration
   - shutting down

   The libvirt.log reports:

   .. code:: sh

      Migration running for X secs; memory progress XX% (X MiB processed, XX MiB
      remaining, XXX MiB total); data progress X% (XXX MiB processed, XXX MiB
      remaining, XXX MiB total); memory bandwidth XXX MB/s;expected downtime XXX
      ms;since X seconds ago
      ...
      Increasing downtime to X ms after X sec elapsed time
      ...
      VM Paused (Lifecycle Event)
      Migration operation has completed
       _post_live_migration() is started

   CPU increments down roughly 10% each step. Run stress-ng (only 1 vm i.e. not
   running in parallel) with virtual memory stress (paging and memory)
   --vm-bytes 1024M:

   .. code:: sh

      $ top -H | grep CPU  (on the host)

3. Repeat the migration stress test with instance tht has the following
   flavor. Expect in live migration, the CPU will throttle down in increments of
   approx. 10 starting from 99 Migrate completes/converges.

   - 4 VCPU, 4 GB RAM, local_image, dedicated cpu policy
   - auto converge enabled by default

   .. code:: sh

      stress-ng --vm 1 --vm-bytes 1024M --vm-keep --vm-method swap&

   CPU increments down roughly 10% each step. Run stress-ng (run 1 for
   2 of 4 vCPU in parallel) with virtual memory stress (paging and memory)
   --vm-bytes 1024M  to busy multple pCPU on the host. Flavor has 4 VCPU,
   4 GB RAM, local_image, dedicated cpu policy auto converge enabled
   by default.

   Live migrate the instance while watching CPU% in the host

   .. code:: sh

      $ top -H | grep CPU  (on the host)

   .. code:: sh

      stress-ng --vm 1 --vm-bytes 1024M  --vm-keep --vm-method swap &
      stress-ng --vm 1 --vm-bytes 1024M --vm-keep --vm-method swap &

   CPU increments down roughly 10% each step (but does not go below 8%).
   Run stress-ng (1 to 3 out of 4 vCPU in parallel) with virtual memory
   stress (paging and memory) --vm-bytes 1024M

4. Repeat the migration stress test with instance tht has the following
   flavor. Expect in live migration, the CPU will throttle down in increment
   of approx. 10 starting from 99 Migrate completes/converges.

   Flavor has 4 VCPU, 4 GB RAM, 25 root disk, local_image, dedicated cpu
   policy, 2048 mem page size, auto converge enabled by default.

   Live migrate the instance while watching CPU% in the host:

   .. code:: sh

      $ top -H | grep CPU  (on the host)

   .. code:: sh

      stress-ng --vm 1 --vm-bytes 1024M --vm-keep --vm-method swap &
      stress-ng --vm 1 --vm-bytes 1024M --vm-keep --vm-method swap &
      stress-ng --vm 1 --vm-bytes 1024M --vm-keep --vm-method swap &

   CPU increments down roughly 10% each step (but does not go below 8%)
   e.g. CPU% decrements eg. 99 to 80, 70, 60, 50, 40, 30, 20, 12.2, 9.9,
   ~7.9/8.3

   .. code:: sh

      hw:wrs:live_migration_max_downtime=600
      hw:wrs:live_migration_timeout=600

   timeout and downtime set in instance metadata (increased to 600 to see the
   throttling).

   .. code:: sh

      700.154 Live-Migrate cancelled for instance test now on host compute-2, reason
      = Live migration timeout after 600 sec
      ...
      13355 root      20   0 7030552  29588  11092 S  8.3  0.0  17:25.57 CPU 3/KVM
      13354 root      20   0 7030552  29588  11092 S  8.3  0.0  10:31.23 CPU 2/KVM
      13355 root      20   0 7030552  29588  11092 S  8.3  0.0  17:25.82 CPU 3/KVM
      13353 root      20   0 7030552  29588  11092 S  7.9  0.0  19:49.28 CPU 1/KVM

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

When live migration autoconverge is enabled, the CPU is throttled (increments
down roughly 10% each step) but does not go below 8%. This increases the
likelihood of migration successfully completing. The live migration settings
are respected.

--------------------
Nova_Convergence_2.0
--------------------

:Test ID: test_nova_livemigration_autoconverge-_4VCPU,_6GB_RAM,_stress-ng_with_1_per_vcpu_and_512M_vm-bytes
:Tags: p2, regression, nova

~~~~~~~~~~~~~~
Test Objective
~~~~~~~~~~~~~~

Live migrate instance under stress where instance has multiple supported
interfaces to ensure auto-converge is throttling but not starving CPU (max CPU
throttling 92%) stress-ng (running up to 4 in parallel).

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. Boot instances with multiple supported interfaces (from cinder):

   - Flavor has 4 VCPU, 6 GB RAM, local_image,
   - Dedicated cpu policy
   - Auto converge enabled by default

2. Instance metadata optionally set to hw:wrs:live_migration_downtime 180, and
   live migration timeout 180.

3. Run live migrations. Monitor CPU using top (on the source host) to see CPU%
   for each pCPU in the host. It should throttle each vCPU by (maybe 20%
   initially) then by 10% each step (starting from 99%). CPU should never
   throttle below 8% (ie. max 92 percent throttling):

   .. code:: sh

      $ top -H | grep CPU (on the host)

   Stress run 4 in parallel:

   .. code:: sh

      stress-ng --vm 1 --vm-bytes 512M --vm-keep --vm-method swap &
      stress-ng --vm 1 --vm-bytes 512M --vm-keep --vm-method swap &
      stress-ng --vm 1 --vm-bytes 512M --vm-keep --vm-method swap &
      stress-ng --vm 1 --vm-bytes 512M --vm-keep --vm-method swap &

   CPU increments down roughly 10% each step to allow converging.
   Confirm CPU is not throttled to 0 eg. only throttles down then converges
   (8% min):

   .. code:: sh

      97046 root      20   0 9160524  30356  11092 S 11.9  0.0   1:05.36 CPU 0/KVM
      97047 root      20   0 9160524  30356  11092 S 11.9  0.0   1:03.47 CPU 1/KVM
      97048 root      20   0 9160524  30356  11092 S 11.9  0.0   1:05.16 CPU 2/KVM
      97049 root      20   0 9160524  30356  11092 S 11.9  0.0   1:05.72 CPU 3/KVM

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

When live migration autoconverge is enabled, the CPU is throttled (increments
down roughly 10% each step) but does not go below 8%. This increases the
likelihood of migration successfully completing.
