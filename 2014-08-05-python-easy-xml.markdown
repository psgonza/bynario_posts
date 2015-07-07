title: Python: Parsing XML files
date: 2014-08-05 13:14
slug: 2014-08-05-python-easy-xml
category: posts
tag: - python , coding , dictionary , tables

{% img center https://dl.dropboxusercontent.com/u/14814182/blog/python-xml.jpg  'xml' %}

I would like to continue with another small python example. As in the previous post,this is more like  a "note to self" thing than a educational post (this is not [stackoverflow](http://stackoverflow.com/questions/tagged/python)), but anyway,  I guess it could be handy for someone(it has been for me...)

This time, I have been testing two xml submodules availables in Python 2.7 XML package:

* [xml.dom](https://docs.python.org/2/library/xml.dom.html#module-xml.dom): the DOM API definition 
* [xml.etree.ElementTree](https://docs.python.org/2/library/xml.etree.elementtree.html#module-xml.etree.ElementTree): the ElementTree API, a simple and lightweight XML processor 


Both of them are pretty easy to use, and I haven't found any real difference in time execution (at least for basic filters) between both submodules.  

I will use this xml file as an example:

``` 
<?xml version="1.0"?>
<data>
	<node>
		<attribute>Front End</attribute>
		<res_ids>100</res_ids>
		<nEName>BALDR</nEName>
		<ipList>192.168.0.5</ipList>
		<ipv6List>fe80::1:f6f1:fe01:12</ipv6List>
		<so>Ubuntu</so>
		<kernel>3.0.93</kernel>
	</node>
	<node>
		<attribute>Web</attribute>
		<res_ids>12</res_ids>
		<nEName>THOR</nEName>
		<ipList>192.168.0.20</ipList>
		<ipv6List>fe80::1:f6f1:fe01:12</ipv6List>
		<so>Ubuntu</so>
		<kernel>3.0.93</kernel>
	</node>
	<node>
		<attribute>Storage</attribute>
		<res_ids>200</res_ids>
		<nEName>VALI</nEName>
		<ipList>192.168.0.10</ipList>
		<ipv6List>fe80::1:f6f1:fe01:12</ipv6List>
		<so>Ubuntu</so>
		<kernel>3.0.93</kernel>
	</node>
	<node>
		<attribute>DB</attribute>
		<res_ids>230</res_ids>
		<nEName>LOKI</nEName>
		<ipList>192.168.0.110</ipList>
		<ipv6List>fe80::1:f6f1:fe01:12</ipv6List>
		<so>Fedora</so>
		<kernel>3.0.93</kernel>
	</node>
	<node>
		<attribute>Backup</attribute>
		<res_ids>300</res_ids>
		<nEName>OTHER</nEName>
		<ipList>192.168.0.103</ipList>
		<ipv6List>fe80::1:f6f1:fe01:12</ipv6List>
		<so>Debian</so>
		<kernel>3.0.93</kernel>
	</node>
</data>
```

In the tree, we can see that we have one "data" object containing several "nodes", each one with 7 "attributes". 

Now let's extract the nodes and all the attributes using xml.dom:

``` 
#!/usr/bin/env python
from xml.dom import minidom

def parse_file(filename):
    xmldoc = minidom.parse(filename)
    for node in xmldoc.getElementsByTagName('node'):
        print str(node.getElementsByTagName("attribute")[0].firstChild.nodeValue),
        print str(node.getElementsByTagName("res_ids")[0].firstChild.nodeValue),
        print str(node.getElementsByTagName("nEName")[0].firstChild.nodeValue),
        print str(node.getElementsByTagName("ipList")[0].firstChild.nodeValue)
        print str(node.getElementsByTagName("ipv6List")[0].firstChild.nodeValue)
        print str(node.getElementsByTagName("so")[0].firstChild.nodeValue)
        print str(node.getElementsByTagName("kernel")[0].firstChild.nodeValue)
     
parse_file("./fakexml.xml")
```

And now, the same thing using xml.etree.ElementTree:

``` 
#!/usr/bin/env python
from xml.etree import ElementTree as ET

def parse_file(filename):
    tree = ET.ElementTree(file=filename)
    attrList = [ entry for entry in tree.findall('./node') ]
    for e in attrList:
        print e.find('attribute').text,
        print e.find('res_ids').text,
        print e.find('nEName').text,
        print e.find('ipList').text,
        print e.find('ipv6List').text,
        print e.find('so').text,
        print e.find('kernel').text
   
parse_file("./fakexml.xml")
```

The output from any of the two options would be the same one:

```
$> python print_table_from_xml_dom.py
Front End 100 BALDR 192.168.0.5 fe80::1:f6f1:fe01:12 Ubuntu 3.0.93
Web 12 THOR 192.168.0.20 fe80::1:f6f1:fe01:12 Ubuntu 3.0.93
Storage 200 VALI 192.168.0.10 fe80::1:f6f1:fe01:12 Ubuntu 3.0.93
DB 230 LOKI 192.168.0.110 fe80::1:f6f1:fe01:12 Fedora 3.0.93
Backup 300 OTHER 192.168.0.103 fe80::1:f6f1:fe01:12 Debian 3.0.93


$> python print_table_from_xml_element.py
Front End 100 BALDR 192.168.0.5 fe80::1:f6f1:fe01:12 Ubuntu 3.0.93
Web 12 THOR 192.168.0.20 fe80::1:f6f1:fe01:12 Ubuntu 3.0.93
Storage 200 VALI 192.168.0.10 fe80::1:f6f1:fe01:12 Ubuntu 3.0.93
DB 230 LOKI 192.168.0.110 fe80::1:f6f1:fe01:12 Fedora 3.0.93
Backup 300 OTHER 192.168.0.103 fe80::1:f6f1:fe01:12 Debian 3.0.93
```

As I mentioned before, for this small exercise, there is no difference in terms of time of execution between both alternatives. I created a "slightly bigger" xml file (100x bigger), and the times were still pretty much the same, so I would say it is not a matter of the size, but the complexity of the filters.

To wrap up, let's put this code together with the one in the previous post, where we created a table from the data in a dictionary.

I use ElementTree instead of xml.dom, but both of them would work. Here's the code:

``` 
#!/usr/bin/env python
import sys
from collections import defaultdict
from xml.etree import ElementTree as ET

if sys.argv[1:]:
    inputF = sys.argv[1]
else:
    print "ERROR. Execution failed: missing file"
    sys.exit(1)

total = defaultdict(list)

# Column width
AttrColLen=25
ValueColLen=20
colwidth1="{0:<"+str(AttrColLen)+"}"
colwidth2="{0:<"+str(ValueColLen)+"}"


# Printer function
def print_table(data):
    for key, values in sorted(total.items()):
        print "|" + AttrColLen*"-" + ((ValueColLen+2)*len(values))*"-" + "-|"
        print  "| " + colwidth1.format(key) + "|",
        for i in xrange(len(values)):
            print colwidth2.format(values[i])+"|",
        print ""
    print "|" + AttrColLen*"-" + ((ValueColLen+2)*len(values))*"-" + "-|"

# Parse the xml, and add the items to the dictionary
def parse_file(filename):
    colwidth1="{0:<30}"
    colwidth2="{0:<25}"

    tree = ET.ElementTree(file=filename)
    attrList = [ entry for entry in tree.findall('./node') ]
    for e in attrList:
        total["Attribute"].append(e.find('attribute').text)
        total["RES_ID"].append(e.find('res_ids').text)
        total["networkElementName"].append(e.find('nEName').text)
        total["IPv4"].append(e.find('ipList').text)
        total["IPv6"].append(e.find('ipv6List').text)
        total["SO"].append(e.find('so').text)
        total["Kernel"].append(e.find('kernel').text)


parse_file(inputF)
print_table(total)
```

And here is the result:


```
$> python print_table_from_dict.py fakexml.xml
|----------------------------------------------------------------------------------------------------------------------------------------|
| Attribute                | Front End           | Web                 | Storage             | DB                  | Backup              |
|----------------------------------------------------------------------------------------------------------------------------------------|
| IPv4                     | 192.168.0.5         | 192.168.0.20        | 192.168.0.10        | 192.168.0.110       | 192.168.0.103       |
|----------------------------------------------------------------------------------------------------------------------------------------|
| IPv6                     | fe80::1:f6f1:fe01:12| fe80::1:f6f1:fe01:12| fe80::1:f6f1:fe01:12| fe80::1:f6f1:fe01:12| fe80::1:f6f1:fe01:12|
|----------------------------------------------------------------------------------------------------------------------------------------|
| Kernel                   | 3.0.93              | 3.0.93              | 3.0.93              | 3.0.93              | 3.0.93              |
|----------------------------------------------------------------------------------------------------------------------------------------|
| RES_ID                   | 100                 | 12                  | 200                 | 230                 | 300                 |
|----------------------------------------------------------------------------------------------------------------------------------------|
| SO                       | Ubuntu              | Ubuntu              | Ubuntu              | Fedora              | Debian              |
|----------------------------------------------------------------------------------------------------------------------------------------|
| networkElementName       | BALDR               | THOR                | VALI                | LOKI                | OTHER               |
|----------------------------------------------------------------------------------------------------------------------------------------|
```

</bye!> ;)

