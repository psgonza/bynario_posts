title: Running telegram-cli in cygwin
date: 2016-08-05
slug: 2016-08-05-running-telegram-cli-in-cygwin
tags: cygwin, windows, telegram
category: posts

[Telegram](https://telegram.org/) is a really interesting messaging application... It is secure (well, let's say [secure enough](http://security.stackexchange.com/questions/49782/is-telegram-secure#49802)), fast and multiplatform... Even there is a CLI client!

I tried to compile the [telegram-cli](https://github.com/vysheng/tg) in my cygwin64 earlier today following their [documentation](https://github.com/vysheng/tg/blob/master/README-Cygwin.md), and I came across a couple of issues.

** 1) Patching failed: **

I couldn't patch `Makefile` and Hunk #1 failed in `loop.c` as well:

```
20:34:26@tg$ patch -p1 < telegram-cli-cygwin.patch
patching file Makefile
Hunk #1 FAILED at 4.
1 out of 1 hunk FAILED -- saving rejects to file Makefile.rej
patching file loop.c
Hunk #1 FAILED at 383.
Hunk #2 succeeded at 737 with fuzz 2 (offset 103 lines).
1 out of 2 hunks FAILED -- saving rejects to file loop.c.rej
```

Probably the patch file is not updated with the latest telegram-cli changes, so I dediced to patch it manually...

I updated the first lines of `Makefile` with the changes stated in the [patch file](https://gist.github.com/ied206/d774a445f36004d263ab):

```
1 srcdir=.
2
3 CFLAGS=-g -O2
4 LDFLAGS= -L/usr/local/lib -L/usr/lib -L/usr/lib -L/usr/lib
5 CPPFLAGS= -I/usr/local/include -I/usr/include -I/usr/include  -I/usr/include/python3.4m -I/usr/include
6 DEFS=-DHAVE_CONFIG_H
7
8 COMPILE_FLAGS=${CFLAGS} ${CPFLAGS} ${CPPFLAGS} ${DEFS} -Wall -Werror -Wextra -Wno-missing-field-initializers 
   -Wno-deprecated-declarations -fno-strict-aliasing -fn      o-omit-frame-pointer -ggdb -Wno-unused-parameter
9 EXTRA_LIBS=-ljansson -lconfig -lz -levent -lm   -lreadline -llua-5.2  -lpython3.4m -lssl -lcrypto
10 LOCAL_LDFLAGS=-ggdb -levent ${EXTRA_LIBS} -ldl -lpthread -lutil
11 LINK_FLAGS=${LDFLAGS} ${LOCAL_LDFLAGS}
```

Since `loop.c` was partially patched already, I tried to manually fix `Hunk #1`, but nothing worked, so I guess it is no longer neccessary... 

I managed to generate the binary file after patching `Makefile`... But it still refused to work.

** 2) Error assertion "0" failed: file "tgl/mtproto-utils.c", line 101, function: BN2ull **

There were no compilation errors, but there was something wrong in `tgl/mtproto-utils.c`:

```
21:00:14@tg$ bin/telegram-cli -k tg-server.pub
Telegram-cli version 1.4.1, Copyright (C) 2013-2015 Vitaly Valtman
Telegram-cli comes with ABSOLUTELY NO WARRANTY; for details type `show_license'.
This is free software, and you are welcome to redistribute it
under certain conditions; type `show_license' for details.
Telegram-cli uses libtgl version 2.1.0
Telegram-cli includes software developed by the OpenSSL Project
for use in the OpenSSL Toolkit. (http://www.openssl.org/)
I: config dir=[/home/user/.telegram-cli]
[/home/user/.telegram-cli] created
[/home/user/.telegram-cli/downloads] created
> SIGNAL received
No libexec. Backtrace disabled
assertion "0" failed: file "tgl/mtproto-utils.c", line 101, function: BN2ull
```

The solution seems to be remove (or comment out) lines 101 and 115 in `tgl/mtproto-utils.c`:

```
[...]
101     //assert (0); // As long as nobody ever uses this code, assume it is broken.
[...]
115     //assert (0); // As long as nobody ever uses this code, assume it is broken.
[...]
```

After that, run `make` again, and execute it:

```
21:13:09@tg$ bin/telegram-cli -k tg-server.pub
Telegram-cli version 1.4.1, Copyright (C) 2013-2015 Vitaly Valtman
Telegram-cli comes with ABSOLUTELY NO WARRANTY; for details type `show_license'.
This is free software, and you are welcome to redistribute it
under certain conditions; type `show_license' for details.
Telegram-cli uses libtgl version 2.1.0
Telegram-cli includes software developed by the OpenSSL Project
for use in the OpenSSL Toolkit. (http://www.openssl.org/)
I: config dir=[/home/user/.telegram-cli]
>
```

I hope it helps!

\\\psgonza
