---
layout: post
title:  "How to tap with Wireshark into a remote interface"
lang: en
tags: [en, wireshark, tcpdump]
category: tutorial
published: true
---

I use this approach for troubleshooting, watching traffic on a remote machine locally in GUI is very convenient.

## remote machine

`remote` machine is running any Linux distribution with tcpdump binary (Debian in my case) and I have privileged `admin` user credentials.

install tcpdump, change the capabilities to allow creating and using raw sockets and network administration without superuser privileges:

```bash
sudo apt install tcpdump
sudo setcap cap_net_raw,cap_net_admin=eip /usr/bin/tcpdump
```

limit running tcpdump to the admin user (if the remote machine is a multiuser environment):

```bash
sudo groupadd pcap
sudo usermod -aG pcap admin
sudo chown root:pcap /usr/bin/tcpdump
sudo chmod 750 /usr/bin/tcpdump
```

## local machine

local machine is running Linux with desktop manager (I use Ubuntu), have wireshark installed, ssh **key** access to the `remote` machine.

warm up sudo to avoid typing in the password. it is also possible to add current user to the wireshark group, for that must first choose to allow *non-superusers be able to capture packets* during wireshark installation (which basically does the same thing as above but for `dumpcap` on the local machine), and then `sudo usermod -a -G wireshark $USER`:

```bash
sudo -v
```

**run the packet capture on the remote machine** (change the interface name accordingly):

```bash
ssh admin@remote tcpdump -i eth0 -U -s0 -w - 'port not 22' | sudo wireshark -k -i -
```

in the end, remove the capabilities on the remote machine:

```bash
ssh -t admin@remote sudo setcap -r /usr/bin/tcpdump
```