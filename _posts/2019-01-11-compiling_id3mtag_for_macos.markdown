---
layout: post
title:  "Compiling id3mtag for macOS"
date:   2019-01-11 11:22:01 +0000
#categories: audio mp3 id3 id3v2 macOS c++ make
---

I'm working on a project which involves tagging some mp3 files using the id3v2 spec. Rather than write one myself, I thought (errantly) that it would be easy to find a command line tool for macOS that I could use in a shell script. It's true that there are several out there, but they all had limitations. These range from randomly limiting text fields to 254 characters (a limitation not found in the id3v2 spec), not being able to write comment tags (which is necessary for my project) and not being able to write id3v2 at all. I also came across a promising looking open source tool, written primarily in c++: [`id3mtag`](https://squell.github.io/id3/). 

The tool isn't available in Homebrew and the author does not make a pre-built binary for macOS available. So I thought I'd try to compile it myself. Should be no big deal, I thought. I'm a `make` noob, and I've never written a line of c++ in life. My confidence was not well founded. Here's what happened.

```
$ make -f makefile
c++ -g -Os -pedantic  -fno-rtti -Wformat -Wno-parentheses -c main.cpp
c++ -g -Os -pedantic  -fno-rtti -Wformat -Wno-parentheses -c sedit.cpp
c++ -g -Os -pedantic  -fno-rtti -Wformat -Wno-parentheses -c varexp.cpp
c++ -g -Os -pedantic  -fno-rtti -Wformat -Wno-parentheses -c fileexp.cpp
c++ -g -Os -pedantic  -fno-rtti -Wformat -Wno-parentheses -c charconv.cpp
c++ -g -Os -pedantic  -fno-rtti -Wformat -Wno-parentheses -c mass_tag.cpp
c++ -g -Os -pedantic  -fno-rtti -Wformat -Wno-parentheses -c pattern.cpp
c++ -g -Os -pedantic  -fno-rtti -Wformat -Wno-parentheses -c setid3.cpp
c++ -g -Os -pedantic  -fno-rtti -Wformat -Wno-parentheses -c getid3.cpp
cc -g -Os -pedantic -Wformat -Wno-parentheses -c id3v1.c
c++ -g -Os -pedantic  -fno-rtti -Wformat -Wno-parentheses -c setid3v2.cpp
c++ -g -Os -pedantic  -fno-rtti -Wformat -Wno-parentheses -c getid3v2.cpp
cc -g -Os -pedantic -Wformat -Wno-parentheses -c id3v2.c
cc -g -Os -pedantic -Wformat -Wno-parentheses -c fileops.c
c++ -g -Os -pedantic  -fno-rtti -Wformat -Wno-parentheses -c char_ucs.cpp
c++ -g -Os -pedantic  -fno-rtti -Wformat -Wno-parentheses -c char_utf8.cpp
c++ -g -Os -pedantic  -fno-rtti -Wformat -Wno-parentheses -c setlyr3.cpp
c++ -g -Os -pedantic  -fno-rtti -Wformat -Wno-parentheses -c getlyr3.cpp
c++ -g -Os -pedantic  -fno-rtti -Wformat -Wno-parentheses -c lyrics3.cpp
c++ -g -Os -pedantic  -fno-rtti -Wformat -Wno-parentheses -c setfname.cpp
c++ -g -Os -pedantic  -fno-rtti -Wformat -Wno-parentheses -c setquery.cpp
c++ -g -Os -pedantic  -fno-rtti -Wformat -Wno-parentheses -c dumptag.cpp
c++ -g -Os -pedantic  -fno-rtti -Wformat -Wno-parentheses main.o sedit.o varexp.o fileexp.o charconv.o mass_tag.o pattern.o setid3.o getid3.o id3v1.o setid3v2.o getid3v2.o id3v2.o fileops.o char_ucs.o char_utf8.o setlyr3.o getlyr3.o lyrics3.o setfname.o setquery.o dumptag.o  -o id3
Undefined symbols for architecture x86_64:
  "_iconv", referenced from:
      charset::recode(char*, unsigned long, void const*, unsigned long, char const*, char const*, bool, unsigned long) in charconv.o
  "_iconv_close", referenced from:
      charset::recode(char*, unsigned long, void const*, unsigned long, char const*, char const*, bool, unsigned long) in charconv.o
  "_iconv_open", referenced from:
      charset::recode(char*, unsigned long, void const*, unsigned long, char const*, char const*, bool, unsigned long) in charconv.o
ld: symbol(s) not found for architecture x86_64
clang: error: linker command failed with exit code 1 (use -v to see invocation)
make: *** [id3] Error 1
```

Argh!

Knowing that macOS is based on FreeBSD I attempted to use the [FreeBSD Makefile](https://github.com/squell/id3/blob/master/FreeBSD/Makefile) included with the source code. It failed in a completely different way. However, studying the file I noticed these lines:

```
USES=   iconv

CFLAGS+=  -I${ICONV_INCLUDE_PATH}
LDFLAGS+= -L${ICONV_PREFIX}/lib ${ICONV_LIB}
```

I'm not going to lie and pretend I understand the problem. It seems to be based around Apple providing their own `iconv` in `/usr/lib` (there are three possible culprits on my system, `libiconv.2.4.0.dylib`, `libiconv.2.dylib` and `libiconv.dylib`).

This document was also useful in helping me figure out what the values for the `ICONV_*` variables should be: [https://www.freebsd.org/doc/en/books/porters-handbook/using-iconv.html](https://www.freebsd.org/doc/en/books/porters-handbook/using-iconv.html)

The solution:
-------------

Install iconv (I used brew):

```
$ brew install iconv
```

```
$ iconv --version
iconv (GNU libiconv 1.11)
Copyright (C) 2000-2006 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
Written by Bruno Haible.
```

```
$ which iconv
/usr/bin/iconv
```

Modify the generic [`makefile`](https://github.com/squell/id3/blob/master/makefile) provided by the author. (I added a single line below each `# added` comment)

```
# added
USES= iconv

CFLAGS   ?= -g -Os -pedantic
CFLAGS   += $(WARNINGS:%=-W%)

# added
CFLAGS+=        -I/usr/bin

CXXFLAGS ?= $(DEFFLAGS)
CXXFLAGS += -fno-rtti $(WARNINGS:%=-W%)
LDFLAGS  ?=

# added
LDFLAGS  += -L/usr/bin -liconv
```

Now try to compile:

```
$ make -f makefile
c++ -g -Os -pedantic  -I/usr/bin -fno-rtti -Wformat -Wno-parentheses -c main.cpp
c++ -g -Os -pedantic  -I/usr/bin -fno-rtti -Wformat -Wno-parentheses -c sedit.cpp
c++ -g -Os -pedantic  -I/usr/bin -fno-rtti -Wformat -Wno-parentheses -c varexp.cpp
c++ -g -Os -pedantic  -I/usr/bin -fno-rtti -Wformat -Wno-parentheses -c fileexp.cpp
c++ -g -Os -pedantic  -I/usr/bin -fno-rtti -Wformat -Wno-parentheses -c charconv.cpp
c++ -g -Os -pedantic  -I/usr/bin -fno-rtti -Wformat -Wno-parentheses -c mass_tag.cpp
c++ -g -Os -pedantic  -I/usr/bin -fno-rtti -Wformat -Wno-parentheses -c pattern.cpp
c++ -g -Os -pedantic  -I/usr/bin -fno-rtti -Wformat -Wno-parentheses -c setid3.cpp
c++ -g -Os -pedantic  -I/usr/bin -fno-rtti -Wformat -Wno-parentheses -c getid3.cpp
cc -g -Os -pedantic -Wformat -Wno-parentheses -I/usr/bin -c id3v1.c
c++ -g -Os -pedantic  -I/usr/bin -fno-rtti -Wformat -Wno-parentheses -c setid3v2.cpp
c++ -g -Os -pedantic  -I/usr/bin -fno-rtti -Wformat -Wno-parentheses -c getid3v2.cpp
cc -g -Os -pedantic -Wformat -Wno-parentheses -I/usr/bin -c id3v2.c
cc -g -Os -pedantic -Wformat -Wno-parentheses -I/usr/bin -c fileops.c
c++ -g -Os -pedantic  -I/usr/bin -fno-rtti -Wformat -Wno-parentheses -c char_ucs.cpp
c++ -g -Os -pedantic  -I/usr/bin -fno-rtti -Wformat -Wno-parentheses -c char_utf8.cpp
c++ -g -Os -pedantic  -I/usr/bin -fno-rtti -Wformat -Wno-parentheses -c setlyr3.cpp
c++ -g -Os -pedantic  -I/usr/bin -fno-rtti -Wformat -Wno-parentheses -c getlyr3.cpp
c++ -g -Os -pedantic  -I/usr/bin -fno-rtti -Wformat -Wno-parentheses -c lyrics3.cpp
c++ -g -Os -pedantic  -I/usr/bin -fno-rtti -Wformat -Wno-parentheses -c setfname.cpp
c++ -g -Os -pedantic  -I/usr/bin -fno-rtti -Wformat -Wno-parentheses -c setquery.cpp
c++ -g -Os -pedantic  -I/usr/bin -fno-rtti -Wformat -Wno-parentheses -c dumptag.cpp
c++ -g -Os -pedantic  -I/usr/bin -fno-rtti -Wformat -Wno-parentheses main.o sedit.o varexp.o fileexp.o charconv.o mass_tag.o pattern.o setid3.o getid3.o id3v1.o setid3v2.o getid3v2.o id3v2.o fileops.o char_ucs.o char_utf8.o setlyr3.o getlyr3.o lyrics3.o setfname.o setquery.o dumptag.o  -L/usr/bin -liconv  -o id3
```

Looks good! Now to install:

```
$ sudo make install
install -d /usr/local/bin /usr/local/man/man1
install -m 644 id3.man /usr/local/man/man1/id3.1
install id3 /usr/local/bin/id3
```

Success!

```
$ id3 --version
id3 0.80 (2016005), Copyright (C) 2003, 04, 05, 06, 15 Marc R. Schoolderman
This program comes with ABSOLUTELY NO WARRANTY.

This is free software, and you are welcome to redistribute it
under certain conditions; see the file named COPYING in the
source distribution for details.
```

After trying multiple different tagging tools (I'm particularly looking at you, `ffmpeg`), `id3mtag` works wonderfully. I suspect there's a better solution to this problem, and perhaps one day I will find out what it is - but for now, I'm satisfied.

Links:
------
id3mtag: [https://squell.github.io/id3/](https://squell.github.io/id3/)
