# Installing Proxmox on Raspberry Pi 4 and 5

With these steps I managed to get Pimox on my Raspberry Pi 4 and 5 in february 2024.

## Step 1 - Flashing the OS

Install "RPi OS Lite 64-bit" with [Raspberry Pi Imager](https://www.raspberrypi.com/software/). It's listed under "Raspberry Pi OS (Other)"

![](https://i.imgur.com/Zw0PEpQ.png)

I set my user and password already here in the Raspberry Pi Imager when asked for config, so it's easy to SSH in later. I suggest you do the same.

## Step 2 - Network config

Assign your Pi a static IP in your router, then SSH into the Pi and launch the network config GUI with `nmtui` and adjust to the static ip.

## Step 3 - Install updates

```sh
apt-get update
apt-get upgrade
```

## Step 4 - Edit your host file

`nano /etc/hosts`
```
127.0.0.1 localhost pimox5
192.168.0.xx pimox5
::1 localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

> [!IMPORTANT]  
> Replace `192.168.0.xx` with the static IP you assigned your Pi in your router.

## Step 5 - Edit your hostname

`nano /etc/hostname`
```
pimox5
```

## Step 6 - Reboot

`reboot`

## Step 7 - Add sources and keys

`echo "deb [arch=arm64] https://global.mirrors.apqa.cn/proxmox/debian/pve bookworm port">/etc/apt/sources.list.d/pveport.list`

`curl https://global.mirrors.apqa.cn/proxmox/debian/pveport.gpg -o /etc/apt/trusted.gpg.d/pveport.gpg`

## Step 8 - Update and install Proxmox

```sh
apt-get update
apt-get upgrade
apt-get full-upgrade
apt-get dist-upgrade
```

```
apt-get install ifupdown2
apt-get install proxmox-ve postfix open-iscsi chrony mmc-utils usbutils
```

## Step 9 - Edit your network interface

`nano /etc/network/interfaces`

```
# interfaces(5) file used by ifup(8) and ifdown(8)
# Include files from /etc/network/interfaces.d:
# source /etc/network/interfaces.d/*

auto lo
iface lo inet loopback

iface eth0 inet manual

auto vmbr0
iface vmbr0 inet static
address 192.168.0.xx/24
gateway 192.168.0.1
bridge-ports eth0
bridge-stp off
bridge-fd 0

iface eth0 inet manual
```

> [!IMPORTANT]  
> Replace `192.168.0.xx` with the static IP you assigned your Pi in your router.

## Step 10 - Add DNS server

For me `/etc/resolv.conf` was empty. I added `nameserver 8.8.8.8` to this file.

## Step 11 - Reboot

Reboot your Pi again with `reboot`.

## Step 12 - Assign a password to your root user

Run `sudo -i` to get a root prompt, then `passwd` to set your password. This will be the login for the Proxmox UI.

## Step 13 - Done

You can now reach your Proxmox UI on `http://pimox5.local:8005` or `http://192.168.0.xx:8006` and login with the username `root` and the password you set in step 11.

> [!IMPORTANT]  
> Replace `192.168.0.xx` with the static IP you assigned your Pi in your router.


## Sources

- https://forum.proxmox.com/threads/proxmox-on-aarch64-arm64.121925/page-3
- https://www.reddit.com/r/Proxmox/comments/vpgn3f/how_to_login_pve_without_root_on_debian_after/

## Now what?

If you want Home Assistant on your Pimox instance you can use the PiMox HAOS VM script from here to set up a VM with Home Assistant OS: https://tteck.github.io/Proxmox/
