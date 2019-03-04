=========================
Log Nova-Api-Proxy Events
=========================

This is part of Nova server actions. An admin user using the remote cli
client can issue nova commands and the events are recorded in the
nova-api-proxy.log. Confirm /var/log/nova_api_proxy.log output for each
of the following where the NFV action can be extracted from the payload
of the message. The log will indicate the NFV action as extracted from
the payload of the message. The ip address, the POST request issued on
the server, the <instanceid> of the server and by whom the request was
issued (user authenticated and their project user admin tenant admin).

-----------------
Test Requirements
-----------------

Tbd

.. contents::
   :local:
   :depth: 1


--------------------
Nova_APIProxyLog_1.0
--------------------

:Test ID: test_proxy_logs_cli_admin_novaactions_payload_remote_1.0
:Tags: p2, regression, nova, DX, admin

~~~~~~~~~~~~~~
Test Objective
~~~~~~~~~~~~~~

As admin user, I want to see nova actions performed via remote cli from
off-box, logged to the nova-api-proxy.log
The nova-api-proxy.log will indicate the NFV action as extracted from the
payload of the message for these actions.


~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. Connect to the controller user the remote cli client (off box) as an admin
   user.
2. Perform the following nova actions:

   .. code:: sh

      $ nova stop <instanceid>
      $ nova start <instanceid>
      $ nova suspend <instanceid>
      $ nova resume <instanceid>
      $ nova pause <instanceid>
      $ nova unpause <instanceid>

2. Perform nova Resize operations:

   .. code:: sh

      $ nova resize <instanceid> --poll <instanceid> <newflavor>
      $ nova resize-revert <instanceid> (or resize-confirm)

3. Perform instance Soft reboot actions:

   .. code:: sh

      $ nova reboot <instanceid>

4. Perform nova Migration operations on the Instance
   Cold migrate and Confirm:

   .. code:: sh

      $ nova migrate <instanceid>
      $ nova resize-confirm <instanceid>
      $ nova live-migration <instanceid>

5. Perform instance Rebuild operatons:

   .. code:: sh

      $ nova rebuild --poll --name <name> <instanceid>

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

Confirm /var/log/nova_api_proxy.log output for each of the following where the
NFV action can be extracted from the payload of the message. The log will
indicate the NFV action as extracted from the payload of the message, the ip
address, the POST request issued on the server, the <instanceid> of the server
and by whom the request was issued. In this test it will be the admin user
authenticating (project user admin tenant admin).


--------------------
Nova_APIProxyLog_2.0
--------------------

:Test ID: test_proxy_logs_cli_admin_novaactions_remote_2.0
:Tags: p2,regression,nova,DX,admin

~~~~~~~~~~~~~~
Test Objective
~~~~~~~~~~~~~~

As admin user, I want to see nova actions performed via remote cli from
off-box, logged to the nova-api-proxy.log. The nova-api-proxy.log will not
indicate these NFV actions that are not included in the payload of the
message. It will include the POST request issued by the project admin.

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

Connect to the controller user the remote cli client (off box) as an admin
user.

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. Connected to the controller user the remote cli client (off box) as an
   admin user, perform the following nova actions:

   .. code:: sh

      nova shelve <instanceid>
      nova unshelve <instanceid>
      nova lock <instanceid>
      nova unlock <instanceid>
      nova reset-state <instanceid>
      nova rescue <instanceid>

2. Abort an on-going live migration or Force on-going live migration to
   complete:

   .. code:: sh

      $ nova migration-list
      $ live-migration-abort <instanceid> <migrationid> (to abort the on-going live
        migration)
      $ live-migration-force-complete <instanceid> <migrationid> (to force the
        on-going live migration to complete)
      $ nova scale 1f8a9a0d-e33a-4d2e-9609-be976ce5a513 cpu down

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

Confirm /var/log/nova_api_proxy.log output for each of the following POST
operations where the NFV action is *NOT* included in the payload of the
message. The log will include the ip address, the POST request issued on
the server, the <instanceid> of the server and by whom the request was
issued (user authenticated and their project user admin tenant admin).
The log will not indicate the NFV action (as it is not included in the
payload of the message).


--------------------
Nova_APIProxyLog_3.0
--------------------

:Test ID: test_proxy_logs_cli_tenant_novaactions_remote
:Tags: p2, regression, nova, tenant

~~~~~~~~~~~~~~
Test Objective
~~~~~~~~~~~~~~

As a tenant user using the remote cli client (off box), run nova actions to
confirm nova_api_proxy.log output for each. Perform Nova commands with NFV
action (listed below) as the tenant user. Attempt Nova actions even though
not allowed by policy: NFV action should be logged in the
nova-api-proxy.log.

As a tenant user using the remote cli client (off box), run nova actions to
confirm nova_api_proxy.log output for each. Perform Nova commands with NFV
actions (listed below) as the remote tenant user (off box). Perform nova
actions ensuring that the nova-api-proxy.log includes the POST request and
the NFV action as extracted from the payload of the message.

