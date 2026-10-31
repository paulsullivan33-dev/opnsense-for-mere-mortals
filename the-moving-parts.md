# The moving parts

OPNsense looks, from the web interface,
like one program. Underneath it is a small
cast of specialists, each doing one job,
all taking their orders from the same
configuration file. This chapter names the
parts and says which part holds what, so
that later chapters — and later mysteries
on your own firewall — resolve into "a
named part did this".

## The base: FreeBSD

OPNsense is built on **FreeBSD**, a
complete Unix-like operating system with
its own kernel, drivers and utilities. Two
consequences matter to a user:

- **Hardware support is FreeBSD's
  hardware support.** If FreeBSD has a
  driver for your network card, OPNsense
  can use it; if not, it cannot. This is
  why chapter 4 cares so much about which
  network chips are in the box.
- **The firewall engine is FreeBSD's
  `pf`**, the packet filter originally
  written for OpenBSD. When the
  documentation and this guide talk about
  rules, states and NAT, `pf` is the
  machine underneath doing the work.

OPNsense is also a *package* on that base:
the project notes you can remove the
`opnsense` package and be left with an
almost standard FreeBSD system. In
practice nobody does — but it explains the
layout: FreeBSD provides the ground floor,
OPNsense the building on top.

## The configuration: one XML file

Every setting — interfaces, firewall
rules, DHCP ranges, users, certificates,
VPNs — lives in a single XML file,
`/conf/config.xml`. The source tree makes
the arrangement literal: the web
framework's configuration names its
directory in one line, `configDir =>
'/conf'`, and reads and writes
`config.xml` inside it.

One file to hold everything sounds
fragile, and the system treats it with
corresponding respect:

- Before each change, the previous version
  is kept; the GUI's History page lists
  past versions, shows the difference
  between any two as a diff, and can
  revert (chapter 8).
- If the file is found invalid at boot,
  the system attempts to restore the last
  known good copy, and failing that, the
  factory configuration. Both behaviours
  are visible in the source code that
  loads it.
- A factory reset is, mechanically,
  emptying `/conf` and rebooting into the
  shipped defaults.

Chapter 6 reads the shipped version of
this file closely; chapter 8 covers its
life in service. For now, hold the
shape: **the GUI is an editor; this
file is the firewall.**

## The conductor: configd

Editing an XML file does not restart a
service. Between the configuration and the
running system sits **configd**, a backend
daemon (rewritten in Python by the
OPNsense project) that receives orders —
"reload the filter", "restart the DNS
resolver" — and carries them out with the
privileges the web interface itself does
not have. The source tree carries one
actions file per service
(`actions_filter.conf`, `actions_dns.conf`,
`actions_ids.conf`, and so on): a tidy
map of everything the system can be asked
to do.

This separation was one of the founding
reasons for the project (chapter 3): the
web pages should not themselves run as
root. The GUI asks; configd acts.

## The face: the web interface and the API

The web interface is a PHP application
(built on the Phalcon framework, in an
MVC — model, view, controller — shape).
Its newer pages are generated from **XML
models** in the source tree, one per
subsystem: `Firewall`, `Unbound`, `Kea`,
`IPsec`, `Wireguard`, `IDS`, `Interfaces`,
and two dozen more. Each model defines the
settings, their types and their valid
values — which is why the GUI can refuse
an impossible setting before it ever
reaches the file.

Alongside the pages sits a **REST API**
covering the same ground. Anything this
guide describes clicking, a script can
generally do over the API with an API key
— including the firewall rules, which were
rebuilt as a modern, API-first
implementation (the older rule pages still
exist as an optional plugin,
`os-firewall-legacy`, for those who want
them).

And when the web interface cannot help —
before an address is configured, after a
bad change locks you out — there is the
**console menu**, a numbered text menu on
the machine's own screen or serial port:
assign interfaces, set an address, reset
the root password, restore a
configuration, upgrade, reboot. Chapter 7
walks it.

## The workers: services that answer the network

These are ordinary, well-known programs,
configured by OPNsense from the one file:

- **DHCP servers.** Three generations
  coexist: the newer **Kea** DHCP server,
  the long-serving **ISC DHCP**, and
  **Dnsmasq**, which can serve both DHCP
  and DNS forwarding. The factory
  configuration enables Dnsmasq (with a
  LAN range of 192.168.1.100–199, read
  straight from the shipped defaults in
  chapter 6) — you choose which engine
  serves your network in chapter 11.
- **Unbound**, a validating, recursive,
  caching DNS resolver from NLnet Labs, is
  the standard DNS service and is enabled
  by default on a new install. It answers
  your machines' name lookups itself,
  walking the DNS hierarchy rather than
  forwarding everything to your provider.
  (The series' Unbound guide covers the
  resolver itself in depth.)
- **strongSwan** provides IPsec VPNs, in
  its modern `swanctl` "Connections" form;
  **OpenVPN** and a kernel **WireGuard**
  implementation round out the VPN shelf
  (chapter 12).
- **Suricata** is the intrusion detection
  and prevention engine, reading traffic
  through FreeBSD's **netmap** for speed
  (chapter 13).
- Smaller hands: `ntpd` for time, the
  **Monit** supervisor watching services,
  NetFlow export for traffic accounting,
  and `sshd` for shell access — off until
  you enable it, and even then the
  factory configuration restricts it to
  the admins group.

## The shop: packages and plugins

Under the GUI, OPNsense is installed and
updated with FreeBSD's package manager,
`pkg`. Extensions arrive as **plugins** —
packages named `os-<something>` — from the
project's plugin collection: this guide's
lab counted **102 plugins** in the
official plugins repository, from a DHCP
server alternative to a full web
application firewall to Tailscale and
ZeroTier clients. Plugins carry a support
tier (chapter 14): some are the core
team's own, most are community work the
core team does not promise to support.

## The picture, assembled

```
        you ──► web GUI / API / console
                        │
                        ▼
              /conf/config.xml  ◄── the whole firewall, one file
                        │
                   configd (asks → acts)
                        │
        ┌───────────────┼────────────────┐
        ▼               ▼                ▼
   pf (filter/NAT)   Unbound, Kea,    Suricata,
   in the kernel     Dnsmasq, NTP     strongSwan…
        │
   FreeBSD on your hardware
```

Keep that picture and the rest of the
guide is commentary: every procedure is
someone editing the file through a door,
configd applying it to a worker, and a
check that the worker did it.
