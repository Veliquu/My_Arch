# Arch Linux Pre-Installation Guide

This guide walks you through preparing your system for Arch Linux installation, from creating bootable media to setting up partitions and mounting filesystems.

## Prerequisites

Before starting, ensure you have:
- A USB drive (4GB minimum, 8GB recommended)
- Stable internet connection
- Basic command-line familiarity
- Your computer's boot menu key (commonly **F12**, F2, F8, or DEL)

---

# ISO image and bootable USB

## Download the ISO
First get ISO image from [Arch Downloads](https://archlinux.org/download/).

## Create Bootable USB
Next create bootable USB with [Rufus](https://rufus.ie/en/) or [Balena Etcher](https://etcher.balena.io/) (or any other software to create bootable USB).

**Using Rufus (Windows):**
1. Insert your USB drive
2. Open Rufus and select your USB device
3. Click "SELECT" and choose the Arch Linux ISO
4. Leave other settings as default
5. Click "START" and wait for completion

**Using Balena Etcher (Cross-platform):**
1. Download and install Balena Etcher
2. Insert USB drive
3. Select the Arch Linux ISO file
4. Select your USB drive
5. Click "Flash!" and wait for completion

## Boot from USB
Boot from your PC from the USB you created. In most cases when powering on your PC press **F12** until you get to the boot menu. After you boot from your USB you will see screen similar to this:

![Boot Menu](https://github.com/user-attachments/assets/7ab13105-7adc-43e4-b5b7-483ae93e1804)

Choose the installation you want to make. You will get to screen like this:

![Arch Linux Command Prompt](https://github.com/user-attachments/assets/afc25cc1-8878-4d00-9321-4fb0f67c2ae8)

Now you can start to prepare the installation. You should see a command prompt: `root@archiso ~ #`

---

# Set the console keyboard layout

The default console keymap is **US**. If you need a different layout, you can change it.

## List Available Layouts
Available layouts can be listed with:
```
# localectl list-keymaps
```

## Set Your Keyboard Layout
To set the keyboard layout, for example Finnish, you need to pass its name to loadkeys:
```
# loadkeys fi
```

**Common keyboard layouts:**
- `us` - US English (default)
- `uk` - UK English  
- `de` - German
- `fr` - French
- `es` - Spanish
- `fi` - Finnish
- `se` - Swedish
- `no` - Norwegian

---

# Connect to the internet

A working internet connection is essential for downloading packages during installation.

## Check Network Interface
To ensure your **network interface** is listed and enabled use ip-link:
```
# ip link
```

## Wireless Setup
For wireless and WWAN, make sure the card is not blocked with rfkill.

![Network Interface Status](https://github.com/user-attachments/assets/713841a1-dfcb-4a11-8202-6a183c9853ab)

If your network card is soft-blocked, you can unblock it with:
```
# rfkill unblock wlan
```

## Connection Methods
Connect to the internet using one of these methods:
- **Ethernet** — plug in the cable (automatic connection)
- **Wi-Fi** — authenticate to the wireless network using `iwctl`

<details>
<summary><h2>Connect with iwctl</h2></summary>
<br>

### Enter iwctl Interactive Mode
To get an interactive prompt do:
```
# iwctl
```

The interactive prompt is then displayed with a prefix of `[iwd]#`.

### List Wireless Devices
To connect Wi-Fi you need to know your wireless device name that you can get with:
```
[iwd]# device list
```

### Power On Device (if needed)
If the device or its adapter is turned off, turn it on:
```
[iwd]# device name set-property Powered on
```
```
[iwd]# adapter adapter set-property Powered on
```

### Scan and Connect
From here onward I will use device name `wlan0` - change it to match your device name.

To initiate scan for networks (command will not output anything):
```
[iwd]# station wlan0 scan
```

Then to list available networks:
```
[iwd]# station wlan0 get-networks
```

To connect to a network (replace SSID with the network's name):
```
[iwd]# station wlan0 connect SSID
```

### Exit iwctl
Finally exit with:
```
[iwd]# exit
```

</details>

## Verify Connection
To verify you are connected, ping:
```
# ping archlinux.org
```

If you get replies, your internet connection is working properly.

---

# Update the system clock

When you have internet connection **systemd-timesyncd** will automatically sync the default time.

## Verify System Clock
Use **timedatectl** to ensure the system clock is synchronized:
```
# timedatectl
```

## Set Timezone (if needed)
In my case my time zone was wrong and needed to change it to Europe/Helsinki. You can set the time zone:
```
# timedatectl set-timezone Europe/Helsinki
```

**Common timezone examples:**
- `Europe/London` - UK
- `America/New_York` - US Eastern
- `America/Los_Angeles` - US Pacific  
- `Asia/Tokyo` - Japan
- `Australia/Sydney` - Australia

## Manual Time Setting (if necessary)
You can also manually set your time:
```
# timedatectl set-time "yyyy-MM-dd hh:mm:ss"
```

Example:
```
# timedatectl set-time "2024-12-25 14:30:00"
```

---

# Partition the disks

When recognized by the live system, disks are assigned to a **block device** such as `/dev/sda`, `/dev/nvme0n1` or `/dev/mmcblk0`.

## Identify Storage Devices
To identify these devices, use lsblk or fdisk:
```
# fdisk -l
```

For example this is what mine shows:

![Disk Layout](https://github.com/user-attachments/assets/70b48c7b-8488-478c-bbec-d3082d04b70e)

Here we can see USB where I booted `/dev/sda1` and my PC's SSD `/dev/nvme0n1`.

## Partition Requirements
When partitioning your disk, you will need at least two partitions:
- **Root partition** (`/`) - Main system partition
- **Boot partition** (`/boot`) - EFI system partition

In this example I will also make a 3rd partition for **SWAP** (optional but recommended).

## Start Partitioning Process
To start the partitioning use (change `/dev/nvme0n1` to your disk):
```
# fdisk /dev/nvme0n1
```

When you are in fdisk the command line would look like this:
```
Command (m for help):
```

## Fdisk Commands Reference
- `p` - Print current partition table
- `g` - Create new GPT partition table (deletes all existing partitions)
- `n` - Create new partition
- `t` - Change partition type
- `d` - Delete partition
- `w` - Write changes and exit
- `q` - Quit without saving
- `m` - Show help menu

## Step-by-Step Partition Creation

### 1. Create EFI Boot Partition (1GB)
For the first partition (`/boot`) you can use default values for the first 2 options (just press ENTER) and set the last one to 1G:
```
Command (m for help): n
Partition number (1-128, default 1): 1
First sector (2048-1953525134, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-1953525134, default 1953523711): +1G
```

If the partition has vfat signature, remove it:
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

### 2. Create SWAP Partition (Optional)
Next I create the **SWAP** partition. I will make 16G **SWAP** partition since I have 64G RAM:
```
Command (m for help): n
Partition number (2-128, default 2): 
First sector (2099200-1953525134, default 2099200): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2099200-1953525134, default 1953523711): +16G
```

Mark the partition as a Linux Swap partition, setting the partition type as **19**:
```
Command (m for help): t
Partition number (1,2, default 2): 2
Partition type or alias (type L to list all): 19
```

**SWAP size recommendations:**
- 4GB RAM or less: Equal to RAM size
- 4-16GB RAM: Half of RAM size
- 16GB+ RAM: 8GB or more (depending on usage)

### 3. Create Root Partition (Remaining Space)
Lastly we create the root partition:
```
Command (m for help): n
Partition number (3-128, default 3): 3
First sector (10487808-1953525134, default 10487808):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (10487808-1953525134, default 1953523711):
```

You can just press enter to all the sectors since we are using rest of the disk for this partition.

Mark the partition as a Linux Root (x86-64) partition, setting the partition type as **23**:
```
Command (m for help): t
Partition number (1,2,3, default 3): 3
Partition type or alias (type L to list all): 23
```

### 4. Save Partition Layout
To save the partition layout use command `w`:
```
Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

---

# Format the partitions

Once the partitions have been created, each newly created partition must be formatted with an appropriate file system.

## Check Created Partitions
You can see the partitions with `fdisk -l` to verify they were created correctly.

## Format Root Partition (ext4)
I will be creating **Ext4** file system for the root partition, so to format that I run:
```
# mkfs.ext4 /dev/nvme0n1p3
```

## Format EFI Boot Partition (FAT32)
For the boot partition, I created EFI system partition, format it to FAT32 using mkfs.fat:
```
# mkfs.fat -F 32 /dev/nvme0n1p1
```

## Initialize Swap Partition (if created)
If you created a partition for swap, initialize it with **mkswap**:
```
# mkswap /dev/nvme0n1p2
```

**Note:** Replace `/dev/nvme0n1p1`, `/dev/nvme0n1p2`, `/dev/nvme0n1p3` with your actual partition names. These correspond to:
- `p1` - EFI boot partition
- `p2` - Swap partition  
- `p3` - Root partition

---

# Mount the file systems

Now we need to mount the formatted partitions to prepare for installation.

## Mount Root Partition
Mount root volume to `/mnt`:
```
# mount /dev/nvme0n1p3 /mnt
```

## Mount EFI Boot Partition
For UEFI systems, mount the EFI system partition:
```
# mount --mkdir /dev/nvme0n1p1 /mnt/boot
```

The `--mkdir` flag creates the `/mnt/boot` directory automatically.

## Enable Swap (if created)
If you created a swap volume, enable it with swapon:
```
# swapon /dev/nvme0n1p2
```

## Verify Mounts
Check that everything is mounted correctly:
```
# lsblk
```

You should see your partitions mounted at `/mnt` and `/mnt/boot`, with swap showing as `[SWAP]`.

---

# Pre-Installation Verification Checklist

Before proceeding to the actual installation, verify that all steps are completed:

- [ ] **Internet connection** is working (`ping archlinux.org`)
- [ ] **Keyboard layout** is set correctly (if not using US)
- [ ] **System clock** is synchronized (`timedatectl`)
- [ ] **Partitions** are created and formatted properly
- [ ] **Root partition** is mounted at `/mnt`
- [ ] **EFI partition** is mounted at `/mnt/boot`
- [ ] **Swap** is enabled (if applicable)
- [ ] **All mounts** verified with `lsblk`

## Troubleshooting Common Issues

### WiFi Connection Problems
- Ensure wireless card is not hardware-blocked
- Check network name for special characters
- Verify password is entered correctly

### Partition Creation Errors
- Ensure you're using the correct disk device name
- Check if disk has existing partitions that need clearing with `g` command
- Verify disk has sufficient space for all partitions

### Mount Errors
- Ensure partitions are properly formatted before mounting
- Check that device names match your actual partitions
- Verify no existing mounts conflict with `umount` if needed


---

# Next Steps: [Installation](https://github.com/Veliquu/My_Arch/blob/main/Installation/Installation.md) 
