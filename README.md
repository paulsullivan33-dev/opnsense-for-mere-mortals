# OPNsense for Mere Mortals

Every network has a border — the point
where your machines stop and the internet
begins — and something has to stand on
it: handing out addresses, deciding which
traffic may cross, remembering every
conversation so the answers find their
way home. Usually that something is the
free box from your internet provider.
**OPNsense** is what people install when
they want the border guarded by software
they can inspect, on hardware they chose:
a complete **firewall and routing
platform** built on **FreeBSD**, managed
from its own web interface, keeping its
entire configuration in **one XML file**
you can back up, diff and carry to new
hardware.

This guide explains what OPNsense is and
how its parts fit together, how to choose
hardware and an image, how to prove the
image you downloaded is the one the
project published, what a fresh system
looks like before anyone touches it, and
then the working core: the one-file
configuration, the firewall rules, NAT,
DHCP and DNS, VPNs, intrusion prevention,
plugins, updates, and the two-machine
high-availability trick. It is honest
throughout about a limitation most
firewall guides never face: **this
guide's lab could not run the system.**
OPNsense is an operating system for
dedicated hardware or a virtual machine,
and the build machine could do neither —
it cannot boot a second operating system
at all.

So the lab did what it honestly could,
and this guide stands on that:

- It downloaded the current release
  image — **OPNsense 26.7** (July 15,
  2026), the nano image — from an
  official mirror and verified it the
  way the project intends: the SHA-256
  checksum against the official list,
  the OpenSSL signature on that list
  against the release public key (itself
  checked against the key printed in the
  release README), and the image's own
  signature over the unpacked file. All
  three returned their proofs.
- It took the image apart byte by byte —
  MBR, BSD disklabel, the UFS2
  filesystem's fingerprint — and reports
  the exact point where a Linux machine
  has no tools to read further.
- It read the **factory configuration**
  entire, in the project's own source
  tree (`opnsense/core`, October 2026),
  so every "by default" in these pages
  traces to the file itself — the
  192.168.1.1 LAN, the hundred-address
  DHCP range, the two shipped firewall
  rules, the wizard trigger, the web GUI
  that does not run as root.

Everything past that — the installer,
the wizard, the dashboard, every click —
is taught from the project's official
documentation and labelled documented-only
in `criteria.md`, which keeps the full
ledger. Nothing here pretends to a
screenshot it never took.

## What this guide covers

- [**What OPNsense is**](what-opnsense-is.md) —
  the border, the four jobs of a firewall,
  and what this one replaces
- [**The moving parts**](the-moving-parts.md) —
  FreeBSD and `pf`, the one configuration
  file, configd, the GUI and API, and the
  services that answer the network
- [**The fork**](the-fork.md) — why
  OPNsense split from pfSense in 2015,
  told fairly, and what actually differs
  today
- [**Choosing hardware**](choosing-hardware.md) —
  the project's sizing tiers, why the
  network cards matter more than the CPU,
  and the shapes people actually buy
- [**Getting the images**](getting-the-images.md) —
  dvd, vga, serial and nano; downloading
  26.7; and the checksum-plus-signature
  verification, run for real
- [**Anatomy of the nano image**](anatomy-of-the-nano-image.md) —
  the MBR and BSD disklabel parsed byte
  by byte, the UFS2 fingerprint, where
  reading stops — and the factory
  configuration, read entire from source
- [**Installing and first configuration**](installing-and-first-configuration.md) —
  the Importer, the live environment, the
  installer, the console menu, the setup
  wizard and the dashboard
- [**config.xml**](config-xml.md) — the
  whole firewall in one file: backups,
  history with diffs, automatic off-box
  backup, and carrying a firewall to new
  hardware
- [**Firewall rules**](firewall-rules.md) —
  states, direction, the floating/group/
  interface order, first-match-wins, and
  the two rules every system is born with
- [**NAT and port forwards**](nat.md) —
  the translation that is already on,
  forwards and their linked rules,
  reflection, and why NAT is not a
  security feature
- [**DHCP and DNS**](dhcp-and-dns.md) —
  three DHCP engines and the shipped
  range; Unbound the recursive resolver;
  overrides and lease registration
- [**VPNs**](vpn.md) — IPsec, OpenVPN and
  WireGuard; road warriors and
  site-to-site; and the rules a tunnel
  still answers to
- [**Intrusion prevention**](intrusion-prevention.md) —
  Suricata, IDS versus IPS, rulesets and
  policies, and the honest costs
- [**Plugins and updates**](plugins-and-updates.md) —
  102 plugins in tiers of support, and
  the two-majors-a-year update train
- [**High availability**](high-availability.md) —
  CARP, pfsync and config sync: two
  firewalls presenting one border
- [**The neighbours**](the-neighbours.md) —
  pfSense, OpenWrt, the provider router,
  hand-built Linux and the cloud-managed
  stacks, compared fairly
- [**Limits**](limits.md) — x86-64 only,
  Wi-Fi is not its job, you are the update
  department, and the limit that isn't
  technical
- [**Troubleshooting**](troubleshooting.md) —
  failed checksums, dark consoles,
  lockouts, rules that "don't work",
  split-brain, and silent major upgrades
- [**Glossary**](glossary.md) — the
  terms, collected in one place

## What it doesn't cover

Driving the system day to day beyond the
first configuration: deep per-feature
tuning (traffic shaping, captive portal,
multi-WAN load balancing, the web proxy)
is pointed at in the manual rather than
taught. Building OPNsense from source,
developing plugins, and the Business
Edition's commercial add-ons are out of
scope. Migrating *from* pfSense is
described as what it is — a rebuild with
the old file as a reference — not a
procedure.

## How to use this guide

Read chapters 1 and 2 for the model, and
chapter 6 before you judge any "by
default" claim — it shows its working.
Chapters 5 and 7 take you from a mirror
to a running firewall; chapter 8 is the
one to read twice, because the backup
habits there are what make every later
experiment safe. The lab's transcripts
live in `opnsense-lab/transcripts/` and
its source list in `opnsense-lab/sources.md`.

---

## Part of the series

This guide is one of the [For Mere Mortals](https://paulsullivan33-dev.github.io/mere-mortals/) guides: plain-language guides to the tools programmers and makers use.

Related guides:

- [OpenWrt for Mere Mortals](https://paulsullivan33-dev.github.io/openwrt-for-mere-mortals/)
- [Home Networking for Mere Mortals](https://paulsullivan33-dev.github.io/home-networking-for-mere-mortals/)
- [Unbound for Mere Mortals](https://paulsullivan33-dev.github.io/unbound-for-mere-mortals/)
- [WireGuard for Mere Mortals](https://paulsullivan33-dev.github.io/wireguard-for-mere-mortals/)

## License

The writing in this guide is licensed under [CC BY 4.0](https://github.com/paulsullivan33-dev/opnsense-for-mere-mortals/blob/main/LICENSE). Code examples and lab scripts are licensed under the [MIT License](https://github.com/paulsullivan33-dev/opnsense-for-mere-mortals/blob/main/LICENSE-CODE).
