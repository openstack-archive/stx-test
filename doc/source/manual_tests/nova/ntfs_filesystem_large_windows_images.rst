========================================
NTFS Filesystem and Large Windows Images
========================================

NTFS tools will be available directly on all nodes with this feature.
The device under test will only support USB (and not partition the server
disk).

USB formatted with NTFS filesystem must be recognized.
This is required for > 4GB window images (ie. where file size limit exceeds
the FAT32 limit of 4GiB)

The tests will confirm that the required rpms are installed in TiS by default
so that the ntfs tools (both ntfs-3g and ntfsprogs) are available directly on
the nodes.

This HLTP will also include tests as follows:

1. Formatting a newly created partition or USB device with NTFS format.
   This will be tested on nodes with different personalities ie. worker,
   controller:

   - The device eg. USB can be formatted (from a windows system) in NTFS format

2. The NTFS device can be mounted and a large file created/deleted:

   - Confirm Large file(s) >4GB can be written or deleted from the mount
     point.

3. Label update on the NTFS formatted device.
4. Large windows test image file (win2016_cygwin_compressed.qcow2 7.48GB) can
   be copied to the NTFS mount point.
5. Unmount of the NTFS device can be performed (if not busy).
6. Windows image can be imported (glance-image create) and instance can be
   successfully launched using the glance image created from the file at the NTFS
   mount point.

NTFS RPM Tools

In order to support NTFS format the following rpms will be required on the
node(s):

- ntfs-3g-2017.3.23-1.el7.x86_64.rpm and
- ntfsprogs-2017.3.23-1.el7.x86_64.rpm

The following file should exist on the node /usr/sbin/mkfs.ntfs. Useful
Commands:

.. code:: sh

   sudo lsblk -o NAME,FSTYPE,SIZE,MOUNTPOINT,LABEL

.. code:: sh

   $ sudo mount -t ntfs /dev/<device> /media/ntfs

or alternatively:

.. code:: sh

   $ sudo ntfs-3g /dev/<device> /media/ntfs
   $ sudo parted -l /dev/<device>
   $ df -h /media/ntfs
   $ sudo umount /media/ntfs

Natively, you cannot store files larger than 4 GB on a FAT file system. The 4
GB barrier is a hard limit of FAT you cannot copy a file that is larger than 4
GiB to any plain-FAT volume. NTFS is able to support this though. Test
creation of large file(s) >4GB to usb drive formatted in NTFS /create large
file(s):

.. code:: sh

   $ dd if=/dev/zero of=testfile bs=1024 count=5120000

Test copy (scp) of large img file eg. windiows file, to the NTFS mount point:

.. code:: sh

   <host>:/media/ntfs$ df -h /media/ntfs
   Filesystem      Size  Used Avail Use% Mounted on

   /dev/sXX#        <size>G   <used>G   <avail>G  <usedperc.>% /media/ntfs

-----------------
Test Requirements
-----------------

Tbd

.. contents::
   :local:
   :depth: 1

USB formatted with NTFS filesystem.

---------------------
Nova_NTFS_Windows_1.0
---------------------

:Test ID: test_copy_of_large_image_file\_>4GB_file_to/from_NTFS_mount_point_(eg_windows_2016_image)
:Tags: p1, regression, nova
:Sub-domain: Storage

~~~~~~~~~~~~~~
Test Objective
~~~~~~~~~~~~~~

Copy large iso (one that would exceed a FAT32 limit) to/from NTFS mount point.

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. Execute the following commands:

   .. code:: sh

      $ mkdir -p /media/NTFS
      <host>:/$ sudo mount -t ntfs /dev/sdx# /media/ntfs

   Confirm ntfs filesystem type

   .. code:: sh

      <host>:/media/ntfs$ sudo  parted -l
      ...
      Number  Start   End     Size    File system  Name                 Flags
      .
      .
      #        29.9GB  75.0GB  45.1GB  ntfs         LVM Physical Volume

   Use dd to create a large file (>4GB) or copy large windows image file to the
   ntfs mount point:

   - eg. large file(s)copied to NTFS mount point
   - eg. create large file (1.0 GB)

   .. code:: sh

      dd if=/dev/zero of=partition bs=1024 count=1024000

   e.g. create larger file (5.0 GB) with dd:

   .. code:: sh

      <host>:/media/ntfs$ dd if=/dev/zero of=testfile bs=1024 count=5120000
       5120000+0 records in
       5120000+0 records out
       5242880000 bytes (5.2 GB) copied, 90.0053 s, 58.3 MB/s

