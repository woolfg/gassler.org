---
title: "Hardware acceleration on Fedora 37 with AMD Radeon 680M"
date: "2023-03-15"
slug: "fedora-hvec-amd"
tags: [stackoverflowish,fedora,linux]
notOnHome: true
summary: "How to get hardware acceleration on Fedora 37 with AMD Radeon 680M to e.g. use DaVinci Resolve and Shotcut"
---

I recently got a new laptop: Lenovo ThinkPad T14s AMD Ryzen 7 PRO 6850U with a AMD Radeon 680M.
As a Linux and Fedora 37 user, I wanted to have hardware acceleration to e.g. use [DaVinci Resolve](https://www.blackmagicdesign.com/products/davinciresolve).

It tool me quite a while to get in working, but here is what I did:

- [Install the RPM Fusion repositories](https://rpmfusion.org/Configuration) (free should be sufficient)
- Swap the mesa driver to drivers provided by RPM Fusion
  - run `sudo dnf swap mesa-va-drivers mesa-va-drivers-freeworld --allowerasing`
  - run `sudo dnf swap mesa-vdpau-drivers mesa-vdpau-drivers-freeworld --allowerasing`
- Install the ROCm OpenCL drivers by `sudo dnf install rocm-opencl`

That's it and `clinfo` should show all details. After this, [Shotcut](https://shotcut.org/) was able to
find hvec (hardware acceleration) and use it as well. Furthermore, [DaVinci Resolve](https://www.blackmagicdesign.com/products/davinciresolve) started as it was
able to detect a GPU.