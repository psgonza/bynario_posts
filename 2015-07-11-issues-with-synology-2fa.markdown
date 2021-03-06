title: Issues with Synology 2FA
slug: 2015-07-11-issues-with-synology-2fa
date: 2015-07-11 00:00
category: posts
tags: synology, nas, ntp

Hi there

After a few months in the shelf, today I reconnected my [Synology DS215j](https://www.synology.com/en-global/products/DS215j), and to my surprise, I wasn't able to login via DiskStation GUI due to a problem with my autenthication:

{% img center /img/synology_ntp_mismatch.jpg 'DSM Error' %}

It was weird because:

  a) (you are going to love this one...) ***"It was working before"***  
  b) I could login to my NAS via SSH with the same user/pass, so it was clearly a problem with the 2 Factor Authethication

Once in SYNOLOGY CLI, I realized the system time was wrong:

{% img center /img/synology_2fa_error.png 'ntp mismatch' %}

As you can (hopefully) see in the picture, the time in the SYNOLOGY CLI (23:58:13) differs from the one in web GUI (23:54) (and the system time in my MBP as well)

So I connected to the NAS with root password (rather sooner than later I would need to "enable" sudo and block root user in SSH) and updated the date by running:

```
ntpdate -u pool.ntp.org
```

(If this isn't working, try killing ```ntpd``` and run ```ntpdate -u pool.ntp.org``` again)

After that, I managed to login using my 2FA code as expected... 

Bytes!

PS: First post using github as text editor... Let's see how this goes.
