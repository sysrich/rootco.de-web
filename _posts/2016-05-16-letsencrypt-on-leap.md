---
layout: post
title: LetsEncrypt on openSUSE Leap
subtitle: with a little help from SaltStack
date: '2016-05-16 14:19:00'
---
I've been running my personal blog on [rootco.de](http://rootco.de) for a few months now. The server is a minimal install of [openSUSE Leap 42.1](https://software.opensuse.org/421/en) running on a nice physical machine hosted at the awesome [Hetzner](https://www.hetzner.de/en/), who offer openSUSE Leap as an OS on all of their Physical and Virtual server hosting. I use the standard [Apache](https://httpd.apache.org/) available in Leap, with [Jekyll](https://jekyllrb.com/) to generate this blog. You can actually see the source to this [Jekyll blog on GitHub](https://github.com/sysrich/rootco.de-web). And to manage it all I use the awesome [SaltStack](https://saltstack.com/) and keep all of my [Salt configuration in GitHub also](https://github.com/sysrich/salt-states) so you can see exactly how my system is setup.

Why am I sharing all of this? Well this weekend there was something I needed to fix.  
http://rootco.de was running without **HTTPS**.

## So What?
This site is a blog about Free Software & Open Source stuff, why on earth does it need to be running HTTPS?.

