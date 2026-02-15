---
title: Installing Fonts on Linux
date: 2026-02-15
categories:
- Data Science, Machine Learning, Multimedia Processing
tags:
- Reference
---

First, download your desired font. Fonts are usually TTF (`.ttf`), OTF (`.otf`). For illustration, let's say you downloaded `MyFont.ttf`.

## Install for your user only

Place the font file into `˜/.local/share/fonts`. If this folder does not exist, you can create it.

```sh
mkdir -p ˜/.local/share/fonts
cp MyFont.ttf ˜/.local/share/fonts/
```

## Install system-wide

Copy the font to `/usr/share/fonts` or `/usr/local/share/fonts`. You'll need root privileges:

```sh
sudo cp MyFont.ttf /usr/share/fonts/
```

## Update the font cache

After copying the font file, update the font cache so the system recognizes it:

```sh
fc-cache -fv
```

## Verify Installation

You can check if the font is installed by listing fonts:

```sh
fc-list
```
