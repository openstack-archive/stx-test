============
LINUX_USER
============

.. contents::
   :local:
   :depth: 1

-----------------------
SECURITY_LINUX_USER_01
-----------------------

Test ID: SECURITY_LINUX_USER_01
Test Title:  test_TiS linux user wrsroot password change propagation
Tags: P1, Security, Regression, AUTOMATABLE

~~~~~~~~~~~~~~~~~~~~~
Testcase Objective:
~~~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions:
~~~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~~
Test Steps:
~~~~~~~~~~~

change linux user wrsroot password

~~~~~~~~~~~~~~~~~~~~~
Expected Behaviour:
~~~~~~~~~~~~~~~~~~~~~

password changed successfully

-----------------------
SECURITY_LINUX_USER_02
-----------------------

Test ID: SECURITY_LINUX_USER_02
Test Title:  test_Verify file permission after initial install
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

verify file permission for opt/platform files permission is correct
verify file permission for /etc/(system)-config files permission is correct

~~~~~~~~~~~~~~~~~~~~~
Expected Behaviour:
~~~~~~~~~~~~~~~~~~~~~

All test steps are pass

-----------------------
SECURITY_LINUX_USER_03
-----------------------

Test ID: SECURITY_LINUX_USER_03
Test Title:  test_Verify File permission after reboot nodes.
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

verify file permission for opt/platform files permission is correct
verify file permission for /etc/(system)-config files permission is correct

~~~~~~~~~~~~~~~~~~~~~
Expected Behaviour:
~~~~~~~~~~~~~~~~~~~~~

All test steps are pass

-----------------------
SECURITY_LINUX_USER_04
-----------------------

Test ID: SECURITY_LINUX_USER_04
Test Title:  test_Verify that system admin user is capable of changing
password quality
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

Password quality configuration is validated using pam_cracklib library.

To change password quality configuration on the controller,
edit /etc/pam.d.common-password

The password quality validation is configured via the first non-comment line:

password    required     pam_cracklib.so retry=1 minlen=8 lcredit=-1 difok=3


To increase the minimum password length, change the minlen parameter.

To change the minimum number of characters that must change between subseqent
passwords, change the diffok parameter.

To require at least one uppercase character in the password, add 'ucredit=-1'

To require at lease one digit, add 'dcredit=-1'

~~~~~~~~~~~~~~~~~~~~~
Expected Behaviour:
~~~~~~~~~~~~~~~~~~~~~

All test steps are pass

-----------------------
SECURITY_LINUX_USER_05
-----------------------

Test ID: SECURITY_LINUX_USER_05
Test Title:  test_Verify account stays locked after swact
Tags: P1, Security, Regression, AUTOMATABLE

~~~~~~~~~~~~~~~~~~~~~
Testcase Objective:
~~~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions:
~~~~~~~~~~~~~~~~~~~~~

1. add new user: linux_user_16
2. make linux_user_16 locked by attempting login multi times with incorrect
    password

~~~~~~~~~~~~~~~~~~~~~
Test Steps:
~~~~~~~~~~~~~~~~~~~~~

Step-1: swact controllers, try to login linux_user_16

~~~~~~~~~~~~~~~~~~~~~
Expected Behaviour:
~~~~~~~~~~~~~~~~~~~~~

user account still stay locked


