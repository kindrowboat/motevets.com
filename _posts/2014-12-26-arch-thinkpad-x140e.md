---

layout:     post
title:      "Arch on a Lenovo ThinkPad x140e"
date:       2014-12-26 22:37:00
categories: linux

---

Recently at work I had been virtualizing Arch Linux on a MacBook Pro using
VirtualBox.  While there were many perks (easy backups, no hardware issues),
when I left that assignment (and had to give back the Mac), I couldn't justify
spending a grand on a Mac which would spend most of its time running Linux in a
virtual machine, so I set out to find a cheap, modern, Linux compatible laptop.
Ultimately, I landed on the Lenovo Thinkpad x140e which was [certified by
Canonical to run Ubuntu][canonical certification].  Unfortunately, for Arch
Linux, much of the hardware did not work out of the box.  This post covers the
modifications I performed to get all of the hardware working including:

- graphics
- network adapters (wired and wireless)
- input devices (trackpad and trackpoint)
- sound
- Fn keys for volume and brightness

<!--more-->

This guide assumes that you've gone through all of the steps in the [Arch
installation guide][arch install].  For what it's worth, this guide is window
manager agnostic (or even atheist if you're just in terminal), though I'm using
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
install `xf86-input-synaptics` and
[configure it to your liking][configure synaptics].

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
volume and mute buttons, and is not really needed — see below) by adding a file
`/etc/modprobe.d/alsa-base.conf` with

```
options snd_hda_intel 1 enable=1 index=0
options snd_hda_intel 2 enable=0 index=1
```

and restarting.  You'll still need to unmute all the channels using something
like `alsamixer`. Install `alsa-utils` through pacman and use `alsamixer` to
unmute the channels and adjust the volume.

**ACPI Events (volume and brightness keys)**

In the past I've configured the window manager to handle volume and brightness
function keys.  However, using the Linux [ACPI daemond][acpid] allows your
machine to handle these keys strokes before you even start X!  First install
`acpid` through pacman, and `start` and `enable` `acpid` using `systemctl`.
Next, you'll be making the events for the brightness and volume key presses.

`/etc/acpi/events/bl_d`

```
event=video/brightnessdown
action=/etc/acpi/handlers/bl -
```

`/etc/acpi/events/bl_u`

```
event=video/brightnessup
action=/etc/acpi/handlers/bl +
```

`/etc/acpi/events/vol_u`

```
event=button/volumedown
action=/etc/acpi/handlers/vol -
```

`/etc/acpi/events/vol_d`

```
event=button/volumeup
action=/etc/acpi/handlers/vol +
```

`/etc/acpi/events/mute`

```
event=button/mute
action=/usr/bin/amixer set Master toggle
```

Next make the directory `/etc/acpi/handlers`, in which create the following
files:

`/etc/acpi/events/bl`

```
#!/bin/sh
bl_dev=/sys/class/backlight/radeon_bl0
step=5
case $1 in
  -) echo $(($(&lt; $bl_dev/brightness) - $step)) &gt;$bl_dev/brightness;;
  +) echo $(($(&lt; $bl_dev/brightness) + $step)) &gt;$bl_dev/brightness;;
esac
```

`/etc/acpi/events/vol`

```
#!/bin/sh
step=5
case $1 in
  -) amixer set Master $step-;;
  +) amixer set Master $step+;;
esac
```

... and make these files executable with `sudo chmod 755 /etc/acpi/handlers/*`.

_Note that the `vol` handler require `amixer` which is part of the `alsa-utils`
package._

**Conclusion**

That's about it.  There's always more to be done, from customizing your wifi to
auto-connect, to setting up external displays, to installing an configuring a
window manager, but after following these steps, all of your hardware should be
up and running properly.


[canonical certification]: http://www.ubuntu.com/certification/hardware/201309-14195/
[arch install]: https://wiki.archlinux.org/index.php/Installation_guide
[Awesome]: https://wiki.archlinux.org/index.php/Awesome
[dhcpcd]: https://wiki.archlinux.org/index.php/dhcpcd
[wl]: https://wiki.archlinux.org/index.php/Broadcom_wireless#broadcom-wl
[configure synaptics]: https://wiki.archlinux.org/index.php/Synaptics
[acpid]: https://wiki.archlinux.org/index.php/acpid
