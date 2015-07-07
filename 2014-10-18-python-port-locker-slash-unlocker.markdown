title: Python port locker/unlocker
date: 2014-10-18 17:58:30 +0200
slug: 2014-10-18-python-port-locker-slash-unlocker
tags: python,coding,iptables

If you run you own server (VPS, cloud instances, bare metal, no matter what), I guess you check your log files almost on daily basis, and probably you already run any the multiple available tools (the list is pretty long, but in my case I use [logwatch](http://sourceforge.net/projects/logwatch/)) that helps you out to keep an eye in what is going on in your machine.

There is one thing in particular that drives me crazy. As soon as you expose a machine to the internet, people will be constantly trying to get SSH access:

{% img center https://dl.dropboxusercontent.com/u/14814182/blog/logwatch_print.JPG  'logwatch' %}

Obviously, there are tools that will make your life easier (like [fail2ban](http://www.fail2ban.org/wiki/index.php/Main_Page), totally recommended) and some security tweaks (just Google for "linux server hardening") that you should be doing if you manage your own server, but even with all those things in place, I really don't like people trying to access to my server 24/7... Call me crazy, if you like.

A simple solution would have been just to change the SSH port to a something different, like 22222, and this might work for you. Unfortunately, I spend most of the time behind a proxy server, and I can only access the most common ports (the same very ones that are scanned on regular basis), so I needed to keep ssh service running in port 22.

Last year I coded a small bash script (just for myself) that allowed me to open and close SSH port on demand. It is been working fine since then, and I don't get annoyed by the ssh auth errors in logwath anymore. Now I migrated it to Python, and I thought it could be useful for anyone else, so here it is...

The script is not big deal, it will open/close certain ports that I preconfigured by reading a text file. Nothing new under the sun.

The only "particularity" of this script is that, instead of creating a service (webservice??) that allowed me to send the desired ssh state to the server (this would add another "attack vector"), I decided the script would pull the data from another location. I decided to use a (public) file in my dropbox folder. 

If you want it to give it a try, these are the steps:

1 - Create a file in your dropbox folder and get the public link. Let's say that you get something like this: 

``` 
https://www.dropbox.com/s/XXXXXXXXXXXXXXXX/file
```

2 - Add to this file any of the options you defined in the script. ie: 

``` 
open ssh
close vpn
open  httpd
```

3 - Upload this script to your server (github link available in the code snippet header):

``` 
#!/usr/bin/env python
#===============================================================================
#
#          FILE:  portLocker.py
#         USAGE:  python portLocker.py
#   DESCRIPTION:  port blocker/unblocker via dropbox file
#        AUTHOR:  http://bynario.com
#       CREATED:  05/08/2014 17:37:23 CEST
#===============================================================================
import time,sys,os,subprocess

dbFile="https://www.dropbox.com/s/XXXXXXXXXXXXXXXX/file"
logFile="/var/log/locker_log.log"
htmlFile="/usr/share/nginx/www/sshd.html"
servicesL={'ssh':'22','vpn':'1723','httpd':'80'}
actionsL=('open','close')

def createWWW(filename):
    try:
        with open(filename, 'w') as f: f.write(getTime())
    except Exception as e:
        print2Log(logFile,"Error opening "+filename+": "+str(e))

def print2Log(filename,text):
    try:
        with open(filename, 'w') as f:
            f.write(getTime() + " - " + text+"\n")
    except Exception as e:
        print2Log(logFile,"Error opening "+filename+": "+str(e))

def print2WWW(filename,service,status):
    try:
        with open(filename, 'a') as f:
            f.write(" - Service: " + service + " - Status: " + status)
    except Exception as e:
        print2Log(logFile,"Error opening "+filename+": "+str(e))

def getTime():
    return time.strftime("%Y-%m-%d %H:%M")

def runCommand(command,retVal=None):
    try:
        p = subprocess.Popen(command.split(), stdout=subprocess.PIPE,stderr=subprocess.PIPE,shell=False)
        (output, err) = p.communicate()
        if retVal: return output
        else: return None
    except Exception as e:
        print2Log(logFile,"Error executing "+str(command)+": "+str(e))

def checkDBFile():
    action="/usr/bin/curl -s -L " + dbFile
    command=runCommand(action,retVal=True)
    return command

def runIptables(service,action):
    print getTime() + " Running iptables... Service: " + service + " Action:" + action
    check_command="/sbin/iptables -C INPUT -p tcp -m tcp --dport " + servicesL[service] +" -j DROP"
    try:
        check_output = subprocess.check_call(check_command.split(),stderr=subprocess.STDOUT)
    except subprocess.CalledProcessError as e:
        if action == "close": 
            command="/sbin/iptables -A INPUT -p tcp --dport " + servicesL[service] + " -j DROP"
        else: 
            return True
    else:
        if action == "open":
            command="/sbin/iptables -D INPUT -p tcp --dport " + servicesL[service] + " -j DROP"
        else: 
            return

    runCommand(command)

if __name__ == "__main__":
    createWWW(htmlFile)

    for line in checkDBFile().splitlines():
        if not line: continue
        if len(line.split()) != 2:
            print2Log(logFile,"Error in dropbox file: wrong format")
            sys.exit(1)
        else:
            act = line.split()[0]
            serv = line.split()[1]

            if servicesL.has_key(serv) and act in actionsL:
                runIptables(serv,act)
                print2WWW(htmlFile,serv,act)
            else:
                print
                print2Log(logFile,"Service or Action found in file not valid... Skipping: " + act + "-" + serv)
```

4 - Now check these variables in the script:

```
dbFile="https://www.dropbox.com/s/XXXXXXXXXXXXXXXX/file"
logFile="/var/log/locker_log.log"
htmlFile="/usr/share/nginx/www/sshd.html"
servicesL={'ssh':'22','vpn':'1723','httpd':'80'}
```

Adapt the script to fit your environment:

* Replace _dbFile_ by the URL you got from Dropbox
* Set the path you need for the _logFile_
* The script will write the status of the services to my web server, so I can check the status easily. Set your path in _htmlFile_.
* Define the list of services you want to manage in _servicesL_. This has to match with the info you define in the dropbox file.

5 - Create an entry in crontab with the periodicity you need. In my case, opening/closing ssh port is not critical, so, as I don't want Dropbox people to get angry at me, I set it up to 5 minutes :)

```
 */5 * * * * python /path/to/portLocker.py &>> /var/log/locker_log.log
```

Now all you need to do is modify your dropbox file, and after a few minutes, check if you get the result you expected ;)

There is one main improvement that could be done, and it is adding some kind of signature to the file in order to ensure nobody is messing with us, but I have pretty good reasons not to do it (at the moment):

* I update the dropbox file from mobile devices every now and then, and adding a signature to the file would make this way too complex.
* This file is stored in my dropbox folder, so **theoretically**, nobody but me is able to modify it.
* The security of the server does not rely on this script. This is not meant to be "[security through obscurity](https://en.wikipedia.org/wiki/Security_through_obscurity)".

If you are after something more professional, I recommend you to check [Latch](https://www.elevenpaths.com/technology/latch/index.html).

And that's all folks... It is working pretty well for me, and I hope it works for you... 

See you around
