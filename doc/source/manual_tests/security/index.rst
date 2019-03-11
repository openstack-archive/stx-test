Feature Overview:
==============
Test Security basic features
This test plan covers Security regression for 2019_05 release.  It covers
basic functionality for the following features:

- External_ldap
- HTTPS
- Keystone_User
- Ldap
- Linux_User_Feature
- Remote-CLI
- IMA
- TPM install


Test Requirements:
===============
IP4 & IP6 system installed successful


===============
Test Cases:
===============



Test ID: SECURITY_BASHLOG_01
Test Title: Validate bash.log behaviour on node
Tags: P4, Security, Regression, AUTOMATABLE

Testcase Objective:
--------------------------------

Test Pre-Conditions:
--------------------------
NA

Test Steps:
----------------
1. Validate bash.log behaviour on a controller node
    a. Confirm append-only attribute of bash.log
    - Run a 'sudo lsattr /var/log/bash.log'. Confirm that bash.log is set
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

Expected Behaviour:
-----------------------------
All test steps are pass



Test ID: SECURITY_EXTERNAL_LDAP_02
Test Title: add external ldap user to a group
Tags: P4, Security, Regression, AUTOMATABLE

Testcase Objective:
--------------------------------

Test Pre-Conditions:
--------------------------
NA

Test Steps:
----------------
add user to the group in external ldap

Expected Behaviour:
-----------------------------
verify that the user have the correct credential via keystone



Test ID: SECURITY_EXTERNAL_LDAP_03
Test Title: verify external ldap user via keystone user-list
Tags: P4, Security, Regression, AUTOMATABLE

Testcase Objective:
--------------------------------

Test Pre-Conditions:
--------------------------
NA

Test Steps:
----------------
issue keystone user-list

Expected Behaviour:
-----------------------------
verify that the users in external ldap are listed



Test ID: SECURITY_EXTERNAL_LDAP_04
Test Title:  verify users have correct credential after swact.
Tags: P4, Security, Regression, AUTOMATABLE

Testcase Objective:
--------------------------------

Test Pre-Conditions:
--------------------------
NA

Test Steps:
----------------
swact controller

Expected Behaviour:
-----------------------------
verify keystone user credential can be used. verify you can add new keystone
users.



Test ID: SECURITY_HTTPS_05
Test Title:  test_Query keystone for https endpoints
Tags: P1, Security, Regression, AUTOMATABLE

Testcase Objective:
--------------------------------

Test Pre-Conditions:
--------------------------
NA

Test Steps:
----------------
Enable HTTPS
Verify curl cli from external .

amanoker@yow-cgts2-lx:/folk/cgts_logs/logs/CGTS-10317$
curl -i https://128.224.151.85:18002/v1/event_log?include_suppress=True
-X GET -H "Content-Type: application/json" -H "Accept: application/json"
-H "X-Auth-Token:gAAAAABbwMtHOw0TRCatKH-aWZBkK72o05RKaexK1LkdPhRsMTd4C5wXt8qit
PJ066pQzUE3P5zO8xzkAmzAfcbcPuGD7ghxJpAApGwrYX4iy3cBall7E-a7MxF9vY3i2i6nPJkBtCk
jMXuKhHZMA3r3bHax2Kz5StaGPad8AE8UtmxTes7AhnY"
curl: (60) SSL certificate problem: self signed certificate
More details here: http://curl.haxx.se/docs/sslcerts.html

curl performs SSL certificate verification by default, using a "bundle"
of Certificate Authority (CA) public keys (CA certs). If the default
bundle file isn't adequate, you can specify an alternate file
using the --cacert option.
If this HTTPS server uses a certificate signed by a CA represented in
the bundle, the certificate verification probably failed due to a
problem with the certificate (it might be expired, or the name might
not match the domain name in the URL).
If you'd like to turn off curl's verification of the certificate, use
the -k (or --insecure) option.

