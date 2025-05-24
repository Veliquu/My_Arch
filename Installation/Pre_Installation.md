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


