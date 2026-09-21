---
title: "I Knew systemd. I Didn't Really Understand the .service File."
date: 2026-09-21
lastmod: 2026-09-21
draft: false
description: "I'd used systemctl for years without ever looking inside a unit file. Writing one for PulsePoint forced the issue."
tags: ["linux", "systemd", "sysadmin"]
ShowToc: true
TocOpen: false
cover:
  image: "1.jpg"
  alt: "A Linux terminal at a command prompt"
  caption: "Photo by [Gabriel Heinzer](https://unsplash.com/@6heinz3r) on [Unsplash](https://unsplash.com/photos/xbEVM6oJ1Fs)"
  relative: true
---

When you've been working with Linux for a while, `systemd` isn't exactly unfamiliar.

I've used `systemctl` plenty of times. Start a service, stop a service, restart it, check its status, enable it at boot. Nothing particularly mysterious about any of that.

But recently I found myself having to create a `systemd` service file from scratch for one of my own projects, [PulsePoint](https://pulsepoint.whoismikey.net/).

That's when I realised something slightly embarrassing.

I knew how to *use* `systemd`, but I'd never really stopped to look at what was inside one of those `.service` files.

I'd followed instructions before. Copy this here, change that there, run `systemctl daemon-reload`, enable the service and off you go.

It worked, but I didn't really understand what I'd just created.

So I decided it was probably time to fix that.

## First, what exactly is systemd?

If you're using a modern Linux distribution, there's a good chance you're already using `systemd`, whether you've thought about it or not.

At its simplest, `systemd` is the init system used to start and manage the system. During the boot process, the Linux kernel starts the first userspace process, which is given process ID 1. On a system using `systemd`, that process is `systemd`.

From there, `systemd` takes responsibility for bringing up the rest of the system and managing various things while it is running.

One of the things it manages is **services**.

These are the background processes that keep various parts of your system running. An SSH server is a good example. A web server is another.

And if you've ever run something like this:

```bash
systemctl status sshd
```

you've already interacted with `systemd`. (On Debian and Ubuntu that unit is usually called `ssh` rather than `sshd`.)

## `systemctl` is the bit we're probably most familiar with

For everyday administration, `systemctl` is the command I'm most likely to use.

For example:

```bash
sudo systemctl start sshd
```

starts a service.

```bash
sudo systemctl stop sshd
```

stops it.

```bash
sudo systemctl restart sshd
```

restarts it.

And:

```bash
systemctl status sshd
```

shows me what's happening with the service. That one doesn't need `sudo`, although you'll see more of the service's log output if you use it.

There's also an important distinction between **active** and **enabled**.

If a service is active, it is running right now.

If a service is enabled, it is configured to start automatically when the system boots.

Those aren't the same thing.

You can have a service that's running but isn't enabled, meaning it will be running now but won't necessarily start the next time you reboot.

That's where this comes in:

```bash
sudo systemctl enable sshd
```

And if you don't want it to start automatically:

```bash
sudo systemctl disable sshd
```

There are also `reload` and `restart`, which are worth understanding.

A reload asks the service to reload its configuration without completely restarting it, assuming the service supports that. A restart actually stops and starts the service again.

In practice, I've probably used `restart` more often than `reload`, but it's useful to know that they're doing different things.

So far, so good.

This is the part of `systemd` I was already comfortable with.

Then I needed to create my own service.

## So what is a `.service` file?

This is the part I'd never actually looked at.

A `.service` file is a **unit file**.

And this introduces another `systemd` term that's worth understanding: **units**.

`systemd` doesn't only manage services. It manages different types of objects, known collectively as units.

A service is one type of unit. There are also units for things such as timers, mounts and sockets.

Each unit has a unit file that describes it.

In other words:

> **A unit is something that `systemd` manages. A unit file describes how `systemd` should manage it.**

That distinction made a lot more sense to me once I actually had to create one.

## What's inside a service file?

A `.service` file is just a text file, but it follows a particular structure.

The three sections you're likely to encounter in a basic service file are:

```text
[Unit]

[Service]

[Install]
```

At first glance, that doesn't tell you very much.

So here's a cut-down version of the one I wrote for PulsePoint. I've changed the paths, stripped out the environment variables, the restart policy and the security hardening, and trimmed the `gunicorn` command after the first option, which leaves the bones of it:

```ini
[Unit]
Description=Gunicorn daemon for PulsePoint
After=network.target

[Service]
Type=notify
User=pulsepoint
Group=pulsepoint
WorkingDirectory=/srv/pulsepoint/backend
ExecStart=/srv/pulsepoint/venv/bin/gunicorn --workers 3 ...

[Install]
WantedBy=multi-user.target
```

That's a handful of lines. Taken one section at a time, they stop looking arbitrary.

### `[Unit]`

The `[Unit]` section contains information that applies generally to the unit.

It can describe what the unit is and its relationships with other units.

`Description=` is the human-readable name, and it's what shows up when I run `systemctl status`.

`After=network.target` is about ordering. If the network is being brought up as part of the same startup, my service should come up after it.

It's worth knowing that `After=` controls ordering and nothing else. It isn't a dependency, and reaching `network.target` doesn't necessarily mean the network is fully configured and routable. That's a different target called `network-online.target`, and a rabbit hole for another day.

The important thing for me was understanding that this section isn't specifically about how the application itself runs. It's about the unit and its place within the wider `systemd` system.

### `[Service]`

This is the section that is specific to a service.

It contains information about how that service should actually be run, and it's where most of my file ended up.

`User=` and `Group=` mean the process runs as a dedicated `pulsepoint` account rather than as root. `WorkingDirectory=` is the directory the process starts in.

`ExecStart=` is the command `systemd` actually runs. It's the one directive a service can't really do without.

`Type=` is the one I actually had to go and look up. Mine is set to `notify`, which means the service tells `systemd` when it has finished starting up, rather than `systemd` assuming it's ready the moment the process launches. The common default is `simple`, where `systemd` treats the job as done as soon as the process has been started.

That mattered here because Gunicorn supports systemd's readiness notification. With `notify`, `systemd` waits until the workers are genuinely up before it considers the service started. With `simple`, it would call the job done the instant the process launched, whether or not anything was ready to serve a request.

That distinction between sections is useful because not every unit is a service.

A socket unit has a `[Socket]` section, for example, while a service has `[Service]`.

### `[Install]`

The `[Install]` section deals with how the unit is linked into the system when it is enabled.

`WantedBy=multi-user.target` is the line doing that work in mine.

This is the part that becomes relevant when you run:

```bash
sudo systemctl enable your-service
```

At which point `systemd` creates a symlink to my unit file inside `multi-user.target.wants/`. That symlink is what causes the service to start when the system reaches its normal multi-user state.

Enabling a service isn't some abstract flag being set somewhere. It's a symlink.

It's also why a service file isn't simply a collection of commands for starting an application. It's a description that tells `systemd` how the service fits into the system and how it should be managed.

Once I understood that, the structure of a `.service` file stopped looking quite so mysterious.

## Where do these files live?

There are several locations involved in `systemd` unit files.

The main system unit files are generally found under:

```text
/usr/lib/systemd/system/
```

There is also:

```text
/run/systemd/system/
```

which is used for unit files created at runtime.

And then there's:

```text
/etc/systemd/system/
```

This is the location that is particularly relevant when you're creating your own system-wide service or customising a service.

That's where my own service ended up.

Those three locations aren't interchangeable. They have an order of precedence, and `/etc/systemd/system/` beats `/run/systemd/system/`, which beats `/usr/lib/systemd/system/`. That's the real reason my file went into `/etc` rather than just convention. It also means you can override a unit that came from a package without touching the packaged file, because yours wins.

And suddenly the whole process made a little more sense.

I wasn't creating some mysterious configuration file that `systemd` magically understood.

I was creating a unit file that described a service and putting it somewhere `systemd` knows to look.

Which is also where `daemon-reload` finally made sense to me.

`systemd` doesn't sit watching those directories for changes. Once it has read a unit file, it works from what it has in memory. Drop a new file in, or edit one that's already there, and `systemd` carries on with the old version until you tell it otherwise.

```bash
sudo systemctl daemon-reload
```

That's the whole job of that command. Go and read the unit files again.

It was one of the incantations I'd been copying out of tutorials for years, and it turns out to be about the most obvious thing in the world once you know where the files live.

## The bit I had been missing

I'd been treating the `.service` file as something I needed to copy from a tutorial until it worked.

That's not particularly unusual. When you're trying to get an application running, you don't always have time to stop and understand every configuration file you're dealing with.

But once I actually looked at the structure, it became much easier to understand what I was doing.

`systemd` is managing the service.

`systemctl` is the tool I use to tell `systemd` what I want it to do.

And the `.service` file describes the service itself.

Those three things fit together quite neatly.

## So, do I understand systemd now?

Better than I did before.

I'm still not going to pretend that I've suddenly become a `systemd` expert. There is considerably more to it than the handful of commands and sections I've covered here.

But there's a big difference between knowing that something works and understanding roughly *why* it works.

That distinction matters when you're troubleshooting.

It's one thing to know that you can run:

```bash
sudo systemctl restart my-service
```

It's another to understand that `systemd` is reading a unit file which describes that service, managing it as a unit, and using that configuration to determine how the service fits into the system.

And that's exactly why I like taking these little detours into things I've been using for years.

Creating my own service for PulsePoint forced me to look a little deeper.

And, honestly, I probably should have done it sooner.