amanoker@yow-cgts2-lx:/folk/cgts_logs/logs/CGTS-10317$
curl -i https://128.224.151.85:18002/v1/event_log?include_suppress=True
-X GET -H "Content-Type: application/json" -H "Accept: application/json" -H
"X-Auth-Token:gAAAAABbwMtHOw0TRCatKH-aWZBkK72o05RKaexK1LkdPhRsMTd4C5wXt8qitPJ
066pQzUE3P5zO8xzkAmzAfcbcPuGD7ghxJpAApGwrYX4iy3cBall7E-a7MxF9vY3i2i6nPJkBtCkj
MXuKhHZMA3r3bHax2Kz5StaGPad8AE8UtmxTes7AhnY" -k
HTTP/1.1 200 OK
Content-Length: 85549
Content-Type: application/json
X-Openstack-Request-Id: req-b7ab7bfd-d487-45fa-be32-26a4f82a2693
Date: Fri, 12 Oct 2018 16:27:41 GMT
Strict-Transport-Security: max-age=63072000; includeSubDomains

Expected Behaviour:
-----------------------------
All test steps are pass



Test ID: SECURITY_HTTPS_06
Test Title:  test_Validate that services are only listening on https
Tags: P3, Security, Regression, AUTOMATABLE

Testcase Objective:
--------------------------------

Test Pre-Conditions:
--------------------------
NA

Test Steps:
----------------
1) Log on to active controller, use netstat to determine listening ports
2) Validate that services (e.g. sysinv, Nova, Neutron, patching, cinder,
glance, ceilometer, and heat) are listening to correct ports
3) On an external host (e.g. linux workstation) telnet to the listening port
4) Validate that telnet connects, type 'GET / \n' and no info is returned
5) From and external host, use telnet to connect to each service
(telnet <host> <port>)
6) Validate that telnet connects, type 'GET / \n' and no info is returned

Expected Behaviour:
-----------------------------
All test steps are pass



Test ID: SECURITY_HTTPS_07
Test Title:  test_Validate that services respond over https
Tags: P1, Security, Regression, AUTOMATABLE

Testcase Objective:
--------------------------------

Test Pre-Conditions:
--------------------------
NA

Test Steps:
----------------
From and external host, use curl to access HTTPS REST API for each service

Expected Behaviour:
-----------------------------
Validate that services (e.g. sysinv, Nova, Neutron, patching, cinder, glance,
ceilometer, and heat) respond correctly



Test ID: SECURITY_KEYSTONE_USER_08
Test Title:  test_Change keystone admin password on active controller.
Tags: P1, Security, Regression, AUTOMATABLE

Testcase Objective:
--------------------------------

Test Pre-Conditions:
--------------------------
NA

Test Steps:
----------------
AS TITLE

Expected Behaviour:
-----------------------------
Password changed successfully



Test ID: SECURITY_LDAP_09
Test Title:  test the new ldap user on both controllers, swacting in between.
Tags: P1, Security, Regression, AUTOMATABLE

Testcase Objective:
--------------------------------

Test Pre-Conditions:
--------------------------
NA

Test Steps:
----------------
step-1: add ldap user user-1 on controller-0
expected result: verify user is added
step-2: swact to controller-1 and login as users-1 and login to controller-0
as user-1
expected result: verify login is successful.
step-3: add ldap user user-2 on controller-1 and swact to controller-0
expected result: verify user2 is added successfully, swact is successful.
step-2: login to controller-0 as user-2 and login to controller-1 as user-2
expected result: verify login is successful.

Expected Behaviour:
-----------------------------
All test steps are pass



Test ID: SECURITY_LDAP_10
Test Title:  test_verify ldap user can change password
Tags: P1, Security, Regression, AUTOMATABLE

Testcase Objective:
--------------------------------

Test Pre-Conditions:
--------------------------
NA

Test Steps:
----------------
step-1: on controller, create new ldap user ldapuser2 via ldapusersetup
expected result: on compute, login as ldapuser2 using default password
which is equal to username (ldapuser2) and verify password must be changed
on first login. verify login is successful.
step-2: change password again, via 'passwd' command and logout
expected result: login to another node using latest password.

Expected Behaviour:
-----------------------------
All test steps are pass



Test ID: SECURITY_LDAP_11
Test Title:  test_Verify LDAP users after controller-0 reinstall and swact
Tags: P1, Security, Regression, AUTOMATABLE

Testcase Objective:
--------------------------------

