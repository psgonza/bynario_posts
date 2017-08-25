title: Embedding modules in Python scripts
date: 2015-09-21 10:10:10
slug: 2015-09-21-embedding-modules-in-python-scripts
tags: python, scripts
category: posts

I am a huge fan of [pexpect](https://github.com/pexpect/pexpect "pexpect"). For those of you who don't know it, it is a module "based" on the same idea as [Expect](http://www.tcl.tk/man/expect5.31/expect.1.html) but in Python.  As its author said:

>  I loved Expect, but I hated TCL, so I wrote this 100% pure Python module that does the same thing

[Noah Spurrier](http://www.noah.org/python/)

I think it's a great tool for ssh automatization whenever priv/pub keys are not an option. 

In my case, since I usually work in isolated systems where I cannot even install local python mudules, and given that I am using just a small part of all the funcionalities provided by this module, I came up with the idea of embedding an old version of pexpect module into my scripts, so I don't have to worry about SCP'ing several files each time I need to run any of them.

The idea is to embed the base64 code of the `pexpect.py` module into the script, and then create the module on the fly whenever the script is executed (if it is not there already).

Simple stuff.... This is the procedure for Python 2.x:

1) Get the base64 code of the [module](https://raw.githubusercontent.com/psgonza/bynario/master/pexpect.py):

```
LOCAL $ gzip -c pexpect.py | base64
H4sICFG7GFUAA2luY19wZXhwZWN0LnB5AKxbe3PbNrbX58C43RG8laWnXQ7veud7F3FlhNtbdkj
[...]
8Ys0SwQW+DuQhDqXlYSw8fuPPtrNbsrDtjn65LC5Nf8xR4nZvEa2O5y1R0uUHA4/+n+riV9boSgB
AA==
$
```

2) Get MD5 checksum:

```	
LOCAL $ md5sum pexpect.py
1d9643479e2bf16939fcdf007f4bf9f9 *pexpect.py
```

3) Add the necessary modules to the script:
	
```
import sys
from hashlib import md5 
from cStringIO import StringIO    
from base64 import b64decode 
from gzip import GzipFile
```

4) Import the module, or create it if it doesn't exist:

```
try:      
	import pexpect
except ImportError:  
	#Copy and paste the base64 code from step 1
	pexpect_mod = """   
	H4sICFG7GFUAA2luY19wZXhwZWN0LnB5AKxbe3PbNrbX58C43RG8laWnXQ7veud7F3FlhNtbdkj  
	[...]
	8Ys0SwQW+DuQhDqXlYSw8fuPPtrNbsrDtjn65LC5Nf8xR4nZvEa2O5y1R0uUHA4/+n+riV9boSgB    
	AA==   
	"""

	#MD5 sum from step 2
	pexpect_mod_md5 = "1d9643479e2bf16939fcdf007f4bf9f9"
	
	#Decode the module stored in pexpect_mod and load it in a variable
	with GzipFile(mode='r', fileobj=StringIO(b64decode(pexpect_mod))) as pexpect_mod_fd:
	    pexpect_mod_data = pexpect_mod_fd.read()
	
	#Dump the variable into a file
	with open("pexpect.py","w+b") as pexpect_fd:
	    pexpect_fd.write(pexpect_mod_data)
	
	#Double-check pexpect.py is the same file you have in your local machine    
	with open("pexpect.py", 'rb') as pexpect_fd:
		if pexpect_mod_md5 == md5(pexpect_fd.read()).hexdigest():
	 		#Import the module
			import pexpect
	    else:
			#Exit if the file is not identical 
			print("Error creating pexpect.py module. MD5 checksum incorrect. Exiting")
			sys.exit(-1)
```

And voila, just you run your script in your remote machine, and there you have the module in your current directory:
     
```
REMOTE $ ls -1 pexpect.py*
pexpect.py*
pexpect.pyc*
```

Needless to say you could follow this same approach for any other file you would need to "attach" to your scripts... 

Have fun!
