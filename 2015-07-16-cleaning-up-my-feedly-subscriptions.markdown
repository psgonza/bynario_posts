title: Cleaning up my feedly subscriptions
date: 2015-07-16 10:10:10
slug: 2015-07-16-cleaning-up-my-feedly-subscriptions
tags: python, rss, scripts
category: posts

(I wrote this post in my way back to the old continent... According to the map, we were flying over [Jarenga river](https://en.wikipedia.org/wiki/Yarenga_River), in the mother Russia) 

I don't know you, but I have been using RSS clients since, well, foverer I'd say... First [Akregator](https://www.kde.org/applications/internet/akregator/) in KDE, then Google Reader (I still wonder why Google had to shut it down) and finally, as most of the people did, I moved to [Feedly](http://feedly.com), and I am pretty happy with it, no complains at all.

A few days ago I was taking a look at my RSS subscriptions and there were more than 400 websites that I have been adding during all these years, so I deciced to clean it up a little bit using Python.

I came up with the script you can see below... It is fairly simple (and it uses only standard library modules) but does the job for me. 

In order to use it, you just need to export your feedly subscriptios (you will get an opml file) and execute the script. You will get a new file without the "dead" links ready to be imported.

Usage:

` # python3 feedly_checker.py myfeedlyrss.opml `


Code:

```
#!/usr/bin/python3
import os
import sys
from xml.dom import minidom

import http.client
from urllib.parse import urlparse

errors = []

def usage():
    print("Usage: python3 " + sys.argv[0] + " <feedly_export_file>.opml")
    sys.exit(0)


def parse_feed(xmlfile):
    feeds = []
    for _ in range(len(xmlfile)):
        tmp = {}
        if xmlfile[_].getAttribute('type') == "rss":
            tmp[xmlfile[_].getAttribute('text')] = xmlfile[_].getAttribute('xmlUrl')
            feeds.append(tmp)
    return feeds


def check_http_response(url):
    for name, rss in url.items():
        addr = urlparse(rss)
        try:
            conn = http.client.HTTPConnection(addr[1], 80, timeout=3)
            conn.request("HEAD", addr[2])
            res = conn.getresponse()
            if res.status >= 400:
                errors.append(rss)
                print("Name: %s, URL:%s, Error Code:%d" % (name, rss, res.status))
        except http.client.HTTPException:
            errors.append(rss)
        finally:
            conn.close()


def load_xml(filename):
    long_input_file = os.path.realpath(filename)
    try:
        xmldoc = minidom.parse(long_input_file)
        item_res = xmldoc.getElementsByTagName('outline')
    except Exception:
        print("Error parsing xml file")
        sys.exit(1)
    return item_res


def clean_xml(filename, errors):
    xmldoc = minidom.parse(filename)
    doc_root = xmldoc.getElementsByTagName('outline')

    for j in doc_root:
        if j.getAttribute('type') == "rss" and j.getAttribute('xmlUrl') in errors:
            j.parentNode.removeChild(j)
            print("Removing... %s" % (j.getAttribute('xmlUrl')))

    try:
        f = open(filename + "_clean", "w")
        f.write(xmldoc.toxml())
        print("Creating %s_clean file..." % filename)
    except IOError:
        print("Error creating %s_clean file" % filename)
    else:
        f.close()

if len(sys.argv) != 2:
    usage()
else:
    input_file = sys.argv[1]

outlineItems = load_xml(input_file)

results = parse_feed(outlineItems)

print("%d entries found in %s" % (len(results), input_file))

for site in results:
    check_http_response(site)

print("%d wrong entries found" % (len(errors)))

clean_xml(input_file, errors)

sys.exit(0)
```

It's been a quick excersice (POC?), so there is ***a lot*** of room for improvement... 

1. It is not fast, because it is single-thread, and it has to wait for the http connection to timeout. 
2. Any error code above 400 will make the link to be removed... Some error codes 5xx are "temporary server errors", but they will be deteled as well... It'd need some extra work

\\\psgonza
