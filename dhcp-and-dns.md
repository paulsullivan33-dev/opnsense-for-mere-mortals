# DHCP and DNS: addresses and names

Two services make a network feel like
home: something hands each device an
address when it arrives (**DHCP**), and
something answers when a device asks
"where is *printer*?" or "where is the
internet?" (**DNS**). On OPNsense both
live on the firewall. This chapter covers
what is running by default — read from
the shipped configuration, not from
habit — and the choices you actually
have to make.

## DHCP: three engines, one job

OPNsense carries three DHCP servers,
because the project is mid-journey
between generations:

- **Kea** — the modern DHCP server from
  ISC, present as its own configuration
  model in current releases and the
  direction of travel.
- **ISC DHCP** — the long-serving classic,
  still fully supported.
- **Dnsmasq** — the small combined DNS
  forwarder and DHCP server, beloved of
  home networks for a decade.

The factory configuration's own answer is
instructive: it enables **Dnsmasq** for
DHCP on the LAN, with a range of
**192.168.1.100 to 192.168.1.199** out of
the 192.168.1.0/24 network — a hundred
addresses for guests and gadgets, leaving
.2–.99 free for things you address by
hand. (Its DNS side listens on the
unusual port 53053 in the defaults;
Unbound, below, owns port 53.) Whichever
engine you choose, the concepts are the
same:

- The **range** is the pool of addresses
  handed to strangers. Keep it smaller
  than the subnet; the gaps are where
  your deliberate addressing lives.
- A **static mapping** (reservation) pins
  a device's hardware (MAC) address to a
  fixed IP, so the printer is always
  192.168.1.20 while still configuring
  itself automatically. Reserve anything
  other things point at: servers, cameras,
  the access points themselves.
- The lease also hands out the **gateway**
  and **DNS server** — which is to say,
  OPNsense itself, unless you say
  otherwise. This is the quiet mechanism
  by which the firewall becomes a
  network's source of truth.

One scale note for chapter 15: with two
firewalls, DHCP can run in failover, the
two servers sharing one pool and knowing
each other's leases — the CARP how-to
covers its settings, including that
clients must be given the *virtual*
address as gateway and DNS.

## DNS on the box: Unbound

Since version 17.7, the standard DNS
service — enabled by default on a new
install, says the manual, and the factory
configuration agrees — is **Unbound**, a
validating, recursive, caching resolver.
Three adjectives, three commitments:

- **Recursive**: instead of forwarding
  your lookups to your provider (or to a
  large public resolver), Unbound walks
  the DNS hierarchy itself — root servers,
  then the .com registry, then the
  domain's own servers. The provider sees
  far less of your household's reading
  habits.
- **Caching**: answers are remembered for
  their allowed lifetime, so the second
  device asking the same question gets an
  instant, local answer. A house full of
  machines phoning the same update servers
  feels this immediately.
- **Validating**: with DNSSEC enabled in
  its settings, Unbound checks the
  cryptographic signatures on answers and
  refuses forged ones — protection against
  cache poisoning that a simple forwarder
  does not give you.

The series' *Unbound for Mere Mortals*
guide teaches the resolver itself; here,
the OPNsense-specific doors:

- **Overrides** (Services ▸ Unbound DNS ▸
  Overrides) are your local phone book:
  *host overrides* make `nas.internal`
  mean 192.168.1.30, with aliases and
  automatic reverse (PTR) records. This is
  also the clean fix for chapter 10's
  reflection problem: the name resolves to
  the inside address for inside askers,
  so no hairpin is needed.
- **Access lists** decide who may ask;
  interfaces gain entries automatically.
- **Registering DHCP leases** (a general
  setting) puts devices' chosen hostnames
  into DNS as they get addresses — the
  small magic by which every gadget on
  the network becomes reachable by name.
- **Reporting ▸ Unbound DNS** in the GUI
  shows the resolver's live statistics —
  the top requested domains on your own
  network are frequently a surprise.

Dnsmasq's presence beside Unbound
confuses newcomers, so plainly: they are
alternatives with an overlap. The
defaults give the *resolver* job to
Unbound on port 53 and DHCP to Dnsmasq;
using Dnsmasq *as* the DNS service too
(forwarding mode), or replacing DHCP with
Kea, are both supported rearrangements —
but run exactly one DNS service on port
53, or the second one will fail to start
and the log will say why.

## The honest trade-off

Centralising names on the firewall means
the firewall is now on the critical path
for *everything feeling broken*: when DNS
is down, "the internet is down", whatever
the link is doing. The mitigations are
the chapter-8 habits — back up the
config, know the console menu, keep the
old router's address written down — plus
one diagnostic reflex: when a site "won't
load", try an address you know by number
before blaming the connection.
