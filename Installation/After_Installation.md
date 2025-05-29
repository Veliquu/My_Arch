If you are not able to use ethernet you firstly need to connect to the internet. We installet networkmanager previously so we can use `nmcli` to connect Wi-Fi.
```
# List nearby Wi-Fi networks:
$ nmcli device wifi list

# Connect to a Wi-Fi network:
$ nmcli device wifi connect SSID_or_BSSID password password
```
# AUR helper
FirsT i will install AUR helper. I will install `paru` but you can also install `yay`.  
To do that I need to install git:
```
$ sudo pacman -S git
```
Next you need to clone **Paru** repository:
```
$ git clone https://aur.archlinux.org/paru.git
```
This command will download the contents of the Paru GitHub repository in a local directory named paru.  
Change into the paru directory:
```
$ cd paru
```
Finally, build and install Paru AUR helper in Arch Linux using the following command:
```
$ makepkg -si
```
# Desktop enviroment
Now you should deside which desktop enviroment you want to install ( you can find all supported enviroments [here](https://wiki.archlinux.org/title/Desktop_environment).  
For example to get **GNOME** youi would install it like this:
```
$ sudo pacman -S gnome gdm
```
**GNOME** package install the desktop enviroment and **gdm** is GNOME display manager which manages graphical user logins.  
After enabling **gdm** 
```
$ systemctl enable gdm
```
You can restar your pc and you will be greeted in **gdm**.  
Now you have **GNOME** as you GUI.

## Qtile
I will personally install **qtile** which is tiling window manager. Tiling window managers differs from full GUI. Where something like **GNOME** you get basicly every graphical interface you would need, in tiling window managers you need to build everyhting yourself.  
So to install **Qtile** I will start installing **Qtile** package, **SDDM** as my display manager and **Alacritty** to be my terminal.
```
$ sudo pacman -S qtile sddm alacrityy
```
Ater installation I will need to enable **SDDM** so after reboot the display manager is in use.
```
$ systemctl enable sddm
```
So after rebooting I'm greeted with this screen.  
![Image](https://github.com/user-attachments/assets/fb3a2f88-98fe-4ec9-8736-ff2004870dae)  
