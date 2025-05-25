# Comprehensive Arch Linux Installation Guide

This guide covers the complete Arch Linux installation process after you've completed the pre-installation setup (partitioning, formatting, and mounting filesystems).

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

Having both kernels means if one fails to boot, you can use the other as backup.

## Install Firmware
Install firmware files for hardware devices:
```
# pacman -S linux-firmware
```

This package contains firmware files for various hardware components like WiFi cards, graphics cards, and other devices.

---

# GPU Drivers

## Identify Your GPU
First, identify what kind of GPU you have:
```
# lspci | grep -E "(VGA|3D)"
```

Example output:
![GPU Information](https://github.com/user-attachments/assets/15f1cafa-9b1c-4f2c-b7a4-6f7cceb40034)

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

## Generate Locales
Generate the locale files:
```
# locale-gen
```

This creates the actual locale files based on your selections in `/etc/locale.gen`.

## Set System Locale
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

## Set Timezone
```
# ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
```

Example:
```
# ln -sf /usr/share/zoneinfo/Europe/Helsinki /etc/localtime
```

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

### Network Issues
- Ensure NetworkManager is enabled: `sudo systemctl status NetworkManager`
- For WiFi issues, try: `sudo systemctl restart NetworkManager`

### Graphics Issues
- Verify correct GPU drivers are installed
- Check Xorg logs: `journalctl -u display-manager`

### Permission Issues
- Verify user is in correct groups: `groups username`
- Check sudo configuration: `sudo visudo`

Congratulations! You now have a fully functional Arch Linux system ready for customization and daily use.
