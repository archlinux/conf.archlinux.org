---
title: "Versioned Kernel Installs"
date: 2019-10-12T16:54:46+02:00
---

# Versioned kernel installs

# Problems
Upon installing a new kernel, the module tree is removed, meaning that subsequent loading of modules fails (e.g. pluging an USB drive)
If there is an issue with the new kernel, no old one to boot on, while -lts or any other kernel might not be a suitable fallback

# Issues with versioned kernel installs
Bootloader? Currently we install /boot/vmlinuz-linux and generate an associated /boot/initramfs-linux.img. How to handle versioned version without people having to rewrite their bootloader on each kernel update?
Symlinking seems nice (default names link to newest version), but does not work on FAT, e.g. systemd-boot
kernel-install from systemd can manage multiple kernels apparently, but then what about non-UEFI systems
Do all bootloaders have a way to specify glob pattern in kernel image/initramfs so that they grab all versioned kernels?
No, generating config is unavoidable


Changes planned regarding the port to Dracut would let us easily leave at least old kernels + initramfs around, making rescue boots possible:
/boot/* no longer packaged
Hooks in mkinitcpio and and Dracut (might use kernel-install, or not) copy kernels from /lib/modules to /boot/, in different naming schemes

# Seblu versioned kernels with kernel-install and mkinitcpio:
    - https://git.seblu.net/archlinux/linux-seblu-meta
    - https://git.seblu.net/archlinux/kernel-install-poc

