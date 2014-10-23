GNOME Shell Extension Installer
===============================

A bash script to install and search extensions from [extensions.gnome.org](https://extensions.gnome.org/)

## Help

```
Usage: gnome-shell-extension-installer EXTENSION_ID [ EXTENSION_ID... ] [ GNOME_VERSION ] [ OPTIONS ]
 
Options: 
  -s or --search "STRING"	Interactive search for extensions matching STRING. 
  --yes 			Skip all prompts (don't affect search mode). 
  -h or --help 			Print this message.
 
Usage examples: 
  # Install "Small Panel Icon" for GNOME Shell 3.12 answering yes to all questions: 
  gnome-shell-extension-installer 861 3.12 --yes
 
  # Search for extensions matching "User Themes": 
  gnome-shell-extension-installer -s "User Themes"
```

## Installation

### Manual installation

```
wget -O gnome-shell-extension-installer "https://github.com/ianbrunelli/gnome-shell-extension-installer/raw/master/gnome-shell-extension-installer"
chmod +x gnome-shell-extension-installer
mv gnome-shell-extension-installer /usr/bin/
```

### AUR (Arch Linux)

[This is the AUR page](https://aur.archlinux.org/packages/gnome-shell-extension-installer). You know what do do.