2. Use scp to copy large windows 2016 iso file to the NTFS mount point:

   .. code:: sh

      <host>:/media/ntfs$ scp
      wrsroot@<ip>:/home/wrsroot/images/[win2016_cygwin_compressed.qcow2
      /media/ntfs/win2016_cygwin_compressed.qcow2


   Note: Host cpu usage alarm appears during scp operation.
   Per info from Jack "There is no more development on ntfs-3g. It's been bought
   by Tuxera. Any future performance fixes would only available in commercial
   version."

   .. code:: sh

      set/clear   100.101 Platform CPU Usage threshold exceeded; threshold: 95%,
      actual: 99.00%.

3. Remove the large file from the ntfs mount point and confirm the use/Avail
space returns:

   .. code:: sh

      <host>:/media/ntfs$ df -h /media/ntfs

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

1. Confirms the ntfs as the file system type.

   .. code:: sh

      $ sudo parted -l

2. Confirm the filesystem used/available when the large file has been created:

   .. code:: sh

      compute-0:/media/ntfs$ df -h /media/ntfs
      Filesystem      Size  Used Avail Use% Mounted on
      /dev/sdX#        42G  7.6G   35G  18% /media/ntfs

3. Confirm the use/Avail space returns when the large file is removed:

   .. code:: sh

      <host>:/media/ntfs$ df -h /media/ntfs
      Filesystem      Size  Used Avail Use% Mounted on
      /dev/sda6        42G   66M   42G   1% /media/ntfs

---------------------
Nova_NTFS_Windows_2.0
---------------------

:Test ID: test_glance_image-create_the_large_windows_image_on_the_ntfs_mount_point
:Tags: p1, regression, nova
:Sub-domain: Storage

~~~~~~~~~~~~~~
Test Objective
~~~~~~~~~~~~~~

Glance image-create the large windows image on the ntfs mount point

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. As admin user, use the glance image-create command to create a windows
   image, i.e. create the glance image using a large windows file copied
   to an ntfs mount point.

   .. code:: sh

      $ glance image-create --property os_type=windows --name win_2016wendy
        --disk-format qcow2 --file /media/NTFS/win2016_cygwin_compressed.qcow2
        --container-format bare --visibility public

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

The image is created from the large windows file at the mount point:

.. code:: sh

   | Property         | Value                                |
   | checksum         | c1fb0a12c3dc11c056976fedcd26e6dd     |
   | container_format | bare                                 |
   | created_at       | 2018-05-07T20:49:03Z                 |
   | disk_format      | qcow2                                |
   | id               | 332cf194-ad31-4c06-984f-320a512ab7d2 |
   | min_disk         | 0                                    |
   | min_ram          | 0                                    |
   | name             | win_2016wendy                        |
   | os_type          | windows                              |
   | owner            | ed347b35d6384b8eab70dad27c74c2cf     |
   | protected        | False                                |
   | size             | 8039104512                           |
   | status           | active                               |
   | store            | file                                 |
   | tags             | []                                   |
   | updated_at       | 2018-05-07T20:49:29Z                 |
   | virtual_size     | None                                 |
   | visibility       | public

---------------------
Nova_NTFS_Windows_3.0
---------------------

:Test ID: test_nova_boot_windows_image_(ie._boot_from_glance_image_created_from_file_on_ntfs_mount_point)
:Tags: p2, regression, nova
:Sub-domain: Storage

~~~~~~~~~~~~~~
Test Objective
~~~~~~~~~~~~~~

Nova instantiates from the windows glance image (ie. can boot from the glance
image that is on the ntfs mount point).

~~~~~~~~~~~~~~~~~~~
Test Pre-Conditions
~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~
Test Steps
~~~~~~~~~~

1. As tenant user (eg. tenant2), use the $nova boot command to boot an
   instance from the windows image; creating volume eg. size 32 GiB, 4 VCPU,
   4GB RAM, Disk 10GB (i.e. Booting from the windows glance image created
   from the file on the ntfs mount point):

   .. code:: sh

      nova boot --flavor <flavorname> <instancename> --nic net-id=<net-id>
      --image <glanceimagename>

As tenant user, using Horizon, launch an instance directly from a large
win2016 glance image (that was created from the NTFS mount point).
Specify a flavor with eg. RAM 4GB, VCPUs 4GB and disk  30GB

2. Attempt to launch an instance directly from the win2016 glance image but
   with a flavor that has a disk that is too small, e.g. using flavor with
   RAM 2GB, VCPUS 2, Disk 8GB.

3. Attempt to launch and instance from a cinder volume created from the
   windows 2016 image, but the volume size is too small for the image.
   e.g. specify cinder volume size 8GB or 28GB. Confirm error feedback
   on instance launch if the volume size specified is too small to launch
   from the windows image specified.

4.Launch the instance from a cinder volume (created from the windows 2016
  glance image) where the volume size is sufficient for e.g.:

  - cinder volume size 32GiB
  - cinder volume size 29GiB

~~~~~~~~~~~~~~~~~
Expected Behavior
~~~~~~~~~~~~~~~~~

1. Confirm the windows instance launches from the glance image if the disk
   size specified is large enough.

2. Confirm error feedback on instance launch if the disk size specified is too
   small to launch with the windows image specified. The feedback indicates
   that disk is too small:

   .. code:: sh

      Message Build of instance <id> aborted: Flavor\'s disk is too small for
      requested image. Flavor disk is 8589934592 bytes, image is 31138512896 bytes.
      Code 500

3. Horizon reports error 500 error instance is in error state:

   .. code:: sh

      Build of instance <id> aborted: Volume <vol_id> creation did not finish after
      46 seconds or 16 attempts. Status is error.
      Error creating volume. Message from driver: Image 332cf194-ad31-4c06-984

   The cinder-volume.log reports the following error.

   .. code:: sh

    ERROR oslo_messaging.rpc.server ImageUnacceptable: Image
    332cf194-ad31-4c06-984f-320a512ab7d2 is unacceptable: Image virtual size is
    29GB and doesn\'t fit in a volume of size 8GB.

4. Confirm the windows instance launches with cinder if the volume size is
   large enough to launch.
