Title: Interview question: chmod chmod?
Date: 2017-05-09 13:13:17 +0200
Slug: 2017-05-09-Interview-question-chmod-chmod
category: posts
tag: linux

Not sure why, today I remembered about a question from a job interview I made probably 5 years ago... It was a pretty easy question, and my answer was pretty dumb.

Q: "Ok, imagine you remove exectuion permissions to chmod... How would you put them back???"
A: "quick and dirty, I'd copy the binary from the machine sitting next to it". 

How about that? I remember it was very late in the evening (I was working in Tokyo at that time and they called me from Europe), but anyway, there was no excuse. I think they guy who interviewed me is still laughing :)

There are multiple ways to solve it. Using perl, python, go, C++, etc allows you to change the permissions of any file, including chmod... Let me show you how I'd do it in python: 

´´´
# Current permissions: 755
root@94d9407f1002:/# # ls -l /bin/chmod
-rwxr-xr-x 1 root root 56112 Feb 18  2016 /bin/chmod

# Let's remove execution permission
root@94d9407f1002:/# chmod -x /bin/chmod

# New permissions: 644 
root@94d9407f1002:/# ls -l /bin/chmod
-rw-r--r-- 1 root root 56112 Feb 18  2016 /bin/chmod

# Checking execution is not allowed
root@94d9407f1002:/# chmod +x /bin/chmod
bash: /bin/chmod: Permission denied
 
# Here comes python to the rescue
root@94d9407f1002:/# python3
Python 3.5.2 (default, Nov 17 2016, 17:05:23)
[GCC 5.4.0 20160609] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import os
>>> os.chmod("/bin/chmod",0o755)
>>> exit()

# Confirming it worked as expected
root@94d9407f1002:/# ls -lrt /bin/chmod
--wxrw--wt 1 root root 56112 Feb 18  2016 /bin/chmod
´´´

Not sure why this came to my mind today... But in case someone asks you this very same question, don't f$ck it up as I did. Just breathe normally, and think...

///psgonza
