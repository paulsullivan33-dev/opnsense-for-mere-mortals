# Limits

An honest chapter, as this series
insists: what OPNsense cannot do, does
badly, or will make you do yourself.
Everything here was met in this guide's
own reading and lab work, not collected
from complaint threads.

## The lab's limit, first

This guide never ran the system. The
build machine cannot boot another
operating system, so the installer, the
wizard, the dashboard and every click in
them are documented-only, taught from the
project's manual. What was verified —
the image, its checksum, its signature
chain, its byte anatomy, the factory
configuration in the source tree — is
itemised in `criteria.md`. A guide that
pretends to clicks it never made would be
worthless precisely where a firewall
guide must be exact.

## Platform limits

- **x86-64 only, in practice.** The
  install manual says the software is
  available for the x86-64 architecture;
  ARM exists in the source but ships no
  official images. The drawer of ARM
  single-board computers — the natural
  habitat of OpenWrt — is closed to it.
- **It is not a Wi-Fi access point** in
  any modern sense. FreeBSD's wireless
  drivers trail current standards; use
  separate access points (chapter 4).
- **Hardware support is FreeBSD's.**
  A network chipset FreeBSD drives badly,
  OPNsense drives badly; no GUI fixes a
  driver (chapter 4's Intel advice is
  this limit wearing a hat).

## Operational limits

- **You are the update department.**
  Fortnightly minors, two majors a year —
  the security posture is the selling
  point, and it is also a standing
  appointment. An unmaintained OPNsense
  decays exactly like any unmaintained
  firewall.
- **Major upgrades are offline and
  console-first.** No web or SSH during
  the process (chapter 14). A firewall
  installed somewhere you cannot reach
  physically needs its out-of-band plan
  made *before* it needs it.
- **The configuration file is not
  portable across the fork.** pfSense
  files, and OPNsense files across very
  old versions, are references, not
  restores (chapter 3). Hardware changes
  also have their edges: a restore onto
  different interface names pauses for
  reassignment rather than guessing.
- **State is finite.** Every connection
  costs memory; the state table's
  reserved share is configurable, not
  infinite. Torrents and scanners can
  fill it on small hardware, and the
  symptom — new connections failing on a
  box that looks idle — is a classic.

## Feature-shape limits

- **IPS is a tripwire with a CPU bill**
  (chapter 13): inline inspection costs
  throughput, TLS content stays opaque,
  and rulesets need tuning by a human who
  reads the alerts.
- **The legacy baggage is real.** Two
  rule implementations coexist (the
  modern one, and the legacy pages as a
  plugin); two and a half DHCP servers;
  IPsec carries both a modern Connections
  model and legacy tunnels with a
  migration story. Each coexistence is
  kindness to existing users, and each is
  one more thing a newcomer must learn
  is doubled.
- **Plugins are a tiered promise**
  (chapter 14). The community catalogue
  is a strength and a supply chain; the
  core team supports the core.

## The limit that isn't technical

OPNsense makes you the network
administrator of everyone behind it.
When it breaks at 21:00, the household's
internet is down and the support desk is
you, a console cable, and chapter 18.
That is the true cost line in the
spreadsheet, and it is why this guide
keeps pointing at the exits: config
backups, the importer, the old router on
the shelf. Buy the system with your eyes
open and it is a bargain; buy it as a
talisman and it is a liability with a
nice dashboard.
