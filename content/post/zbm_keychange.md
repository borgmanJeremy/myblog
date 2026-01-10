+++ 
title = "Change encryption password with ZFSBootMenu"
date = 2025-01-06
author = "Jeremy Borgman"
tags = ['ZFS', 'zbm', 'encryption']
+++

# Introduction
I've been enjoing ZfsBootMenu for some time now. One issue I encounted was trying to rotate my encryption key. When I first set up ZBM I followed the [upstream encryption instructions](https://docs.zfsbootmenu.org/en/v3.1.x/general/native-encryption.html) However I did not find a clear guide on how to change the password later.


# Process
## Change Passphrase
Since the passphrase is stored in /etc/zfs/zroot.key, the first step is to edit that file and replace the old passphrase with the new passphrase. Note the permission of this file is supposed to be 000, so you may need to change it to be editable, then change it back to 000.

## Change ZFS key
```
zfs change-key -o keylocation=file:///etc/zfs/zroot.key zroot
```

## Rebuild initramfs
As part of the original encryption instructions, the zroot.key file is embedded in the initramfs. Since this file was edited we need to rebuild the initramfs:

```
mkinitcpio -P
```

If you have multiple OS partitions the initramfs will need to be rebuilt for each of them. 

For example my second dataset is an ubuntu dataset so I will adjust the mount point, setup a chroot, enter the chroot, change the key, rebuild initrafmfs:

```
zfs set mountpoint=/mnt/ubuntu zroot/ROOT/ubuntu
zfs mount zroot/ROOT/ubuntu
mount -t proc proc /mnt/ubuntu/proc
mount -t sysfs sys /mnt/ubuntu/sys
mount -B /dev /mnt/ubuntu/dev
mount -t devpts pts /mnt/ubuntu/dev/pts
chroot /mnt/ubuntu /bin/bash
```

Now edit the key file in the chroot at /etc/zfs/

And then rebuild the initramfs in the chroot (must set path first).
```
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
/sbin/update-initramfs -u -k all
```

Then unmount:

```
umount /mnt/ubuntu/proc
umount /mnt/ubuntu/sys
umount -l /mnt/ubuntu/dev/pts
umount -l /mnt/ubuntu/dev
```

Then set back the mountpoint
```
sudo zfs set mountpoint=/ zroot/ROOT/ubuntu
```
Now when you reboot both datasets should be encrypted with the new key.
