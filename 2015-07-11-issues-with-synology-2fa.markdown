tittle: Issues with Synology 2FA
slug: 2015-07-11-issues-with-synology-2fa
date: 2015-07-11 00:00
category: posts
tag: synology, nas, ntp
Status: draft

Hi there

After a few months in the shelf, today I reconnected my Synology DS215, and to my surprise, I wasn't able to login via DiskStation GUI due to a problem with my autenthication:

{% img center https://dl.dropboxusercontent.com/u/14814182/blog/synology_ntp_mismatch.jpg 'DSM Error' %}

It was weird because:
a) (you are going to love this one...) "It was working before" :) I have the same user, password and mobile app. 
b) I could login to my NAS via SSH with the same user/pass, so it was clearly a problem with the 2 Factor Authethication

Once in the CLI, I realized  the system time was wrong:

{% img center https://dl.dropboxusercontent.com/u/14814182/blog/synology_2fa_error.png 'ntp mismatch' %}

As you can see in the picture, the time in the SYNOLOGY CLI (23:58:13) differs from the one in web GUI (23:54) (and the system time in my MBP as well)

So I connected to the NAS with root password (rather sooner than later I would need to "enable" sudo and block root user in SSH) and updated the date by running:

```
ntpdate -u 24.56.178.140
```

(If this is working, try killing ```ntpd``` and run ntpdate again)

After that, I managed to login using my 2FA code as expected... 

Bytes

PS: First post using github as text editor... Let's see how this goes.
