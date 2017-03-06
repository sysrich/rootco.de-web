---
layout: post
title: openSUSE Leap on the Omnia as a csync host
date: '2017-03-05 12:40:12'
---
openSUSE 42.2 in dropdown of omnia webUI

lxc-start -F -n k2so

"Welcome to your new installation of openSUSE Leap 42.2!
Please configure a few basic system settings:"
Locale set to 131 - en_GB.UTF-8
set root password with passwd

#Errors out of the gate
auditd.service and systemd-modules-load.service fail by default, disable them
systemctl disable auditd.service
rm /usr/lib/modules-load.d/sg.conf

#Nice and clean
Gave k2so a static lease 32:57:60:10:58:23
Removed the mountpoint for /mnt/data from router - datadisk is 100% k2so's, not Omnias
zypper in --no-recommends salt-minion
echo "144.76.82.9 salt salt.rootco.de" >> /etc/hosts
hostnamectl set-hostname k2so.dyn.rootco.de
systemctl enable salt-minion
systemctl start salt-minion
Highstate

#encrypt and configure disk
blocked by device not visible in lxc, fixed with container conf

# rootco.de configuration
lxc.cgroup.devices.allow = b 8:16 rwm
lxc.cgroup.devices.allow = b 8:17 rwm
lxc.cgroup.devices.allow = c 10:234 rwm
lxc.hook.start = /usr/local/bin/lxc-hook-sdb

<!--FIX ME . THIS SHOULD BE IN SALT FOR K2SO-->
/srv/lxc/k2so/rootfs/usr/local/bin/lxc-hook-sdb
  #!/bin/sh
  mknod -m 644 /dev/sdb b 8 16
  mknod -m 644 /dev/sdb1 b 8 17
  # needed for btrfs
  mknod /dev/btrfs-control c 10 234

Installing btrfsprogs,cryptsetup,snapper from salt state
Can't use YaST because it scans using btrfsprogs which causes a crash as lxc doesn't own it's root btrfs fs
zypper rm yast2

cryptsetup luksFormat /dev/sdb1
cryptsetup open --type luks /dev/sdb1 backups
mkfs.btrfs /dev/mapper/backups
cryptsetup close backups

TODO - proper decrypt method
echo "backups UUID=328dd1df-4df5-498d-b0c5-02c19700d335  none" >> /etc/crypttab
echo "/dev/mapper/backups  /backups  btrfs  defaults 0 0" >> /etc/fstab

TODO - PROPER SUBVOLUME AND SNAPPER CONFIG
TODO - backup router config to obiwan/k2so
TODO - csync crons
TODO - little script on omnia to make sure LXC container is running and light up User1 accordingly
  Blue flashing - container booting - set by lxc start-script on omnia
  Green - container booted - set by container after boot
  Red flashing - container broken - set by a monitor script on omnia
