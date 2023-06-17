---
title: ImportError libffi.so.7 cannot open shared object file No such file or directory on Manjaro
date: 2021-12-13
tags: [ linux, python, virtualenv, venv, manjaro ]
---

This error happened after I try to run a Django project on Linux(Manjaro) after an sistem update.

```
ImportError: libffi.so.7: cannot open shared object file: No such file or directory
```

# What Have I try:

---

## Install libffi using pacman

```
pacman -Sy libffi
```

The installation is successful but the error persists.

## Delete and recreate virtualenv

it did not help neither.

# Solution

---

Install That specific `libffi.so.7`

I use `yay` to search for suitable package to install

```sh
$ yay libffi
7 aur/cffi-lua 0.2.1-1 (+0 0.00)
    A portable C FFI for Lua, based on libffi
16 aur/libx32-libffi 3.2.1-1.1 (+0 0.00)
    A portable, high level programming interface to various calling conventions (x32 ABI)
15 aur/libffi-git 3.4.2.r20.g0f2dd36-1 (+0 0.00)
    Portable foreign function interface library
14 aur/libffi-static 3.4.2-4 (+0 0.00)
    Portable foreign function interface library
13 aur/lib32-libffi-minimal-git 3.3-1 (+1 0.00)
    A portable, high level programming interface to various calling conventions (32-bit)
12 aur/lib32-libffi6 3.2.1-1 (+1 0.01)
    A portable, high level programming interface to various calling conventions (ABI version 6)
11 aur/libffi-minimal-git 3.2.1.r334.g4fdbb05-1 (+1 0.00)
    Portable foreign function interface library
10 aur/libjffi 1.3.6-2 (+1 0.40)
    Java bindings for libffi
9 aur/libffi6 3.2.1-1 (+6 0.02)
    A portable, high level programming interface to various calling conventions (ABI version 6)
# THIS ONE
8 aur/libffi7 3.3-2 (+7 3.72)
    Portable foreign function interface library (ABI version 7)
7 aur/lib32-libffi5 3.0.10-2 (+8 0.00)
    A portable, high level programming interface to various calling conventions (ABI version 5)
6 aur/pure-ffi 0.16-1 (+8 0.00)
    An interface to libffi which enables you to call C functions from Pure and vice versa.
5 aur/mingw-w64-libffi 3.4.2-1 (+19 0.01)
    Portable foreign function interface library (mingw-w64)
4 aur/libffi5 3.0.10-1 (+32 0.00)
    A portable, high level programming interface to various calling conventions (ABI version 5)
3 multilib/lib32-libffi 3.4.2-3 (18.5 KiB 38.7 KiB) (Installed)
    Portable foreign function interface library (32-bit)
2 community/haskell-libffi 0.1-26 (39.6 KiB 212.3 KiB)
    A binding to libffi
# NOT this one
1 core/libffi 3.4.2-4 (44.9 KiB 94.4 KiB) (Installed)
    Portable foreign function interface library
```

Notice number 8, that is the package that you need to install because the latest libffi now is on version 8, and your python needs the 7 version.

Install that and you are set.


# A Helpful Note For Debugging

---

use `locate`  to locate if your system already has `libffi.so.7`

```
$ locate libffi.so.7
/home/ynrfin/.dropbox-dist/dropbox-lnx.x86_64-136.4.4345/libffi.so.7
/home/ynrfin/.dropbox-dist/dropbox-lnx.x86_64-136.4.4345/libffi.so.7.1.0
/opt/dropbox/libffi.so.7
/opt/dropbox/libffi.so.7.1.0

```

using `locate libffi` returns more result on my computer. So you should be specific about what you want to locate

Hope this helps.
