# What OPNsense is

Your home or office network has a border:
the point where your machines stop and the
internet begins. Something has to stand on
that border — handing out addresses,
deciding which traffic may cross, and
keeping the conversations straight. In most
homes that job belongs to the free router
box from an internet provider. **OPNsense**
is what people install when they want that
job done by software they can inspect,
configure in depth, and run on hardware
they chose themselves.

OPNsense is a **firewall and routing
platform**: a complete operating system,
built on **FreeBSD**, that turns an
ordinary small computer with two or more
network ports into the border guard for a
network. It is open source, free to
download, and managed almost entirely from
a web page it serves itself. The project's
own description is "an open source,
easy-to-use and easy-to-build FreeBSD based
firewall and routing platform", and its
name is a small manifesto: *open (source)
makes sense*.

This chapter gives you the mental model.
By the end you should be able to say what
OPNsense replaces, what it does all day,
and — just as important — what it is not.

## The job, in one paragraph

A firewall/router does four everyday jobs:

- **Routing** — moving packets between
  networks: yours (the **LAN**, local area
  network) and the provider's (the **WAN**,
  wide area network).
- **Address translation (NAT)** — letting
  a whole houseful of devices share one
  public internet address, and keeping
  track of which device started each
  conversation so the answers come back to
  the right place.
- **Filtering** — deciding, packet by
  packet, what may cross. OPNsense's
  filter is **stateful**: it remembers
  conversations it has allowed, so replies
  are recognised as part of a conversation
  rather than judged as strangers.
- **Services** — the border is a natural
  place to hand out addresses (**DHCP**),
  answer name lookups (**DNS**), accept
  encrypted tunnels (**VPN**), and keep
  time (**NTP**). OPNsense bundles all of
  these.

A consumer router does these jobs too, in
a smaller and quieter way. The difference
is not the list of jobs; it is how much of
each job you are allowed to see and
change, how the software is updated, and
whose computer it runs on.

## What it replaces, and what it doesn't

OPNsense replaces the *routing* part of
your provider router or a consumer mesh
system's base station. It does not
manufacture Wi-Fi: it has only basic
support for wireless cards, and nearly
everyone who runs it keeps separate
wireless access points behind it (chapter
4 returns to this). It is also not a
switch — it expects a small computer with
at least two wired network ports, one
facing the internet and one facing your
network, and it is happiest when those
ports are good ones.

It also takes over, if you let it, jobs
that might currently live elsewhere on
your network: the DHCP server on your old
router, the DNS resolver your provider
runs for you, the VPN server on a spare
machine. Chapters 11 and 12 explain those
choices one by one.

## Who runs it

- Home users who outgrew their provider
  router: they want per-device rules,
  a real VPN back into the house, or an
  honest log of what crossed the border.
- Small businesses and branch offices,
  which is the market OPNsense's sponsor
  **Deciso** sells support and appliances
  for.
- Schools, labs and non-profits that need
  commercial-grade features — redundancy
  between two firewalls, intrusion
  prevention, detailed reporting — without
  a per-seat licence.

There is also a paid **Business Edition**
with a more conservative update track; the
free **Community Edition** in this guide
is the same core software on the faster
release schedule.

## The shape of the thing

Three facts organise everything else in
this guide:

1. **It is a whole operating system.**
   You do not install OPNsense onto a
   running computer the way you install an
   app. You write an image to a disk or
   card, and the machine *becomes* the
   firewall (chapters 5 to 7).
2. **Almost everything is one file.**
   The entire configuration — interfaces,
   rules, users, certificates — lives in a
   single XML file, `/conf/config.xml`.
   Back that file up and you can rebuild
   the firewall on new hardware
   (chapter 8).
3. **It is managed from its own web
   interface**, with a text console menu
   for installation, recovery and the
   dozen emergencies a web page cannot
   help with. There is also a REST API —
   the web interface and the API are two
   doors into the same configuration.

## What this guide's lab could and couldn't do

A word about evidence, because this series
keeps a ledger. OPNsense is an operating
system for dedicated hardware or a virtual
machine, and this guide's build machine
could do neither: it has no way to boot a
second operating system. So no OPNsense
system was ever run for this guide.
Instead, the lab did what it honestly
could (chapter 6 has the details):

- It downloaded the current release image
  from an official mirror and **verified
  it the way the project intends** — the
  SHA-256 checksum against the official
  checksum list, and the OpenSSL signature
  on that list against the project's
  published public key (chapter 5).
- It **took the image apart byte by
  byte** — partition tables, the BSD
  disklabel, the filesystem's fingerprint
  — up to the exact point where a Linux
  machine has no tools to read further,
  and it says where that point is.
- It read the **factory configuration**
  the system ships with, in the project's
  own source tree, field by field — so
  every "by default" claim in this guide
  traces to the file itself.
- Everything past that — clicking through
  the installer, the setup wizard, the
  dashboard — is taught from the project's
  official documentation, and labelled
  documented-only in `criteria.md`.

That is a weaker position than most guides
in this series enjoy, and the guide says
so rather than dressing it up. What it
means in practice: the facts here are
checkable, but the *feel* of driving the
system is described, not experienced.

## The one warning that belongs up front

A firewall is other people's connectivity.
When your router is a hobby box, a mistake
costs you an evening. When it is the
border for a household or an office, a
mistake costs everyone their internet —
and a careless rule can quietly remove the
protection the box exists to provide.
Chapter 7 therefore starts from the
recovery tools (the configuration
importer, the console menu, the config
backup) before it touches the installer,
and this guide will keep pointing at the
exit doors. Keep a way back: the old
router on a shelf, the config file backed
up, the console reachable.
