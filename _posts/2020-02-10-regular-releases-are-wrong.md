---
layout: post
title: Regular Release Distributions Are Wrong
subtitle: Why I No Longer Use openSUSE Leap
date: '2020-02-10 23:58:12'
---
For those who don't already know, [openSUSE](https://www.opensuse.org) is a Linux Distribution Project with two Linux distributions, Tumbleweed and Leap. 

* *Tumbleweed* is what is known as a **Rolling Release**, in that the distribution is constantly updating. Unlike Operating Systems with specific versions (e.g.. Windows 7, Windows 10, iOS 13, etc..) there is just 'openSUSE Tumbleweed' and anyone downloading it fresh or updating it today gets all the latest software.
* *Leap* is what is known as a a **Regular Release**, in that it does have specific versions (e.g.. 15.0, 15.1, 15.2) released in a regular cadence. In Leap's case, this is annually. Leap is a variation of the Regular Release known as an **LTS Release** because each of those 'minor versions' (X.1, .2, .3) are intended to include only minor changes, with a major new version (e.g.. 16.0) expected only every few years. 

It's a [long documented fact](https://rootco.de/2016-03-28-why-use-tumbleweed/) that I am a big proponent of Rolling Releases and use them as my main operating system for Work & Play on my Desktops/Laptops.  
However in the 4 years since writing that last blog post I always had a number of Leap machines in my life, mostly running as servers.  

As of today, my last Leap machine is no more, and I do not foresee ever going back to Leap or any Linux distribution like it.

This post seeks to answer why I have fallen out of love with the Regular Release approach to developing & using Operating Systems and provide an introduction to how you too could rely on Rolling Releases (specifically Tumbleweed & MicroOS) for everything.

# Disclaimer

First, a few disclaimers apply to this post. I fully realise that I am expressing a somewhat controversial point and am utterly expecting some people to disagree and be dismissive of my point of view. That's fine, we're all entitled to our opinions, this post is mine.  

I am also distinctly aware that, my views expressed run counter to the business decisions of the customers of my employer, who do a very good job of selling a very commercially successful Enterprise Regular Release distribution.  
**The views expressed here are my own and not those of my employer.**  

And I have no problem that my employer is doing a very good job making a lot of money from customers who are currently making decisions I feel are 'wrong'.  
If my opinion is correct then I hope I can help my employer make even more money when customers start making decisions I feel are more 'right'.

# Regular & LTS Releases Mean Well

Regular & LTS Releases (hereafter referred to as just Regular Releases) have all of the best intentions. The Open Source world is made up of thousands if not millions of discreet Free Software & Open Source Projects, and Linux distributions exist to take all of that often chaotic, ever-evolving software and condensing it into a consumable format that is then put to very real work by its users. This might be something as 'simple' as a single Desktop or Server Computer, or something far larger and complex such as a SystemZ Mainframe or 100+ node Kubernetes Cluster.

The traditional mindset for distribution builders is that the regular release gives a nice, predictable, plan-able schedule in which the team can carefully select appropriate software from the various upstream projects.  
This software can then be carefully curated, integrated, and maintained for several, sometimes many, years.  
This maintenance often comes in the form of making minimal changes, seeking only to address specific security issues or customer requests, taking great care not to break systems currently in use.

This mindset is often appreciated by users, who also want from their computing a nice, predictable, reliable experience.. but they also want new stuff to keep up with their peers, either in communities or commercially.. and here begins the first problem. 

# All Change Is Dangerous, but Small Changes Can Be Worse

