title: Python: print table from dictionary of lists
date: 2014-07-31 13:14
slug: 2014-07-31-python-print-table-from-dictionary
tag: python , coding , dictionary , tables

{% img center https://dl.dropboxusercontent.com/u/14814182/blog/python-logo-inkscape.jpg  'python' %}

Hello 

I have been trying to find an easy way to create ascii tables in Python using data from a dictonary of lists. I couldn't find anything that suited me, so I came up with this small piece of code

{% codeblock %}
def print_table(data):
    for key, values in sorted(total.items()):
        print "|" + AttrColLen*"-" + ((ValueColLen+2)*len(values))*"-" + "-|"
        print  "| " + colwidth1.format(key) + "|",
        for i in xrange(len(values)):
            print colwidth2.format(values[i])+"|",
        print ""
    print "|" + AttrColLen*"-" + ((ValueColLen+2)*len(values))*"-" + "-|"
{% endcodeblock %}

It's pretty simple, and does the job. Let me show it to you with an example


``` 
#!/usr/bin/env python
from collections import defaultdict

total = defaultdict(list)

# Column width
AttrColLen=25
ValueColLen=15
colwidth1="{0:<"+str(AttrColLen)+"}"
colwidth2="{0:<"+str(ValueColLen)+"}"

# Fake data 
attribute=['Front End','Web','Storage']
res_ids=['100','12','200']
nEName=['BALDR','THOR','VALI']
ipList=['192.168.0.5','192.168.0.20','192.168.0.10']

# Printer
def print_table(data):
    for key, values in sorted(total.items()):
        print "|" + AttrColLen*"-" + ((ValueColLen+2)*len(values))*"-" + "-|"
        print  "| " + colwidth1.format(key) + "|",
        for i in xrange(len(values)):
            print colwidth2.format(values[i])+"|",
        print ""
    print "|" + AttrColLen*"-" + ((ValueColLen+2)*len(values))*"-" + "-|"



# Populating the dictionary
for iter1 in attribute:
    total["Attribute"].append(iter1)
for iter2 in res_ids:
    total["RES_ID"].append(iter2)
for iter3 in nEName:
    total["networkElementName"].append(iter3)
for iter4 in ipList:
    total["IPv4"].append(iter4)	

# Let's print the dictionary
print total

```

Obviously there are other (better!!) ways to print a dictionary (like [pprint](https://docs.python.org/2/library/pprint.html)),anyway I just wanted to show you how the dictionary looks like 

```
defaultdict(<type 'list'>, {'Attribute': ['Front End', 'Web', 'Storage'],
'RES_ID': ['100', '12', '200'], 
'networkElementName': ['BALDR', 'THOR', 'VALI'], 
'IPv4': ['192.168.0.5', '192.168.0.20', '192.168.0.10']})
```

Now let's replace:

```
# Let's print the dictionary
print total
```

By:

```
# Let's print the dictionary
print_table(total)
```

This is the result:

```
|-----------------------------------------------------------------------------|
| Attribute                | Front End      | Web            | Storage        |
|-----------------------------------------------------------------------------|
| IPv4                     | 192.168.0.5    | 192.168.0.20   | 192.168.0.10   |
|-----------------------------------------------------------------------------|
| RES_ID                   | 100            | 12             | 200            |
|-----------------------------------------------------------------------------|
| networkElementName       | BALDR          | THOR           | VALI           |
|-----------------------------------------------------------------------------|
```

It looks pretty nice, and the code is really simple...

I hope it helps

Byte!




