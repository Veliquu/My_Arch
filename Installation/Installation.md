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

