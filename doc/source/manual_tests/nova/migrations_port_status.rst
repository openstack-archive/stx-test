=========================
Migrations and PortStatus
=========================

The neutron port status should always be active after a successful migration
(cold, live, block).

-----------------
Test Requirements
-----------------

Tbd

.. contents::
   :local:
   :depth: 1

------------------------------
Nova_Migrations_PortStatus_1.0
------------------------------

:Test ID: test_neutron_port_status_after_successful_migration_ACTIVE_(cold,_live,_block)
:Test Title: Tbd
:Tags: p2,regression,nova

~~~~~~~~~~~~~~
Test Objective
~~~~~~~~~~~~~~

Confirm neutron port status after successful migration, The "Port State"
should be "ACTIVE" after successful migrations (cold, live, block).

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. Test with cold migration, on a system with at least 2 workers
   Launch instance. Once active, look at the status of the related
   neutron ports – they should be reported as ACTIVE

   .. code:: sh

      $ neutron port-list --device_id=e80a0560-42f7-4d81-bbed-68b52664566b -c id
        -c status -c binding:host_id
      +--------------------------------------+--------+
      | id                                   | status |
      +--------------------------------------+--------+
      | 3cf1a5eb-3eed-4596-b021-7e7051f98c5c | ACTIVE |
      | ca9a0b07-219d-4b8e-9f0d-c3e64cf61024 | ACTIVE |
      | d5c463ef-21fc-4a29-a5b6-c6daf4e7743c | ACTIVE |
      +--------------------------------------+--------+

2. Cold migrate the instance to another worker.
3. Check the related neutron ports to ensure they are reporting ACTIVE.
4. Repeat from step 1 to step 3.
5. Test with live migration, on a system with at least 2 workers. Launch an
   instance. Once active, look at the status of the related neutron ports,
   they should be reported as ACTIVE.

   .. code:: sh

      $ neutron port-list --device_id=e80a0560-42f7-4d81-bbed-68b52664566b -c id
        -c status -c binding:host_id

      | id                                   | status |
      | 3cf1a5eb-3eed-4596-b021-7e7051f98c5c | ACTIVE |
      | ca9a0b07-219d-4b8e-9f0d-c3e64cf61024 | ACTIVE |
      | d5c463ef-21fc-4a29-a5b6-c6daf4e7743c | ACTIVE |

6. Live migrate an instance to another worker.
7. Check the related neutron ports to ensure they are reporting ACTIVE.
8. Repeat from step 5 to step 7.
9. Test with live block migration, on a system with at least 2 workers. Launch
   an instance, once active, look at the status of the related neutron ports,
   they should be reported as ACTIVE.

   .. code:: sh

      $ neutron port-list --device_id=e80a0560-42f7-4d81-bbed-68b52664566b -c id
        -c status -c binding:host_id

      | id                                   | status |
      | 3cf1a5eb-3eed-4596-b021-7e7051f98c5c | ACTIVE |
      | ca9a0b07-219d-4b8e-9f0d-c3e64cf61024 | ACTIVE |
      | d5c463ef-21fc-4a29-a5b6-c6daf4e7743c | ACTIVE |

10. Live block migrate an instance to another worker.
11. Check the related neutron ports to ensure they are reporting ACTIVE.
12. Repeat from step 9 to step 11.

Expected Behavior:
-----------------------------
1. Expect the neutron ports to be ACTIVE after cold migration.
2. Expect the neutron ports to be ACTIVE after live migration.
3. Expect them to be ACTIVE after live block migration.
