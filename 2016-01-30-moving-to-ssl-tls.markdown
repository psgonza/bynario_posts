Title: Moving to SSL/TLS
Date: 2016-01-30 2:3:4 +0200
Slug: 2016-01-30-moving-to-ssl-tls
category: posts
tags: bynario, ssl

10 minutes... That's all it takes to configure [HTTPS](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol_Secure) thanks to [Let's Encrypt](https://letsencrypt.org/). So from now on, this blog will be using https by default..

There is a lot of information about how to configure it available in Internet (you can find a usefull piece of information [here](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-centos-7)) so I won't even try to add anything new.

Let's Encrypt certs have a ninety-day lifetime, so make sure you renew your certifications each 3 months (some examples [here](https://letsencrypt.org/howitworks/)).

As a final note, if you are running a static blog generator as myself, don't forget to modify your config file and replace "http://" by "https://".

\\\psgonza
