Feature Overview:
==============
This test plan covers Installation and Configuration regression.  It covers
basic functionality for
the following features:

- License
- Installation and commissioning
- Secure Boot
- tboot
- IPv4 and IPv6 installation

Test Requirements:
===============
IP4 & IP6 system installed successful



===============
Test Cases:
===============


Test ID: INSTALLATION_AND_CONFIGURATION_INSTALL_01
Test Title: test_AIO-DX https ipv4 install
Tags: P1, Installation and configuration, Regression, AUTOMATED

Testcase Objective:
--------------------------------
To ensure DX https ipv4 installation working well on StarlingX

Test Pre-Conditions:
--------------------------
NA

Test Steps:
----------------
Install IPv4 DX system

Expected Behaviour:
-----------------------------
Installation successfully



Test ID: INSTALLATION_AND_CONFIGURATION_INSTALL_02
Test Title: test_AIO-DX https ipv4 low-latency pxeboot_setup install
Tags: P1, Installation and configuration, Regression, AUTOMATED

Testcase Objective:
--------------------------------
To ensure DX https ipv4 low-latency installation working well on StarlingX

Test Pre-Conditions:
--------------------------
NA

Test Steps:
----------------
Install DX https ipv4 low-latency system

Expected Behaviour:
-----------------------------
Installation successfully



Test ID: INSTALLATION_AND_CONFIGURATION_INSTALL_03
Test Title: test_Standard https ipv4 install
Tags: P1, Installation and configuration, Regression, AUTOMATED

Testcase Objective:
--------------------------------
To ensure standard https ipv4 installation working well on StarlingX

Test Pre-Conditions:
--------------------------
NA

Test Steps:
----------------
Install IPv4 standard system

Expected Behaviour:
-----------------------------
Installation successfully



Test ID: INSTALLATION_AND_CONFIGURATION_INSTALL_04
Test Title: test_Standard https ipv6 install
Tags: P1, Installation and configuration, Regression, AUTOMATED

Testcase Objective:
--------------------------------
To ensure standard https ipv6 installation working well on StarlingX

Test Pre-Conditions:
--------------------------
NA

Test Steps:
----------------
Install IPv6 standard system

Expected Behaviour:
-----------------------------
Installation successfully



Test ID: INSTALLATION_AND_CONFIGURATION_LICENSE_05
Test Title: test_Post install license-install with corrupted license file
Tags: P3, Installation and configuration, Regression

Testcase Objective:
--------------------------------
To ensure Post install license-install with corrupted license file working
as expected on StarlingX

Test Pre-Conditions:
--------------------------
NA

Test Steps:
----------------
After config-controller, run "system license-install <license-file>" with
corrupted license file

Expected Behaviour:
-----------------------------
Verification should fail and Error msg should be clearly addressed.



Test ID: INSTALLATION_AND_CONFIGURATION_LICENSE_06
Test Title: test_ Post install license-install with expired license file
Tags: P2, Installation and configuration, Regression

Testcase Objective:
--------------------------------
To ensure Post install license-install with expired license file working as
expected on StarlingX

Test Pre-Conditions:
--------------------------
NA

Test Steps:
----------------
After config-controller, run "system license-install <license-file>" with
expired license file

Expected Behaviour:
-----------------------------
Verification should fail and Error msg should be clearly addressed



Test ID: INSTALLATION_AND_CONFIGURATION_LICENSE_07
Test Title: test_Run lab_setup with missing license file
Tags: P2, Installation and configuration, Regression

Testcase Objective:
--------------------------------
To ensure running lab_setup with missing license file working as expected on
StarlingX

Test Pre-Conditions:
--------------------------
NA

Test Steps:
----------------
run config_controller without license file
    config complete

run lab_setup.sh without license operation
    proper warning popup like "license missing..."

Expected Behaviour:
-----------------------------
As above



Test ID: INSTALLATION_AND_CONFIGURATION_LICENSE_08
Test Title: test_Verify License configuration on configui tool
Tags: P1, Installation and configuration, Regression

Testcase Objective:
--------------------------------
To Verify License configuration on configui tool working as expected on
StarlingX

Test Pre-Conditions:
--------------------------
NA

Test Steps:
----------------
using config_gui to generate config file

Expected Behaviour:
-----------------------------
There is no license related questions asked



Test ID: INSTALLATION_AND_CONFIGURATION_LICENSE_09
Test Title: test_Verify post install license-install
Tags: P1, Installation and configuration, Regression

Testcase Objective:
--------------------------------
To test_Verify post install license file working as expected on StarlingX

Test Pre-Conditions:
--------------------------
NA

Test Steps:
----------------
Run "config-controller" without config.ini
    config process should run through

Run "system license-install <license-file>"
    license installed as expected

Expected Behaviour:
-----------------------------
As above



