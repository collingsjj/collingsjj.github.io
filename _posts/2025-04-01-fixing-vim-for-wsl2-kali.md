---
layout: post
title: Fixing vim for WSL2 kali
categories:
- Debugging
- WSL2
tags:
- kali-linux
- vim
- x11
- wslg
description: Vim was taking literal minutes to open on my WSL2 kali install. I document
  my process on fixing the issue - which ended up being an X11 problem.
date: 2025-04-01 15:07 -0600
---
## The Issue

I noticed that `vim` was taking an extremely long time to open when I went to edit some aliases in my WSL2 install of kali-linux. Normal time would be nearly instantaneous; milliseconds, 
at most. This was taking actual minutes to open my `.bash_aliases` file. Yes, I do make great use of aliases, but there's no way it should be taking _that_ long to open. Something was 
clearly wrong here, so I decided to take a look.

## Initial Findings

The first thing I wanted to check was if there were any leftover `vim` configurations outside of the `vim` config file itself. The only reference I could find was in the `.bashrc` file:

```terminal
# Set default editor
export VISUAL=vim
export EDITOR="$VISUAL"
```

This looked fine; `vim` is my default editor of choice (sorry `emacs` lovers). I decided to take things to Google; after sifting through the results and refining the search terms, I came 
across a [closed issue on the WSL github](https://github.com/microsoft/WSL/issues/5223#issuecomment-652309457) with the insight (thanks [clementnuss](https://github.com/clementnuss)):

> I guess the problem here is that `vim` tries to access the clipboard, which appears to be reaching out to the X server running (or not) on Windows because of the `DISPLAY` environment variable.
> 
> If the variable is set and an X11 server is running on Windows and reachable, then `vim` starts correctly.
> If the variable is set and the X11 server is not reachable, then it times out after 30sec and the startup is therefore slow.
> If the variable is not set, `vim` starts correctly.

## Debugging X11

As I hadn't used an X11 app in a while, I hadn't noticed that it wasn't working in kali. Sure enough though, when I checked it out, my `DISPLAY` var was failing:

```terminal
echo $DISPLAY
Error: Can't open display: <IP-ADDRESS>:0
```
(I removed the actual IP shown here; I don't want to dox myself, even though the IP was a reserved one for private networks)

I wasn't sure where the IP was coming from, but searching this error brought up a lot more information. The common theme running through them was a solution that involved adding this to 
the `.bashrc` file:

```terminal
export DISPLAY=$(cat /etc/resolv.conf | grep nameserver | awk '{print $2; exit;}'):0.0
```

Checking the catted file, sure enough the IP from the error was the one in `/etc/resolv.conf`. It would appear I had issues with X11 apps before and this solution fixed things for me. 
This is a perfect example of why you document these kinds of things! The issue now was finding where this was being set; I didn't have this line in my `.bashrc` file but it was clearly 
being set somewhere. A quick grep revealed the culprit:

```terminal
$ grep -r "DISPLAY=" .
./.profile:#    DISPLAY=$(route.exe print | grep 0.0.0.0 | head -1 | awk '{print $4}'):0.0
```

Opening this file indeed revealed the line I was looking for; though the command was different, I confimed this gave the same IP and thus set `DISPLAY` to the same value. To confirm 
this was the only culprit, I commented out the line and then restarted wsl in an elevated powershell window:

```terminal
wsl --shutdown
```

Upon bringing kali back up, I confirmed `DISPLAY` was using the correct value:

```terminal
echo $DISPLAY
:0
```

Fantastic! All that was left was to confirm the X11 issue resolved; a common means of doing so is to run the [`xeyes`](https://www.commandlinux.com/man-page/man1/xeyes.1.html) program, 
an X11 tool that's worth looking into if you're not familiar:

```terminal
xeyes
Error: Can't open display: :0
```

...Not what I was hoping for. What about `vim`? Trying to open a new text document presented no problems, though I surmised this was because X11 was throwing an error rather than timing 
out like before. Original problem solved! But leaving the system in a known bad state would be irresponsible. Back to Google I go!

## Fixing WSL2 Can't open display

I immediately ran into issues; this error was too similar to the error above with the IP address. As such, the most common solution was to add the line I had just removed. I had to dig 
into search results and filter them to only include the most recent results. I eventually found [this issue](https://github.com/microsoft/wslg/issues/1251) opened on the 
[`WSLg`](https://github.com/microsoft/wslg) github:

> "Error: Can't open display: :0" fails to run a simple GUI program correctly #1251

The thread here eventually lead to a [wiki entry](https://github.com/microsoft/wslg/wiki/Diagnosing-%22cannot-open-display%22-type-issues-with-WSLg#x11-display-socket) discussing the 
X11 display socket. Turns out, my kali was failing to create the link necessary to enable `WSLg` at startup. Using the recommended steps to re-create the link, I was able to confirm 
this indeed resolved the issue:

```terminal
sudo rm -r /tmp/.X11-unix
ln -s /mnt/wslg/.X11-unix /tmp/.X11-unix
xeyes
```

Voila! `xeyes` opened in a window on my Windows 11 setup. Unfortunately, this fix doesn't survive a reboot, nor does it explain _why_ it's happening in the first place; `WSLg` should 
work "out of the box," so to speak, as it's installed with all WSL2 distros. Turns out, this is [a known issue](https://github.com/microsoft/wslg/issues/1156); the 
[solution](https://github.com/microsoft/WSL/issues/6999#issuecomment-2303010704) right now is to create a file `/etc/tmpfiles.d/wslg.conf` as root with the command to mount the 
`WSLg` display:

```terminal
#  This file is part of the debianisation of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU General Public License as published by
#  the Free Software Foundation; either version 2 of the License, or
#  (at your option) any later version.

# See tmpfiles.d(5) for details

# Type Path           Mode UID  GID  Age Argument
L+     /tmp/.X11-unix -    -    -    -   /mnt/wslg/.X11-unix
```

I then restarted `wsl` in Powershell and checked that the symlink was working:

```terminal
ls -la  /tmp/.X11-unix
lrwxrwxrwx 1 root root 19 Apr  1 14:50 /tmp/.X11-unix -> /mnt/wslg/.X11-unix
```

A final test with `xeyes` and I saw what I was hoping for:

![xeyes working in wsl2](/assets/img/posts/xeyes-working-wsl2.png){: w"160" h="131" }

And there we have it; X11 working at startup!

## Wrap-Up

These kinds of debugs are always interesting to me. A problem with `vim` lead to the discovery of a bigger problem with X11 that revealed an even _bigger_ problem with `WSLg`. 
Going down rabbit hole after rabbit hole is a fun - though admittedly frustrating, at times - journey. Seeing as this clearly wasn't the first time I addressed this issue, 
I'm glad I took the time to document this one so that I can refer to it again when `WSLg` (hopefully) gets updated to fix this problem, inevitably breaking this fix and requiring 
me to look into this all over again!