---

title:  "Arch on a Lenovo ThinkPad x140e"
date:   2014-12-26 22:37:00
categories: Linux

---

Recently at work I had been virtualizing Arch Linux on a MacBook Pro using Virtual Box.
While there were many perks (easy backups, no hardware issues), when I left that
assignment (and had to give back the Mac), I couldn't justify spending a grand
on a Mac which would spend most of its time running Linux in a virtual machine,
so I set out to find a cheap, modern, Linux compatible laptop.  Ultimately, I
landed on the Lenovo Thinkpad x140e which was certified by Canonical to run
Ubuntu.  Unfortunately (or fortunately if you're me and like to tinker) a lot
didn't work right out of the box.  These are the changes I added to get a fully
functioning Arch Linux development environment.  For an Arch installation, this
assumes that you've gone through all of the steps in the [Arch installation
guide][arch install].  This guide with window manager agnostic (or even
atheist if you're just in terminal), though for what it's worth, I'm using
[Awesome].


**Graphics**

The integrated Radeon HD 8330 card is supported by the proprietary driver,
`catalyst` (not recommended) or the open source `xf86-video-ati` driver.  If
using the latter, also install the `mesa-libgq` for 3D acceleration and
`mesa-vdpau` for accelerated video decoding.

**Internet**

The ethernet card is supported by a driver in the kernel, you'll just need to
install a dhcp client (e.g. [dhcpcd]) to handle getting your computer to talk to
the router.

Wireless is a little tougher, the Broadcom chipset used is not yet supported by
any of the open source drivers, and so you'll need to follow the instructions
for [installing broadcom-wl drivers][wl].  Namely:

1. install `broadcom-wl` from the AUR
2. create a blacklist file named something like `/etc/modprobe.d/wireless.conf`
   with the following:

   ```
   blacklist brcm80211
   blacklist b43
   blacklist ssb
   ```

3. create a `modules-load.d` file named something like
   `/etc/modules-load.d/wireless.conf` with:

   ```
   wl
   ```

4. restart the machine.

You should then be able to connect to wireless through `wifi-menu` and `netctl`.

**Trackpad and Trackpoint**

I personally am not a fan of the track pad, but if you'd like yours to work
install `xf86-input-synaptics` and [configure it to your liking][configure
synaptics].

Trackpoint should work out of the box, though if you'd like middle button
scrolling, add the file `/etc/X11/xorg.conf.d/20-thinkpad.conf` with:

```
Section "InputClass"
    Identifier  "Trackpoint Wheel Emulation"
    MatchProduct        "TPPS/2 IBM TrackPoint|DualPoint Stick|Synaptics Inc. Composite TouchPad / TrackPoint|ThinkPad USB Keyboard with TrackPoint|USB Trackpoint pointing device"
    MatchDevicePath     "/dev/input/event*"
    Option              "EmulateWheel"          "true"
    Option              "EmulateWheelButton"    "2"
    Option              "Emulate3Buttons"       "false"
    Option              "XAxisMapping"          "6 7"
    Option              "YAxisMapping"          "4 5"
EndSection
```

and restart.

**Sound**

While the integrated sound card is supported out of the box, the system will
most likely set the wrong default card and so you won't be able to hear
anything, you can correct this by disabling the other sound device (which is
actually just a virtual device exposed by `thinkpad_acpi` to represent the
volume and mute buttons, and is not really needed (see below) by adding a file
`/etc/modprobe.d/alsa-base.conf` with

```
options snd_hda_intel 1 enable=1 index=0
options snd_hda_intel 2 enable=0 index=1
```

and restarting.  You'll still need to unmute all the channels using something
like `alsamixer`.

**ACPI Events (volumene and brightness keys**

In the past I've configured the window manager to handle volume keys and 


[arch install]: https://wiki.archlinux.org/index.php/Installation_guide
[Awesome]: https://wiki.archlinux.org/index.php/Awesome
[dhcpcd]: https://wiki.archlinux.org/index.php/dhcpcd
[wl]: https://wiki.archlinux.org/index.php/Broadcom_wireless#broadcom-wl
[configure synaptics]: https://wiki.archlinux.org/index.php/Synaptics
