# ISO image and bootable USB
First get ISO image from [Arch Downloads](https://archlinux.org/download/).  
Next create bootable USB with [Rufus](https://rufus.ie/en/) or [Balena Etcher](https://etcher.balena.io/) (or any other software to create bootable USB).  

Boot from your PC from the USB you created. In my case when powering on your PC press **F12** until you get to the boot menu. After you boot from you USB you will se scrren similar to this:  
![Image](https://github.com/user-attachments/assets/7ab13105-7adc-43e4-b5b7-483ae93e1804)  

Choose the installation youi want to make. You will get to screen like this:  
![Image](https://github.com/user-attachments/assets/afc25cc1-8878-4d00-9321-4fb0f67c2ae8)  
Now you can start to prepare the installation.
  
# Set the console keyboard layout
The default console keymap is **US**. Available layouts can be listed with:  
```
# localectl list-keymaps
```
To set the keyboard layout, for example finnish, you need to pass its name to loadkeys:  
```
# loadkeys fi
```

# Connect to the internet
To ensure your **network interface** is listed and enabled use ip-link:  
```
# ip link
```
For wireless and WWAN, make sure the card is not blocked with rfkill.  
![Image](https://github.com/user-attachments/assets/713841a1-dfcb-4a11-8202-6a183c9853ab)  
If your network card is soft-blocked, you can unblock it with:  
```
# rfkill unblock wlan
```
Connect to the internet
- Ethernet—plug in the cable.
- Wi-Fi—authenticate to the wireless network using `iwctl`.

<details>
<summary><h2>Connect with iwctl</h2></summary>
<br>
To get an interactive prompt do:
  
      # iwctl

The interactive prompt is then displayed with a prefix of `[iwd]#`.   
To connect Wi-Fi you need to know your wireless device name that you can get with: 
```
[iwd]# device list
```
If the device or its adapter is turned off, turn it on: 
```
[iwd]# device name set-property Powered on
```
```
[iwd]# adapter adapter set-property Powered on
```

Onward I will use my device name `wlan0` change it to what your device name is.  
To initiate scan for networks (command will not output anything):
```
[iwd]# station wlan0 scan
```
Then to list available networks: 
```
[iwd]# station wlan0 get-networks
```
To connect to a network (replace SSID with the networks name):
```
[iwd]# station wlan0 connect SSID
```
Finally exit with: 
```
[iwd]# exit
```
</details>

To verify you are connected, ping:
```
# ping archlinux.org
```

# Update the system clock
When you have internet connection **systemd-timesyncd** will automaticvally sync default time.  
Use **timedatectl** to ensure the system clock is synchronized: 
```
# timedatectl
```
In my case my time zone was wrong and needed to change it to Europe/Helsinki. You can set the time zone:
```
# timedatectl set-timezone Europe/Helsinki
```
You can also manually set your time:
```
# timedatectl set-time "yyyy-MM-dd hh:mm:ss"
```

# Partition the disks
When recognized by the live system, disks are assigned to a **block device** such as `/dev/sda`, `/dev/nvme0n1` or `/dev/mmcblk0`. To identify these devices, use lsblk or fdisk.
```
# fdisk -l
```
For example this is what mine shows:  
![Image](https://github.com/user-attachments/assets/70b48c7b-8488-478c-bbec-d3082d04b70e)  
Here we can see USB where I booted `/dev/sda1` and my PCs SSD `/dev/nvme0n1`.

When partitioning your disk, you will need atleast two partitions root `/` and boot `/boot`. In this example I will also make 3rd partition for `SWAP`.  
To start the partitioning use (change /dev/nvme0n1 to your disk):
```
# fdisk /dev/nvme0n1
```
When you are in fdisk the command line would look like this:
```
Command (m for help):
```
Using `p` for the command you can check the disk and with `g` you can delete all partitions if you have any.  
To create new partition use `n` command.  
For the first partition (/boot) you can use default values for the first 2 options (just press ENTER) and set the last one to 1G.
```
Partition number (1-128, default 1): 1
First sector (2048-1953525134, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-1953525134, default 1953523711): +1G
```
If the partition have vfat signature just remove it.
```
Partition #1 contains a vfat signature.

Do you want to remove the signature? [Y]es/[N]o: Y
The signature will be removed by a write command.
```
Mark the partition as an EFI system partition:
```
Command (m for help): t
Selected partition 1
Partition type or alias (type L to list all): 1
Changed type of partition 'Linux filesystem' to 'EFI System'.
```
Next I create the `SWAP` partition. I will make 16G `SWAP` partition since I have 2TB ssd.
```
Command (m for help): n
Partition number (2-128, default 2): 
First sector (2099200-1953525134, default 2099200): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2099200-1953525134, default 1953523711): +16G
```
Mark the partition as an Linux Swap partition, setting the partition type as **19**:
```
Command (m for help): t
Partition number (1,2, default 2): 2
Partition type or alias (type L to list all): 19
```
Lastly we create the root partition.
```
Command (m for help): n
Partition number (3-128, default 3): 3
First sector (10487808-1953525134, default 10487808):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (10487808-1953525134, default 1953523711):
```
You can just press enter to all the sectors since we are yousing rest of the disk for this partition.  
Mark the partition as an Linux Root (x86-64) partition, setting the partition type as **23**:
```
Command (m for help): t
Partition number (1,2, default 2): 3
Partition type or alias (type L to list all): 23
```
  
To save the partition layout use command `w`.
```
Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

# Format the partitions
Once the partitions have been created, each newly created partition must be formatted with an appropriate file system.  
You can see the partitions with `fdsik -l`. 
I will be creating **Ext4** file system so to format that I run: 
```
# mkfs.ext4 /dev/nvme0n1p3
```
If you created a partition for swap, initialize it with **mkswap**:
```
# mkswap /dev/nvme0n1p2
```
And for the boot I created EFI system partition, format it to FAT32 using mkfs.fat.
```
# mkfs.fat -F 32 /dev/nvme0n1p1
````

# Moun the file systems
Mount root volume to `/mnt`
```
# mount /dev/nvme0n1p3 /mnt
```
For UEFI systems, mount the EFI system partition:
```
# mount --mkdir /dev/nvme0n1p1 /mnt/boot
```
If you created a swap volume, enable it with swapon:
```
# swapon /dev/nvme0n1p2
```

# [Installation]()
