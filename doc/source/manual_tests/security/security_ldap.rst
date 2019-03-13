=====
LDAP
=====

.. contents::
   :local:
   :depth: 1

-----------------
SECURITY_LDAP_01
-----------------

Test ID: SECURITY_LDAP_01
Test Title:  test the new ldap user on both controllers, swacting in between.
Tags: P1, Security, Regression, AUTOMATABLE

~~~~~~~~~~~~~~~~~~~~~
Testcase Objective:
~~~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions:
~~~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~~~~~~~~~~~~
Test Steps:
~~~~~~~~~~~~~~~~~~~~~

step-1: add ldap user user-1 on controller-0
expected result: verify user is added
step-2: swact to controller-1 and login as users-1 and login to controller-0
as user-1
expected result: verify login is successful.
step-3: add ldap user user-2 on controller-1 and swact to controller-0
expected result: verify user2 is added successfully, swact is successful.
step-2: login to controller-0 as user-2 and login to controller-1 as user-2
expected result: verify login is successful.

~~~~~~~~~~~~~~~~~~~~~
Expected Behaviour:
~~~~~~~~~~~~~~~~~~~~~

All test steps are pass

-----------------
SECURITY_LDAP_02
-----------------

Test ID: SECURITY_LDAP_02
Test Title:  test_verify ldap user can change password
Tags: P1, Security, Regression, AUTOMATABLE

~~~~~~~~~~~~~~~~~~~~~
Testcase Objective:
~~~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions:
~~~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~~~~~~~~~~~~
Test Steps:
~~~~~~~~~~~~~~~~~~~~~

step-1: on controller, create new ldap user ldapuser2 via ldapusersetup
expected result: on compute, login as ldapuser2 using default password
which is equal to username (ldapuser2) and verify password must be changed
on first login. verify login is successful.
step-2: change password again, via 'passwd' command and logout
expected result: login to another node using latest password.

~~~~~~~~~~~~~~~~~~~~~
Expected Behaviour:
~~~~~~~~~~~~~~~~~~~~~

All test steps are pass

-----------------
SECURITY_LDAP_03
-----------------

Test ID: SECURITY_LDAP_03
Test Title:  test_Verify LDAP users after controller-0 reinstall and swact
Tags: P1, Security, Regression, AUTOMATABLE

~~~~~~~~~~~~~~~~~~~~~
Testcase Objective:
~~~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions:
~~~~~~~~~~~~~~~~~~~~~

standard system installed

~~~~~~~~~~~~~~~~~~~~~
Test Steps:
~~~~~~~~~~~~~~~~~~~~~

step-1: add ldap user user-1 on controller-0
expected result: verify user is added
step-2: reinstall controller-0 and swact to controller-1 and login as users-1
and login to controller-0 as user-1
expected result: verify login is successful.

~~~~~~~~~~~~~~~~~~~~~
Expected Behaviour:
~~~~~~~~~~~~~~~~~~~~~

All test steps are pass


