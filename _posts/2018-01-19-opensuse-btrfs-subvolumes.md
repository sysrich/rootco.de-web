---
layout: post
title: Creating openSUSE-style btrfs root partition & subvolumes
date: '2018-01-19 09:54:10'
---
openSUSE's [YaST](https://yast.github.io) installer creates a detailed btrfs root filesystem configuration that has been designed to be flexible and secure while still efficient when used with tools like [Snapper](https://snapper.io).

However this does lead to complications for some advanced users who wish to recreate this manually, such as when doing complex system recovery, custom automated provisioning or other tinkering. (NOTE: for Full System Recovery it is often better to use a tool like [ReaR](https://en.opensuse.org/SDB:Disaster_Recovery)).

The below steps are the steps to manually create an openSUSE-style btrfs partition believed to be correct at time of writing (19 Jan 2018).

This guide should be valid for **openSUSE Tumbleweed 20180117**, **openSUSE Leap 15**, and **SUSE Linux Enterprise 15** or later. However care should be taken to double check for new or removed subvolumes in SUSE distributions as this document gets older.

Older versions of SUSE distributions will need to adjust these instructions to handle the [old /var/* subvolume layout](https://en.opensuse.org/SDB:BTRFS#Old_.2Fvar.2F.2A_subvolume_layout_.28pre_Jan_2018.29) previously used.

It should go without saying that this guide should only be followed by people who feel that they know what they are doing. It's normally a lot easier to use openSUSE's default tools like YaST and ReaR.

## Step-by-Step

For this example we will use `/dev/sda` as our example disk and `/dev/sda1` as our example partition for a btrfs root filesystem.

**1:** Create a partition table and the partition to be used as our root filesystem using your favourite tool (eg. `yast`, `parted`, `fdisk`)

**2:** Format `/dev/sda1` with a btrfs filesystem

```
mkfs.btrfs /dev/sda1
```

**3:** Mount the new partition somewhere so we can work on it. We'll use `/mnt` in this example.

```
mount /dev/sda1 /mnt
```

**4:** Create the default subvolume layout (this assumes an intel architecture, the /boot/grub2/* paths are different for different architectures)

```
btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@/.snapshots
mkdir /mnt/@/.snapshots/1
btrfs subvolume create /mnt/@/.snapshots/1/snapshot
mkdir -p /mnt/@/boot/grub2/
btrfs subvolume create /mnt/@/boot/grub2/i386-pc
btrfs subvolume create /mnt/@/boot/grub2/x86_64-efi
btrfs subvolume create /mnt/@/home
btrfs subvolume create /mnt/@/opt
btrfs subvolume create /mnt/@/srv
btrfs subvolume create /mnt/@/tmp
mkdir /mnt/@/usr/
btrfs subvolume create /mnt/@/usr/local
btrfs subvolume create /mnt/@/var
```

**5:** Disable copy-on-write for var to improve performance of any databases and VM images within

```
chattr +C /mnt/@/var
```

**6:** Create `/mnt/@/.snapshots/1/info.xml` file for snapper's configuration. Include the following content, replacing `$DATE` with the current system date/time.

```
<?xml version="1.0"?>
<snapshot>
  <type>single</type>
  <num>1</num>
  <date>$DATE</date>
  <description>first root filesystem</description>
</snapshot>
```

**7:** Set snapshot 1 as the default snapshot for your root file system, unmount it, and remount it.

```
btrfs subvolume set-default $(btrfs subvolume list /mnt | grep "@/.snapshots/1/snapshot" | grep -oP '(?<=ID )[0-9]+') /mnt
unmount /mnt
mount /dev/sda1 /mnt
```

**8:** You should be able to confirm the above worked by doing `ls /mnt` which should repond with an empty result.

Congratulations, at this point the filesystem is 'created' with the correct structure. But you need to know how to mount it properly to make use of it.

**9:** You now need to create a skeleton of the filesystem to mount all of our subvolumes

```
mkdir /mnt/.snapshots
mkdir -p /mnt/boot/grub2/i386-pc
mkdir -p /mnt/boot/grub2/x86_64-efi
mkdir /mnt/home
mkdir /mnt/opt
mkdir /mnt/srv
mkdir /mnt/tmp
mkdir -p /mnt/usr/local
mkdir /mnt/var
```

**10:** Mount all of the subvolumes

```
mount /dev/sda1 /mnt/.snapshots -o subvol=@/.snapshots
mount /dev/sda1 /mnt/boot/grub2/i386-pc -o subvol=@/boot/grub2/i386-pc
mount /dev/sda1 /mnt/boot/grub2/x86_64-efi -o subvol=@/boot/grub2/x86_64-efi
mount /dev/sda1 /mnt/home -o subvol=@/home
mount /dev/sda1 /mnt/opt -o subvol=@/opt
mount /dev/sda1 /mnt/srv -o subvol=@/srv
mount /dev/sda1 /mnt/tmp -o subvol=@/tmp
mount /dev/sda1 /mnt/usr/local -o subvol=@/usr/local
mount /dev/sda1 /mnt/var -o subvol=@/var
```

**11:** You're done, you've now successfully created an openSUSE-style btrfs root filesystem structure and mounted it for use. You can now use it for whatever you'd like, such as the manual injection of files from an existing openSUSE installation. 

Once populated, care should be made to ensure the `/mnt/etc/fstab` also includes the appropriate entries for each of the subvolumes except `@/.snapshots/1/snapshot` which should not be mounted as it provides your initial installed system.

Have a lot of fun
