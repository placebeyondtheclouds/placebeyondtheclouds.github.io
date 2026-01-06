---
layout: post
title:  "Local cloud gaming (or any 3D accelerated app)"
lang: en
tags: [en, proxmox, cloud, gpu, sunshine, moonlight]
category: tutorial
published: false
---

My notes on how to set up GPU-accelerated desktop on a remote server.

## components

- Proxmox node, (6.8.12-11 kernel)
- GPU (P40 here)
- 50Mbps connection

## host setup

set up virtual machine with win11-24h2, `nano /etc/pve/qemu-server/102.conf`, important parameters:
```
hostpci0: 0000:07:00,pcie=1,rombar=0
args: -global q35-pcihost.pci-hole64-size=512G
balloon: 0
bios: ovmf
boot: order=scsi0;ide2;ide0;net0
cores: 28
cpu: x86-64-v2-AES
efidisk0: local-lvm:vm-102-disk-0,efitype=4m,pre-enrolled-keys=1,size=4M
ide0: local:iso/virtio-win-0.1.240.iso,media=cdrom,size=612812K
machine: pc-q35-9.2+pve1
memory: 32768
meta: creation-qemu=9.2.0,ctime=1766016326
ostype: win11
scsi0: local-lvm:vm-102-disk-1,cache=writeback,discard=on,iothread=1,size=128G,ssd=1
scsihw: virtio-scsi-single
tpmstate0: local-lvm:vm-102-disk-2,size=4M,version=v2.0

```

`lspci -nn | grep 'NVIDIA'`
`lspci -n -v -s 07:00.0`

`nano /etc/modprobe.d/kvm.conf`:
```
options kvm ignore_msrs=1
```

`nano /etc/modprobe.d/blacklist.conf`:
```
blacklist nouveau
blacklist nvidia
```

`nano /etc/modprobe.d/vfio.conf`:
```
softdep nouveau pre: vfio-pci
softdep nvidia pre: vfio-pci
options vfio-pci ids=10de:1b38
```

`nano /etc/modules`:
```
vfio
vfio_iommu_type1
vfio_pci
```

`nano /etc/default/grub`, add `pci=realloc` to GRUB_CMDLINE_LINUX_DEFAULT

`update-initramfs -u && update-grub`




`nano gpu-reset.sh && chmod u+x gpu-reset.sh` if needed:
```
#!/bin/bash
echo 1 > /sys/bus/pci/devices/0000\:07\:00.0/remove
echo 1 > /sys/bus/pci/rescan
```
`qm start 102`, install windows

also, mount guest hard disk on the host, if needed
```
apt install kpartx
kpartx -a /dev/pve/vm-102-disk-3
ls /dev/mapper/
mount /dev/mapper/pve-vm--102--disk--3p1 /mnt/tmp
umount /mnt/tmp
kpartx -d /dev/pve/vm-102-disk-3
```


## rebinding drivers between nvidia drivers (used for Nvidia container toolkit) and vfio

```
-------
lspci -v | grep -i nvidia
lspci -k -s 07:00.0 -n
lspci -nnk -d 10de:1b38
lspci -k -s 07:00.0


#nvidia to vfiopci
echo "0000:07:00.0" > /sys/bus/pci/devices/0000\:07\:00.0/driver/unbind
echo '10de 1b38'    > /sys/bus/pci/drivers/nvidia/remove_id
echo '10de 1b38'    > /sys/bus/pci/drivers/vfio-pci/new_id
#echo '0000:07:00.0' > /sys/bus/pci/devices/vfio-pci/bind
echo '10de 1b38'    > /sys/bus/pci/drivers/vfio-pci/remove_id



#vfiopci to nvidia
echo "0000:07:00.0" > /sys/bus/pci/devices/0000\:07\:00.0/driver/unbind
echo '10de 1b38'    > /sys/bus/pci/drivers/vfio-pci/remove_id
echo '10de 1b38'    > /sys/bus/pci/drivers/nvidia/new_id
#echo "0000:07:00.0" > /sys/bus/pci/drivers/nvidia/bind
echo '10de 1b38'    > /sys/bus/pci/drivers/nvidia/remove_id
```


## guest setup

install [virtio drivers](https://pve.proxmox.com/wiki/Windows_VirtIO_Drivers)

install https://github.com/Raphire/Win11Debloat

install NVIDIA driver 

test with Vulkan cube, take it from https://github.com/Juice-Labs/Juice-Labs/releases/download/2023.08.10-2103.0633b794/JuiceClient-windows.zip

install https://github.com/LizardByte/Sunshine

Sunshine is [configured](https://docs.lizardbyte.dev/projects/sunshine/latest/md_docs_2getting__started.html) via the web ui, which is available on https://localhost:47990 by default.

enable steam access in the host firewall https://steamcommunity.com/discussions/forum/1/1736595227842166572/



## clients

client https://github.com/moonlight-stream/moonlight-qt/releases

download appimage, on ubuntu also need: `sudo apt install libfuse2`. Ctrl+Alt+Shift+Q to exit fullscreen