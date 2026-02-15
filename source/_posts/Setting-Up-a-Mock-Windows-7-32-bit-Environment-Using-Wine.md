---
title: Setting Up a Mock Windows 7 32-bit Environment Using Wine
date: 2026-01-19
categories:
  - DevOps, SysAdmin
tags:
  - Reference
---

Setting up a mock Windows 7 32-bit environment using Wine is a common way to run older Windows applications on Linux/macOS. Here's a step-by-step guide.

## Install Wine (if not already installed)

On Debian/Ubuntu:

```bash
sudo dpkg --add-architecture i386
sudo apt update
sudo apt install wine32 wine64 winetricks
```

On Fedora:

```bash
sudo dnf install wine
```

On macOS:

```bash
brew install --cask wine-stable
```

## Create a Fresh 32-bit Wine Prefix

A Wine prefix is like a virtual `C:` drive. By default, Wine uses `~/.wine`, but you can
create as many custom prefixes as you want.

```bash
export WINEPREFIX=~/wine-win7-32
WINEARCH=win32 winecfg
```

- `WINEARCH=win32` ensures a 32-bit prefix.
- `WINEPREFIX` sets the location (can be any folder name).
- On first run, `winecfg` initializes the prefix and brings up the configuration window.

## Set Windows Version to Windows 7

In the `winecfg` window that appears:

- Go to the **"Applications"** tab.
- Set the **"Windows Version"** dropdown to **Windows 7**.
- Click **Apply** and **OK**.

## Use Winetricks for Dependencies
Often, apps need extra libraries or fonts. Use **winetricks**:

```bash
export WINEPREFIX=~/wine-win7-32
winetricks
```

From the GUI you can install components (like .NET, Visual C++, etc).

## Use the Environment

To **run an installer** within this environment:

```bash
export WINEPREFIX=~/wine-win7-32
wine setup.exe
```
- Replace `setup.exe` with your installer.
- Always export `WINEPREFIX` before running anything for this prefix.

To launch cmd.exe:

```bash
export WINEPREFIX=~/wine-win7-32
wine cmd
```

or, explicitly:

```bash
export WINEPREFIX=~/wine-win7-32
wine 'C:\windows\system32\cmd.exe'
```

To launch Word 2007 (after installation):

```bash
export WINEPREFIX=~/wine-win7-32
wine 'C:\Program Files\Microsoft Office\Office12\WINWORD.EXE'
```
