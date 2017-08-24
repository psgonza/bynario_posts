title: Mail notification system in 2 minutes
date: 2015-12-27 11:11:11
slug: 2015-12-27-mail-notification-system-in-2-minutes
tags: linux, monitoring, ssmtp
category: posts

Quick post... In case you'd like to set up a quick mail notification system for your linux machines, keep reading, otherwise, have a merry Xmas...

I'd like to receive a notification everytime there is a sucessfull ssh login in my VPS, and [swatch](http://linux.die.net/man/1/swatch) + [ssmtp](http://linux.die.net/man/8/ssmtp) make it possible very easily... The idea is to receive a notification in my email every time there is a line in `/var/log/auth.log` containing the strings "sshd" and "Accepted password". ie:

```
Dec 24 12:42:06 bynario sshd[3258]: Accepted password for myuser from 95.11.176.11 port 31738 ssh2

```

NOTE: Using your gmail account might give you some [troubles](http://serverfault.com/questions/635139/how-to-fix-send-mail-authorization-failed-534-5-7-14)... I decided to use a different one.

- Install the sw:

```
sudo apt-get install ssmtp swatch
```

- Make sure no one else can read your email password:

```
sudo chown root:mail /etc/ssmtp/ssmtp.conf
sudo chmod 640 /etc/ssmtp/ssmtp.conf
sudo usermod -a -G mail myuser
```
(`myuser` is the name of my local user)

- Create `/etc/swatch.conf` and add the pattern and action:

```
watchfor /sshd.*Accepted password/
    exec echo "Subject:Bynario.com SSHD accepted password\n\n$_\n" | /usr/sbin/ssmtp <your_email>
```

what does it do? It will be tailing the auth.log file looking for any string matching "sshd.*Accepted password" and in case it finds something, it will send an email to <your_email> with the subject "Bynario.com SSHD accepted password" and the line matching the filter in the email body.

- Let's define the email account... Edit `/etc/ssmtp/ssmtp.conf`: 

```
root=<your_email>

UseTLS=YES
UseSTARTTLS=YES
AuthMethod=LOGIN

mailhub=<your_snmp_server.com>:587
hostname=<your_server.com>

AuthUser=<your_email>
AuthPass=<your_password>

FromLineOverride=YES

```

(The TSL options will depend on your email provider)

- Edit `/etc/ssmtp/revaliases`: 

```
root:<your_email>:<your_snmp_server.com>:587
myuser:<your_email>:<your_snmp_server.com>:587
```

- And finally, start swatch:

```
sudo swatch --config-file=/etc/swatch.conf --tail-file=/var/log/auth.log --tail-args=--follow=name --daemon
```

It would be a good idea to create an rc script or add that line to /etc/rc.local, so it is automatically started if the system is rebooted:

```
bynario:~$ cat /etc/rc.local
/usr/bin/swatch --config-file=/etc/swatch.conf --tail-file=/var/log/auth.log --tail-args=--follow=name --daemon
exit 0
```

And that's it... You should have an email in your account every time there is a new ssh connection... pretty handy.

\\psgonza
