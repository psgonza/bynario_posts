title: Cacti monitoring over SSH
date: 2012-06-23 09:26
slug: 2012-06-23-cacti-monitoring-over-ssh
tag:  linux , sysadmin , devop , monitoring

Recently I had to configure [Cacti](http://www.cacti.net/) in order to monitor several blades in a cluster (nothing fancy, just CPU load, memory usage, load average and disk space) but this time NET-SNMP wasn't an option. I knew that I could use server scripts to access the remote blades using SSH, so after googling a while, I found this proejct from the Percona guys:
    
[Percona Monitoring Plugins for Cacti](http://www.percona.com/doc/percona-monitoring-plugins/cacti/)
    
What can I say... Amazing. Everything working in a few minutes.

Although all the information about Cacti and Percona plugins can be found in their web sites, I decided to put it all together and take a few screenshots, so if you happen to do something similar, this is what I did:

Let's assume that you have a fresh a Debian box (In my case, I created a Debian virtual machine, running testing for this) 

1) Install cacti (and its dependences) using aptitude 
   

```
aptitude install cacti
The following NEW packages will be installed:
  apache2-mpm-prefork{a} apache2-utils{a} apache2.2-bin{a}
  apache2.2-common{a} cacti dbconfig-common{a} libaio1{a}
  libapache2-mod-php5{a} libaprutil1-ldap{a} libcairo2{a} libdatrie1{a}
  libdbd-mysql-perl{a} libdbi-perl{a} libdbi0{a} libfontenc1{a}
  libhtml-template-perl{a} libmysqlclient16{a} libnet-daemon-perl{a}
  libonig2{a} libpango1.0-0{a} libpango1.0-common{a} libphp-adodb{a}
  libpixman-1-0{a} libplrpc-perl{a} libpng12-0{a} libqdbm14{a} librrd4{a}
  libthai-data{a} libthai0{a} libx11-6{a} libx11-data{a} libxau6{a}
  libxcb-render-util0{a} libxcb-render0{a} libxcb1{a} libxdmcp6{a}
  libxfont1{a} libxft2{a} libxrender1{a} mysql-client-5.1{a}
  mysql-client-5.5{ab} mysql-common{a} mysql-server{a} mysql-server-5.5{ab}
  mysql-server-core-5.5{ab} php5-cli{a} php5-common{a} php5-mysql{a}
  php5-snmp{a} php5-suhosin{a} rrdtool{a} x-ttcidfont-conf{a} x11-common{a}
  xfonts-encodings{a} xfonts-utils{a}
```

2) Setup your timezone in php.ini

There are three different "php.ini" files in Debian:

```
/etc/php5/cli/php.ini
/etc/php5/apache2/php.ini
/etc/php.ini
```

Cacti is using PHP over the CLI, so it should be enough if you update "/etc/php5/cli/php.ini", but I would update all of them. Add this line to the file:

```
date.timezone = "<time zone>" 
```

For example: 

```
date.timezone = "Asia/Tokyo"
```

The list of timezones is available [here](http://www.php.net/manual/en/timezones.php)

3) Apache+MySQL+PHP+Cacti working out of the box with just a few commands

At this point you should be able to login in cacti on your apache webserver (http://VM_IP/cacti) 
    
The default user/password is admin/admin. (the system will ask you to change it after the first login)  

I won't explain Cacti configuration, because they already have very good documentation in their [web site](http://www.cacti.net/documentation.php)
    
4) Setting up the SSH authentication

This chapter is pretty well explained in Percona web site. 

Basically we have to:   
- Set up an SSH keypair for SSH authentication.   
- Create a user on the remote server you want to graph.  
- Install the public key into that user.s authorized_keys file.   

Check the detailed information here [Percona: SSH-Based Templates](http://www.percona.com/doc/percona-monitoring-plugins/cacti/ssh-based-templates.html#installing-ssh-based-templates)

5) Installing "Percona Monitoring Plugins for Cacti"     
    
