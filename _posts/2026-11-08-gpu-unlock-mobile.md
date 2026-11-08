---
layout: post
title:  "Unlock RTX 4050 for +30% boost on hashcat"
lang: en
tags: [gpu_unlock, hashcat, runbook, Ubuntu]
category: tutorial
published: true
---

It is possible to unlock full speed for tensor cores in consumer series of Nvidia GPUs using a kernel module. My laptop's GPU is not yet fully supported by the unlock mod, but I tried it and the mod works. A little modification is needed before installing the mod.


### install

first check my gpu ID

```bash
lspci | grep -i nvidia
lspci -nnk -s 0000:32:00.0
```

it would say

```
32:00.0 VGA compatible controller [0300]: NVIDIA Corporation AD107M [GeForce RTX 4050 Max-Q / Mobile] [10de:28a1] (rev a1)
```

we need this `10de:28a1` ID.

then clone the repo:

```bash
git clone https://github.com/Zagitta/gpu-unlock
cd gpu-unlock
git checkout --detach 7eb51bccae0a0b7e599e084d971170e9a1e82d0c
```

`kernel/gpu_unlock/recipes.c` has a line
```
AD107(0x2882), AD107(0x28a0), AD107(0x28e0), AD107(0x28e1),
```

to which we need to add my GPU ID
```
AD107(0x2882), AD107(0x28a0), AD107(0x28a1), AD107(0x28e0), AD107(0x28e1),
```

like this:
```bash
sed -i 's/AD107(0x28a0),/AD107(0x28a0), AD107(0x28a1),/' kernel/gpu_unlock/recipes.c
```

then `make install`, `sudo nano /etc/default/grub` and add `gpu_unlock.targets=all` to GRUB_CMDLINE_LINUX_DEFAULT, `sudo update-grub`, `sudo shutdown -P now`.

if it loads successfully, the display would freeze on the first initialization (i had to input my LUKS password blindly) and then continues to load. can be verified with

```bash
sudo journalctl -k -b --no-pager | grep -i "gpu_unlock: unlocked"
```

### reinstall

if we need to edit or add another GPU ID on the same machine, edit the `kernel/gpu_unlock/recipes.c` and then (on Ubuntu):

```bash
make deb
sudo apt install --reinstall ./build/deb/gpu-unlock-dkms_2.0.0_all.deb
sudo update-initramfs -u -k "$(uname -r)"
sudo shutdown -P now
```

### power management

dynamic power management can interfere with this mod. to disable the power management:

```bash
cat /proc/driver/nvidia/params | grep DynamicPowerManagement
#would say 2
echo 'options nvidia NVreg_DynamicPowerManagement=0x00' | sudo tee /etc/modprobe.d/nvidia-power-management.conf
sudo update-initramfs -u -k "$(uname -r)"
sudo shutdown -r now
```

to roll back this change:

```bash
sudo rm /etc/modprobe.d/nvidia-power-management.conf
sudo update-initramfs -u -k "$(uname -r)"
sudo shutdown -r now
```

### results

the benchmark is

```bash
docker run  --gpus='"device=0"' -it --rm dizcza/docker-hashcat hashcat -b
```

before:
```
---------------------
* Hash-Mode 900 (MD4)
---------------------

Speed.#01........: 27950.4 MH/s (104.02ms) @ Accel:160 Loops:1024 Thr:896 Vec:8

-------------------
* Hash-Mode 0 (MD5)
-------------------

Speed.#01........: 15466.0 MH/s (80.68ms) @ Accel:120 Loops:1024 Thr:512 Vec:8

----------------------
* Hash-Mode 100 (SHA1)
----------------------

Speed.#01........:  5406.8 MH/s (92.53ms) @ Accel:64 Loops:1024 Thr:384 Vec:1

---------------------------
* Hash-Mode 1400 (SHA2-256)
---------------------------

Speed.#01........:  2059.8 MH/s (101.27ms) @ Accel:40 Loops:1024 Thr:256 Vec:4

```


after:
```
---------------------
* Hash-Mode 900 (MD4)
---------------------

Speed.#01........: 37694.0 MH/s (92.42ms) @ Accel:192 Loops:1024 Thr:896 Vec:8

-------------------
* Hash-Mode 0 (MD5)
-------------------

Speed.#01........: 20551.0 MH/s (96.88ms) @ Accel:96 Loops:1024 Thr:1024 Vec:8

----------------------
* Hash-Mode 100 (SHA1)
----------------------

Speed.#01........:  7257.6 MH/s (91.77ms) @ Accel:128 Loops:1024 Thr:256 Vec:1

---------------------------
* Hash-Mode 1400 (SHA2-256)
---------------------------

Speed.#01........:  2729.7 MH/s (91.46ms) @ Accel:12 Loops:1024 Thr:1024 Vec:4
```