Firstly, whether the change is a security update or a new feature, that change is going to be made by humans. Humans are flawed, and no matter how great we all get with fancy [release processes](https://openbuildservice.org) and [automated testing](https://open.qa), we will never avoid the fact that humans make mistakes.

Therefore, the nature of the change has to be looked at. *"Is this change too risky?"* is a common question, and quite often highly desired features take years to deliver in regular releases because the answer is *"yes"*.

When changes are made, they are made with the intention of minimising the risks introduced by changing the existing software. That often means avoiding updating software to an entirely new **version** but instead opting to **backport** the smallest necessary amounts of code and merging them with (often much) older versions already in the Regular Release. 
We call these patches, or updates, or maintenance updates, but we avoid referring them to what they really are ... **franken-software**

![franken-software](/pics/frankensoftware.png)

No matter how skilled the engineers are doing it, no matter how great the processes & testing are around their backporting, fundamentally the result is a hybrid mixed together combination of **old** and **new** software which was never originally intended to work together.  
In the process of trying to avoid risk, backports instead introduce entirely new vectors for bugs to appear. 

# Regular Releases Neglect The Strengths Of Open Source

Linus's Law states *"given enough eyeballs, all bugs are shallow."*  
I believe this to be a fundamental truth and one of the strongest benefits of the Open Source development model. The more people involved in working on something, the more eyeballs looking at the code, the better that code is, not just in a 'lack of bugs' sense but also often in a 'number of working features' sense.

And yet the process of building Regular Releases actively avoids this benefit. The large swath of contributors in various upstream projects are familiar with codebases often months, if not years, ahead of the versions used in Regular Releases.  
Even inside Community Distribution Projects, it is my experience that the vast majority of volunteer distribution developers are more enthused in targetting 'the next release' rather than backporting complex features into an old codebase months or years old.  
This leaves a small handful of committed volunteers, and those employees of companies selling commercial regular releases. These limited resources are often siloed, with only time and resources to work on their specific distribution, with their backports and patches often hard to reuse by other communities.

With Regular Releases, there are not many eyes. Does that mean all bugs are deep? 

I am not suggesting the people building these Releases do not do a good job, they most certainly do. But when you consider the best possible job all Regular Release maintainers possibly could do and compare it to the much broader masses of the entire open source ecosystem, how did we ever think this problem was light enough for such narrow shoulders? 

# A Different Perspective

I've increasingly come to the realisation that not only is change unavoidable in software, it's desired. It not only happens in Rolling Releases, but it still happens in Regular Releases.  
Instead of trying to avoid it why don't we embrace it and deal with any problems that brings?

Increasingly I hear more and more users demanding "we want everything stable forever, and secure, but we want all the new features too".

Security and Stability cannot be achieved by standing still.  
New Features cannot be easily added if good engineering practice is discouraged in the name of *appearing* to be stable.  
In software as complicated as a Linux distribution, quite often, the right way to make a change requires a significant amount of changes across the entire codebase.  
Or in other words **"in order to be able to change any ONE thing, you must be able to change EVERYTHING"**.

Rolling Releases can already do that, and you can [read my earlier blog post if you haven't already](https://rootco.de/2016-03-28-why-use-tumbleweed/) for some reasons why Tumbleweed is the best at that.  
But it's not enough, otherwise I would have been running Tumbleweed on my servers 4 years ago.

# I Am A Lazy Sysadmin

Before my life as a distribution developer I was a sysadmin, and like all sysadmins I am lazy. I want my servers to be as *'zero-effort'* as possible.  
Once they're working I don't want to have to patch them, reboot them, touch them, look at them, or ideally even think about them ever again. And this is a hard proposition if I am running my server on a regular Rolling Release like Tumbleweed. 

Tumbleweed is made up of over 15,000 packages, all moving at the rate of contribution. Worse, Tumbleweed is designed as a *multi-purpose* operating system.  
You can quite happily set it up to be a *mail server, web server, proxy server, virtualisation host, and heck, why not even a desktop* **all at the same time**.  
This is one of Tumbleweed's greatest strengths, but in this case it is also a weakness.  
openSUSE has to make sure all these things could possibly work together. That often means installing more 'recommended packages' on a system than absolutely necessary, 'just-in-case' the user wants to use all the possible features their combination of packages could possibly allow.  
And with this complexity comes an increase of risk, and an increase of updates, which themselves bring yet more risk. Perfectly fine for my desktop (for now), but that's far too much work for a Server, especially when I typically need a Server to **just do one job**. 

# Minimal Risk, Maximum Benefits with openSUSE MicroOS

openSUSE MicroOS is the newest member of the openSUSE Family. From a code perspective, it is a **derivative of openSUSE Tumbleweed**, using exactly the same packages and integrated into its release process, but from a philosophical perspective, MicroOS is a totally different beast.

Whereas Tumbleweed & other traditional distributions are *multi-purpose*, MicroOS is designed from the ground up to be **single-purpose**. It's a Linux distribution you deploy on a bit of hardware, a VM, a Cloud instance, and once it is there it is intended **to do just one job.**

* The package selection is *lean*, with all you need to run on bare metal if you install from the [ISO](http://download.opensuse.org/tumbleweed/iso/openSUSE-MicroOS-DVD-x86_64-Current.iso), or even smaller if you choose a [VM Image](http://download.opensuse.org/tumbleweed/appliances/) for a platform where hardware support isn't necessary. Fewer packages mean fewer reasons to update, helping lower the risks of change traditionally introduced by a rolling release. Recommended packages are disabled by default.  
* Being a [transactional system](https://kubic.opensuse.org/blog/2018-04-04-transactionalupdates/), it has a read-only root filesystem, further cutting down the risk of changes to the system, ensuring that any unwanted change that does happen can be rolled back. Not only that, but such roll-backs can be automated with [health-checker](https://github.com/kubic-project/health-checker).  
* With [rebootmgr](https://github.com/SUSE/rebootmgr), I can even schedule maintenance windows to ensure my updates only take effect during times I'm happy with.  

Auto updating, auto rebooting, auto rolling back? **I can be a lazy sysadmin** even with a rolling release! 

# Just One Job Per Machine?

MicroOS is designed to do just one job, and this is fine for machines or VMs where all you would want to do is something like a self-maintaining webserver. That is as a simple as running`transactional-update pkg in nginx`.  
But that can be rather limiting. Wouldn't it be nice if there was some way of running services that minimised the introduction of risk to the base operating system, and could be updated independently of that base operating system?

Oh, right, that already exists, and they're called containers.

**MicroOS makes a perfect container host**, so much so we deliver VM Images and a System Role on the ISO which already comes configured with **podman**.

Especially now openSUSE has a [growing collection of official containers](https://registry.opensuse.org/cgi-bin/cooverview), a good number of services are a simple `podman run podman pull registry.opensuse.org/opensuse/foo` away.

In my case, I have moved all of my old Leap servers to now use MicroOS with Containers, this includes

* NextCloud, using upstream containers [[1]](https://github.com/sysrich/salt-states/blob/master/rootco/containerhost/r2d2/nextcloud.service)[[2]](https://github.com/sysrich/salt-states/blob/master/rootco/containerhost/r2d2/ncserver.service)[[3]](https://github.com/sysrich/salt-states/blob/master/rootco/containerhost/r2d2/ncnginx.service)[[4]](https://github.com/sysrich/salt-states/blob/master/rootco/containerhost/r2d2/ncdb.service)
* Jekyll, using an upstream container [[5]](https://github.com/sysrich/salt-states/blob/master/rootco/containerhost/d0/rootco-jekyll.service)
* Nginx, using the official openSUSE container [[6]](https://github.com/sysrich/salt-states/blob/master/rootco/containerhost/d0/rootco-web.service)
* A custom salt-master container [[7]](https://build.opensuse.org/package/show/home:RBrownSUSE:containers/salt-master-image)
* A custom SSH container acting as both a jump-host and backup target for all my other machines [[8]](https://build.opensuse.org/package/show/home:RBrownSUSE:containers/backer-srv-image)
* [and some other containers which I hope to make official openSUSE Containers once I'm happy they'll work for everyone](https://build.opensuse.org/project/show/home:RBrownSUSE:containers) 

All of the custom containers are built on openSUSE's tiny `busybox` container which is just small and tiny and magical and I have no idea why anyone would use *alpine*.

**All of my servers are now running MicroOS.** The software is newer with all the latest features, but through a combination of containerisation and transactional-updates I find myself spending significantly less time maintaining my servers compared to Leap.

I can update the containers when I want, using container tags to pin the services to use specific versions until *I* decide when I want to update them.  
I can easily add new containers to the to the MicroOS hosts just by adding another systemd service file running `podman` to `/etc/systemd/system`.  
And I never need to worry about my base operating system which just takes care of itself, rebooting in the maintenance window I defined in `rebootmgr`.

I'm going to be writing further blog posts about my life with **MicroOS** and **podman** , but meanwhile I couldn't be happier, and sincerely hope more people take this approach to running infrastructure.  
**Why?** Well, we're still only human, but when things do go wrong it'll be even easier with more people looking at any problems :)

Now all I need to do is see if any of these benefits make sense for a [desktop....](https://hackweek.suse.com/projects/microos-desktop)


