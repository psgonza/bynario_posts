title: Monitoring remote systems: Using GTalk notifications
date: 2012-08-05 13:52
slug: 2012-08-05-monitoring-remote-systems-using-gtalk-notifications
category: posts
tag: xmpp , notifications

Since a few weeks ago I am using a [Raspberry Pi](http://raspberrypi.org) as bittorrent client instead of my old, big and noisy Pentium III 1Ghz

{% img center https://dl.dropbox.com/u/14814182/pics/DSC_0321_sm.jpg 'admin' %}

I am pretty happy with the change (besides a few issues with the USB/Network controller). I managed to get rid of a big PC case, there are no fans spinning anymore (noiseless), and the power consumption is unbeatable, and I only paid around $40. Win-Win situation, isn't it?.

But this post is not about my rasPi (ok, maybe just a little bit), but about how to received XMMP notification from a remote system. In this case, I was interested in sending a brief report of my VPS status (CPU,MEM,HD, etc) every 12 hours to my Gtalk account, so I can see how it is doing.

Most of the alternatives are based on sending emails, but I wasn't very keen on setting up a MTA only to send a few reports every X hours, so I decided to use XMMP notifications instead.

Googling a little bit I found this:

```
apt-cache show sendxmpp
Package: sendxmpp
Priority: optional
Section: net
Installed-Size: 88
Maintainer: Guus Sliepen <guus@debian.org>
Architecture: all
Version: 1.20-1
Depends: perl, libnet-xmpp-perl
Filename: pool/main/s/sendxmpp/sendxmpp_1.20-1_all.deb
Size: 15026
MD5sum: 2a3fcdc995442ebd1f7d8a336fc88164
SHA1: 90dd5cc21c9b97872dec5a55ce9d55ea7d728d65
SHA256: eb8a2fd09ffc65c04cc54a8a718faf550b5864a3135a9d045d2657cd43ed0449
Description: commandline XMPP (jabber) utility
 sendxmpp is a perl script to send XMPP (jabber) messages, similar to what
 mail(1) does for mail. XMPP is an open, non-proprietary protocol for instant
 messaging. See www.jabber.org for more information.
Tag: implemented-in::perl, interface::commandline, protocol::jabber, role::program, scope::utility
```

Just what I needed.

PROs: 
- Easy to use and integrate in simple scripts.
- Perl script available in Debian repositories.
- Lightweight. There are no daemons running. The script is executed whenever is needed.
- Secure. Using SSL.

CONS:
- Only available to GTalk users (obviously).
- Limited size of the message. (just because of the limited size of the screen of your mobile device)
- In order to avoid SPAM, the account where you are sending the message from, has to be in the contact list of the receiver.

As, it was working like a charm in my Raspberry Pi, I decided to do the same in my VPS. Let me describe in a few steps how to configure it:

Installation? Debian's way.

```
 apt-get install sendxmpp
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following extra packages will be installed:
  libauthen-sasl-perl libdigest-sha1-perl libio-socket-ssl-perl libnet-libidn-perl libnet-ssleay-perl libnet-xmpp-perl
  libxml-stream-perl
Suggested packages:
  libdigest-hmac-perl libgssapi-perl libio-socket-inet6-perl libnet-dns-perl
The following NEW packages will be installed:
  libauthen-sasl-perl libdigest-sha1-perl libio-socket-ssl-perl libnet-libidn-perl libnet-ssleay-perl libnet-xmpp-perl
  libxml-stream-perl sendxmpp
0 upgraded, 8 newly installed, 0 to remove and 0 not upgraded.
Need to get 581 kB of archives.
After this operation, 2,543 kB of additional disk space will be used.
Do you want to continue [Y/n]? Y
```

Pretty easy configuration. Under your home directory, create a .sendxmpprc file

```
# cat .sendxmpprc
<your_email>@gmail.com <your_password>
```


Let's test it. First sending a message from the standard input:
```
echo "hello there" | /usr/bin/sendxmpp  -t -u <your_email> -o gmail.com <recipient Google ID> 
```

Notification received:

{% img center https://dl.dropbox.com/u/14814182/pics/gtalk1.jpg %}

And now, sending the text from a file:

```
echo "this is a message send from a text file" > text.txt
/usr/bin/sendxmpp -t -u <your_email> -o gmail.com <recipient Google ID> -m text.txt
```

Notification received:

{% img center https://dl.dropbox.com/u/14814182/pics/gtalk2.jpg %}

Working!

Integrating it in a simple bash script:

```
#!/bin/bash

REPORT="/tmp/.report"

[  -f $REPORT ] && rm -f $REPORT
echo  "VPS report at `date`:" >> $REPORT
echo  "Machine stats : `uptime`" >> $REPORT
echo  "Memory stats : `free -m`" >> $REPORT
echo  "Hard Disk Space: `df -h`" >> $REPORT
echo "" >> $REPORT


#Sending the report to GTALK
/usr/bin/sendxmpp -t -u <your_email> -o gmail.com <recipient Google ID> -m $REPORT 
```


And setting up the task in cron:

```
crontab -e
57 17 * * * sh /scripts/send_report.sh
```

The notification received in my Gtalk client:

{% img center https://dl.dropbox.com/u/14814182/pics/gtalk3.jpg %}

This is just an example, but I am pretty sure you could find a lot of applications to this basic notification system

Enjoy!


