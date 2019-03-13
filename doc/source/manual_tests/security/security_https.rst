=====
HTTPS
=====

.. contents::
   :local:
   :depth: 1

------------------
SECURITY_HTTPS_01
------------------

Test ID: SECURITY_HTTPS_01
Test Title:  test_Query keystone for https endpoints
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

Enable HTTPS
Verify curl cli from external .

  curl -i https://128.224.151.85:18002/v1/event_log?include_suppress=True
  -X GET -H "Content-Type: application/json" -H "Accept: application/json"
  -H "X-Auth-Token:gAAAAABbwMtHOw0TRCatKH-aWZBkK72o05RKaexK1LkdPhRsMTd4C5w
  Xt8qitPJ066pQzUE3P5zO8xzkAmzAfcbcPuGD7ghxJpAApGwrYX4iy3cBall7E-a7MxF9vY3
  i2i6nPJkBtCk  jMXuKhHZMA3r3bHax2Kz5StaGPad8AE8UtmxTes7AhnY"
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

~~~~~~~~~~~~~~~~~~~~
Expected Behaviour:
~~~~~~~~~~~~~~~~~~~~

All test steps are pass

------------------
SECURITY_HTTPS_02
------------------

Test ID: SECURITY_HTTPS_02
Test Title:  test_Validate that services are only listening on https
Tags: P3, Security, Regression, AUTOMATABLE

~~~~~~~~~~~~~~~~~~~~
Testcase Objective:
~~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions:
~~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~~~~~~~~~~~
Test Steps:
~~~~~~~~~~~~~~~~~~~~

1) Log on to active controller, use netstat to determine listening ports
2) Validate that services (e.g. sysinv, Nova, Neutron, patching, cinder,
    glance, ceilometer, and heat) are listening to correct ports
3) On an external host (e.g. linux workstation) telnet to the listening port
4) Validate that telnet connects, type 'GET / \n' and no info is returned
5) From and external host, use telnet to connect to each service
    (telnet <host> <port>)
6) Validate that telnet connects, type 'GET / \n' and no info is returned

~~~~~~~~~~~~~~~~~~~~
Expected Behaviour:
~~~~~~~~~~~~~~~~~~~~

All test steps are pass

------------------
SECURITY_HTTPS_03
------------------

Test ID: SECURITY_HTTPS_03
Test Title:  test_Validate that services respond over https
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

From and external host, use curl to access HTTPS REST API for each service

~~~~~~~~~~~~~~~~~~~~
Expected Behaviour:
~~~~~~~~~~~~~~~~~~~~

Validate that services (e.g. sysinv, Nova, Neutron, patching, cinder, glance,
ceilometer, and heat) respond correctly

------------------
SECURITY_HTTPS_04
------------------

Test ID: SECURITY_HTTPS_04
Test Title:  test_https enabled - https-certificate-install successful
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

This test verifies that https can be successfully enabled post-install and
that server-certificate could be successfully installed
1) Install a standard (2+2) system
2) Enable https post-install

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
      system alarm-list

  * Before proceeding to the next step, the above alarms should be cleared

3) Install a signed certificate:

  [wrsroot@controller-1 ~(keystone_admin)]$ sudo https-certificate-install
  -c server-with-key.pem
  Enter password for the CA-Signed certificate file [Enter <CR> for
  no password]:
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
  openstack endpoint list

5) Verify that Horizon is only accessble via https and that the correct
certificate is presented

~~~~~~~~~~~~~~~~~~~~
Expected Behaviour:
~~~~~~~~~~~~~~~~~~~~

All test steps are pass


