Title: Problems installing IPython Notebook in Cygwin (W7)
Date: 2015-07-08 10:41:04
Slug: 2015-07-08-problems-installing-ipython-notebook-in-cygwin
category: posts
tag: cygwin,windows,ipython

While trying to "pip install [ipython[notebook]](http://ipython.org/)" in my Cygwin installation (W7), I got this error due to libzmq:

```
 Collecting ipython[notebook]

[...]

 Running setup.py install for pyzmq

[...]

gcc -Wno-unused-result -DNDEBUG -g -fwrapv -O3 -Wall -Wstrict-prototypes -ggdb -O2 -pipe -Wimplicit-function-declaration 
-fdebug-prefix-map=/usr/src/ports/python3/python3-3.4.3-1.i686/build=/usr/src/debug/python3-3.4.3-1 
-fdebug-prefix-map=/usr/src/ports/python3/python3-3.4.3-1.i686/src/Python-3.4.3=/usr/src/debug/python3-3.4.3-1 
-DZMQ_USE_POLL=1 -DHAVE_LIBSODIUM=1 -Ibundled/zeromq/include -Ibundled -Ibundled/libsodium/src/libsodium/include 
-Ibundled/libsodium/src/libsodium/include/sodium -I/usr/include/python3.4m -c bundled/zeromq/src/address.cpp 
-o build/temp.cygwin-2.0.4-i686-3.4/bundled/zeromq/src/address.o

gcc: error: spawn: No such file or directory
error: command 'gcc' failed with exit status 1

----------------------------------------

Command "/usr/bin/python3 -c "import setuptools, tokenize;__file__='/tmp/pip-build-l4nkh9zq/pyzmq/setup.py';
exec(compile(getattr(tokenize,'open', open)(__file__).read().replace('\r\n', '\n'), __file__, 'exec'))" 
install --record /tmp/pip-jicpjqak-record/install-record.txt --single-version-externally-managed --compile" 
failed with error code 1 in /tmp/pip-build-l4nkh9zq/pyzmq
```

It took me a while to figure it out... In case you were in a similar situation, this is what I did:

1) Install some tools (via [apt-cyg](https://github.com/transcode-open/apt-cyg) or setup.exe)
```
apt-cyg install libtool automake autoconf   
```
(you will need the usual tools for compiling C/C++ code, such as gcc,g++,make, etc,etc)

2) Export `PKG_CONFIG_PATH`
```
export PKG_CONFIG_PATH=/usr/local/lib/pkgconfig
```

3) Download and compile [libsodium](https://github.com/jedisct1/libsodium/releases)
```
./configure
make && make install
```

4) Download and compile [libzmq](https://github.com/zeromq/libzmq)
```
./autogen.sh
./configure
make && make install
```

5) Finally, try to install ipython[notebook] again:
```
pip install ipython[notebook]
```

It worked for me... Hope it helps you.

Later
