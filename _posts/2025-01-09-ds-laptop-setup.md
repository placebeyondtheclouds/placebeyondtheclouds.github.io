---
layout: post
title:  "My Laptop setup for deep learning"
lang: en
tags: [en]
published: true
---

My notes from last year (2024) on how I set up a lenovo thinkbook gen6 laptop with ubuntu 24.04 as a hypervisor for QEMU/KVM virtualization with GPU passthrough to guests and hibernation.

## end result:

- Ubuntu 24.04 host with encrypted LUKS partitions, hibernation, and GPU passthrough to guests
- guest VMs running in Virtual Machine Manager with (one at a time) and without NVIDIA GPU passthrough, Intel Arc Graphics (MTL) GPU for the video output.


## Github repo

[Linux GPU laptop setup for data science and software development](https://github.com/placebeyondtheclouds/lenovo-laptop-with-ubuntu-and-libvirt)

