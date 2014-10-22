GNOME Shell Extension Installer
===============================

A bash script to install extensions from extensions.gnome.org

# Help

```
Usage: gnome-shell-extension-installer EXTENSION_ID [ EXTENSION_ID... ] [ GNOME_VERSION ] [ OPTIONS ]
 
Options: 
  --yes 		Skip all prompts (be careful on using this!). 
  -h or --help 		Print this message.
 
Usage example: 
  gnome-shell-extension-installer 861 3.14 --yes 
  # Installs "Small Panel Icon" for GNOME Shell 3.14 answering yes to all questions.
```

# Installation

## Manual installation

```
wget -O gnome-shell-extensions-installer "https://github.com/ianbrunelli/gnome-shell-extension-installer/raw/master/gnome-shell-extension-installer"
chmod +x gnome-shell-extensions-installer
mv gnome-shell-extensions-installer /usr/bin/
```

## AUR (Arch Linux)

[This is the package](https://aur.archlinux.org/packages/gnome-shell-extensions-installer). You know what do do.
