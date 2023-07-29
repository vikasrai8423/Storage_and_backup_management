# Logical Volume Manager

### To perform the practical add 2 HDD in VM

<p align="center">
    <img src=https://github.com/shubnimkar/Storage-and-Backup-Management/assets/46809421/52a7f9ba-f08e-434c-8f59-e7f2ede02561>
</p>

## 1. Identify the new hard disks:

This is achieved using the `lsblk` command. `lsblk` lists information about all available or the specified block devices. It reads the sysfs filesystem and udev db to gather information.


    [root@nfs-client ~]# lsblk
    
<p align="center">
<img src=https://github.com/shubnimkar/Storage-and-Backup-Management/assets/46809421/025521c0-9ffd-4869-92a6-69b4068dfb3a>
<p>

Here, `sdb` and `sdc` are the new hard disks that you want to add.

## 2. Create physical volumes:

This is done with `pvcreate`. This initializes physical volume(s) for later use by the Logical Volume Manager (LVM). Each physical volume can be a disk partition, whole disk, meta-device, or loopback file.


    pvcreate /dev/sdb /dev/sdc
   
![681f7298-7039-41bb-8089-f4c9acba7db5](https://github.com/shubnimkar/Storage-and-Backup-Management/assets/46809421/fd168069-b8d3-4ee1-bcde-cb4bd9ac7eb6)


pvdisplay command for view volumes

    pvdisplay

![4f00aaea-b9e6-4605-90e0-5c3c0cf19796](https://github.com/shubnimkar/Storage-and-Backup-Management/assets/46809421/c7a80e87-0474-4ccb-913f-cd6ff16d6db8)


## 3. Create a volume group:

`vgcreate` creates a new volume group named DEMO on physical volumes /dev/sdb and /dev/sdc.


    vgcreate HPCSA /dev/sdb /dev/sdc

![18f6c737-6356-4308-b855-1b5731ee5451](https://github.com/shubnimkar/Storage-and-Backup-Management/assets/46809421/810b9b5c-bb59-48da-ac5a-0df692949df2)


vgdisplay to view

    vgdisplay
![baddf797-79d1-4e46-a9c9-f720612f32a4](https://github.com/shubnimkar/Storage-and-Backup-Management/assets/46809421/b0134517-aec7-4356-b32b-62c58378c2cc)


## 4. Create a logical volume:

`lvcreate` creates a logical volume in a volume group. In this case, it's creating a logical volume named `demo_lab` with a size of 1G in the volume group `DEMO`.


    lvcreate -n demo_lab --size 1G DEMO

![448a0eef-2727-421a-bc8b-5ae1b4764b17](https://github.com/shubnimkar/Storage-and-Backup-Management/assets/46809421/a24d92b7-7b47-4b2d-8619-323a8d42a2a0)

lvdisplay to view

    lvdisplay

![f19c322f-66e4-4ce9-b309-29429f40d6b7](https://github.com/shubnimkar/Storage-and-Backup-Management/assets/46809421/c672fb27-3b2d-4df7-ab42-4a6e770c136b)


## 5. Partition the new logical volume:

`fdisk` is a dialogue-driven command-line utility that creates and manipulates partition tables and partitions on a hard disk.


    fdisk /dev/mapper/DEMO-demo_lab

Here you're asked to press `n` to create a new partition.
then press p for primary
then mention size or press enter , enter
then again will ask for option press w to write and exit

![ad514619-d9bb-480d-96f2-09dcff57ae4b](https://github.com/shubnimkar/Storage-and-Backup-Management/assets/46809421/2a99d319-401a-49ef-9edc-97eb73ae2b0b)


    Error:
    WARNING: Re-reading the partition table failed with error 22: Invalid argument.
    The kernel still uses the old table. The new table will be used at
    the next reboot or after you run partprobe(8) or kpartx(8)
    Syncing disks.
    
    [root@localhost ~]# partprobe /dev/HPCSA/hpcsa_lab

![93e6964c-04d1-4bef-aedf-a721a73e5abe](https://github.com/shubnimkar/Storage-and-Backup-Management/assets/46809421/4e154707-709e-41f5-b8b2-4d6a938cbfb1)


## 6. Create a filesystem:

`mkfs.ext4` is used to create an ext4 filesystem on the partition.


    mkfs.ext4 /dev/mapper/DEMO-demo_lab

![dfc69b79-f46a-4b6b-b907-0c0854be6e98](https://github.com/shubnimkar/Storage-and-Backup-Management/assets/46809421/19e23763-850f-4d44-a2bd-e8e4adb6ba63)


## 7. Create a mount point and mount the logical volume:

Create a directory that will serve as the mount point, and then use the `mount` command to mount the logical volume.


    mkdir our-demo
    mount /dev/mapper/DEMO-demo_lab our-demo
    
![1b4accf9-d037-4f7d-991a-397d5cf65364](https://github.com/shubnimkar/Storage-and-Backup-Management/assets/46809421/edf3fe04-15a7-4bf7-94a9-e27e4edeab1c)



## 8. Extend the logical volume:

`lvextend` allows you to extend the size of a logical volume. Here, you're extending the logical volume `hpcsa_lab` by an additional 2G.


    lvextend -L +2G /dev/mapper/DEMO-demo_lab

![a6039f32-78c5-43e4-848f-b68b01e68acf](https://github.com/shubnimkar/Storage-and-Backup-Management/assets/46809421/dd8f1db1-fcc4-4378-9b34-4c608fe6de3c)


## 9. Resize the filesystem:

`resizefs` resizes the filesystem on the logical volume to use all of the available space.


    resize2fs /dev/mapper/DEMO-demo_lab

![7afca6b4-66b4-4c1a-8e3c-5f60931ba297](https://github.com/shubnimkar/Storage-and-Backup-Management/assets/46809421/5039c443-4077-4006-9ae2-0d24eae2bfc5)


## 10. Create a snapshot:

`lvcreate` with `-s` creates a snapshot logical volume, which is a read-only copy of another logical volume.


    lvcreate -L 1GB -s -n demo_lab_snap /dev/mapper/DEMO-demo_lab

![8366f228-41c9-4b4e-ab77-cffd64501280](https://github.com/shubnimkar/Storage-and-Backup-Management/assets/46809421/e8dcb527-846f-4a44-8d87-801ce7db93f6)


## 11. Merge the snapshot:

`lvconvert` with `--merge` will merge the snapshot back into its origin volume. If both origin and snapshot volume are not open the merge will start immediately, otherwise, it will be delayed until the origin volume becomes inactive.

    lvconvert --merge /dev/mapper/DEMO-demo_lab_snap

![f54fe5f0-e836-4e98-8714-6761af42f2ec](https://github.com/shubnimkar/Storage-and-Backup-Management/assets/46809421/2554002e-ccde-434c-b4dd-901907f09a68)


Remember to add the disks to `/etc/fstab` if you want them to be mounted automatically at system startup.

![eff5a421-b11a-4110-931c-67ba9aec36e9](https://github.com/shubnimkar/Storage-and-Backup-Management/assets/46809421/d6d7072b-5019-44c3-b120-22b906322477)

## 12. Mirror

 It creates mirror of data in all physical  drives
 
    lvcreate -L 2GB -m1 -n testmirror DEMO

![7ebcaf7c-d8c7-418b-bc33-a15946050200](https://github.com/shubnimkar/Storage-and-Backup-Management/assets/46809421/17bb1f36-7fd6-420a-b02f-4a711560b190)

  Logical volume "testmirror_lv" created.

## 13. Remove a logical volume

The command lvremove can be used to remove logical volumes. We should make sure a logical volume does not have any valuable data stored on it before we attempt to remove it. Moreover, we should make sure the volume is not mounted.

    lvremove /dev/mapper/DEMO-demo_lab
    
<p align="center">
<img src=https://github.com/shubnimkar/Storage-and-Backup-Management/assets/46809421/512e7080-6899-4561-8bad-54fc92ad032b>
<p>

Now to remove the DEMO volume

    lvremove /dev/mapper/DEMO
    
![4ef8ce8c-4da7-4e11-988e-936149b04e55](https://github.com/shubnimkar/Storage-and-Backup-Management/assets/46809421/84dde010-c3d5-4a56-aa2f-bc8b8d06c837)

