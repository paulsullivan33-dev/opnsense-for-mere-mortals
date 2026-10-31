# Choosing hardware

OPNsense is software in search of a small
computer. This chapter is the shopping
list — honest, because the wrong box is
the most common OPNsense disappointment
and it is almost always the network cards,
not the processor. By the end you should
be able to size a machine for a connection
speed and a feature list, and say what to
avoid.

## The minimum shape

- **x86-64.** OPNsense is published for
  the 64-bit PC architecture only. (ARM
  builds exist in the source tree, but no
  official ARM images are released — the
  mirror directories for current releases
  carry amd64 only.)
- **Two wired network ports, minimum.**
  One becomes the WAN (internet side), one
  the LAN (your network). More ports earn
  their keep: a separate network for
  guests or cameras, a dedicated link to a
  second firewall (chapter 15).
- **A console you can reach.** Either a
  monitor and keyboard (VGA) or a serial
  port — one of them must work when the
  network does not, because that is
  precisely when you will need it.

## The project's own sizing table

The manual's hardware page gives three
tiers. Quoted faithfully:

| Tier | CPU | RAM | Install target |
|---|---|---|---|
| Minimum | 1 GHz dual-core | 3 GB | SD/CF card, 4 GB or more (nano image) |
| Reasonable | 1 GHz dual-core | 4 GB | 40 GB SSD |
| Recommended | 1.5 GHz multi-core | 8 GB | 120 GB SSD |

And it ties speed to tier approximately:
up to about 150 Mbps on the minimum spec,
150–350 Mbps on reasonable, and 350–750+
Mbps on recommended, with all standard
features on. Treat these as the project's
rules of thumb, not promises — the same
page notes what bends them:

- **States cost memory.** Each tracked
  connection takes about 1 kB of RAM, and
  the filter reserves a slice of memory
  (10% by default, per the rules manual)
  for the state table. Busy networks with
  hundreds of thousands of connections
  need RAM accordingly.
- **Features cost CPU.** A caching proxy
  and the intrusion prevention system
  (chapter 13) are the heavy ones; an IPS
  examining every packet can halve what a
  box would otherwise route, and it needs
  disk for its alert database — which is
  why the minimum tier, with its memory
  disks, excludes it.
- **VPNs cost CPU per packet**, unless
  the processor has AES acceleration
  (AES-NI), which most PC processors of
  the last decade do.

## Network cards: the chapter's real subject

The manual is blunt: Intel network
chipsets are "reliable, fast and not
error-prone", delivering higher throughput
at lower CPU load, and the community's
experience matches it. Practical guidance:

- **Prefer Intel-based ports** (the common
  server chipsets — i210, i350, 82576 and
  their descendants — are the usual
  recommendations). They are what the
  official appliances use class after
  class.
- **Be wary of the cheapest Realtek
  ports** found on bargain mini-PCs:
  FreeBSD's drivers for them have a long
  history of lower throughput and
  occasional hangs under load. Many units
  work; the failure mode, when it comes,
  is miserable to diagnose.
- **USB network adapters** are a last
  resort: fine for a lab, poor citizens
  on a border that must not wobble.
- **Check the FreeBSD hardware list** for
  your exact card before buying — OPNsense
  includes all FreeBSD drivers, so
  FreeBSD's compatibility is OPNsense's.

## The shapes people actually buy

- **A fanless mini-PC with two or four
  Intel ports**, sold precisely for this
  job by many small vendors. Quiet, sips
  power, mounts behind a cabinet. Check
  the chipset, not the brand name.
- **An official appliance from Deciso**
  (the OPNsense shop) or a partner:
  known-good hardware, the system
  preinstalled, and money that funds the
  project — the manual itself points
  buyers there first.
- **A retired small office PC** with an
  added Intel dual-port card. The classic
  free option; mind the power bill and
  the fan noise for something that runs
  all year.
- **A virtual machine**, for labs and for
  people whose server already lives in a
  closet or a cloud. Officially supported
  as an install target (the images run
  from virtual disks), and chapter 7's
  importer trick — booting a new version
  against your existing configuration
  without overwriting it — was made for
  exactly this. A VM firewall protecting
  the network its host sits on deserves
  a careful think about what happens when
  the host reboots.

## Storage, briefly

The nano image (chapter 5) exists for
flash cards: it runs with logs and caches
in memory to spare the card's write life,
at 3 GB in size, adapting to the card
after first boot. For anything with the
intrusion prevention system, reporting
history, or a proxy cache, use a real SSD
— the sizing table's 40–120 GB is
generous for a firewall, and the manual's
installer chapter steers you to **ZFS**
where capacity allows, calling it in most
cases the best option for reliability.

## The Wi-Fi truth

OPNsense can drive some wireless cards as
an access point, but FreeBSD's wireless
drivers lag the state of the art — modern
Wi-Fi generations and speeds are not its
strength, and the documentation treats
wireless as a minor feature. The standard
deployment, recommended here without
reservation: **OPNsense routes, separate
access points broadcast.** Keep your
existing mesh or access points, put them
in bridge/access-point mode behind the
LAN port, and let each tool do its own
job.