Because every single web service that can be HTTPS, should be HTTPS. There are [lots](http://arstechnica.com/business/2011/03/https-is-more-secure-so-why-isnt-the-web-using-it/) [of](http://mashable.com/2011/05/31/https-web-security/#.djdB6AMOsq4) [good articles](https://developers.google.com/web/fundamentals/security/encrypt-in-transit/why-https?hl=en) going back years as to why, but the simplest reasons is that it helps ensure the content you visit when you go to my blog is the content **I** intended for my blog. It's very hard for someone to tamper with the content delivered from a HTTPS website. While I'm not (yet) currently hosting any interactive services on my server, if I do I want to ensure they're secured by HTTPS so the data I'm sending to my server is done so as securely as possible.

And in this day and age, there is rarely an excuse to not use HTTPS for everything. Certificates used to be expensive and complicated to setup, but thanks to the wonderful project [LetsEncrypt](https://letsencrypt.org) anyone can now get certificates for their domains for **FREE**.

## Getting Stated with LetsEncrypt
I started as anyone should, by reading [the Getting Started Guide](https://letsencrypt.org/getting-started/).  
As there is not (yet) a `certbot` package for openSUSE Leap, I had to use the `certbot-auto` wrapper script.  
The documentation recommends you install it using the following commands:

```shell
$ git clone https://github.com/certbot/certbot
$ cd certbot
$ ./certbot-auto --help
```

As I'm actually using SaltStack to manage my system, all I did instead was add the following to my Salt State for the rootco.de Web Server.

```YAML
certbot:
  git.latest:
    - name: https://github.com/certbot/certbot
    - target: /opt/certbot
    - user: root
```
You can see the [git commit HERE](https://github.com/sysrich/salt-states/commit/2b3b9ea2bf988d6210119ee6d40648174319f49e).

I then ran the following on my salt master to tell SaltStack to pull down the changes and apply them to the rootco.de Web Server.

```shell
$ git -C /srv/salt pull 
$ salt 'luke.rootco.de' state.highstate --state-output=changes
```

*NOTE: I could have just waited, I actually have the above running as a cronjob every 30 minutes to make sure my server configuration stays as I have defined it in SaltStack*

I then sanity checked the contents of `/opt/certbot` before proceeding. I really hate randomly downloading code from GitHub, so I spent a bit of time making sure what I downloaded made sense and matched what I expected, while wishing someone would take the time to package this up on the [openSUSE Build Service](https://build.opensuse.org) so I could trust them and stop worrying. Once I was happy, I ran the following command to request a certificate for `rootco.de` and `www.rootco.de`:

```shell
$ /opt/certbot/certbot-auto certonly --webroot -w /srv/www/htdocs \
-d rootco.de -d www.rootco.de
```
The wizard automatically detected I was running openSUSE, installed a few packages it needed, then asked me for an email address, and that was it! I had my certificate created and on my server at `/etc/letsencrypt/live/rootco.de/fullchain.pem`. I followed the advice to backup `/etc/letsencrypt` as it constains lots of important configuration and the certificates/keys for my system. Now I had to get Apache to actually use the certificate.

## Configuring Apache on Leap for LetsEncrypt

Because I was a little rusty, I reminded myself of the [openSUSE Leap Apache Documentation](https://doc.opensuse.org/documentation/leap/reference/html/book.opensuse.reference/cha.apache2.html#sec.apache2.ssl.configuration). Good thing too, because I can completely forgotten that to tell Apache to use SSL you needed to run the following command:

```shell
$ a2enflag SSL
```

With that set, I went about setting up an Apache vhost configuration for SSL on rootco.de:

```Apache
<VirtualHost _default_:443>
	DocumentRoot "/srv/www/htdocs"
	ErrorLog /var/log/apache2/error_log
	TransferLog /var/log/apache2/access_log
	SSLEngine on
	Path to the LetsEncrypt created certificate fullchain.pem
	SSLCertificateFile /etc/letsencrypt/live/rootco.de/fullchain.pem 
	# Path to the LetsEncrypt created private key privkey.pem
	SSLCertificateKeyFile /etc/letsencrypt/live/rootco.de/privkey.pem
	CustomLog /var/log/apache2/ssl_request_log   ssl_combined
</VirtualHost>
```

Now, because I absolutely hate making any change to the rootco.de Server directly and want everything managed by SaltStack, I actually put this in my Salt States folder, and modified the rootco.de Web Server state to automatically deploy the file to the appropriate place on the server. You can see [the git commit for that HERE](https://github.com/sysrich/salt-states/commit/ab3b8fcc6aeae57c02f2ad40cb423a0e6a65a579). As I am impaitent and didn't want to wait for my automatic deployment, I again manually refreshed the Salt States on my master and used salt to deploy this new configuration:

```shell
$ git -C /srv/salt pull 
$ salt 'luke.rootco.de.' state.highstate --state-output=changes
```

A quick `systemctl restart apache2` on the server later and I was in business - https://rootco.de was live!

## That's great but...
LetsEncrypt certificates have a duration of 90 days. I want https://rootco.de to be running for a lot longer than that, so I needed to find a solution.

The LetsEncrypt Documentation talked about a `renew` function so I gave it a quick try:

```shell
$ /opt/certbot/certbot-auto renew --dry-run
```

This seemed to work fine so I added a simple cron job to run the following every 60 days:

```shell
$ /opt/certbot/certbot-auto renew >/dev/null 2>&1
```

I picked 60 days as LetsEncrypt certificates only let you renew 30 days before expiration, and I want to hit their servers as little as possible while still ensuring the certificate always gets updated before it expires. `>/dev/null 2>&1` is there because I really don't care about the logs - if it works, I will never need to look at it, and if it's broken I'm going to have to be running the command manually anyway to figure out what went wrong.

Doing a cronjob in SaltStack is so easy, I used it [rather than editing the crontab for root myself](https://github.com/sysrich/salt-states/commit/eac0c5c9fe56281a922ca8a486c1b3ed30bb7a25).

So now my server is running with HTTPS, with a nice shiny LetsEncrypt certificate for both https://rootco.de and https://www.rootco.de, and the whole thing will auto renew ever 60 days. And if something goes horribly wrong and my server gets messed up, all of this is easily redeployable using SaltStack, which is a nice extra bonus for me.

## One last thing

LetsEncrypt is awesome. This service is really revolutionary and is the sort of thing which shouldn't be taken for granted, so please help them out by [Donating to LetsEncrypt](https://letsencrypt.org/donate).

