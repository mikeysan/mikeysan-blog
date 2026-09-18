---
title: "My Time with Firejail: How to Uninstall Without Blowing Up Your System"
author: "Mikey San"
date: 2025-09-22T16:31:10.345Z
lastmod: 2026-09-18T13:13:51+01:00

description: "Firejail's firecfg sandboxed my whole desktop, screenshots included. How to uninstall it cleanly on Arch-based Linux without leaving broken symlinks behind."
summary: "Firejail's firecfg sandboxed my whole desktop, screenshots included. How to uninstall it cleanly on Arch-based Linux without leaving broken symlinks behind."
subtitle: "Step 1: Clean All Symlinks (Critical)"

image: "1.png" 
images:
 - "1.png"
 - "2.png"

aliases:
  - "/my-time-with-firejail-how-to-uninstall-without-blowing-up-your-system-0f1685396cc1"

tags:
  - firejail
  - linux
  - sandboxing
  - arch linux
  - firefox

---


![](1.png)

I recently installed a package called “Firejail” on my Linux system (CachyOS with KDE Plasma). The intention was to add some hardening to any instance of Firefox I ran. Unfortunately, Firejail appeared to pretty much jail almost all aspects of my system-including one particularly annoying one: screenshots (Spectacle).

Needless to say, it had to go.

### When Good Intentions Meet Reality

The irony wasn’t lost on me. Here I was, trying to secure one application, and I’d inadvertently locked myself out of basic desktop functionality. I’ll admit-I attempted to configure Firejail correctly from the start. The problem is that Firejail comes with numerous default profiles for many popular applications, and when you run the `firecfg` tool, it creates symlinks for all applications that have profiles, effectively sandboxing them system-wide.

This wasn’t what I’d expected. I wanted Firefox jailed, not my entire workflow.

The breaking point came when every screenshot attempt failed utterly. I’d already wrestled with Firefox startup errors earlier-managed to sort those out and convinced myself to persevere with the setup. But screenshots? That crosses into “deal-breaker” territory. I take far too many screenshots for work and documentation to tolerate that nonsense. In fact, you can blame it for not seeing one now.

I would rather go way overboard and install a virtual machine to run just Firefox than deal with such comprehensive system interference. And honestly, that threat isn’t off the table.

![](2.png)

### The Uninstall Challenge

This led me down another mini rabbit hole: how to uninstall Firejail without breaking everything. Through a combination of documentation diving and methodical trial and error, I cobbled together a proper removal procedure.

The key insight-one that could save you considerable frustration-is that **you must clean up Firejail’s integration before removing the package itself**. Skip this step, and you’ll likely end up with broken symlinks and confused desktop files scattered across your system.

### The Complete Removal Process

### Step 1: Clean All Symlinks (Critical)

Before doing anything else, run:

```bash
sudo firecfg --clean
```

This removes all symbolic links from `/usr/local/bin` that point to firejail. Miss this step, and you're asking for trouble.

### Step 2: Remove Modified Desktop Files

Firejail creates modified desktop files that need addressing:

```bash
# List what's been modified
ls -la ~/.local/share/applications/

# Create backup (recommended)
mkdir ~/desktop-backup
mv ~/.local/share/applications/*.desktop ~/desktop-backup/
```

The guide I was following called for deleting the files in this location. I found this step somewhat nerve-wracking-deleting all desktop files feels risky. Moving them to a backup location proved wise; you can always restore specific ones if needed later.

### Step 3: Clean User Access Database

```bash
# Check if it exists
cat /etc/firejail/firejail.users

# Remove if present
sudo rm /etc/firejail/firejail.users
```

### Step 4: Verify No Lingering Symlinks

```bash
# Check for remaining symlinks
ls -la /usr/local/bin/ | grep firejail
# Verify applications point to correct locations
which -a firefox spectacle
```

These should now point to `/usr/bin/` locations, not `/usr/local/bin/`.

### Step 5: Uninstall the Package

```bash
sudo pacman -Rns firejail
```

The `-Rns` flags remove the package, unnecessary dependencies, and configuration files.

### Step 6: Final Verification

```bash
# Confirm complete removal
which firejail # Should return nothing 

# Check for remaining files
find /etc -name "*firejail*" 2>/dev/null
find /usr -name "*firejail*" 2>/dev/null

# Test functionality
spectacle # Should work without authorisation errors
```

### Step 7: Restart Your Session

Log out and back in, or reboot. This ensures no firejail processes linger and all changes take effect.

### Lessons from the Trenches

The most critical insight from this exercise: if you use Firejail, it’s assumed you’d prefer to run applications sandboxed, and firecfg does its best to achieve that by creating symlinks and changing desktop files system-wide. This isn’t a bug-it’s intended behaviour.

But it highlights a fundamental design choice that caught me off guard. A better installation approach might involve zero system impact by default, requiring users to explicitly specify which applications they want sandboxed. The current system makes assumptions about user intent that don’t always align with reality.

As for alternative sandboxing methods for Firefox-I haven’t decided yet. The threat to use a full VM just for browsing remains very much on the table. Sometimes the nuclear option is the most honest one.

Thankfully, I discovered the screenshot issue long before uncovering any other affected packages. Small mercies-at least I caught it early, before the integration had time to complicate my workflow further.

*For what it’s worth, the uninstall process worked flawlessly. Sometimes the real victory is knowing when to retreat gracefully.*

*Originally published at* [*http://whoismikey.uk*](https://whoismikey.uk/2025/09/22/my-time-with-firejail-how-to-uninstall-without-blowing-up-your-system/) *on September 22, 2025.*