Test Pre-Conditions:
--------------------------
standard system installed

Test Steps:
----------------
step-1: add ldap user user-1 on controller-0
expected result: verify user is added
step-2: reinstall controller-0 and swact to controller-1 and login as users-1
and login to controller-0 as user-1
expected result: verify login is successful.

Expected Behaviour:
-----------------------------
All test steps are pass



Test ID: SECURITY_LINUX_USER_12
Test Title:  test_TiS linux user wrsroot password change propagation
Tags: P1, Security, Regression, AUTOMATABLE

Testcase Objective:
--------------------------------

Test Pre-Conditions:
--------------------------
NA

Test Steps:
----------------
change linux user wrsroot password

Expected Behaviour:
-----------------------------
password changed successfully



Test ID: SECURITY_LINUX_USER_13
Test Title:  test_Verify file permission after initial install
Tags: P1, Security, Regression, AUTOMATABLE

Testcase Objective:
--------------------------------

Test Pre-Conditions:
--------------------------
NA

Test Steps:
----------------
verify file permission for opt/platform files permission is correct
verify file permission for /etc/(system)-config files permission is correct

Expected Behaviour:
-----------------------------
All test steps are pass



Test ID: SECURITY_LINUX_USER_14
Test Title:  test_Verify File permission after reboot nodes.
Tags: P1, Security, Regression, AUTOMATABLE

Testcase Objective:
--------------------------------

Test Pre-Conditions:
--------------------------
NA

Test Steps:
----------------
verify file permission for opt/platform files permission is correct
verify file permission for /etc/(system)-config files permission is correct

Expected Behaviour:
-----------------------------
All test steps are pass



Test ID: SECURITY_LINUX_USER_15
Test Title:  test_Verify that system admin user is capable of changing
password quality
Tags: P1, Security, Regression, AUTOMATABLE

Testcase Objective:
--------------------------------

Test Pre-Conditions:
--------------------------
NA

Test Steps:
----------------
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

Expected Behaviour:
-----------------------------
All test steps are pass



Test ID: SECURITY_LINUX_USER_16
Test Title:  test_Verify account stays locked after swact
Tags: P1, Security, Regression, AUTOMATABLE

Testcase Objective:
--------------------------------

Test Pre-Conditions:
--------------------------
1. add new user: linux_user_16
2. make linux_user_16 locked by attempting login multi times with incorrect
password

Test Steps:
----------------
Step-1: swact controllers, try to login linux_user_16

Expected Behaviour:
-----------------------------
user account still stay locked



Test ID: SECURITY_CLI_17
Test Title:  test_Verify basic Heat templates via Remote CLI
Tags: P1, Security, Regression, AUTOMATABLE

Testcase Objective:
--------------------------------

Test Pre-Conditions:
--------------------------
NA

Test Steps:
----------------
AS TITLE

Expected Behaviour:
-----------------------------
All test steps are pass



Test ID: SECURITY_HTTPS_18
Test Title:  test_https enabled - https-certificate-install successful
Tags: P1, Security, Regression, AUTOMATABLE

Testcase Objective:
--------------------------------

Test Pre-Conditions:
--------------------------
NA

Test Steps:
----------------
This test verifies that https can be successfully enabled post-install and
that server-certificate could be successfully installed
1) Install a standard (2+2) system
2) Enable https post-install

[wrsroot@controller-1 ~(keystone_admin)]$
[wrsroot@controller-1 ~(keystone_admin)]$ system show
+------------------+--------------------------------------+
| Property | Value |
+------------------+--------------------------------------+
| system_mode | duplex |
| sdn_enabled | True |
| description | yow-cgcs-pv-1: setup by lab_setup.sh |
| software_version | 17.07 |
| system_type | Standard |
| created_at | 2017-09-14T19:11:42.099406+00:00 |
| uuid | 8568e042-f6d4-41d3-8045-1ce7a55a4f81 |
| updated_at | 2017-09-18T18:15:36.462770+00:00 |
| contact | None |
| location | None |
| https_enabled | False |
| timezone | UTC |
| name | yow-cgcs-pv-1 |
+------------------+--------------------------------------+
[wrsroot@controller-1 ~(keystone_admin)]$

