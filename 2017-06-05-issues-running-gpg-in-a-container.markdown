Title: Issues running gpg in a container
Date: 2017-06-05 00:01:17 +0200
Slug: 2017-06-05-issues-running-gpg-in-a-container
category: posts
tags: docker,networking

In case it helps...

I am giving a try to [this docker componse example](https://github.com/pahaz/docker-compose-django-postgresql-redis-example), and for some strange reason, docker was stuck in this block of code in the Dockerfile for one of the components:

```
# grab gosu for easy step-down from root
ENV GOSU_VERSION 1.7
RUN set -x \
        && apt-get update && apt-get install -y --no-install-recommends ca-certificates wget && rm -rf /var/lib/apt/lists/* \
        && wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$(dpkg --print-architecture)" \
        && wget -O /usr/local/bin/gosu.asc "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$(dpkg --print-architecture).asc" \
        && export GNUPGHOME="$(mktemp -d)" \
        && gpg --keyserver ha.pool.sks-keyservers.net --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4 \
        && gpg --batch --verify /usr/local/bin/gosu.asc /usr/local/bin/gosu \
        && rm -r "$GNUPGHOME" /usr/local/bin/gosu.asc \
        && chmod +x /usr/local/bin/gosu \
        && gosu nobody true
```

Specifically, in this line:

```
gpg --keyserver ha.pool.sks-keyservers.net --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4 
```

It is a really simple command... I could ping the keyserver, I checked the website and everything looked ok, but it wasn't working, neither in the host machine or inside a container... The response was the same: timeout while getting the keys

So... Here comes netstat to the rescue:

```
# netstat -natop | grep gpg
tcp        0      1 192.168.1.10:34340      104.236.209.43:11371    SYN_SENT    8653/gpg2keys_hkp    on (7,10/3/0)
```

SYN_SENT? So it was trying to stablish the connection, but there wasn't any response from the remote host... **_In that port_**

It turns out I recently upgraded my internet connection, and the new router allows you to customize the firewall security levels (you know, low, medium, high and paranoid... yeah, I am using the latter). I noticed that port 11371 was not defined in as a "known service", so I wasn't able to reach it from within my home network.  

As soon as I allowed the connection to that port:

```
# gpg --keyserver ha.pool.sks-keyservers.net --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4
gpg: solicitando clave BF357DD4 de hkp servidor ha.pool.sks-keyservers.net
gpg: /root/.gnupg/trustdb.gpg: se ha creado base de datos de confianza
gpg: clave BF357DD4: clave pública "Tianon Gravi <tianon@tianon.xyz>" importada
gpg: no se encuentran claves absolutamente fiables
gpg: Cantidad total procesada: 1
gpg:               importadas: 1  (RSA: 1)
```

So make sure all your ip flows are open, even for such a small thing as this one...

Take care out there!
