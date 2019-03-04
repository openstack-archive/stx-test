========
Snapshot
========

An instance can be booted from a volume snapshot under various conditions:

- with/without snapshot description.
- with force lock flag set to false (should not allow creating snapshot from
  volume in-use).
- with force lock flag set to true (should allow creating snapshot from volume
  in-use).

-----------------
Test Requirements
-----------------

Tbd

.. contents::
   :local:
   :depth: 1

-----------------
Nova_Snapshot_1.0
-----------------

:Test ID: test_instance_boot_from_volume_snapshot_(using_cli),_storage_Local_CoW_Image,_without_snapshot_description
:Test Title:
:Tags: p1,regression,nova

~~~~~~~~~~~~~~
Test Objective
~~~~~~~~~~~~~~

Volume Snapshots created without description. Local_CoW Image Backed storage
and instantiation.

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

Precondition: worker(s) with local_image backing.

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. Create a Volume i.e. create volume from an image (where image optionally
   has additional metadata such as hw_cpu_policy dedicated or shared) and
   specifying Nova as the availability zone.

2. Run cinder list command to get the volumeid

   .. code:: sh

      $ cinder list

   Test Scenario a. Create Snapshot Without Display Description.

3. Snapshot create without --display-description parameter
   Create snapshot from volume without any display-description specified

   .. code:: sh

      $ cinder snapshot-create --display-name WendySnap1 <volumeid>

4. List and show the newly created snapshot

   .. code:: sh

      $ cinder snapshot-list

5. Run cinder snapshot-show <snapshotid> to view the properties and values of the
   snapshot.

   .. code:: sh

      $ cinder snapshot-show <snapshotid>

   e.g.

   .. code:: sh

      $ cinder snapshot-show <id> displays property as follows:

        created_at
        display_description
        display_name
        id
        metadata
        os-extended-snapshot-attributes:progress
        os-extended-snapshot-attributes:project_id
        size
        status
        volume_id
        wrs-snapshot:backup_status

6. Look for failures in cinder log on snapshot creation.

   .. code:: sh

      $ grep <snapshotid> /var/log/cinder/<*>.log

7. Launch as instance from the newly created volume snapshot where the storage
   type specified in the flavor is local_image.

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

No failures in the cinder log on snapshot creation. Instantiation from the
snapshot is successful.

-----------------
Nova_Snapshot_2.0
-----------------

:Test ID: test_instance_boot_from_volume_snapshot_(using_cli),_storage_Local_CoW_Image,_with_snapshot_description
:Test Title: Tbd
:Tags: p2, regression, nova

~~~~~~~~~~~~~~
Test Objective
~~~~~~~~~~~~~~

Volume Snapshots created with description. Instantiation.

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

Workers(s) with local_image backing.

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. Create Volume. Create volume from an image (where image optionally has
   additional metadata such as hw_cpu_policy dedicated or shared) and
   specifying Nova as the availability zone.

2. Run cinder command to get the volumeid.

   .. code:: sh

      $ cinder list

3. Test scenario b. Snapshot create with Display Description. 
   Create snapshot from volume using option --display-description:

   .. code:: sh

      $ cinder snapshot-create --display-name WendySnap1 --display-description
        somedescription <volumeid>

4. List the newly created snapshot:

   .. code:: sh

      $ cinder snapshot-list

5. Run cinder snapshot-show <snapshotid> to view the properties and values
   of the snapshot:

   .. code:: sh

      $ cinder snapshot-show <snapshotid>

   Displays property as follows:

   .. code:: sh

      created_at
      display_description
      display_name
      id
      metadata
      os-extended-snapshot-attributes:progress
      os-extended-snapshot-attributes:project_id
      size
      status
      volume_id
      wrs-snapshot:backup_status

4. Look for failures in cinder log on snapshot creation.

   .. code:: sh

      $ grep <snapshotid> /var/log/cinder/<*>.log

5. Launch as instance from the newly created volume snapshot where
   the storage type specified in the flavor is local_image.

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

No failures in the cinder log on snapshot creation. Instantiation from
the snapshot is successful.

-----------------
Nova_Snapshot_3.0
-----------------

:Test ID: test_instance_boot_from_volume_snapshot_(using_cli),_storage_Local_CoW_Image,_with_--force_false
:Test Title:
:Tags: p2, regression, nova

~~~~~~~~~~~~~~
Test Objective
~~~~~~~~~~~~~~

Attempts to create a snapshot from volume that is in-use should be detected
and result in an error where the --force flag is false.

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. Create new image as admin user.
2. As tenant user create volume from the image.
3. Launch instance from the volume to attach. To get the volume-id run:

   .. code:: sh

      $ cinder list

4. Test Scenario c. Snapshot created with Force flag false. Attempt to create
   a snapshot from volume using the optional –force flag set to False
   (the default) where the instance has been attached to the volume already.
   E.g.:

   .. code:: sh

      $ cinder snapshot-create --display-name WendySnapForceFalse --force
        False --display-description withforceflagFalse <volumeid>

  Try also allcaps FALSE for the force flag. E.g.

   .. code:: sh

      $ cinder snapshot-create --display-name WendySnapForceFalse --force
        FALSE --display-description withforceflagFalse <volumeid>

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

Ensure the volume snapshot can not be created and an appropriate error is
returned. Where the force flag is set to false, attempts to create a snapshot from
volume that is in-use should be detected and result in an error:

   .. code:: sh

      ERROR: Invalid volume: Volume <volumeid> status must be available, but current
      status is: in-use. (HTTP 400)

-----------------
Nova_Snapshot_4.0
-----------------

:Test ID: test_instance_boot_from_volume_snapshot_(using_cli),_storage_Local_CoW_Image,_with_--force_true
:Test Title:
:Tags: p2, regression, nova

~~~~~~~~~~~~~~
Test Objective
~~~~~~~~~~~~~~

Creating a snapshot from volume that is in-use should be detected and still be
allowed if the --force flag is true.

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. Create new image as admin user.
2. As tenant user create volume from the image.
3. Launch instance from the volume to attach. Run:

   .. code:: sh

      $ cinder list to get the volumeid

4. Test Scenario d. Snapshot created with Force flag true
Create a snapshot from volume using the optional –force flag set to True (or
eg. TRUE)
(The force flag True is used to snapshot a volume even if it is attached to an
instance

   .. code:: sh

      $ cinder snapshot-create --display-name WendySnapForceTrue --force TRUE
        --display-description withforceflagTRUE <volumeid>

5. Ensure the volume snapshot can be listed and shown from the cli

   .. code:: sh

      $ cinder snapshot-list

6. Run cinder snapshot-show <snapshotid> to view the properties and values of the
   snapshot:

   .. code:: sh

      $ cinder snapshot-show <snapshotid>

   Displays property as follows:

   .. code:: sh

        created_at
        display_description
        display_name
        id
        metadata
        os-extended-snapshot-attributes:progress
        os-extended-snapshot-attributes:project_id
        size
        status
        volume_id
        wrs-snapshot:backup_status

6. Look for failures in cinder log on snapshot creation.

   .. code:: sh

      $ grep <snapshotid> /var/log/cinder/<*>.log

7. Launch as instance from the newly created volume snapshot where the storage
   type specified in the flavor is local_image.

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

Cinder snapshot lists and shows appropriate values and no errors in cinder log
on creation. The volume snapshot is created successfully.

   .. code:: sh

      | Property | Value
      | display_description | withforceflagTRUE
      | display_name | WendySnapForceTrue
      | id | <id>
      | metadata | {}
      | size | 1
      | status | creating
      | volume_id | <volumeid>

An instance successfully launches from the new volume snapshot.
