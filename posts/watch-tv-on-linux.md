---
title: Watch TV on Linux
date: 2022-12-08
tags:
  - linux
  - manjaro
  - tv
  - software
  - english
layout: layouts/post.njk
---

## Introduction

The goal is to be able to watch [over the air](https://en.wikipedia.org/wiki/Over-the-air) digital TV on my laptop or Desktop.

My distro is Manjaro Linux and, since I'm in Austin, Texas, the local digital TV standard is [ATSC](https://en.wikipedia.org/wiki/ATSC_standards).

## Hardware

In order to catch ATSC channels, I got a [Hauppauge WinTV HVR-950Q USB device](https://www.linuxtv.org/wiki/index.php/Hauppauge_WinTV-HVR-950Q). This device supports analog TV as well but I'm using it just for its digital TV tuner.

## Packages

We need these packages in order to make this work:

- `w_scan2` is needed to scan for channels
- We will use VLC as the video player
- `aribb24` is needed to decode [TS streams](https://en.wikipedia.org/wiki/MPEG_transport_stream) (see [this bug report](https://bugs.archlinux.org/task/76535) for details)

***Note:*** Since `w_scan2` is available only as an [AUR package](https://aur.archlinux.org/packages/w_scan2), I'm using [pikaur](https://github.com/actionless/pikaur) and not pacman to install these packages. Alternatively, you can [build AUR packages manually](https://wiki.archlinux.org/title/Arch_User_Repository#Installing_and_upgrading_packages) or use [another AUR helper](https://wiki.archlinux.org/title/AUR_helpers).

```bash
pikaur -S \
  w_scan2 \
  vlc \
  aribb24
```

## Scan for channels

Now we can scan for channels by using the following command:

```bash
w_scan2 -c US -L > tv.xspf
```

About the parameters:

- `-c US`: Specifies the country in order to identify the right frequencies and modulation to scan
- `-L`: Uses the VLC format for the output

## VLC

The previous command creates a file named `tv.xspf` that we can now open with VLC to watch:

![VLC](/img/vlc-air-tv.png)

Enjoy!
