====
IMA
====

.. contents::
   :local:
   :depth: 1

----------------
SECURITY_IMA_01
----------------

Test ID: SECURITY_IMA_01
Test Title:  test_Alter a monitored file by adding a line to it
Tags: P1, Security, Regression, AUTOMATABLE

~~~~~~~~~~~~~~~~~~~~
Testcase Objective:
~~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions:
~~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~~~~~~~~~~~
Test Steps:
~~~~~~~~~~~~~~~~~~~~

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

~~~~~~~~~~~~~~~~~~~~
Expected Behaviour:
~~~~~~~~~~~~~~~~~~~~

All test steps are pass

----------------
SECURITY_IMA_02
----------------

Test ID: SECURITY_IMA_02
Test Title:  test_Verify basic Heat templates via Remote CLI
Tags: P1, Security, Regression, AUTOMATABLE

~~~~~~~~~~~~~~~~~~~~
Testcase Objective:
~~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions:
~~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~~~~~~~~~~~
Test Steps:
~~~~~~~~~~~~~~~~~~~~

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

~~~~~~~~~~~~~~~~~~~~
Expected Behaviour:
~~~~~~~~~~~~~~~~~~~~

All test steps are pass

----------------
SECURITY_IMA_03
----------------

Test ID: SECURITY_IMA_03
Test Title:  test_Create a new file owned by wrsroot user and execute it
(non-root)
Tags: P1, Security, Regression, AUTOMATABLE

~~~~~~~~~~~~~~~~~~~~
Testcase Objective:
~~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions:
~~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~~~~~~~~~~~
Test Steps:
~~~~~~~~~~~~~~~~~~~~

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

~~~~~~~~~~~~~~~~~~~~
Expected Behaviour:
~~~~~~~~~~~~~~~~~~~~

All test steps are pass


