---
title: Best dd options to write an ISO file to a USB flash drive
date: 2022-05-05
tags:
  - english
  - linux
  - software
  - manjaro
  - ubuntu
layout: layouts/post.njk
---

Every time I have to write an ISO file to a USB flash drive, I spend several minutes googling the right `dd` options to force sync writes, to use the right buffer size and to display the progress on the screen.

Thanks to my hobby of trying Linux distros and to re-install my desktop setup a couple of times a year, I repeat this process often enough, so here are my usual options for `dd`.

## Buffer size

Using a low buffer size will make this process to take forever. I have found that usually 4M is the right value: `bs=4M`

## Sync write

By default, `dd` makes use of the kernel buffer to write to the device. This is great if we are copying a small file but an ISO image is usually bigger than 1GB, so we want to use a value that allows `dd` to display the real data rate and ETA: `oflag=sync`

## Show the progress

By default, `dd` doesn't display the current progress status to the output. We can use `status=progress` to display it.

## Other options

- Remember to use `sudo` if you are writing to a device
- `if=` is the parameter for the input ISO file
- `of=` is the parameter for the ouput device

## Summary

Finally, these are all the options that usually work best for me:

```
sudo dd \
  if=image.iso \
  bs=4M \
  oflag=sync \
  of=/dev/DEVICE \
  status=progress
```

Example:

```
sudo dd \
  if=manjaro-mate-21.2.5-minimal-220314-linux515.iso \
  bs=4M \
  oflag=sync \
  of=/dev/sdb \
  status=progress
```

I hope this is helpful to more people and it's also a reminder for my future self.
