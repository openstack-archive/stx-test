=======
BASHLOG
=======

.. contents::
   :local:
   :depth: 1

--------------------
SECURITY_BASHLOG_01
--------------------

Test ID: SECURITY_BASHLOG_01
Test Title: Validate bash.log behaviour on node
Tags: P4, Security, Regression, AUTOMATABLE

~~~~~~~~~~~~~~~~~~~
Testcase Objective:
~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions:
~~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~~
Test Steps:
~~~~~~~~~~~

1. Validate bash.log behaviour on a controller node
    a. Confirm append-only attribute of bash.log
    Run a 'sudo lsattr /var/log/bash.log'. Confirm that bash.log is set
    to append only.

-----a-------e-- bash.log <-- append-only attr on

-------------e-- user.log <-- append-only attr off

b. Attempt to edit bash.log, modify the existing data and save the file
- Validate that this is blocked.
c. Attempt to remove the append-only attribute of bash.log
- Using chattr, run 'sudo chattr -a bash.log', in order to remove
the append-only
attribute
- Validate this is rejected
2. Repeat test #1 on a compute node
3. Repeat test #1 on a storage node

~~~~~~~~~~~~~~~~~~~
Expected Behaviour:
~~~~~~~~~~~~~~~~~~~

All test steps are pass


