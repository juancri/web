---
title: My Linux Desktop setup
date: 2021-05-17
tags:
  - linux
  - software
  - english
layout: layouts/post.njk
---

## Introduction

I've been using Linux for more than 20 years now (23 if I recall correctly). It took me several years to make Linux my primary desktop driver, though. Here's a snapshot of my current setup, from the distro to the tools I use for software development.

## The distro

My first distro was [SuSE](https://en.wikipedia.org/wiki/SUSE_Linux) 5.1, thanks to a CD included in an edition of PC Magazine. I have to say that it blew my mind. The possibility to run an operating system other than Microsoft Windows was incredible. After a while, I switched to [Red Hat](https://en.wikipedia.org/wiki/Red_Hat_Linux), then [Debian](https://en.wikipedia.org/wiki/Debian), [Gentoo](https://en.wikipedia.org/wiki/Gentoo_Linux) and [Ubuntu](https://en.wikipedia.org/wiki/Ubuntu).

It was not easy to use Linux during the late 90s and early 2000s as my primary desktop environment. When laptops began to be a thing, simple tasks like connecting to a new wireless network was not easy on Linux. Ubuntu changed that and it was really thought as a friendly desktop experience. That was the point when I decided to fully move from Windows to Linux.

After Ubuntu, I jumped to [Arch](https://en.wikipedia.org/wiki/Arch_Linux). I have tried multiple derivates but finally have sticked to *vanilla* Arch for the last 8 years or so.

## The desktop

![Screenshot](/img/linux-screenshot.png)

Until not long ago, my desktop environment was [MATE](https://en.wikipedia.org/wiki/MATE_(software)) (pronounced *ma-te* as the [drink](https://en.wikipedia.org/wiki/Mate_(drink)) in Spanish). It might be that I'm just old and I miss the early 2000s, but MATE + [compiz](https://en.wikipedia.org/wiki/Compiz) was a really good mix.

A few months ago, however, I switched to bspwm. [Tiling window managers](https://en.wikipedia.org/wiki/Tiling_window_manager) can be scary at first and indeed have a steep learning curve but, in the long therm, they provide a very good and personalized experience.

Here's a list of software I install as soon as I have a new computer or installation:

## Software

### Desktop environment

- [bspwm](https://github.com/baskerville/bspwm): The window manager
- [compton](https://github.com/chjj/compton): The compositor (now I realize that I should migrate to [picom](https://github.com/yshui/picom))
- [synapse](https://en.wikipedia.org/wiki/Synapse_(software)): The launcher
- [sxhkd](https://github.com/baskerville/sxhkd): The hotkey daemon
- [polybar](https://github.com/polybar/polybar): The status bar
- [flameshot](https://github.com/flameshot-org/flameshot): The screenshot utility
- [feh](https://feh.finalrewind.org/): The wallpaper setter

### Desktop software

- [Google Chrome](https://en.wikipedia.org/wiki/Google_Chrome)
- [Deluge](https://en.wikipedia.org/wiki/Deluge_(software))
- [Pinta](https://en.wikipedia.org/wiki/Pinta_(software))
- [mpv](https://mpv.io/)

### Development software and tools

- [Visual Studio Code](https://en.wikipedia.org/wiki/Visual_Studio_Code)
- [git](https://en.wikipedia.org/wiki/Git)
- [HTTPie](https://httpie.io/)
- [bash-git-prompt](https://github.com/magicmonty/bash-git-prompt)
- [meld](https://meldmerge.org/)

### Other tools

- [pikaur](https://github.com/actionless/pikaur)
- [pandoc](https://en.wikipedia.org/wiki/Pandoc)
- [midnight commander](https://en.wikipedia.org/wiki/Midnight_Commander)
- [eom](https://github.com/mate-desktop/eom)
- [neovim](https://neovim.io/)

## Installation

Since I have several computers and it's a common process to install or re-install, I decided to create my own install scripts. Usually, Arch Linux is installed manually following the [installation guide](https://wiki.archlinux.org/title/installation_guide), but I value my time so I try to automate as much as I can. I created this repository: [juancri/my-arch-setup](https://github.com/juancri/my-arch-setup) that includes scripts to install all the software previously listed and apply some tweaks and customizations.

Feel free to explore this repo and use it as an inspiration to customize your own setup `;)`