[wrsroot@controller-1 ~(keystone_admin)]$ system modify --https_enabled=true
+------------------+--------------------------------------+
| Property | Value |
+------------------+--------------------------------------+
| system_mode | duplex |
| sdn_enabled | True |
| description | yow-cgcs-pv-1: setup by lab_setup.sh |
| software_version | 17.07 |
| system_type | Standard |
| created_at | 2017-09-14T19:11:42.099406+00:00 |
| uuid | 8568e042-f6d4-41d3-8045-1ce7a55a4f81 |
| updated_at | 2017-09-15T19:17:01.822657+00:00 |
| contact | None |
| location | None |
| https_enabled | True |
| timezone | UTC |
| name | yow-cgcs-pv-1 |
+------------------+--------------------------------------+
HTTPS enabled with a self-signed certificate.
This should be changed to a CA-signed certificate with
'https-certificate-install'.

* Wait for config-out-of-date alarms to clear:
[wrsroot@controller-1 ~(keystone_admin)]$ system alarm-list
+----------+--------------------------------------------+-------------------+
----------+------------------+
| Alarm ID | Reason Text | Entity ID | Severity | Time Stamp |
+----------+--------------------------------------------+-------------------+
----------+------------------+
| 250.001 | controller-1 Configuration is out-of-date. | host=controller-1 |
major | 2017-09-18T18:15 |
| | | | | :36.569036 |
| | | | | |
| 250.001 | controller-0 Configuration is out-of-date. | host=controller-0 |
major | 2017-09-18T18:15 |
| | | | | :36.517086 |
| | | | | |
+----------+--------------------------------------------+-------------------+
----------+------------------+

* Before proceeding to the next step, the above alarms should be cleared

3) Install a signed certificate:

[wrsroot@controller-1 ~(keystone_admin)]$ sudo https-certificate-install
-c server-with-key.pem
Enter password for the CA-Signed certificate file [Enter <CR> for no password]:
Enter [sudo] password for wrsroot:
Installing certificate file server-with-key.pem

WARNING: Installing an invalid or expired certificate
will cause a service interruption.
OK to Proceed? (yes/NO): yes

WARNING: For security reasons, the original certificate,
containing the private key, that you provided, will be removed,
once the private key is processed.
OK to Proceed? (yes/NO): yes
In regular mode...
done
[wrsroot@controller-1 ~(keystone_admin)]$

4) Verify that all public endpoints are changed to https
Ex:
[wrsroot@controller-1 ~(keystone_admin)]$ openstack endpoint list |grep https
| d0759c6e894e4164af99fb321fa0ad03 | RegionOne | heat | orchestration | True |
public | https://128.224.151.182:8004/v1/%(tenant_id)s |
| 39870774310e4b738fc810ef3d34b4df | RegionOne | ceilometer | metering |
True | public | https://128.224.151.182:8777 |
| d25215e7d3c44c0e9a522e9aace62c00 | RegionOne | nova | compute | True |
public | https://128.224.151.182:8774/v2.1/%(tenant_id)s |
| 617784eb0b5345fd87c7bb1164021571 | RegionOne | keystone | identity | True |
public | https://128.224.151.182:5000/v3 |
| 055e71bb7f5b4ee3ae5b0479a4fd7a19 | RegionOne | vim | nfv | True | public |
https://128.224.151.182:4545 |
| fde3bffef4974429adecb0270f907ae3 | RegionOne | neutron | network | True |
public | https://128.224.151.182:9696 |
| 7bc38a082b9b458199cf27971c98e1a5 | RegionOne | cinder | volume | True |
public | https://128.224.151.182:8776/v1/%(tenant_id)s |
| f3a978f25ef54306982fc99ec5ab3162 | RegionOne | cinderv2 | volumev2 | True |
public | https://128.224.151.182:8776/v2/%(tenant_id)s |
| d47315aae5ba48f0b565d9489f7a95ca | RegionOne | patching | patching | True |
public | https://128.224.151.182:15491 |
| a024349222d3463680ff6cb723933cee | RegionOne | aodh | alarming | True |
public | https://128.224.151.182:8042 |
| e1bd3fd055504df08bd00c29a3eb6f25 | RegionOne | sysinv | platform | True |
public | https://128.224.151.182:6385/v1 |

