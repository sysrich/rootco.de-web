---
layout: post
title: Hackweek 0x10  Conclusion
subtitle: The Birth of the Kubic Desktop
date: '2017-11-16 17:36:10'
---
Well it's over, SUSE's 16th [hackweek](https://hackweek.suse.com). Here's a quick post to sum up how my Hackweek went compared to [my plans](https://rootco.de/2017-11-10-hackweek-2017-day-0/).

# Small Bits and Pieces

I had a few small things on my list to look at. I didn't get around to look at [WeKan](https://wekan.github.io/), but I understand other colleagues did and got it working on [openSUSE](https://www.opensuse.org). I plan to learn from them about whether or not the tool has much potential for my user cases.

I did get around to having fun with the large interactive whiteboard in SUSE's Nürnberg office. Working with a bunch of colleagues we had a lot of fun, first booting it with [SLE 15 Beta 2](https://www.suse.com/betaprogram/sle-beta/) but deciding that running an Enterprise operating system wasn't really the best idea and replacing that with openSUSE Tumbleweed.

We found both Linux distributions booted a lot faster than Windows and had exceptionally good support for all of the involved hardware (including the touchscreen). However, just like Windows, it seems with this specific embedded system in the touch screen, we couldn't get the embedded HDMI interface to set the resolution to 4K, despite the screen supporting 4K.
Given this wouldn't work in either Windows or *SUSE Linux after a day of hacking around we gave up and the screen is likely to be sent back to the manufacturer.

# What about Kubic Desktop?

So my big idea was taking the container focused [openSUSE Kubic](https://github.com/kubic-project/community) and seeing if I could turn it into a [GNOME](https://www.gnome.org) powered Chromebook-like operating system, with user applications coming from [Flatpak](http://flatpak.org).

The short version is **IT WORKS**

![it works!](https://rootco.de/pics/kubicdesktop.jpeg)

It was an interesting journey. On the Kubic side of things I quickly abandoned the idea of building everything 'clean' in OBS first and instead just took the existing [Tumbleweed Kubic](https://software.opensuse.org/distributions/tumbleweed) Media and manually hacking the resulting installation into something that I wanted.

Our support for [transactional updates](https://www.youtube.com/watch?v=oUREPvOObTw) got a very good workout, being used and abused to install well over 1606 packages from Tumbleweed onto the Kubic host to provide a fully fledged desktop install.

It surprised me how little I had to modify, with minor tweaks to the btrfs subvolume configuration required to get gdm working. I learned alot about PackageKit and gnome-software getting it configured just the way I wanted.

By the end, my resulting prototype VM was so promising I repeated the process on my [GPD Pocket](http://www.gpd.hk/pocket.asp). I'm now running a Kubic Desktop installation, with automated operating system updates, and if an update ever did go wrong would have seamless rollback to the _exact_ state of the system root filesystem.

It was really nice to prove that, even though the project as a whole is still in a very early stage, [Tumbleweed Kubic](https://software.opensuse.org/distributions/tumbleweed) is a powerful flexible platform for hacking on and getting used to this new, transactional way of looking at an operating system and it's software management.

With a little more time and polish, I'm supremely confident we could build this into a 'grandmother-friendly' openSUSE desktop operating system, where the user wouldn't even need to know that such a thing as a 'root account' existed, but they would still benefit from the [fully tested rolling nature of openSUSE Tumbleweed](https://rootco.de/2016-03-28-why-use-tumbleweed/).

# A journey into Flatpaks

A fancy desktop operating system is nothing without apps. And as this idea aimed to have a system where it's users shouldn't need to worry about doing something as root, I needed a solution that provided apps that could be installed and run reliably in userspace.

As openSUSE does not (yet) have support for isolated snaps (due to Canonicals AppArmor patches still not fully available in the upstream Kernel), that left AppImage and Flatpak for consideration as a source of these applications.

As AppImage has a pretty terrible user experience story, without any kind of standard 'AppStore' interface, requiring users download executables from random websites, then to `chmod a+x` a file in the commandline before executing said file, that ruled them out as an option for this project; Though it was nice to see an AppImage running first time on the first Kubic Desktop prototypes.

But as originally planned, [Flatpak](http://flatpak.org) seemed to have the answers to everything I needed. With solid gnome-software integration I knew I could build something that would offer users all of the applications in [Flathub](http://flathub.org). Despite my previous concerns of [these technologies](https://www.youtube.com/watch?v=SPr--u4n8Xo) I had high hopes.

# "Everything is possible if you think in Opportunities"

The famous slogan of the international purveyor of furniture in flat-packed form, [Ikea](http://ikea.com), is really appropriate for Flatpak.

After a few days of playing with Flatpaks, I'm starting to see the opportunities and possibilities. It was really nice to be able to easily install [Atom](https://atom.io), [Signal](https://signal.org), and more with ease, when due to issues like licenses & crazy build environment requirements we haven't (yet) been able to offer that software as part of openSUSE Tumbleweed.

But the ecosystem as a whole is flawed, possibly broken, and is in no way a software delivery system I can put any faith in at the moment. Flatpaks like Audacity would totally fail to start, claiming not to be able to read configuration files that were clearly present on the system. Other Flatpaks like Gydl complained about a lack of network, which was clearly a fault of that specific application when others were streaming YouTube just fine. And applications like libreoffice are clearly advertised on Flathub, just to not appear in gnome-software due to missing metadata.

With so many failures on a relatively small software selection, Flathub really, desperately, need to impose quality controls on their repository and stop shipping broken packages. And when they do, as they are right now, they need to have a way of actually reporting bugs against those packages. It's nice their website has a guide on how to report Legal or Security issues, but that's no good if users can't get the apps to work in the first place.

As I played around with apps I got the feeling more and more that applications needed much tinkering and tuning to be able to fully support the Flatpak-way of doing things, and I think that is going to hold the Flatpak ecosystem back, unless tools like gnome-builder somehow become the predominant tooling for application development on Linux.

I was also dreadfully dissapointed with the performance, with applications taking significantly longer to load on my GPD Pocket compared to the same app provided by openSUSE Tumbleweeds rpms. On faster machines I didn't notice, but it really put a dent in my idea of using Flatpaks for 'netbook' style usecases.

So I see the potential there, but at the moment I'm left feeling it will be unrealised potential.

With all that said, I'd like to extend a huge thank you to the folks in #flatpak IRC on Freenode. They were very friendly, a huge help, and I do believe they will work hard to address these problems I bumped into. Who knows, I may even help them out, they certainly are a nice enough bunch that I feel obligated to do so no matter what happens to the "Kubic Desktop".

# Next Steps

As you will see from the instructions below, installing your own Kubic Desktop isn't a trivial process at the moment. If I have time, or if others feel like moving this idea forward wit me, we really need to look into the following:

- Polishing the Product & Image definitions in OBS for Kubic to remove as many of the installation steps below as possible. This would either mean extending the existing images & installation routine to support a Kubic desktop, or producing new images for a Kubic desktop.
- As part of the above, remove the conflicts that stop additional patterns being installed with the openSUSE-TW-Kubic product.
- Make libzypp-plugin-appdata a recommended dependency so that a system can run with only flatpaks showing in gnome-software instead of always showing the distribution packages also.
- To get a complete GNOME installation I chose to use openSUSE's "gnome" pattern. Finding (or tuning) a GNOME pattern that just installs GNOME without any applications would be nicer, as right now they clutter the installation more than I envision for the idea.
- Fix the transactional-update tool to support all of zyppers additional commandline switches (like --from when using 'dup').
- Working with Flathub so they can setup [openQA](http://open.qa) and start testing their Flatpaks before distributing them.

# Install your own Kubic Desktop

These instructions aren't for the faint of heart, but if you'd like to setup your own prototype Kubic desktop to play with you can follow the guide below

* Download the latest snapshot of [Tumbleweed Kubic](https://software.opensuse.org/distributions/tumbleweed) and boot the media, choosing the 'Installation' option
* Follow the installation wizard as normal, but customise the Partition configuration. You need a `btrfs` root filesystem with two additional subvolumes `@/var/lib/gdm` and `@/var/lib/flatpak`
* Choose to install a `Plain System`
* Once installed and rebooted, login as root and run the following command; `zypper ar http://download.opensuse.org/tumbleweed/repo/oss tw-repo`
* Edit `/etc/zypp/zypp.conf` and set `solver.onlyRequires = false`
* Run `transactional-update pkg in patterns-gnome-gnome flatpak gsettings-backend-dconf`
* Accept any dependency warnings but choosing option `1` and deinstalling the problematic packages
* Once installed then **reboot** - transactional updates don't apply until the next reboot.
* Once rebooted, run the following commands
  - `systemctl set-default graphical` to set the system to boot into a graphical mode in the future
  - `useradd -m $USERNAME` with `$USERNAME` being whatever you want your useraccount to be called to create a user account
  - `passwd $USERNAME` to set the password for that user account
  - `rm -Rf /var/cache/app-info` to remove any trace of AppData from the official Tumbleweed repos to force gnome-software to rely only on Flatpak
  - `transactional-update shell` to create an interactive shell that will let you modify the otherwise read-only root filesystem
  - `rpm -e --nodeps libzypp-plugin-appdata` to remove the libzypp plugin that creates the app-info AppData
  - `zypper al libzypp-plugin-appdata` to prevent anything installing that package again
  - `exit` to exit the interactive shell and create a snapshot which will effectively be a custom 'transactional update' next time you reboot
  - then reboot to activate the above changes
* You should now be presented with standard gdm login screen with your `$USERNAME` account being offered, login
* Before doing anything else, run the following commands to setup access to flatpak and tune gnome-software to run how we want it
  - `gsettings set org.gnome.software install-bundles-system-wide false`
  - `gsettings set org.gnome.software allow-updates false`
  - `gsettings set org.gnome.software download-updates false`
  - `gsettings set org.gnome.software enable-software-sources false`
  - `gsettings set org.gnome.software first-run false`
  - `flatpak remote-add --user --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo`
* Now load up "Software" in GNOME and enjoy your experimental Kubic Desktop experience

If you are interested in working on this idea and taking it beyond the fun little experiments I've started here, please feel free to get in touch using the links at the bottom of this blog.

Thanks for reading!





