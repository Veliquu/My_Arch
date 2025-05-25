# Select the mirrors
Packages to be installed must be downloaded from mirror servers, which are defined in /etc/pacman.d/mirrorlist. On the live system, after connecting to the internet, reflector updates the mirror list by choosing 20 most recently synchronized HTTPS mirrors and sorting them by download rate.

The higher a mirror is placed in the list, the more priority it is given when downloading a package. You may want to inspect the file to see if it is satisfactory. If it is not, edit the file accordingly, and move the geographically closest mirrors to the top of the list, although other criteria should be taken into account.

This file will later be copied to the new system by pacstrap, so it is worth getting right.

# Install Kernel
Use the pacstrap script to install the base package, Linux kernel and firmware for common hardware:
```
# pacstrap -K /mnt base linux linux-firmware
```
Later when we chroot to the new system we will install some tools using pacman.

# Configure the System

Generate **fstab** file (The fstab(5) file can be used to define how disk partitions, various other block devices, or remote file systems should be mounted into the file system.).
```
# genfstab -U /mnt >> /mnt/etc/fstab
```
Check the resulting /mnt/etc/fstab file, and edit it in case of errors.  

Change root into the new system:
```
# arch-chroot /mnt
```
The new look for the terminal looks like this:
```
[root@archiso /]#
```

Now that you are in the new system, we wioll set the time zone (in my case Europe/Helsinki):
```
# ln -sf /usr/share/zoneinfo/Europe/Helsinki /etc/localtime
```
Run hwclock to generate /etc/adjtime:
```
# hwclock --systohc
```

Next we set localization settings: system language and keyboard layout for the new system.  
Edit /etc/locale.gen and uncomment en_US.UTF-8 UTF-8 and other needed UTF-8 locales. In my case I needed to install text editor and I went whitr micro.
```
pacman -S micro

micro /etc/locale.gen
```
Generate the locales by running:
```
# locale-gen
```
Next we create locale.conf and set the LANG variable accordingly (set the language you chose):
```
micro /etc/locale.conf
---
LANG=en_US.UTF-8
```
If you set the console keyboard layout, make the changes persistent in vconsole.conf:
```
micro /etc/vconsole.conf
---
KEYMAP=fi
```
# Tools and root

Now you can install some tools you may need.  
I will be installing
- network manager for wireless network. (networkmanager)
- poreviously installed console text editor. In my case it was **micro** but there are other choises like **nano** and **vim**.
- As I have intel system I will install intel-ucode but if you have adm system you should install amd-ucode. These are for hardware bug and security fixes.
- packages for accessing documentation in man and info pages: man-db, man-pages and texinfo.
- Choose and install a Linux-capable boot loader. In this guide I will go with GRUB

You install these with pacman. In pacman `-S` is for installing packages. For example for the network manager:
```
# pacman -S networkmanager
```

Next you create root password:
```
# passwd
```

# Reboot
First you need to exit the chroot enviroment by typing `exit`.  
I recommend manually unmount all the partitions with `umount -R /mnt`, his allows noticing any "busy" partitions, and finding the cause with fuser (fuser - identify processes using files or sockets).  
Lasty reboot your system by typing `reboot`. Remember to remove the installation medium and then login into the new system with the root account.  

