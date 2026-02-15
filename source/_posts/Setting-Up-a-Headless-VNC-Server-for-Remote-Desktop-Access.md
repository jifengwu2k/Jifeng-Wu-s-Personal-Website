---
title: Setting Up a Headless VNC Server for Remote Desktop Access
date: 2025-08-21
categories:
  - DevOps, SysAdmin
tags:
  - Reference
---

While SSH is a staple tool and almost universally understood among Linux users, setting up **VNC for remote desktop access** - especially headlessly or with virtual framebuffers - remains mysterious to many, and for good reasons:

- Requires understanding of X11 and graphical sessions (DISPLAY variables, X servers, etc.)
- Involves properly launching graphical sessions

However, for many professionals, researchers, and IT administrators, secure remote desktop access is not just a convenience - it's an inelastic requirement. Unlike other features you can work around or delay, robust GUI access to a remote host is sometimes the only way to get mission-critical work done. For example:

- **Work-from-home** mean you need to access graphical applications (like IDEs, simulators, and visualization tools) installed on remote lab or office machines.
- **Remote servers without physical monitors** demand a headless-only setup - you can't just "plug in a monitor and keyboard" to start a desktop session.

In these scenarios, there is **no substitute** - the need is inelastic. You must have a reliable, secure way to run and manage desktops remotely. Frustratingly, most available documentation is scattered, outdated, or omits crucial details.

That's why this guide exists: to provide you with a **direct, practical roadmap for setting up a robust remote desktop workflow**, no matter where you work.

## The Three Components of Headless VNC

Three key components work together to present a remote desktop via VNC:

### Xvfb (X Virtual Framebuffer)

Xvfb is a "headless" X server that implements the X11 protocol and acts like a display server, but renders everything to memory instead of a physical monitor.

### x11vnc

This utility acts as a bridge, allowing VNC clients to view and interact with an existing X server session. It:

- Connects to the X server's framebuffer (in this case, Xvfb's in-memory screen)
- Reads pixels from the X server, and writes mouse/keyboard events
- Presents those images to a VNC client over the network, and listens for events

### GUI Applications

These render to Xvfb's in-memory screen instead of a physical monitor via the `DISPLAY` environment variable.

## Step-by-Step Setup Guide

### Prerequisites (Remote Host)

First, install the necessary packages on the **remote host**:

- `xvfb`
- `x11vnc`
- `xterm` (for demonstration purposes in this tutorial)

**We would have to run `xvfb`, `x11vnc`, and `xterm` concurrently.** For this purpose, I recommend using `tmux` (or any other comparable tool) to manage multiple terminal windows and persistent sessions.

### Start the Virtual Display (Remote Host)

In your first terminal window on the remote host, start Xvfb:

```bash
Xvfb :1 -screen 0 800x600x16
```

This creates a virtual X11 display number `:1` (using TCP port 6001) with resolution `800x600` and 16-bit color depth.

**Xvfb has properly started only if the above command neither logs output nor exits after being invoked.** If it logs messages or exits immediately, there's a problem to solve before moving on. Common issues include:

- Port 6001 already in use (try a different display number like `:2`)
- Permission issues with `/tmp/.X11-unix` directory. The `/tmp/.X11-unix` directory must be readable and writable. **In some environments like WSL, this directory might be read-only, which will prevent Xvfb from starting properly.**

### Launch the VNC Server (Remote Host)

In a second terminal window on the remote host, start x11vnc:

```bash
x11vnc -display :1 -rfbport 5901 -localhost -forever
```

This command:

- Connects to the virtual X server's framebuffer at display `:1`
- Listens for VNC connections on port 5901
- Restricts connections to localhost only (for security, VNC is not encrypted)
  - **Never expose VNC directly to the public Internet!**
- With `-forever`, x11vnc won't exit after the last client disconnects

After running the command:

- It may report **authentication or password-related errors** (which is **acceptable**).
- But it should **NOT** exit after running for a while. If it exits after running for a while, there's a problem to solve before moving on.

### Secure Access with SSH Tunneling (Local Machine)

VNC is not encrypted, so we'll use SSH tunneling on your **local machine** for security.

You can either use basic SSH port forwarding (which may be confusing), or [set up the `pull_remote_port.sh` wrapper](https://github.com/jifengwu2k/port-tunnel-manager) on your **local machine** to pull the remote host's port 5901 to `localhost:5901`. After **cloning its GitHub repository and completing its prerequisites**, run:

```bash
bash pull_remote_port.sh [-p <ssh_port>] -u <remote_user> -h <remote_host> -r 5901 -l 5901
```

This makes the remote host's VNC port available on your local machine via a secure SSH tunnel.

### Test with a Simple Application (Remote Host)

In a third terminal window on the **remote host**, launch a test application:

```bash
# The `DISPLAY=:1` environment variable must be set
# For all GUI apps you wish to appear in the VNC session
export DISPLAY=:1
xterm
```

Now connect to your VNC server on your **local machine** using a VNC client (like TigerVNC, RealVNC, or Remmina) pointing to `localhost:5901`. You should see the xterm window.

### Using Desktop Environments (Remote Host)

For a full desktop experience rather than just single applications, you can install a desktop environment.

#### Lightweight Options

- Openbox
- Fluxbox
- IceWM
- WindowMaker

#### Full Desktop Environments

- GNOME
- KDE Plasma
- MATE
- Cinnamon
- LXDE
- LXQt

### Important Caveat: Single-Instance Applications

Many desktop applications are designed to run **only one instance per user account**:

- Browsers: Firefox, Chrome/Chromium
- Advanced editors: gedit, kate, geany (configurable), code/VSCode
- File managers: nautilus, pcmanfm (configurable), dolphin
- Document viewers: evince
- Many modern GTK/Qt apps that use D-Bus for activation

**If your remote host has a physical desktop, and such an app is already running on your physical desktop**, new instances will try to communicate with the existing instance via D-Bus, **creating windows on your physical desktop instead of in your virtual framebuffer, even after `export DISPLAY=:1`.**

Solution:

- Ensure applications aren't already running on your main desktop before launching them in your VNC session.
- Or use a different user account for VNC access.

## Conclusion

Setting up a headless VNC server might seem complex at first, but by understanding the three components (Xvfb, x11vnc, and your GUI applications) and following this recipe, you can create a reliable remote desktop solution. Remember to always use SSH tunneling for security and be mindful of single-instance applications.

With this setup, you can enjoy full graphical desktop access to your remote Linux machines, opening up possibilities for remote administration, development, and troubleshooting.
