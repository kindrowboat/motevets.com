---

title:  "Connecting to University of Cincinnati's Securewireless with netctl"
date:   2015-01-13 17:14:00
categories: linux

---

If you've set-up an Arch Linux installation, and you've used `wifi-menu` then
you've used [netctl][netctl] before and didn't even know it.  `netctl` is

> ... a CLI-based tool used to configure and manage network connections via
> profiles. It is a native Arch Linux project for network configuration.

I used to think that the `wifi-menu` dialouge was a cute little installer helper
program, but I learned later, that it can be used to automatically generate
profiles in `/etc/netctl` that you can subsequently use to reconnect to network
later with

```bash
# netctl start INTERFACE-SSID
```

... where *INTERFACE* is the name of your wireless interface device (see below)
and *SSID* is the "name" of the network.

I was a little disappointed (but not too shocked) when `wifi-menu` failed to
connect to the University of Cincinnati's `Securewireless` network.  This short
guide will discuss the steps needed to connect to `Securewireless` using
`netctl`, and discuss why these extra steps are needed.

<!--more-->

**Connect to Securewireless — _tl;dr_**

Create and edit the file `/etc/netctl/INTERFACE-Securewireless` as root (using
`sudo`).  Note that _INTERFACE_ should be the name of your wireless interface.
Use `ip link` to find out what it is.  While the interface prefix is not
mandatory, it does help you stay organized, `wifi-menu` adds it by default, and
you'll need it below.

```
Connection='wireless'
Interface=INTERFACE
Security='wpa-configsection'
Description="UC eduroam-like network"
IP='dhcp'
TimeoutWPA=30
WPAConfigSection=(
    'ssid="Securewireless"'
    'key_mgmt=WPA-EAP'
    'identity="UC_USER_NAME"'
    'password="UC_CENTRAL_LOGIN_PASSWORD"'
)
```

Where *`INTERFACE`* is your wireless interface as described above,
*`UC_USER_NAME`* is your 6+2 user name without the domain suffix (e.g. smithbb1
**not** smithbb1@mail.uc.edu), and *`UC_CENTRAL_LOGIN_PASSWORD`* is the central
login password that you use for all of your UC services. (Leave in the quotes
around the actual username and password.)

**Details**

The magic is in the `wpa-configsection`/`WPAConfigSection`.  This allows you to
step outside of simple WEP/WPA/WPA2 shared passphrase paradigm and set the
security stack exactly how you need as if you were setting up `wpa_supplicant`
by hand.  There's a lot you can do here, like connect to an [eduroam][eduroam]
network or use another pre-agreed upon security certificate, but UC's setup is
pretty simple. If you need to see all of the settings you can put in the
WPAConfigSection, see the manual page for `wpa_supplicant` or look at a [sample
wpa_supplicant.conf][wpa_supplicant example].

University of Cincinnati uses WPA Enterprise much like other universities.
According to [UC's IT Handbook (last page)][uc_it_handbook](pdf), `Securewireless`
uses:

* WPA2 Enterprise Security
* Protected Extensible Authentication Protocol (PEAP)
* No enterprise security certificate

Through trial and error, I found the simplest `WPAConfigSection` needed to
successfully connect.  `ssid` is set to `Securewireless`, the name of UC's
network. `key_mgmt=WPA-EAP` tells the WPA supplicant to use and `identity` and
`password` through (Protected) Extensible Authentication Protocol to connect to
the network.

I hope that this either helps you connect to `Securewireless` at UC or points
you in the right direction for creating a profile to connect to your WPA
Enterprise network at your school/work.


[netctl]: https://wiki.archlinux.org/index.php/netctl
[eduroam]: https://wiki.archlinux.org/index.php/WPA2_Enterprise#netctl
[wpa_supplicant example]: http://w1.fi/cgit/hostap/plain/wpa_supplicant/wpa_supplicant.conf
[uc_it_handbook]: http://ucdirectory.uc.edu/studentplanner/ITHandbook.pdf
