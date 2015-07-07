title: "TIP: Script para creacion de usuarios en Linux"
date: 2011-01-14T03:23:00+01:00
slug: 2011-01-14-tip-script-para-creacion-de-usuarios-en.markdown
category: posts
tag:  linux, bash

In case you need to define a lot of users in a Unix/Linux box, this small script could save you some time:

{% codeblock create_bulk_user.sh %}
!/bin/bash
file=.user_list.txt.
for i in `cat $file`
do
user=`echo $i | cut -d.;. -f1`
pass=`echo $i | cut -d.;. -f2`
echo .Creating user: $user.
useradd -m $user && echo .DONE. || echo .FAIL.
echo .Creating entry in login.allow for user: $user.
echo .$user all. >> /etc/login.allow
echo .Setting the password for user: $user.
echo .$pass. | passwd .stdin .$user. && echo .DONE. || echo .FAIL.
echo .$user DONE.
echo
done
{% endcodeblock %}

You just need to create the "user_list.txt" file containing your list of users/passwords in this format:    

{% codeblock %}
usuario1;pass1
usuario2;pass2
{% endcodeblock %}

The script will create the user, set the password, and create an entry in allow.login


Bye!

