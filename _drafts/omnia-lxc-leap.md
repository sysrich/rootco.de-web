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
lxc.hook.start = /usr/local/sbin/lxc-hook-sdb

BELOW PUSHED BY SALT
/srv/lxc/k2so/rootfs/usr/local/sbin/lxc-hook-sdb
  #!/bin/sh
  mknod -m 644 /dev/sdb b 8 16
  mknod -m 644 /dev/sdb1 b 8 17
  # needed for btrfs
  mknod /dev/btrfs-control c 10 234

Installing btrfsprogs,cryptsetup,snapper from salt state
Can't use YaST because it scans using btrfsprogs which causes a crash as lxc doesn't own it's root btrfs fs
zypper rm yast2

BELOW PUSHED BY SALT
cryptsetup luksFormat /dev/sdb1
cryptsetup open --type luks /dev/sdb1 backups
mkfs.btrfs /dev/mapper/backups
mount /dev/mapper/backups /backups
btrfs subvolume create /backups/@
btrfs subvolume list /backups (@ should be 257)
btrfs subvolume set-default 257 /backups
umount /backups
mount /dev/mapper/backups /backups
snapper -c backups create-config /backups
vi /etc/snapper/configs/backups

BELOW PUSHED BY SALT
TODO - backup/manage with salt
  TIMELINE_MIN_AGE="1800"
  TIMELINE_LIMIT_HOURLY="12"
  TIMELINE_LIMIT_DAILY="7"
  TIMELINE_LIMIT_WEEKLY="4"
  TIMELINE_LIMIT_MONTHLY="12"
  TIMELINE_LIMIT_YEARLY="10"

snapper -c backups create --description "TEST filesystem snapshot"
snapper -c backups delete 1

DONE - little scripts on omnia to make sure LXC container is running and light up User1 accordingly
  Blue flashing - container booting - set by lxc start-script on omnia
    Done - /etc/lxc-bootnotify
  White - container booted - set by container after boot
    Done - /usr/local/sbin/backer-unlock
  Red flashing - container broken - set by a monitor script on omnia
    Done - /etc/lxc-alarm + cron

/etc/config/lxc-auto
  config container
        option name k2so
        option timeout 60

Profile.local to warn when /backups not mounted
  
DONE - proper decrypt method and/or script to decrypt with backer-unlock

TODO - as backups will be pushed from root on obiwan and others generate root ssh private keys for all hosts in salt, and have those public keys on k2so via salt
TODO - create new users for backer on k2so, one for csync, rsync, use system root keys to auth to them, use command= to lock down each http://superuser.com/questions/261361/do-i-need-to-have-a-passphrase-for-my-ssh-rsa-key

TODO - backup script to create router config to obiwan/k2so
TODO - restore script for router config
TODO - backup k2so + restore script
TODO - csync/rsync crons - csync of home takes a few minutes

