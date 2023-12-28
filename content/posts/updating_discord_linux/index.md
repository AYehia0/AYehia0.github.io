---
title: "How to update discord on archlinux ?"
date: 2023-11-24T23:18:52+02:00
draft: true
toc:
  enable: true
  auto: false
share:
  enable: true

authors: []
description: "How to manually update discord on archlinux ?"

tags: []
categories: [linux, archlinux, discord]
series: []

hiddenFromHomePage: false
hiddenFromSearch: false

featuredImagePreview: "featured-image.jpg"
---
I hate discord updates!, they do lots of updates which prevents discord desktop from launching.
Bad for me that I use Archlinux, as discord ships thier updates as `.deb`.
<!--more-->

{{< image src="discord_update_screen.png" caption="When discord needs to be updated" >}}

So how to manually update discord without waiting for the package maintainer to update it for you ?

## Cheating way
This method allows you to trick the Discord into thinking it's been updated. First, check for the path of the discord executable
```bash
file $(which discord) 
```
Then edit the `resources/build_info.json` in that path.
```json
{
    "releaseChannel": "stable",
    "version": "0.0.xx" // change to the latest version
}
```
## The proper way
This method involves using the [Arch Build System](https://wiki.archlinux.org/title/Arch_build_system)

1. Download the `asp` through `yay` : `yay -S asp`
2. Export the `PKGBUILD` file from discord : `asp export discord`
3. Edit the `pkgver=0.0.xx` in the `PKGBUILD` with the current version.
4. Build discord : `makepkg -si`

