==============
EXTERNAL_LDAP
==============

.. contents::
   :local:
   :depth: 1

--------------------------
SECURITY_EXTERNAL_LDAP_01
--------------------------

Test ID: SECURITY_EXTERNAL_LDAP_01
Test Title: add external ldap user to a group
Tags: P4, Security, Regression, AUTOMATABLE

~~~~~~~~~~~~~~~~~~~~
Testcase Objective:
~~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions:
~~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~~~~~~~~~~~
Test Steps:
~~~~~~~~~~~~~~~~~~~~

add user to the group in external ldap

~~~~~~~~~~~~~~~~~~~~
Expected Behaviour:
~~~~~~~~~~~~~~~~~~~~

verify that the user have the correct credential via keystone

-----------------------------
SECURITY_EXTERNAL_LDAP_02
-----------------------------

Test ID: SECURITY_EXTERNAL_LDAP_02
Test Title: verify external ldap user via keystone user-list
Tags: P4, Security, Regression, AUTOMATABLE

~~~~~~~~~~~~~~~~~~~~
Testcase Objective:
~~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions:
~~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~~~~~~~~~~~
Test Steps:
~~~~~~~~~~~~~~~~~~~~

issue keystone user-list

~~~~~~~~~~~~~~~~~~~~
Expected Behaviour:
~~~~~~~~~~~~~~~~~~~~

verify that the users in external ldap are listed

-----------------------------
SECURITY_EXTERNAL_LDAP_03
-----------------------------

Test ID: SECURITY_EXTERNAL_LDAP_03
Test Title:  verify users have correct credential after swact.
Tags: P4, Security, Regression, AUTOMATABLE

~~~~~~~~~~~~~~~~~~~~
Testcase Objective:
~~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions:
~~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~~~~~~~~~~~
Test Steps:
~~~~~~~~~~~~~~~~~~~~

swact controller

~~~~~~~~~~~~~~~~~~~~
Expected Behaviour:
~~~~~~~~~~~~~~~~~~~~

verify keystone user credential can be used. verify you can add new keystone
users.


