title: We love you Bash
date: 2016-02-09 11:11:11
slug: 2016-02-09-we-love-you-bash
tag: linux, bash
category: posts
status: draft

We love you Bash… but sometimes you are so damn slow

I'm sure this has happened to you before… I had to create an ldif file containing 70.000 entries like these ones:

```
dn: user=XXXXXXXXXX,rootdn=com
changetype: modify
replace: attr1
attr1: qwerty1
-
replace: attr2
attr2: {"reporting": [{"name":"Total","reportingLevel":"total","subscription":"A", "time":500,"reset":{"main":"30 ","time":"30 "}}}]}
```

(the real data was a little bit larger, this is just an example)

It was supposed to be something really quick and simple, one of those one-liners you write in 3 seconds… I said to myself: **bash + loop + echo + redirection to file = WIN**

Well, it proved me wrong :

The quick and dirty code to be executed in the shell:

```
for i in `seq 1 70000`;
do
echo -e "dn: user=XXXXXXXXXX,rootdn=com
changetype: modify
replace: attr1
attr1: qwerty1
-
replace: attr2
attr2: {\"reporting\": [{\"name\":\"Total\",\"reportingLevel\":\"total\",\"subscription\":\"A\", \"time\":500,\"reset\":{\"main\":\"30 \",\"time\":\"30 \"}}}]}
"  >> modify.ldif
done
```

So I issued that command, and went out for a bite… 30 minutes later, the [fraking](http://en.battlestarwiki.org/wiki/Frak) thing was still running!!!  How was that even possible…

I realized I was opening the file every time I wrote a line, a nonsense… so I just stopped the script and tweaked it a little bit… **Result: pretty much the same…**  Here's some figures with only 5000 iterations per loop with different redirections:

Initial redirection:

```
for i in `seq 1 5000`; do
…
> "  >> modify.ldif; done
real    4m17.606s
user    1m4.028s
sys     1m35.850s
```

Only one redirection outside of the loop:

```
for i in `seq 1 5000`; do
…
> " ; done >> modify.ldif
real    4m30.218s
user    1m8.496s
sys     1m41.390s
```

I also tried to use a file descriptor, just for the sake of it,  but it has no impact:

```
exec 4>> modify.ldif
for i in `seq 1 5000`; do
…
> "  >&4; done
real    4m32.197s
user    1m10.248s
sys     1m39.294s
```

And using the FD outside of the loop:

```
for i in `seq 1 5000`; do
…
> "; done >&4
real    4m31.478s
user    1m11.740s
sys     1m38.678s
```

So all in all, as you can see, Bash IO redirection performance, basically sucks.

A 20 lines python script (with no optimization whatsoever) is able to generate the whole ldif file (creating 70.000 entries to modify) in less than 1 second:

```
#!/usr/bin/env python

data="""dn: user=XXXXXXXXXX,rootdn=com
changetype: modify
replace: attr1
attr1: qwerty1
-
replace: attr2
attr2: {\"reporting\": [{\"name\":\"Total\",\"reportingLevel\":\"total\",\"subscription\":\"A\", \"time\":500,\"reset\":{\"main\":\"30 \",\"time\":\"30 \"}}}]}
"""

with open("modify.ldif",'w') as outputF:
    for iter in xrange(70000):
        outputF.write(data.replace("XXXXXXXXXX ",str(iter)))
```

Executing it:

```
# time python gen_ldif.py
real    0m0.506s
user    0m0.108s
sys     0m0.052s
```

How about that? Sometimes it's just better to use the right tool for the right job...

So yeah, we love you Bash… Even though you are really slow sometimes.

\\\psgonza
