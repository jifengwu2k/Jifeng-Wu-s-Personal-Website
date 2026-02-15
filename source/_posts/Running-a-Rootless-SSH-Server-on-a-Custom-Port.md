---
title: Running a Rootless SSH Server on a Custom Port
date: 2025-12-10
categories:
  - DevOps, SysAdmin
tags:
  - Reference
---

Sometimes you need to spin up an SSH server without changing system configs or needing root. Here's how you can do it, using a custom port and a host key you generate yourself.

## Prerequisites

- `/usr/sbin/sshd` properly configured to facilitate logins and `ChrootDirectory` (usually `/var/empty`) set up **by root**.

> **Note:** If `/usr/sbin/sshd` is not properly configured or you want to reconfigure the logic to facilitate logins (e.g., allow password login), and you *don't* have root access, this method *won't* work. Consider alternatives like [telnet](https://en.wikipedia.org/wiki/Telnet) for shell or [WebDAV](https://en.wikipedia.org/wiki/WebDAV) for file transfers.

## Generate a Custom Host Key

First, create a new Ed25519 host key (keeps the main host keys untouched):

```bash
ssh-keygen -t ed25519 -f ~/my_host_ed25519_key -N ''
```

- This creates the private key: `~/my_host_ed25519_key`
- And the public key: `~/my_host_ed25519_key.pub`
- `-N ''` means no passphrase

## Start `sshd` on a Custom Port in Foreground

You can launch the SSH daemon on, for example, port 2222, using your key:

```bash
/usr/sbin/sshd \
    -D \
    -p 2222 \
    -o "HostKey=$(realpath ~/my_host_ed25519_key)" \
    -o PasswordAuthentication=yes \
    -o PermitRootLogin=yes \
    -o PubkeyAuthentication=yes
```

**Explanation:**

- `-D` - Don't daemonize (run in the foreground)
- `-p 2222` - Use a non-privileged, custom port
- `-o HostKey=...` - Use just the specified host key

Now you or a teammate can connect like:

```bash
ssh -p 2222 <user>@<host>
```
