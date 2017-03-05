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
Gave k2so a static lease
Removed the mountpoint for /mnt/data from router - datadisk is 100% k2so's, not Omnias
zypper in --no-recommends salt-minion
echo "144.76.82.9 salt salt.rootco.de" >> /etc/hosts
hostnamectl set-hostname k2so.dyn.rootco.de
systemctl enable salt-minion
systemctl start salt-minion
Highstate

TODO - encrypt and configure disk
blocked by device not visible in lxc, fixed with echo "lxc.cgroup.devices.allow = b 8:16 rwm" to container conf
mknod 644 /dev/sdb b 8 16
GOT STUCK ^^^ to fix I think
http://lxc-users.linuxcontainers.narkive.com/qOC3Oqq5/adding-a-host-block-device-to-a-container
FRESH LXC container would be nice!

TODO - backup router config to obiwan/k2so
TODO - csync
TODO - little script on omnia to make sure LXC container is running and light up User1 accordingly
