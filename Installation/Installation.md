# Comprehensive Arch Linux Installation Guide

This guide covers the complete Arch Linux installation process after you've completed the pre-installation setup (partitioning, formatting, and mounting filesystems).

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Preparing Arch Installation Environment](#preparing-arch-installation-environment)
   - [Install Base System](#install-base-system)
   - [Generate fstab File](#generate-fstab-file)
   - [Enter chroot Environment](#enter-chroot-environment)
3. [User Accounts](#user-accounts)
   - [Set Root Password](#set-root-password)
   - [Create User Account](#create-user-account)
   - [Set User Password](#set-user-password)
4. [Installing Basic Packages](#installing-basic-packages)
   - [Essential System Packages](#essential-system-packages)
   - [Install CPU Microcode](#install-cpu-microcode)
   - [Enable Essential Services](#enable-essential-services)
5. [Kernel and Firmware](#kernel-and-firmware)
   - [Install Linux Kernels](#install-linux-kernels)
   - [Install Firmware](#install-firmware)
6. [GPU Drivers](#gpu-drivers)
   - [Identify Your GPU](#identify-your-gpu)
   - [Install GPU Drivers](#install-gpu-drivers)
7. [System Configuration](#system-configuration)
   - [Generate Kernel Ramdisks](#generate-kernel-ramdisks)
   - [Configure System Locale](#configure-system-locale)
   - [Configure Sudo Access](#configure-sudo-access)
8. [Setting up GRUB Bootloader](#setting-up-grub-bootloader)
   - [Install GRUB to EFI](#install-grub-to-efi)
   - [Copy Locale Files for GRUB](#copy-locale-files-for-grub)
   - [Generate GRUB Configuration](#generate-grub-configuration)
9. [Final System Setup](#final-system-setup)
   - [Set Hostname](#set-hostname)
   - [Configure Hosts File](#configure-hosts-file)
   - [Set Timezone](#set-timezone)
   - [Generate Hardware Clock](#generate-hardware-clock)
10. [Wrapping Up](#wrapping-up)
    - [Exit chroot Environment](#exit-chroot-environment)
    - [Unmount Partitions](#unmount-partitions)
    - [Reboot System](#reboot-system)
    - [First Boot](#first-boot)
11. [Post-Installation Checklist](#post-installation-checklist)
    - [Next Steps](#next-steps)
    - [Troubleshooting Common Issues](#troubleshooting-common-issues)
12. [Understanding Command Parameters](#understanding-command-parameters)

---

## Prerequisites

Before starting this installation guide, ensure you have completed:
- **Pre-installation setup** (partitioning, formatting, mounting)
- **Internet connection** is active and working
- **System clock** is synchronized
- **Root partition** mounted at `/mnt`
- **EFI partition** mounted at `/mnt/boot/efi`
- **Basic familiarity** with command-line interface

**Required knowledge:** Understanding of Linux file systems, partitioning concepts, and basic terminal commands.

---

# Preparing Arch Installation Environment

Now that the partitions are mounted, we need to install the base packages for Arch Linux.

## Install Base System
```
# pacstrap -i /mnt base
```

**Command breakdown:**
- `pacstrap` - Script that installs packages to the new root directory
- `-i` - Interactive mode (prompts before installing packages)
- `/mnt` - Target directory where we mounted our root partition
- `base` - Meta-package containing essential system components

The `base` package includes core utilities like bash, coreutils, glibc, and other fundamental components needed for a minimal Arch Linux system.

## Generate fstab File
```
# genfstab -U -p /mnt >> /mnt/etc/fstab
```

**Command breakdown:**
- `genfstab` - Generates file system table entries
- `-U` - Use UUIDs instead of device names (more reliable across reboots)
- `-p` - Exclude pseudofs mounts (like /proc, /sys)
- `/mnt` - Root of the new installation
- `>>` - Append output to the fstab file

The **fstab** file defines how disk partitions and other block devices should be mounted into the filesystem at boot time. This ensures your partitions are automatically mounted when the system starts.

## Enter chroot Environment
```
# arch-chroot /mnt
```

**Command breakdown:**
- `arch-chroot` - Arch Linux specific chroot command
- `/mnt` - Directory to chroot into

**chroot** (change root) allows us to work inside the new installation as if it were the running system. This lets us configure the system from within.

---

# User Accounts

## Set Root Password
First we create a password for the root account:
```
# passwd
```

The root account is the administrator account with full system privileges. Always set a strong password for security.

## Create User Account
We also create a user account for daily use. We'll add it to the `users` and `wheel` groups:
```
# useradd -m -g users -G wheel username
```

**Command breakdown:**
- `useradd` - Command to add new user accounts
- `-m` - Create home directory for the user
- `-g users` - Set primary group to "users"
- `-G wheel` - Add user to supplementary "wheel" group
- `username` - Replace with your desired username

**Group explanations:**
- `users` - Standard group for regular users
- `wheel` - Group that traditionally has sudo/administrator privileges

## Set User Password
Next we set a password for your user account:
```
# passwd username
```

Replace `username` with the actual username you created.

---

# Installing Basic Packages

## Essential System Packages
Install essential packages needed for a functional system:
```
# pacman -S base-devel dosfstools grub efibootmgr micro networkmanager openssh os-prober sudo
```

**Package explanations:**
- `base-devel` - Meta-package with development tools (gcc, make, etc.) needed for building packages
- `dosfstools` - Utilities for FAT filesystem (needed for EFI boot partition)
- `grub` - Grand Unified Bootloader (handles system booting)
- `efibootmgr` - Tool to modify UEFI boot entries
- `micro` - User-friendly text editor (alternative to nano/vim)
- `networkmanager` - Network connection management service
- `openssh` - Secure Shell for remote access
- `os-prober` - Detects other operating systems (useful for dual-boot)
- `sudo` - Allows users to run commands with elevated privileges

## Install CPU Microcode
Install microcode updates for hardware bug fixes and security patches:

**For Intel CPUs:**
```
# pacman -S intel-ucode
```

**For AMD CPUs:**
```
# pacman -S amd-ucode
```

**What is microcode?** Microcode provides hardware-level bug fixes and security patches directly from CPU manufacturers. It's loaded during boot to update CPU behavior.

## Enable Essential Services

### Enable NetworkManager
Enable NetworkManager so networking will function after reboot:
```
# systemctl enable NetworkManager
```

**Command breakdown:**
- `systemctl` - Controls systemd services
- `enable` - Set service to start automatically at boot
- `NetworkManager` - Network management service

### Enable SSH (Optional)
If you installed openssh and want remote access:
```
# systemctl enable sshd
```

---

# Kernel and Firmware

## Install Linux Kernels
Install the Linux kernel(s). Having multiple kernels provides backup options:
```
# pacman -S linux linux-headers linux-lts linux-lts-headers
```

**Package explanations:**
- `linux` - Latest stable Linux kernel
- `linux-headers` - Header files for building kernel modules (for main kernel)
- `linux-lts` - Long Term Support kernel (more stable, older version)
- `linux-lts-headers` - Header files for LTS kernel

**Why install both kernels?**
- **Main kernel** - Latest features and hardware support
- **LTS kernel** - More stable, tested longer, good for servers
- **Backup option** - If one kernel fails, you can boot the other

## Install Firmware
Install firmware files for hardware devices:
```
# pacman -S linux-firmware
```

This package contains firmware files for various hardware components like WiFi cards, graphics cards, and other devices that require proprietary firmware to function.

---

# GPU Drivers

## Identify Your GPU
First, identify what kind of GPU you have:
```
# lspci | grep -E "(VGA|3D)"
```

Example output:
```
00:02.0 VGA compatible controller: Intel Corporation Meteor Lake-P [Intel Graphics] (rev 08)
```

## Install GPU Drivers

### For Intel and AMD GPUs
```
# pacman -S mesa
```

**Mesa** is an open-source graphics library that provides OpenGL, Vulkan, and other graphics APIs for Intel and AMD GPUs.

### For NVIDIA GPUs
```
# pacman -S nvidia nvidia-utils
```

**Package explanations:**
- `nvidia` - NVIDIA proprietary driver for current kernel
- `nvidia-utils` - NVIDIA driver utilities and libraries

### For NVIDIA with LTS Kernel
If you installed the LTS kernel, also install:
```
# pacman -S nvidia-lts
```

This ensures NVIDIA drivers work with the LTS kernel.

**Important:** NVIDIA drivers are kernel-specific, so you need different packages for different kernels.

---

# System Configuration

## Generate Kernel Ramdisks
Create initial ramdisk environments for each installed kernel:
```
# mkinitcpio -p linux
# mkinitcpio -p linux-lts
```

**Command breakdown:**
- `mkinitcpio` - Tool to build initial ramdisk
- `-p` - Use preset configuration
- `linux`/`linux-lts` - Kernel preset to build for

The initial ramdisk (initramfs) contains essential drivers and tools needed to mount the root filesystem during boot.

## Configure System Locale
Set up your system language by editing the locale configuration:
```
# micro /etc/locale.gen
```

In the editor, find and uncomment the line:
```
en_US.UTF-8 UTF-8
```

Or uncomment your preferred language/locale. Save and exit the editor.

**Common locales:**
- `en_US.UTF-8 UTF-8` - US English
- `en_GB.UTF-8 UTF-8` - UK English
- `de_DE.UTF-8 UTF-8` - German
- `fr_FR.UTF-8 UTF-8` - French
- `es_ES.UTF-8 UTF-8` - Spanish

### Generate Locales
Generate the locale files:
```
# locale-gen
```
This creates the actual locale files based on your selections in `/etc/locale.gen`.  

  
### Set keyboard layout
If you set the console keyboard layout, make the changes persistent in vconsole.conf:
```
# echo "KEYMAP=fi" > /etc/vconsole.conf
```

### Set System Locale
Create the locale configuration file:
```
# echo "LANG=en_US.UTF-8" > /etc/locale.conf
```

## Configure Sudo Access
Allow users in the wheel group to use sudo:
```
# EDITOR=micro visudo
```

Find and uncomment this line:
```
%wheel ALL=(ALL:ALL) ALL
```

This allows members of the wheel group to execute any command with sudo.

**Security note:** Only users you trust should be in the wheel group, as they gain administrative privileges.

---

# Setting up GRUB Bootloader

Previously we mounted our boot partition to `/mnt/boot/efi`. This makes GRUB setup easier.

## Install GRUB to EFI
```
# grub-install --target=x86_64-efi --bootloader-id=grub_uefi --recheck
```

**Command breakdown:**
- `grub-install` - Install GRUB bootloader
- `--target=x86_64-efi` - Install for 64-bit UEFI systems
- `--bootloader-id=grub_uefi` - Set identifier for boot entry
- `--recheck` - Force checking of device map

## Copy Locale Files for GRUB
```
# cp /usr/share/locale/en\@quot/LC_MESSAGES/grub.mo /boot/grub/locale/en.mo
```

This copies locale files so GRUB messages appear in English (or your chosen language).

## Generate GRUB Configuration
```
# grub-mkconfig -o /boot/grub/grub.cfg
```

**Command breakdown:**
- `grub-mkconfig` - Generate GRUB configuration file
- `-o /boot/grub/grub.cfg` - Output file location

This creates the GRUB menu that appears when you boot, listing available kernels and boot options.

---

# Final System Setup

## Set Hostname
Give your system a name:
```
# echo "your-hostname" > /etc/hostname
```

Replace `your-hostname` with your desired computer name.

**Hostname guidelines:**
- Use lowercase letters, numbers, and hyphens only
- Don't start or end with hyphens
- Keep it short and descriptive
- Examples: `arch-desktop`, `my-laptop`, `server01`

## Configure Hosts File
```
# micro /etc/hosts
```

Add these lines:
```
127.0.0.1    localhost
::1          localhost
127.0.1.1    your-hostname.localdomain your-hostname
```

The hosts file maps hostnames to IP addresses for local resolution.

## Set Timezone
```
# ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
```

Example:
```
# ln -sf /usr/share/zoneinfo/Europe/Helsinki /etc/localtime
```

**Common timezone examples:**
- `America/New_York` - US Eastern
- `America/Los_Angeles` - US Pacific
- `Europe/London` - UK
- `Europe/Berlin` - Germany
- `Asia/Tokyo` - Japan
- `Australia/Sydney` - Australia

## Generate Hardware Clock
```
# hwclock --systohc
```

This generates `/etc/adjtime` and sets the hardware clock from the system clock.

---

# Wrapping Up

## Exit chroot Environment
```
# exit
```

This returns you to the installation media environment.

## Unmount Partitions
```
# umount -a
```

**Command breakdown:**
- `umount` - Unmount filesystems
- `-a` - Unmount all mounted filesystems

## Reboot System
```
# reboot
```

**Important:** Remove the installation USB/DVD before the system restarts.

## First Boot
After rebooting, you'll see a login screen like this:

![Login Screen](https://github.com/user-attachments/assets/86ee3e7f-5c26-49fe-bcf1-928e3b45228a)

You can now login with the user account you created.

---

# Post-Installation Checklist

After first boot, verify these work correctly:

- [ ] **Network connection** - Test internet connectivity
- [ ] **User login** - Login with your user account
- [ ] **Sudo access** - Test `sudo` commands work
- [ ] **System updates** - Run `sudo pacman -Syu`
- [ ] **Graphics** - Verify display works correctly
- [ ] **Audio** - Test sound output (if applicable)
- [ ] **Time/Date** - Verify correct timezone and time

## Next Steps

Your base Arch Linux system is now installed and ready. You can now:

1. **Install a desktop environment** (GNOME, KDE, XFCE, etc.)
2. **Install additional software** using pacman
3. **Configure additional services** as needed
4. **Set up development tools** if needed
5. **Install an AUR helper** (yay, paru) for additional packages

## Troubleshooting Common Issues

### Boot Issues
- If system won't boot, use the LTS kernel option in GRUB
- Check UEFI boot order in BIOS settings
- Verify GRUB was installed correctly
- Ensure Secure Boot is still disabled

### Network Issues
- Ensure NetworkManager is enabled: `sudo systemctl status NetworkManager`
- For WiFi issues, try: `sudo systemctl restart NetworkManager`
- Check network interface status: `ip link`

### Graphics Issues
- Verify correct GPU drivers are installed
- Check Xorg logs: `journalctl -u display-manager`
- For NVIDIA issues, verify nvidia modules are loaded: `lsmod | grep nvidia`

### Permission Issues
- Verify user is in correct groups: `groups username`
- Check sudo configuration: `sudo visudo`
- Ensure wheel group has sudo access

### Package Issues
- Update package database: `sudo pacman -Sy`
- Clear package cache if needed: `sudo pacman -Sc`
- Check for corrupted packages: `sudo pacman -Qkk`

---

# Understanding Command Parameters

This installation guide uses many commands with various parameters. Here's what the common ones mean:

## Single Dash (-) Parameters
Single dashes are used for **short options** (single letter flags):

- `pacstrap -i` → The `-i` flag enables interactive mode (prompts before installing)
- `genfstab -U -p` → `-U` uses UUIDs, `-p` excludes pseudofs mounts
- `useradd -m -g -G` → `-m` creates home directory, `-g` sets primary group, `-G` sets supplementary groups
- `mkinitcpio -p` → The `-p` flag uses preset configuration
- `grub-mkconfig -o` → The `-o` flag specifies output file
- `umount -a` → The `-a` flag unmounts all filesystems

## Double Dash (--) Parameters  
Double dashes are used for **long options** (full word flags):

- `grub-install --target=x86_64-efi` → Specifies installation target architecture
- `grub-install --bootloader-id=grub_uefi` → Sets bootloader identifier
- `grub-install --recheck` → Forces device map checking
- `hwclock --systohc` → Sets hardware clock from system clock
- `ln -sf` → `-s` creates symbolic link, `-f` forces overwrite

## Parameter Examples from This Guide

**Package management:**
- `pacman -S` → Install packages (`-S` means sync/install)
- `pacman -Syu` → Update system (`-y` refreshes database, `-u` upgrades)

**Service management:**
- `systemctl enable` → Enable service to start at boot
- `systemctl status` → Check service status

**File operations:**
- `>>` → Append output to file (not overwrite)
- `>` → Redirect output to file (overwrites existing)

## Why Use Parameters?
- **Efficiency** - Modify command behavior without separate commands
- **Precision** - Specify exactly what you want the command to do
- **Safety** - Some parameters add confirmation prompts or safety checks
- **Flexibility** - Same command can perform different operations based on parameters

**Pro tip:** Most commands support `--help` or `-h` to show available parameters and their meanings.

---

Congratulations! You now have a fully functional Arch Linux system ready for customization and daily use.
