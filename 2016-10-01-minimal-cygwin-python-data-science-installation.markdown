title: Minimal Cygwin Python data science installation
date: 2016-10-01
slug: 2016-10-01-minimal-cygwin-python-data-science-installation
tags: cygwin, windows, python, data science
category: posts

A few days ago a made a fresh installation of Cygwin64 in order to tinker with [Pandas](http://pandas.pydata.org/) (just for fun) but instead of installing [Anaconda](https://www.continuum.io/anaconda-subscriptions/anaconda), I decided to install each component manually... I will write it here because I'm sure I will need it in the future (and it might be handy for someone else). 

First things first, package management in Cygwin is a bit of a pain without [apt-cyg](https://github.com/transcode-open/apt-cyg)... So download it from [here](https://raw.githubusercontent.com/transcode-open/apt-cyg/master/apt-cyg), save it as "apt-cyg" in your $HOME and install it: 

```
install apt-cyg /bin/
```
