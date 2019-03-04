=================
CPU Thread Policy
=================

Resize interactions cpu thread policy and vcpu number under hyperthreaded conditions.

-----------------
Test Requirements
-----------------

Hyperthread capable host.

.. contents::
   :local:
   :depth: 1

------------------------
Nova_CPUThreadPolicy_1.0
------------------------

:Test ID: test_nova_resize_to_CPU_Thread_Policy_require_As_a_user,_attempt_resize_operation_changing_the_VCPU_size_as_well
:Test Title:
:Tags: p2, regression, nova

~~~~~~~~~~~~~~
Test Objective
~~~~~~~~~~~~~~

Tbd

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

Tbd

~~~~~~~~~~
Test Steps
~~~~~~~~~~

Resize an instance from isolate to CPU Thread Policy “require | prefer”

1. As a user, attempt instance resize operation changing the VCPU size as
   well, (e.g. Instance with flavor with 2 VCPU and the extra spec resized):

   .. code:: sh

     hw:cpu_thread_policy=isolate

2. To a flavor with extra spec where the new flavor has 3 VCPU or 1 VCPU:

   .. code:: sh

     hw:cpu_thread_policy=require

3. To a flavor with extra spec where the new flavor has 3 VCPU or 1 VCPU:

   .. code:: sh

     hw:cpu_thread_policy=prefer

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

The resize should not fail (since Mitaka) regardless of whether the flavor
vcpu is not evenly divisible (i.e. where vcpu is not a multiple of 2 and the
hw:cpu_threads_policy=require was specified in the flavor).

------------------------
Nova_CPUThreadPolicy_2.0
------------------------

:Test ID: test_nova_resize_changing_VCPU_settings_and_setting_CPU_Thread_Policy_to_isolate,_prefer
:Test Title:
:Tags: p2, regression, nova
:Sub-domain: Hyperthreading

~~~~~~~~~~~~~~
Test Objective
~~~~~~~~~~~~~~

Resize an instance changing vCPU settings and setting CPU Thread Policy to
`isolate` to `prefer`.

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

Tbd

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. Resize instance from flavor with 2 VCPU and the extra spec `require` to a
   flavor with extra spec `isolate` where the new flavor has 3 VCPU (ie. or
   some number that is not a multiple of 2):::

   .. code:: sh

     hw:cpu_thread_policy=require
     hw:cpu_thread_policy=isolate

2. Repeat resize test but to a new target CPU Thread Policy prefer, resize
   instance from `require` flavor with 2 VCPU to a flavor with extra spec
   `prefer` where the new flavor has 3 VCPU (ie. or some number that is not
   a multiple of 2):

   .. code:: sh

     hw:cpu_thread_policy=prefer

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

The resize should be successful  (if the host has specifient room).

------------------------
Nova_CPUThreadPolicy_3.0
------------------------

:Test ID: test_nova_imagemetadata_property_hw_cpu_threads_policy=isolate_override_test,_conflict_with_require_in_flavor
:Test Title:
:Tags: p2, regression, nova
:Sub-domain: Hyperthreading

~~~~~~~~~~~~~~~~~~
Testcase Objective
~~~~~~~~~~~~~~~~~~

Attempts to launch instance with Image metadata hw_cpu_thread_policy=isolate
and dedicated cpu policy but flavor has conflicting thread policy setting
`require`.

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

Tbd

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. Attempts to launch an instance with Image metadata that has
   `hw_cpu_thread_policy=isolate` and dedicated cpu policy Flavor has
   `hw:cpu_thread_policy=require`.

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

Conflict results:

   .. code:: sh

     Error: Image property 'hw_cpu_thread_policy' is not permitted to override
     CPU thread pinning policy set against the flavor (HTTP 403)

------------------------
Nova_CPUThreadPolicy_4.0
------------------------

:Test ID: test_nova_imagemetadata_property_hw_cpu_threads_policy=require_override_test,_prefer_thread_policy_in_flavor
:Test Title:
:Tags: p2, regression, nova
:Sub-domain: Hyperthreading

~~~~~~~~~~~~~~~~~~
Testcase Objective
~~~~~~~~~~~~~~~~~~

Image metadata hw_cpu_thread_policy=require; flavor has
`hw:cpu_thread_policy=prefer`. The instance will have cpu thread policy
`require`.

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

Tbd

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. Set image metadata to the require cpu thread policy,
   `dedicated`:

   .. code:: sh

     hw_cpu_policy=dedicated

2. Set flavor extra spec to cpu thread policy, `prefer`:

   .. code:: sh

     hw:cpu_thread_policy=prefer

3. Launch the instance with that image and flavor.

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

Instance expected to have CPU Thread Policy require.
