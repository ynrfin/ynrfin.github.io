---
title: uRxvt's .Xresources Not Loaded Properly on Manjaro
date: 2019-06-16 
---

## The Problem

When I open `urxvt` after booting up the linux, it is terrible. The hide scrollbar part is working, but the color is off, way off. I wanted to use solarized-dark color theme, but the display it shows is pink text color on pink backgroud. I couldn't see the text!! The config is working properly only after loading the resource file with `xrdb ~/.Xresources`.

## What I Wanted To Do

I wanted to use `urxvt` to replace `gnome-terminal` as my terminal because it has more feature(I think) than the latter. I have a minimal setting on `~/.Xresources` which I have used on my previous Ubuntu, and it worked. But now it does not worked like how it was.

This is partial view of my config:
```bash
#define S_base03        #002b36

... other #define blocks

*background:             S_base03
!Urxvt.background:            S_base03
*foreground:            S_base0
*fadeColor:             S_base03
*cursorColor:           S_base1
*pointerColorBackground:S_base01
*pointerColorForeground:S_base1

#define S_yellow        #b58900

... other #define blocks

!! black dark/light
*color0:                S_base02
*color8:                S_base03

... other color definition block

URxvt.scrollBar: False

URxvt.depth: 32

Urxvt.letterSpace: 1
```

## Solution

I googled this problem. There are many post on forums that suggest I load it in `.bashrc` file, so it will be run in start up. I found it to be too much to do it that way. Other thing that I found is that previously there are `.Xdefaults` before `.Xresources` to save configuration. Does it use `.Xdefaults` then? but some settings are working, its just not reading properly(I guess). Next thing I search is "where gdm/gnome loaded this file" and I found it to be at `/etc/gdm/Xsession`. This is the part of `Xsession` that load the config:

```bash
# ... other settings

userresources="$HOME/.Xresources"
usermodmap="$HOME/.Xmodmap"
userxkbmap="$HOME/.Xkbmap"

sysresources=/etc/X11/Xresources 
sysmodmap=/etc/X11/Xmodmap 
sysxkbmap=/etc/X11/Xkbmap

rh6sysresources=/etc/X11/xinit/Xresources 
rh6sysmodmap=/etc/X11/xinit/Xmodmap 

# merge in defaults
if [ -f "$rh6sysresources" ]; then
    xrdb -nocpp -merge "$rh6sysresources"
fi

if [ -f "$sysresources" ]; then
    xrdb -nocpp -merge "$sysresources"
fi

if [ -f "$userresources" ]; then
    xrdb -nocpp -merge "$userresources"
fi

# ... other settings

```

Notice in the setting, the value of `userresources` is `.Xresources`. So my file name is correct as there are settings that is applied to uRxvt. I then googled the command `xrdb -nocpp -merge "$userresources"`. I read a few forum post(I link it at the bottom), turns out its the `-nocpp` options that make `#define` block read properly, this options tells `xrdb` not to run the config file through preprocessor. 

## The Fix

To fix that, delete `-nocpp` flag from the config.

```bash
    # From this
    xrdb -nocpp -merge "$userresources"
    # became
    xrdb -merge "$userresources"

```
## Conclusion
I did not found the preprocessor part when I search before. I found it only after I googled the `-nocpp` flag. So, search keyword matter.

---
## References
- [Stack Exchange superuser question](https://superuser.com/questions/1307768/preprocessor-directive-define-doesnt-work-in-xresources)
- [Oleksandr Manenko's Blog Post; GDM doesn't load included files from .Xresources in Arch Linux](https://manenko.com/2015/05/15/gdm-doesnt-load-included-files-from-xresources-in-arch-linux.html)
