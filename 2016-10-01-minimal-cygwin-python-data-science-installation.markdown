title: Minimal Cygwin Python data science installation
date: 2016-10-01
slug: 2016-10-01-minimal-cygwin-python-data-science-installation
tag: cygwin, windows, python, data science
category: posts

A few days ago a made a fresh installation of Cygwin64 in order to tinker with [Pandas](http://pandas.pydata.org/) (just for fun) but instead of installing [Anaconda](https://www.continuum.io/anaconda-subscriptions/anaconda), I decided to install each component manually... I will write it here because I'm sure I will need it in the future (and it might be handy for someone else). 

First things first, package management in Cygwin is a bit of a pain without [apt-cyg](https://github.com/transcode-open/apt-cyg)... So download it from [here](https://raw.githubusercontent.com/transcode-open/apt-cyg/master/apt-cyg), save it as "apt-cyg" in your $HOME and install it: 

```
install apt-cyg /bin/
```

Start installing stuff:
```
apt-cyg install wget curl gcc-core g++ libzmq-devel libzmq5 python3 python3-zmq libfreetype-devel
```

Download and install pip:

```
wget https://bootstrap.pypa.io/get-pip.py
python get-pip.py
```

Some of the libraries included in cygwin repository are a little bit old, so it is better to download and compile them from pip... ie:

```
Cython (0.24.1)  - The Cython compiler for writing C extensions for the Python language.
  INSTALLED: 0.22
  LATEST:    0.24.1
```

Install python stuff from pip:

```
pip install cython
pip install jupyter
pip install numpy
pip install pandas
pip install matplotlib
```

And that's it... It just take a few minutes to have everything installed and compiled, but it is not too complicated.

```
psgonza@host$ ipython
Python 3.4.3 (default, May  5 2015, 17:58:45)
Type "copyright", "credits" or "license" for more information.

IPython 5.1.0 -- An enhanced Interactive Python.
?         -> Introduction and overview of IPython's features.
%quickref -> Quick reference.
help      -> Python's own help system.
object?   -> Details about 'object', use 'object??' for extra details.

In [1]: import pandas as pd
```

One remark, in this installation, there is no [Fortran](https://en.wikipedia.org/wiki/Fortran) support, so in case you need it, you'd need to compile [ATLAS](http://math-atlas.sourceforge.net/atlas_install/), [LAPACK](http://www.netlib.org/lapack/), etc.. Last time I did it a couple of years ago, it took me a while, and I'd say I have never really used it, so this time I skipped it. If you really need them, Cygwin might not be what you should be using (IMO)...

Later!

PS: Using Anaconda is a perfectly valid option as well, but it is built for Windows, so you need to link it with your cygwin python installation. You can check [this link](http://stackoverflow.com/questions/24255407/permanently-set-python-path-for-anaconda-within-cygwin) in case you decide to go that way. 
