---
title: "OPNsense AT&T Gateway Bypass"
date: 2020-10-18T07:45:36-07:00
draft: false

description: "Step by step tutorial on OPNsense BGW210-700 bypass"
featured_image: "/images/opnsense_traffic.png"
tags: ["usg", "opnsense", "att", "fiber", "pfatt"]
---

All I wanted was a graph of real-time broadband utilization of my household.  A fully [Unifi](https://unifi-network.ui.com/)'ed network sure looks pretty, but [USG](https://amzn.to/31i09p3) just couln't do it.  Times have changed.  Many home networks are wired up for 1.0 Gbps, and today's open source firewalls have the features I need.  Like real-time traffic graphs.  My trusty USG served me well for years, but off to the ebay it goes.

In its place, I put together some OPNsense hardware and configured it to bypass AT&T Fiber CPE (BW210-700).  Here's how I did it.ds

First the easy part: purchase and assemble the hardware.


- QOTOM Q555G6 Intel i5 [mini PC](https://amzn.to/2IHdO2y)

- 8GB [DRAM](https://amzn.to/2HcGwro)

- 128GB [SSD](https://amzn.to/2IHGd8F)

- USB [Thumb drive](https://amzn.to/2Hhwk0w)


I bought a barren Qotom mini PC separately from the DRAM and SSD to get the components I thought best.  You can also buy a fully-assembled configuration (like this [one](https://amzn.to/3dDsxqx)) or, if you have an unused PC (with sufficiently beefy CPU and at least 3 NIC cards) lying around, you can use that.  Just make sure the ethernet hardware is [compatible](https://wiki.opnsense.org/manual/hardware.html) with OPNsense.

# Bootable USB drive
Since the raw hardware above comes with no software installed, follow the well written [instructions](https://docs.opnsense.org/manual/install.html) from OPNsense to prepare an USB boot drive - for qotom q555G6, select architecture `amd64`, and image type of `vga`.

![download](/images/opnsense_download.png)

# OPNsense configuration

Initial factory installation enables two network interfaces `LAN` and `WAN`.  We add one more and name it `OPT1`:

![opt1](/images/opnsense_add_opt1.png)

Next, simply follow the [instructions](https://github.com/MonkWho/pfatt) from project pfatt README.  Because it was written with pfSense in mind, be sure to note the instructions specific to OPNsense:

![pfatt](/images/opnsense_pfatt.png)


# opnatt.sh

OPNsense configuration steps are nearly identical to pfSense.  For the qotom hardware, `opnatt.sh` modification are below.

```
ONT_IF='igb1'  # NIC -> ONT / Outside
RG_IF='igb2'   # NIC -> Residential Gateway's ONT port
RG_ETHER_ADDR='99:55:b2:23:a4:45'  # MAC address of Residential Gateway
LOG=/var/log/opnatt.log
```

OPNsense network interface configurations are somewhat straight forward.  All three interfaces `LAN`, `OPT1` and `WAN` are pictured below with pi-hole running as the DNS server at `192.168.1.102`.  

# [LAN]

![pfatt](/images/opnsense_lan.png)

# [OPT1]

![pfatt](/images/opnsense_opt1.png)

# [WAN]

Don't forget to spoof the MAC address `RG_ETHER_ADDR` here. 
cal
![pfatt](/images/opnsense_wan.png)

# Firewall

We're almost done. Home Network Guy does a fantastic job of describing how to setup the OPNsense firewall rules [here](https://homenetworkguy.com/how-to/configure-opnsense-firewall-rules/)

# Lobby Dashboard

Once everything is setup, the lobby dashboard looks like this:

![lobby](/images/opnsense_lobby.png)