Download the plugins from Percona website: [Percona - Download percona-monitoring-plugins](http://www.percona.com/downloads/percona-monitoring-plugins/)

```
wget http://www.percona.com/redir/downloads/percona-monitoring-plugins/percona-monitoring-plugins-1.0.1.tar.gz
```

Extract the file, and copy ss_get_by_ssh.php to Cacti script folder:

```
tar zxvf percona-monitoring-plugins-1.0.1.tar.gz
cp percona-monitoring-plugins-1.0.1/cacti/scripts/ss_get_by_ssh.php /usr/share/cacti/site/scripts/
```

Edit /usr/share/cacti/site/scripts/ss_get_by_ssh.php with your remote user name and the path to the SSH identity from step 4.

```
$ssh_user   = 'cacti';                          # SSH username
$ssh_iden   = '-i /var/www/cacti/.ssh/id_rsa';  # SSH identity
```

6) Importing the template

Inside the tar file you will find different templates:
      
```
/percona-monitoring-plugins-1.0.1/cacti/templates# ls -lrt
total 1644
-rw-r--r-- 1 501 root 169078 Jun 15 04:44 cacti_host_template_percona_openvz_server_ht_0.8.6i-sver1.0.1.xml
-rw-r--r-- 1 501 root  41926 Jun 15 04:44 cacti_host_template_percona_nginx_server_ht_0.8.6i-sver1.0.1.xml
-rw-r--r-- 1 501 root 843345 Jun 15 04:44 cacti_host_template_percona_mysql_server_ht_0.8.6i-sver1.0.1.xml
-rw-r--r-- 1 501 root  89553 Jun 15 04:44 cacti_host_template_percona_mongodb_server_ht_0.8.6i-sver1.0.1.xml
-rw-r--r-- 1 501 root  75491 Jun 15 04:44 cacti_host_template_percona_memcached_server_ht_0.8.6i-sver1.0.1.xml
-rw-r--r-- 1 501 root  53416 Jun 15 04:44 cacti_host_template_percona_jmx_server_ht_0.8.6i-sver1.0.1.xml
-rw-r--r-- 1 501 root 249175 Jun 15 04:44 cacti_host_template_percona_gnu_linux_server_ht_0.8.6i-sver1.0.1.xml
-rw-r--r-- 1 501 root  73879 Jun 15 04:44 cacti_host_template_percona_apache_server_ht_0.8.6i-sver1.0.1.xml
-rw-r--r-- 1 501 root  37198 Jun 15 04:44 cacti_host_template_percona_redis_server_ht_0.8.6i-sver1.0.1.xml
```

In our case, we are only interested in **cacti_host_template_percona_gnu_linux_server_ht_0.8.6i-sver1.0.1.xml**

Console -> Import Templates -> Import template:     

7) Create a new device

Under Console -> Devices, add a new device:

And complete the information (name, IP)


8) Modify the data template and the data source (I will use "Percona Disk Space DT" as example)

Check the Percona Data Templates availables:

Console -> Data Templates

Click in the name (Percona Disk Space DT) and let's edit the template. Go to the bottom of the page, and check "Custom Data":

Add Percona Disk Space DT as new data source for your host. Under Console -> Data Sources, click new:

Set the Remote IP and the partition (or volumen) to monitor


9) Create the graph

We have already created the data source for our remote server, so now, we just have to create a new graph.

Under Console -> Graph Management, add a new graph:

And select our data source (Percona Disk Space DT) for the device:

10) Result

At this point, the graph should be created, and after a few minutes, you should see something like this:

If something goes wrong, take a look at /var/log/cacti/cacti.log. There should be a line similar to this one:

```
<DATE> - CMDPHP: Poller[0] Host[3] DS[25] CMD: /usr/bin/php -q /usr/share/cacti/site/scripts/ss_get_by_ssh.php --host <REMOTE_IP> --type proc_stat --items gg,gh,gi,gj,gk,gl,gm,gn,go, output: gg:50994 gh:0 gi:76580 gj:56353167 gk:1617 gl:0 gm:0 gn:2135 go:-1
```
