====
TPM
====

.. contents::
   :local:
   :depth: 1

----------------
SECURITY_TPM_01
----------------

Test ID: SECURITY_TPM_01
Test Title:  test_TPM install cert-with-key standard system
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

1) Install 2+2 lab with ENABLE_HTTPS = Y
2) Follow the procedure to update the https certificate from the Installation
    Guide to install https private key in TPM
    system certificate-install -m tpm_mode -p Li69nux* ./wc99-103.pem
3) Swact controllers

~~~~~~~~~~~~~~~~~~~~
Expected Behaviour:
~~~~~~~~~~~~~~~~~~~~

1) Verify that horizon is only accessible via https
2) Verify that the private key is not present on the controller-0 and
    controller-1 filesystem
3) After swacting controllers, verify that horizon is only accessible via
    https


