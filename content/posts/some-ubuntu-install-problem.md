---
title: some ubuntu installing problems
date: 2018-04-28 21:11:36
categories:
- 杂项
tags:
- Ubuntu
- GRUB 
---

One day ago, I started upgrading my Ubuntu 16.04 to 18.04, after I see the update from website. The most diffrience 
betwen 16.04 and 18.04 is that the 18.04 use GNOME desktop rather than Unity. I had heard that the GNOME desktop is 
better than Unity, so I tried 18.04 for the new desktop. Unfortunately, after I upgraded my OS, I found that the GNOME 
desktop wasjust so uncomfortable. And after one hour I gone back to 16.04:(, and content of this article is about the 
problems I met while reinstalling the OS.
<!-- more -->

I installed the 16.04 OS just the same as what I done before: burned the IOS to USB, started the computer, and do what the OS suggested to do.

## soft lockup ##
The first thing I met was what called soft lockup, like below
```
 watchdog: BUG: soft lockup cpu#0 stuck for 23s.
```
emmm, it was quite depressed, but fortunately I found the solutions on _ubuntu forums_. The
problem can be solved add grub boot option as below.

When you stay at the GRUB menu(you maybe need to press and hold the left `shift` key right
after starting the system if you only has one OS.). Then Hightlight the kernel you want to 
use and press the `e` key, thus you should able to see and edit the commands associated 
with the hightlighted kernel you do before. Go down to the line starting with `linux` and 
add the parameter `nomodeset` to its end. Now press `Ctrl` + `x` to boot in the specific 
config. And remember it just work temporarily, you should configure the `/etc/default/grub`
file as same as what you do before, finally run `sudo update-grub` to permanently update
GRUB's configuration. 

So in conclusion, what you need to do is add `nomodeset` to kernel boot configure.

## PCIe Bus error ##
After I solved the problem above, finally I was led to a __NEW PROBLEM__:( The PCIe Bus 
error actually appeared before the soft lockup, it was solved by _askubuntu_, just add
`pci=noaer` to the kernel boot config, which I explained at soft lockup.

Finally I could use my Ubuntu again. But what the parameter, like `pci=nomsi` really mean?,
or "just work" is fine? Thanks the internet a lot, I found an post about this, here are
some summary.

1. _nomodeset_, may be the most common one, which is needed for some graphics cards that 
otherwise boot into a black screen or corrupted splash. The newest kernels have moved the
video mode setting into the kernel. So all the programming of hardware specific clock rates
and registers on the video card happen in the kernel rather than in the X driver when the
X server starts. This make it possible to have high resolution nice looking splash(boot) 
screens and flicker free transitions from boot splash to login screen. Unfortunately, on 
some cards this doesn't work properly and you end up with a black screen(or soft lockup
above). Adding the parameter instructs the kernel to not load video drivers and use BIOS
modes instead until X is loaded. Note that this option is sometions needed for nVidia cards
when using the default "nouveau" drivers. Installing proprietary nvidia drivers ususlly
makes this option no longer necessary, so it may not be needed to make this option 
permanet, just for one boot until you installed the nvidia drivers.

2. _pci=_, the argument can be used to change the behaviour of PCI bus device problem and
device behaviour.


### Ubuntu and Windows time difference ###
Operating systems store and retrieve the time in the hardware clock located on your motherland so
that it can keep track of the time even when the system does not have power. Most operating system(Linux,
Unix, MacOS) store the time on the hardware clock as UTC by default, though some system(notably Microsoft 
Windows) store the time on the hardware clock as the "local" time. This causes problems in a dual boot system
if both system view the hardware clock differently.

The advantage of having the hardware clock as UTC is that you don't need to change the hardware clock when moving
bewteen timezones or when Daylight Savings Time(DST) begins or ends as UTC does not have DST or timezone offsets.
And changing Linux to use local time is easier and more reliable than changing Windows to use UTC.

1. _Make Linux use 'local' time_,
    pre-Ubuntu 15.04 systems, you should edit `/etc/default/rcS` and add or change the following section
    `UTC=no`, while Ubuntu 15.04 and above you should run `timedatectl set-local-rtc 1 --adjust-system-clock` in 
    terminal since the Ubuntu use timedatectl to manage time after 15.04.
    
2. _Make Windows use UTC_,
    for windows 10 you can create a *.reg(on the desktop) with the following content and double-click it to import
    it to registry:
    ```
    Windows Registry Editor Version 5.00
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\TimeZoneInformation]
     "RealTimeIsUniversal"=dword:00000001
    ```
