=======
GNOCCHI
=======

Gnocchi is an open-source time series database, the problem that Gnocchi solves is the storage and indexing of time series data and resources at a large scale. This is useful in modern cloud platforms which are not only huge but also are dynamic and potentially multi-tenant. Gnocchi takes all of that into account.
Gnocchi has been designed to handle large amounts of aggregates being stored while being performant, scalable and fault-tolerant. While doing this, the goal was to be sure to not build any hard dependency on any complex storage system.
Gnocchi takes a unique approach to time series storage: rather than storing raw data points, it aggregates them before storing them. This built-in feature is different from most other time series databases, which usually support this mechanism as an option and compute aggregation (average, minimum, etc.) at query time.
Because Gnocchi computes all the aggregations at ingestion, getting the data back is extremely fast, as it just needs to read back the pre-computed results.
 

--------------------
Overall Requirements
--------------------

StarlingX Environemnt setup

----------
Test Cases
----------


.. contents::
   :local:
   :depth: 1

~~~~~~~~~~
Gnocchi_01
~~~~~~~~~~

:Test ID: Gnocchi_01
:Test Title: Logs - gnocchi api.log reports listening (address and port in gnocchi-api.conf)
:Tags: Gnocchi

++++++++++++++
Test Objective
++++++++++++++

api and metric files for running the metric daemon

+++++++++++++++++++
Test Pre-Conditions
+++++++++++++++++++

StarlingX Environment setup

++++++++++
Test Steps
++++++++++
  
1. Confirm new gnocchi api and metricd files on the controllers in the following location
 
 ::

   controller-0:/etc/init.d# ls -l | grep gnocchi
   -rwxrwxr-x. 1 root root  ... gnocchi-api
   -rwxrwxr-x. 1 root root  ... gnocchi-metricd

2. Confirm the new gnocchi config files and py files in the /usr/share/gnocchi location

 ::

   controller-X:/usr/share/gnocchi$ ls
   gnocchi-api.conf
   gnocchi-dist.conf
   gnocchi-api.py
   gnocchi-api.pyc
   gnocchi-api.pyo

3. The gnocchi log & config file locations specified in the /etc/init.d/gnocchi-api file.

 ::

   gnocchi-api.conf (specifies bind address and number of workers)
   CONFIGFILE="/usr/share/gnocchi/gnocchi-api.conf"
   eg. bind='<ipaddr:port>' eg. 192.168.204.2:8041
   workers=# eg. workers=10

4. Confirm log folder as specified in gnocchi-dist.conf 

 ::

   eg. default folder #log_dir is /var/log/gnocchi

5. Confirm new gnocchi log folder and logs have been created

 ::

   eg. LOGFILE="/var/log/gnocchi/api.log"

+++++++++++++++++
Expected Behavior
+++++++++++++++++

1. Files gnocchi api and metricd should appear under /etc/init.d
2. New gnocchi config files and py files in the /usr/share/gnocchi are in place
3. Gnocchi log & config file locations should be specified in the /etc/init.d/gnocchi-api file
4. Folder as specified in gnocchi-dist.conf is confirmed
5. New gnocchi log folder and logs have been created
