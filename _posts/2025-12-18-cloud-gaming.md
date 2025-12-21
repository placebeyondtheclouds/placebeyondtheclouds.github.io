---
layout: post
title:  "Local cloud gaming"
lang: en
tags: [en, proxmox, cloud, gpu, sunshine, moonlight]
category: tutorial
published: false
---



## components

- Proxmox node
- GPU
- 50Mbps connection

## steps

set up virtual machine with win11-24h2, `nano /etc/pve/qemu-server/102.conf`, key parameters:

```
hostpci0: 0000:07:00,pcie=1,rombar=0
args: -global q35-pcihost.pci-hole64-size=512G
balloon: 0
bios: ovmf
boot: order=scsi0;ide2;ide0;net0
cores: 4
cpu: x86-64-v2-AES
efidisk0: local-lvm:vm-102-disk-0,efitype=4m,pre-enrolled-keys=1,size=4M
ide0: local:iso/virtio-win-0.1.240.iso,media=cdrom,size=612812K
machine: pc-q35-9.2+pve1
memory: 16384
meta: creation-qemu=9.2.0,ctime=1766016326
ostype: win11
scsi0: local-lvm:vm-102-disk-1,cache=writeback,discard=on,iothread=1,size=128G,ssd=1
scsihw: virtio-scsi-single
tpmstate0: local-lvm:vm-102-disk-2,size=4M,version=v2.0

```

`qm set 102 `

`nano /etc/modprobe.d/kvm.conf`:
```
options kvm ignore_msrs=1
```

`nano /etc/modprobe.d/blacklist.conf`:
```
blacklist nouveau
blacklist snd_hda_intel
```

`nano /etc/default/grub`:
```
add `pci=realloc` to GRUB_CMDLINE_LINUX_DEFAULT
```

`update-initramfs -u -k all && update-grub`


`lspci -nn | grep 'NVIDIA'`
`lspci -n -v -s 07:00.0`

`nano gpu-reset.sh`:
```
#!/bin/bash
echo 1 > /sys/bus/pci/devices/0000\:07\:00.0/remove
echo 1 > /sys/bus/pci/rescan
```


install https://github.com/Raphire/Win11Debloat

install NVIDIA driver ([P40](https://www.nvidia.com/zh-tw/geforce/drivers/results/158400/))

install https://github.com/LizardByte/Sunshine

Sunshine is [configured](https://docs.lizardbyte.dev/projects/sunshine/latest/md_docs_2getting__started.html) via the web ui, which is available on https://localhost:47990 by default.

enable steam access in the firewall https://steamcommunity.com/discussions/forum/1/1736595227842166572/

client https://github.com/moonlight-stream/moonlight-qt/releases

donwload appimage, on ubuntu: `sudo apt install libfuse2`. Ctrl+Alt+Shift+Q to exit fullscreen