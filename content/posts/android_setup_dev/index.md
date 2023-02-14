---
title: "Setup old android tablet for development"
date: 2022-12-14T22:51:07+02:00
draft: true
toc:
    enable: true
    auto: false
tags: [android, termux, linux, development]
categories: [Development]
featuredImagePreview: "featured-image.webp"
featuredImage: "featured-image.webp"
---
{{< image src="tablet.jpg" alt="anlinux-fedora" position="center" style="border-radius: 8px;" >}}
## Getting Started
Before Starting, it's a good idea to install a Custom ROM for your old device just to get the best out of it.
Head to [XDA](https://www.xda-developers.com/) and search for your device.
But why you might ask ?
- The installed custom ROM would bring you newer Android version + new kernel version. (not always)
- Root access (more control in general).

In my case, I am using [SM-T585](https://www.gsmarena.com/samsung_galaxy_tab_a_10_1_(2016)-8090.php) which came with Android `8.1` and later on I upgraded to `9`.
## Requirements
Make sure you have at least 4gb(more than enough) of free space to install all the required apps :
- [Termux](https://github.com/termux/termux-app) : used as a terminal emulator.
- [AnLinux](https://github.com/EXALAB/AnLinux-App) : to install a linux distro without root access.

Note : AnLinux is just an option for those who don't want to navigate the github repo (easy mode).
## Choosing a distro
Open `AnLinux` and choose your favourite distro, I recommend using `fedora` since it has the most up-to-date packages.
Copy the install command and open `termux` and paste.
When the installation is done you can start fedora by `./start-fedora.sh` 

{{< image src="fedora-start.png" alt="anlinux-fedora" position="center" style="border-radius: 8px;" >}}

## Setup The Environment
If you reached this point, you know what to do next :D, but I am going to continue as a reference for future me.

1. Update the system : `dnf upgrade`.
2. Install essential packages : `dnf install wget git unzip gcc g++ python3 pip net-tools openssh psmisc rsync fuse-sshfs`
3. Install Workspace packages : `dnf install nvim tmux zsh`
4. Change the default shell to zsh.
5. Add aliases.
6. Have fun hacking :D

## Extras
### Termux Customizations 
- Download this [addon](https://f-droid.org/packages/com.termux.styling/)
- Change the theme and font by hold pressing -> `style` -> `choose color/ choose font` 

For me, I am using `Source Code Pro` font with `Wild Cherry` color.
### System Hack
Since I use `vim`, I always swap `ESC` with `CAPS_LOCK`, but on Android I couldn't swap the physical keyboard button so I changed the `CAPS_LOCK` to act like `ESC` lol.

Here is how I did it:
- Open the normal termux session.
- Install `vim` or any text editor.
- Mount the system to Write : `mount -o remount,rw /system`
- Edit `/system/usr/keylayout/Generic.kl` and search for `CAPS_LOCK` and change it to `ESCAPE`
- Mount the system to Read only : `mount -o remount,ro /system` 

### Some useful aliases
I have some aliases for easier navigation and nice overall experience.
```bash
alias v="nvim"
alias install="dnf install"
alias neo="neofetch"
alias mkd="mkdir -p"
alias vc="nvim ~/.config/nvim/init.lua"
alias python="python3"
```
### Connect to the network using SSH
I am using SSH to connect to my main machine on the network : `ssh none@192.168.1.6`, and `scp` to upload/download content.
```bash
# note the IP might change
down () {
    scp -r none@192.168.1.6:$1 $2
}
up () {
    scp -r $1 none@192.168.1.6:$2
}
```
