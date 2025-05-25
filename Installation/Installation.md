# Preparin Arch installatioin enviroment
Now that the partitions are mounted we need to install the base packages for Arch: 
```
# pacstrap -i /mnt base
```
Next we will generate the **fstab** file. The fstab file can be used to define how disk partitions, various other block devices, or remote file systems should be mounted into the file system.
```
# genfstab -U -p /mnt >> /mnt/etc/fstab
```
Now we are ready to **chroot** into the system to fine-tune and finalize the installation.
```
# arch-chroot /mnt
```

# User accounts
First we create password for the root aaccount:
```
# passwd
```
We also create user for ourself. We will add it to the `users` and `wheel` groups. `wheel` group gives our user the sudo rights.
```
# useradd -m -g users -G wheel username
```
Newt we set password for your account:
```
# passwd username
```

# Installing basic packages
With this command we will instal l majority of the packages we'll need, not including desktop enviroment.
```
# pacman -S base-devel dosfstools grub efibootmgr micro networkmanager openssh os-prober sudo
```
I recomend to install **intel-ucode** or **amd-ucode** dependig which cpu you have. These are for hardware bug and security fixes.
If you want to use SSH, add: openssh  
If you did choose to install openssh, enable it: **systemctl enable sshd** 
Lets enable **NetworkManmager** so networking weill function when you reboot:
```
# systemctl enable NetworkManager
```

# Kernel and firmware
Next you need to install linux kernel. I will install two kernels `linux` and `linux-lts` just in case I cannot access the main linux kernel I have a backup to get into my system.
```
# pacman -S linux linux-headers linux-lts linux-lts-headers
```
Next you install linux firmware:
```
# pacman -S linux-firmware
```
# GPU
First you need to make sure what kiond of GPU you have.  
You can find that with: 
```
# lspci
```
This is what output foir me looks like:  
![Image](https://github.com/user-attachments/assets/15f1cafa-9b1c-4f2c-b7a4-6f7cceb40034)  
For that I can determine that I have an Intel gpu.  
For **Intel** and **AMD** GPU you need to install mesa.
```
# pacman -S mesa
```
For Nvidia GPU you need to install these:
```
# pacman -S nvidia nvidia-utils
```
And if you installed LTS kernel earlier you need:
```
# pacman -S nvidia-lts
```

# Generate kernel ramdisks
You generate the kernel ramdisk with `mkinitcpio`. For linux you need use `linux` and for linux-lts you need to use `linux-lts`. In my case I have both so the commands will look like this: 
```
# mkinitcpio -p linux
# mkinitcpio -p linux-lts
```
To set up your system language you need to edit `/etc/locale.gen`and uncomment `en_US.UTF-8 UTF-8`  or the language you need: 
```
# micro /etc/locale.gen
```
Next we generate the locales by running:
```
# locale-gen
```

# Setting up grub
Previously we mounted our boot partition to the `/mnt/boot/efi`. We needed to do that so setting upo grub is easier.  
To install grub we run: 
```
# grub-install --target=x86_64-efi --bootloader-id=grub_uefi --recheck
```
The next command will copy locale files for GRUB’s messaging to our boot directory:
```
# cp /usr/share/locale/en\@quot/LC_MESSAGES/grub.mo /boot/grub/locale/en.mo
```
Generate a config file for GRUB:
```
# grub-mkconfig -o /boot/grub/grub.cfg
```
# Wrapping up
Now we exit from tyhe chroot enviroment:
```
# exit
```
Next we unmount all partitions:
```
# umount -a
```
Lasty we reboot the system. Remember to remove the installation medium and then login into the new system with the root account.
```
# reboot
```
After rebooting you will see this screen and can login with the user you created:  
![Image](https://github.com/user-attachments/assets/86ee3e7f-5c26-49fe-bcf1-928e3b45228a)  

Now you can install what desktop enviroment you want.
