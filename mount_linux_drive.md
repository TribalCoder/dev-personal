# Auto mount a drive in Ubuntu linux

---
Related Stuff:
* setup self signed SSL Certificate: https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-apache-in-debian-10
* How to configure OwnCloud to use the data-disk for storage: https://my.simplercloud.com/index.php?/knowledgebase/article/154/how-to-configure-owncloud-to-use-the-data-disk-for-storage-/

* Locate the partition to mount
The first thing to be done is to locate the partition you want to mount. In this case, we'll be working with an entire drive. To do this, open a terminal window and issue the command:
```
$ sudo fdisk -l
```

* Locate the UUID
Next we need to find the UUID (Universal Unique Identifier) of the drive. To do that, issue the command:
```
$ sudo blkid
```

* Create a mount point
Before we add the entry to fstab, we must first create a mount point for the drive. The mount point is the directory where users will access the data on the drive (as they can't access /dev/sdj itself). So let's create a directory called data with the command:
```
$ sudo mkdir /data
```
You'll want to also change the group ownership of that directory, so that users can access it. For this, you might create a group called data and then add users to the new group. This could be done with the commands:
```
$ sudo groupadd data
$ sudo usermod -aG data USERNAME (Where USERNAME is the name of the user to be added)
```
After you've done that, you could then change the ownership of the mountpoint with the command:
```
$ sudo chown -R :data /data
```

* Create auto mount entry
In order to create the automount entry, issue the command:
```
$ sudo vim /etc/fstab
```
At the bottom of that file, we'll add an entry that contains the information we've discovered. The entry will look like this:
```
UUID=14D82C19D82BF81E /data    auto nosuid,nodev,nofail,x-gvfs-show 0 0
```
Breaking that line down, we have:
- ***UUID=14D82C19D82BF81E*** - is the UUID of the drive. You don't have to use the UUID here. You could just use /dev/sdj, but it's always safer to use the UUID as that will never change (whereas the device name could).
- ***/data*** - is the mount point for the device.
- ***auto*** - automatically mounts the partition at boot 
- ***nosuid*** - specifies that the filesystem cannot contain set userid files. This prevents root escalation and other security issues.
- ***nodev*** - specifies that the filesystem cannot contain special devices (to prevent access to random device hardware).
- ***nofail*** - removes the errorcheck.
- ***x-gvfs-show*** - show the mount option in the file manager. If this is on a GUI-less server, this option won't be necessary.
- ***0*** - determines which filesystems need to be dumped (0 is the default).
- ***0*** - determine the order in which filesystem checks are done at boot time (0 is the default).
Save and close the file.

* Testing the entry
Before you reboot the machine, you need to test your new fstab entry. To do this, issue the command:
```
$ sudo mount -a
```
If you see no errors, the fstab entry is correct and you're safe to reboot.

Congratulations, you've just created a proper fstab entry for your connected drive. Your drive will automatically mount every time the machine boots.

* umount will un-mount mounts
```
$ sudo umount /mnt/sdn
```
---

## Notes from "setup samba"
## setup samba
[https://ubuntu.com/tutorials/install-and-configure-samba#4-setting-up-user-accounts-and-connecting-to-share](https://ubuntu.com/tutorials/install-and-configure-samba#4-setting-up-user-accounts-and-connecting-to-share)
[https://www.fosslinux.com/8703/how-to-setup-samba-file-sharing-server-on-ubuntu.htm](https://www.fosslinux.com/8703/how-to-setup-samba-file-sharing-server-on-ubuntu.htm)

* IP address: 192.168.200.29
```
-- list disks
$ sudo lsblk -e7

-- list partitions
$ sudo parted -l

-- mount drives
$ mkdir -p /mnt/sdb1
$ mkdir -p /mnt/sdb2
$ mount -t auto /dev/sdb1 /mnt/sdb1
$ mount -t auto /dev/sdb2 /mnt/sdb2

-- To check if the partitions are actually mounted, run the df command to report file system disk space usage.
$ df -hT
```
```
$ sudo passwd naynay
$ sudo groupadd naynaygroup
$ sudo adduser naynay naynaygroup
```
* be sure to add your password for samba
```
$ sudo smbpasswd -a naynay
```

* add mount to samba
```
$ sudo mkdir -p /mnt/sdb1/sambaprivate
$ sudo chown -R root:smbgroup /mnt/sdb1/sambaprivate/
$ sudo chmod -R 0770 /mnt/sdb1/sambaprivate/
$ sudo vim /etc/samba/smb.conf
```
* smb.conf entry
```
[SambaPrivate]
path = /mnt/sdb1/sambaprivate
valid users = @smbgroup
guest ok = no
writable = yes
browsable = yes

[NayNay]
path = /mnt/sda1/naynay
valid users = @naynaygroup
guest ok = no
writable = yes
browsable = yes
```
* restart service
```
$ sudo service smbd restart
$ service --status-all
```