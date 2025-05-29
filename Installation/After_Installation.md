# Comprehensive Arch Linux Post-Installation Guide

This guide covers essential post-installation steps for Arch Linux, including network configuration, AUR helper installation, and desktop environment setup.

## Network Configuration

If you cannot use ethernet, you'll need to connect to Wi-Fi using NetworkManager, which should have been installed during the base installation.

### Connecting to Wi-Fi with nmcli

NetworkManager Command Line Interface (`nmcli`) is a powerful tool for managing network connections.

```bash
# List nearby Wi-Fi networks
$ nmcli dev wifi list
```

**Command breakdown:**
- `nmcli` - NetworkManager command line interface
- `dev` (or `device`) - Manages network devices
- `wifi` - Specifies Wi-Fi operations
- `list` - Shows available wireless networks with details like SSID, signal strength, and security type

```bash
# Connect to a Wi-Fi network
$ nmcli dev wifi connect "NETWORK_NAME" password "YOUR_PASSWORD"
```

**Parameters explained:**
- `connect` - Establishes connection to specified network
- `"NETWORK_NAME"` - The SSID (network name) - use quotes if it contains spaces
- `password` - Keyword indicating the following string is the network password
- `"YOUR_PASSWORD"` - Your Wi-Fi password - use quotes to handle special characters

**Alternative connection methods:**
```bash
# Connect using BSSID (MAC address) instead of SSID
$ nmcli dev wifi connect AA:BB:CC:DD:EE:FF password "password"

# Connect to open network (no password)
$ nmcli dev wifi connect "NETWORK_NAME"

# View saved connections
$ nmcli connection show

# Connect to previously saved network
$ nmcli connection up "CONNECTION_NAME"
```

## AUR Helper Installation

The Arch User Repository (AUR) contains community-maintained packages. AUR helpers like `paru` or `yay` simplify installing these packages by automating the build process.

### Installing Git

First, install Git, which is required for cloning AUR repositories:

```bash
$ sudo pacman -S git
```

**Command breakdown:**
- `sudo` - Execute command with root privileges
- `pacman` - Arch Linux package manager
- `-S` - Sync flag, installs packages from repositories
- `git` - Version control system needed for AUR operations

### Installing Paru AUR Helper

Paru is a modern AUR helper written in Rust, offering features like:
- Colored output and detailed package information
- Advanced dependency resolution
- Support for viewing package files before installation
- News display from Arch Linux website

```bash
# Clone the Paru repository from AUR
$ git clone https://aur.archlinux.org/paru.git
```

**Command breakdown:**
- `git clone` - Creates a local copy of a remote repository
- `https://aur.archlinux.org/paru.git` - Official AUR repository URL for Paru

```bash
# Navigate to the cloned directory
$ cd paru
```

```bash
# Build and install Paru
$ makepkg -si
```

**Parameters explained:**
- `makepkg` - Arch Linux tool for building packages from source
- `-s` (--syncdeps) - Automatically install missing dependencies using pacman
- `-i` (--install) - Install the package after successful build

**Additional useful makepkg flags:**
- `-c` (--clean) - Clean up temporary files after build
- `-r` (--rmdeps) - Remove dependencies after installation if they weren't previously installed
- `-f` (--force) - Overwrite existing packages

### Using Paru

After installation, you can use Paru similarly to pacman:

```bash
# Install AUR package
$ paru -S package-name

# Search for packages (both official repos and AUR)
$ paru -Ss search-term

# Update all packages including AUR
$ paru -Syu

# Show package information
$ paru -Si package-name
```

## Desktop Environment Installation

Choose a desktop environment based on your preferences. You can find all supported environments in the [Arch Wiki](https://wiki.archlinux.org/title/Desktop_environment).

### GNOME Desktop Environment

GNOME provides a complete, user-friendly desktop experience with modern design and extensive functionality.

```bash
$ sudo pacman -S gnome gdm
```

**Package explanations:**
- `gnome` - Meta-package that installs the complete GNOME desktop environment, including:
  - GNOME Shell (desktop interface)
  - GNOME applications (Files, Settings, Terminal, etc.)
  - System services and libraries
- `gdm` - GNOME Display Manager, handles:
  - Graphical login screen
  - User session management
  - X11/Wayland session startup

**Alternative GNOME installation options:**
```bash
# Minimal GNOME installation (just the shell)
$ sudo pacman -S gnome-shell

# GNOME with extra applications
$ sudo pacman -S gnome gnome-extra
```

### Enabling the Display Manager

```bash
$ sudo systemctl enable gdm
```

**Command breakdown:**
- `systemctl` - Control systemd services and units
- `enable` - Configure service to start automatically at boot
- `gdm` - The GNOME Display Manager service

**Additional systemctl commands:**
```bash
# Start service immediately
$ sudo systemctl start gdm

# Check service status
$ sudo systemctl status gdm

# Stop service
$ sudo systemctl stop gdm

# Disable automatic startup
$ sudo systemctl disable gdm
```

After enabling GDM, restart your system:
```bash
$ sudo reboot
```

You'll be greeted with the GNOME login screen, and after logging in, you'll have access to the full GNOME desktop environment.

## Qtile Window Manager

Qtile is a tiling window manager written in Python. Unlike full desktop environments, tiling window managers require manual configuration but offer greater customization and efficiency.

### Key Differences: Desktop Environment vs Window Manager

**Desktop Environment (like GNOME):**
- Complete graphical interface out-of-the-box
- Integrated applications and services
- Consistent look and feel
- Beginner-friendly

**Window Manager (like Qtile):**
- Minimal base installation
- Requires manual configuration
- Highly customizable
- More efficient resource usage
- Steeper learning curve

### Installing Qtile Components

```bash
$ sudo pacman -S qtile sddm alacritty
```

**Package explanations:**
- `qtile` - Python-based tiling window manager with features like:
  - Automatic window tiling
  - Multiple workspaces
  - Configurable key bindings
  - Extensible with Python scripts
- `sddm` - Simple Desktop Display Manager, provides:
  - Lightweight login interface
  - Session selection
  - User authentication
  - Theme support
- `alacritty` - GPU-accelerated terminal emulator offering:
  - High performance
  - Cross-platform compatibility
  - Extensive configuration options

### Enabling SDDM

```bash
$ sudo systemctl enable sddm
```

This configures SDDM to start automatically at boot, providing the login interface for your Qtile session.

### Post-Installation Qtile Setup

After installation and reboot, you'll see the SDDM login screen. However, Qtile requires additional configuration:

### Basic Qtile Key Bindings (Default)

- `Super + Return` - Open terminal
- `Super + r` - Application launcher
- `Super + Tab` - Switch between windows
- `Super + j/k` - Move focus between windows
- `Super + h/l` - Resize windows
- `Super + Shift + c` - Close window
- `Super + Ctrl + r` - Restart Qtile

### Update Your System

Always keep your system updated:

```bash
# Update package database and installed packages
$ sudo pacman -Syu

# If using Paru for AUR packages
$ paru -Syu
```
Still in progress. Next I will keep building up my **Qtile** enviroment.