5) Verify that Horizon is only accessble via https and that the correct
certificate is presented

Expected Behaviour:
-----------------------------
All test steps are pass



Test ID: SECURITY_IMA_19
Test Title:  test_Alter a monitored file by adding a line to it
Tags: P1, Security, Regression, AUTOMATABLE

Testcase Objective:
--------------------------------

Test Pre-Conditions:
--------------------------
NA

Test Steps:
----------------
select a file monitored by IMA, preferably a Python script, so it can still be
executed by shell process. I suggest /usr/bin/sm-dump
Copy file with SHA256 attributes:
cd /usr/sbin
cp sw-patch --preserve=all TEMP

You should see IMA signature in file attributes:

getfattr -m - -d /usr/sbin/TEMP
# file: usr/sbin/TEMP
security.ima=0sAwIE1SNPUgEAplPHFqr6WiJUBurR
To edit file, DO NOT USE VI, Perl substitution, etc.. as Signature will be
lost during file edition (they use .tmp file and then overwrite original file
after). Append line via Shell directly:

echo "#commentaire bidon" >> TEMP
on 1 terminal, monitor IMA logs:
tail -f /var/log/ima.log

On another shell, execute modified copy of file:

/usr/bin/TEMP

you should see detection of SHA256 mismatch:

2018-01-12T20:02:29.000 controller-1 audispd: info node=controller-1
type=INTEGRITY_DATA msg=audit(1515787349.787:24718): pid=28966 uid=0 auid=1875
ses=193 op="appraise_data" cause="invalid-signature" comm=bash
name=/usr/bin/sm-dump dev=sda3 ino=798984 res=0

Expected Behaviour:
-----------------------------
All test steps are pass



Test ID: SECURITY_IMA_20
Test Title:  test_Verify basic Heat templates via Remote CLI
Tags: P1, Security, Regression, AUTOMATABLE

Testcase Objective:
--------------------------------

Test Pre-Conditions:
--------------------------
NA

Test Steps:
----------------
create an executable script owned by root user.

vi /root/script
#!/bin/bash
echo 'allo'
ESC :wq!
add execution permission: chmod u+x /root/script
execute script:

/root/script

Monitor /var/log/ima.log

Must see a single trace with missing signature:

2018-01-15T21:13:18.000 controller-0 audispd: info node=controller-0
type=INTEGRITY_DATA msg=audit(1516050798.323:16604): pid=75884 uid=0
auid=1875 ses=23 op="appraise_data" cause="missing-signature"
comm=bash name=/root/script dev=sda3 ino=674149 res=0

Expected Behaviour:
-----------------------------
All test steps are pass



Test ID: SECURITY_IMA_21
Test Title:  test_Create a new file owned by wrsroot user and execute it
(non-root)
Tags: P1, Security, Regression, AUTOMATABLE

Testcase Objective:
--------------------------------

Test Pre-Conditions:
--------------------------
NA

Test Steps:
----------------
create an executable script owned by wrsroot user.

vi /home/wrsroot/script
#!/bin/bash
echo 'allo'
ESC :wq!
add execution permission:
chmod u+x script
execute script:

/home/wrsroot/script

Monitor /var/log/ima.log

Should see NO trace added to log file (unsigned executable is ran by non-root
user)

Expected Behaviour:
-----------------------------
All test steps are pass



Test ID: SECURITY_TPM_22
Test Title:  test_TPM install cert-with-key standard system
Tags: P1, Security, Regression, AUTOMATABLE

Testcase Objective:
--------------------------------

Test Pre-Conditions:
--------------------------
NA

Test Steps:
----------------
1) Install 2+2 lab with ENABLE_HTTPS = Y
2) Follow the procedure to update the https certificate from the Installation
Guide to install https private key in TPM
system certificate-install -m tpm_mode -p Li69nux* ./wc99-103.pem
3) Swact controllers

Expected Behaviour:
-----------------------------
1) Verify that horizon is only accessible via https
2) Verify that the private key is not present on the controller-0 and
controller-1 filesystem
3) After swacting controllers, verify that horizon is only accessible via
https



References:
==========