Attempt Nova actions even though not allowed by policy: NFV action should be
logged in thenova-api-proxy.log. Log also nova actions attempted by tenant
user where the user is not allowed by policy.

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

Tenant user connects using the remote cli (offbox).

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. Perform the following nova actions (sourced as tenant user) confirming the
   nova_api_proxy.log output for each:
   i.e. nova commands with NFV action (listed below) as the tenant user,
   e.g. tenant2

   Nova commands with NFV action, Pause instance, Unpause instance,
   Suspend instance, Resume instance:

   .. code:: sh

      $ nova suspend <instanceid>
      $ nova resume a6a1e06d-afbf-4fa7-99b0-84a67218292c

   Stop instance, Start instance:

   .. code:: sh

      $ nova stop 026f427e-10c8-4981-a10b-415efe4d2211
      $ nova start 026f427e-10c8-4981-a10b-415efe4d2211

   Resize Instance (or stop instance,resize and confirm), Revert | confirm:

   .. code:: sh

      ~(keystone_tenant2)]$ nova resize-confirm a6a1e06d-afbf-4fa7-99b0-84a67218292c

   Resize revert:

   .. code:: sh

      $ nova resize-revert a6a1e06d-afbf-4fa7-99b0-84a67218292c

   Stop/shutdown – start then reboot the instance (soft and hard reboots):

   .. code:: sh

      ~(keystone_tenant2)]$nova stop <instanceid>

   Stop/shutdown – start then reboot the instance (soft and hard reboots):

   .. code:: sh

      ~(keystone_tenant2)]$nova stop <instanceid>
      ~(keystone_tenant2)]$nova start <instanceid>
      ~(keystone_tenant2)]$nova reboot <instanceid>
      ~(keystone_tenant2)]$ nova reboot --poll --hard <instanceid>

   Rebuild instance as tenant user:

   .. code:: sh

      (keystone_tenant2)]$ nova rebuild –poll <instancename> <imageid>

2. Perform the following nova actions (sourced as tenant user) even though not
   allowed by policy. Cold migrate (even though not allowed by policy):

   .. code:: sh

      $ nova migrate <instanceid>

    Live block migration (even though not allowed by policy):

   .. code:: sh

      $ nova live-migration --block-migrate <instanceid>

    Live migration (even though not allowed by policy):

   .. code:: sh

      $ nova live-migration <instanceid>

3. Perform the following nova actions (sourced as tenant user). For these Post
   operations the NFV action is not included in the payload of the message:

   .. code:: sh

      (keystone_tenant2)]$nova lock <instanceid>
      (keystone_tenant2)]$ nova unlock <instanceid>

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

1. The nova-api-proxy.log will indicate the NFV action as extracted from the
   payload of the message:

   - ip address
   - the POST request issued on the server
   - The <instanceid> of the server
   - By whom the request was issued (user authenticated and their project
     (tenant <user>)

   Note:
   The requests come from haproxy in this case and as such the haproxy forwarded
   address ie. the <frontendip> as listed in the haproxy.cfg, should be shown in
   the nova-api-proxy.log (as the remote address)

   The following are the frontend and backend internal settings in the
   haproxy.cfg (/etc/haproxy/haproxy.cfg):

   - frontend nova-ec2-restapi
   - bind <frontendip>:port
   - default_backend nova-ec2-restapi-internal
   - reqadd X-Forwarded-Proto:\ http
   - backend nova-ec2-restapi-internal
   - server s-nova-ec2 <backendinternalip>:port

2. The NFV action should be logged in the nova-api-proxy.log
   The nova-api-proxy.log will indicate the NFV action as extracted from the
   payload of the message including:

   - IP address
   - The POST request issued on the server
   - The <instanceid> of the server
   - By whom the request was issued (user authenticated and their project
     (tenant <user>)

3. The nova-api-proxy.log will include the ip address, the POST request issued
   on the server:

   - The <instanceid> of the server
   - By whom the request was issued (user authenticated and their project
     (tenant <user>)

--------------------
Nova_APIProxyLog_4.0
--------------------

:Test ID: test_new_proxy_logs_exists_in_log_collection
:Tags: p2, regression, nova

~~~~~~~~~~~~~~
Test Objective
~~~~~~~~~~~~~~

New nova-api-proxy.log exists after installation and log collection includes
this log. A new log "nova-api-proxy.log" will be created in the /var/log folder
and the log collector will include this log.

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. On a lab that has been installed, navigate to the /var/log location and
   confirm the new log file nova-api-proxy.log exists. The new log file exists in
   this location.

2. Perform log collection

   .. code:: sh

      $ collect  all

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

Confirm collection completes successfully and that the new file
nova-api-proxy.log is included in the collected logs (for both controllers)