Test ID: INSTALLATION_AND_CONFIGURATION_INSTALLATION_10
Test Title: test_ Installation and Commissioning: Attempt to add hosts prior
to controller-0 unlock (negative test)
Tags: P4, Installation and configuration, Regression

Testcase Objective:
--------------------------------
To test Attempting to add hosts prior to controller-0 unlock working
as expected on StarlingX

Test Pre-Conditions:
--------------------------
NA

Test Steps:
----------------
Install controller-0
   Controller-0 should show offline very briefly and then show locked-online.

While controller-0 is locked, attempt to add hosts via system
host-bulk-add(CLI)
    This should fail with "Bulk_add requires enabled controller. Please
unlock controller-0, wait for it to enable and then retry

While controller-0 is locked, attempt to add hosts via horizon
    This should fail with Provisioning request for new host
'<mac address> is not permitted while there is no unlocked-enabled controller.
Unlock controller-0, wait for it to enable and then retry

While controller-0 is locked, attempt to add hosts via CLI via system
host-update
    This should fail with Provisioning request for new host '<mac address>
is not permitted while there is no unlocked-enabled controller.
Unlock controller-0, wait for it to enable and then retry

Expected Behaviour:
-----------------------------
As above



Test ID: INSTALLATION_AND_CONFIGURATION_INSTALLATION_11
Test Title: test_Installation and Commissioning: Validate controller-0 state
after unlock
Tags: P4, Installation and configuration, Regression, AUTOMATED

Testcase Objective:
--------------------------------
To test Validating controller-0 state after unlock working as expected on
StarlingX

Test Pre-Conditions:
--------------------------
NA

Test Steps:
----------------
Install controller-0 (on an AIO-SX or AIO-DX system)
    The controller should be locked-online

Unlock controller-0
    Validate the host status in horizon reports: "Unlocking active controller,
please stand-by while it reboots

Validate that the controller only reboots once after unlock

Expected Behaviour:
-----------------------------
As above



Test ID: INSTALLATION_AND_CONFIGURATION_TBOOT_12
Test Title: test_01: SB(Dis) ESP(En) Con0(UEFI,TXT on) Con1(UEFI TXT on)
Com(UEFI TXT on) Stg(UEFI TXT on)
Tags: P4, Installation and configuration, Regression

Testcase Objective:
--------------------------------
To ensure the tboot configuration as title shown working well on StartlingX

Test Pre-Conditions:
--------------------------
NA

Test Steps:
----------------
01: SB(Dis) ESP(En) Con0(UEFI,TXT on) Con1(UEFI TXT on) Com(UEFI TXT on)
Stg(UEFI TXT on)
All nodes: TBoot functional

Expected Behaviour:
-----------------------------
System boot successfully



Test ID: INSTALLATION_AND_CONFIGURATION_SECURITYBOOT_13
Test Title: test_UEFI Secure Boot enabled TiS WRS Cert in firmware install
compute successful
Tags: P2, Installation and configuration, Regression

Testcase Objective:
--------------------------------
 To ensure the security boot configuration as title shown working well on
StartlingX

Test Pre-Conditions:
--------------------------
NA

Test Steps:
----------------
Enable UEFI secure boot on compute host
Install TiS WRS cert from firmware of compute host
Attempt to install compute host from controller-0

Expected Behaviour:
-----------------------------
compute host must install successfully from controller-0 when TiS WRS cert
is present in compute host's firmware



Test ID: INSTALLATION_AND_CONFIGURATION_SECURITYBOOT_14
Test Title: test_UEFI Secure Boot enabled TiS WRS Cert in firmware install
controller-1 successful
Tags: P2, Installation and configuration, Regression

Testcase Objective:
--------------------------------
 To ensure the security boot configuration as title shown working well on
StartlingX

Test Pre-Conditions:
--------------------------
NA

Test Steps:
----------------
Enable UEFI secure boot on controller-1
Install TiS WRS cert from firmware of controller-1
Attempt to install controller-1 from controller-0

Expected Behaviour:
-----------------------------
Controller-1 must install successfully from controller-0 when TiS WRS cert
is present in controller-1 firmware



Test ID: INSTALLATION_AND_CONFIGURATION_SECURITYBOOT_15
Test Title: test_UEFI Secure Boot enabled TiS WRS Cert in firmware install
low latency CPE controller-0 successful
Tags: P2, Installation and configuration, Regression

Testcase Objective:
--------------------------------
 To ensure the security boot configuration as title shown working well on
StartlingX

Test Pre-Conditions:
--------------------------
NA

Test Steps:
----------------
Enable UEFI secure boot on CPE controller-0 host
Install TiS WRS cert into firmware
Attempt to install a low latency controller-0 CPE host

Expected Behaviour:
-----------------------------
Low latency CPE controller-0 host must install successfully in UEFI secure
boot mode when TiS WRS cert is present in firmware



