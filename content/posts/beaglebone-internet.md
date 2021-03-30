---
author: "Kalyan"
title: "Sharing internet connection with Beaglebone"
date: "2021-03-30"
description: "Sharing internet connection of host machine with Beaglebone"
tags: ["beaglebone"]
ShowToc: true

---

## Sharing internet connection with Beaglebone

The beaglebone can be connected over serial and the fun part is that the usb post emulates the lan connection. So we don't need to have another lan connection to `ssh` into the board.

The present versions the debian seems to recognize the SD card and boot from the SD Card directly. The instructions seem to suggest that we should hold the `USERBUTTON` while providing power in-order to boot from SD Card.

Connect the Beaglebone with the USB Micro and wait for sometime to let the Debian image to boot. 

### Host

In order to check the device you can enter

`ifconfig`

It should look something like this

```bash
enx402e71cff462: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.7.1  netmask 255.255.255.0  broadcast 192.168.7.255
        inet6 fe80::91d8:e48:fa2e:b206  prefixlen 64  scopeid 0x20<link>
        ether 40:2e:71:cf:f4:62  txqueuelen 1000  (Ethernet)
        RX packets 86  bytes 11773 (11.7 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 149  bytes 29354 (29.3 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
.
.
.
wlo1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.0.157  netmask 255.255.255.0  broadcast 192.168.0.255
```

The important information that we need is `enx402e71cff462` and `wlo1`. Take a note of that we will be using it in a few minutes.

Enter these lines in the terminal. Make sure to use the device ID that you have by typing `ifconfig`

```bash
sudo ifconfig enx402e71cff462 192.168.7.1
sudo sysctl net.ipv4.ip_forward=1
sudo iptables --table nat --append POSTROUTING --out-interface wlo1 -j MASQUERADE
sudo iptables --append FORWARD --in-interface enx402e71cff462 -j ACCEPT
```

### Beaglebone

On the beaglebone side, enter these commands.

```bash
sudo route add default gw 192.168.7.1
echo "nameserver 8.8.8.8" >> /etc/resolv.conf
```

Thats it. You should have the Internet on Beaglebone.

