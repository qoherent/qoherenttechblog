---
title: "Getting a pluto working on ubuntu server."
date: 2023-08-12T01:16:34-04:00
draft: false
cover:
    # image img/example.jpg
    alt: 'Getting a pluto working on ubuntu server.'
    caption: 'Not as simple as desktop!'
tags: ['pluto','sdr','ubuntu']
categories: ['how-to']
---


Pluto SDR doesnt automatically connect to ubuntu server as it is a USB device that emulates an ethernet conenction, and ubuntu server does not let USB devices install themselves automatically.

 

To get it set up, first verify the pluto is connected via **lsusb**:

```bash
lsusb
```

 ![](images/11_23_pluto_ubuntu_0.png)

With the pluto plugged in run:

```bash
ip link show
```

Take note of what is shown.

Unplug the pluto then run it again. Note what is missing.

```bash
ip link show
```

Plug it in again then run it one more time to see the new connection

```bash
ip link show
```

Note the name of the interface - in this case **enx00e022f21353**

  ![](images/11_23_pluto_ubuntu_1.png)

Manually set ifconfig for **enx00e022f21353**.

```bash
sudo ifconfig enx00e022f21353 192.168.3.2 netmask 255.255.255.0 up
```

Make special note to use the same IP subnet of your pluto (this one is set to the default of 192.168.3.1, so any IP on 192.168.3.x will do).

 

run ifconfig to see that the new interface exists.

```bash
ifconfig
```

 ![](images/11_23_pluto_ubuntu_2.png)

test with a ping

```bash
ping 192.168.3.1
```

if it returns a result, you are good to go!

 

 ![](images/11_23_pluto_ubuntu_3.png)

 
NOTE, this method will NOT persist across reboots. To do that, do the following method:


```bash
sudo nano /etc/netplan/02-usb-device.yaml
```
add these contents:
 

```bash
network:
  version: 2
  renderer: networkd
  ethernets:
    enx00e022f21353:  # use your correct interface name here
      dhcp4: no
      addresses: [192.168.3.2/24]
```

 

```bash
sudo netplan apply
```

```bash
ping 192.168.3.1
```

 