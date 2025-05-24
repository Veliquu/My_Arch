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

