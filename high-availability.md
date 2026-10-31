# High availability: two firewalls, one border

Everything in this guide so far has a
single point of failure shaped like a
small computer. **High availability**
(HA) removes it: two OPNsense machines,
one active (**master**) and one hot spare
(**backup**), presenting the network with
a single border that survives the death —
or the update — of either box. This
chapter explains the three mechanisms
that cooperate to do it, following the
project's own CARP how-to, and ends with
the honest price.

## The three mechanisms

- **CARP** (Common Address Redundancy
  Protocol, IP protocol 112, from the
  OpenBSD world by way of FreeBSD) lets
  several machines share a **virtual IP
  address**. Each shared address belongs
  to a virtual host group (**VHID**); the
  machines exchange multicast heartbeats,
  and the one with the best (lowest)
  **advertising skew** holds the address.
  When the master's heartbeats stop, the
  backup takes the address over — same
  IP, same MAC identity as far as the
  network is concerned — typically within
  seconds.
- **pfsync** continuously copies the
  master's **state table** to the backup,
  so conversations in progress at the
  moment of failure continue through the
  backup without reconnecting. The how-to
  advises a dedicated, direct cable
  between the two machines for this: the
  state stream is both bulky and
  security-sensitive (a forged state is a
  forged conversation).
- **XMLRPC sync** keeps the *backup's
  configuration* a copy of the master's:
  you choose what replicates — rules, NAT,
  DHCP settings, virtual IPs and more —
  and thereafter you administer the master
  only. It is one-directional by design:
  never configure it on the backup.

## The addressing picture

Each machine keeps its own real address
on every network, and the *shared*
addresses are additional. The how-to's
worked example, which this guide reuses
because it is admirably concrete:

| | Master | Backup | Shared (CARP) |
|---|---|---|---|
| LAN | 192.168.1.10 | 192.168.1.20 | **192.168.1.1** |
| WAN | 172.18.0.101 | 172.18.0.102 | **172.18.0.100** |
| Sync link | 10.0.0.1 | 10.0.0.2 | — |

Notice what follows: **clients must use
the shared addresses** — gateway and DNS
pointed at 192.168.1.1, not at either box.
Outbound NAT must be switched from
automatic to a manual rule translating to
the shared WAN address (chapter 10's
manual mode exists for this), so replies
return to an address the survivor owns.
And the boring prerequisite the how-to
shouts about: **interface assignments
must be identical on both machines** —
the sync maps settings by position, and
a DMZ that is the third interface on one
box and the second on the other will mix
the two machines' addresses into nonsense.

CARP addresses need their own welcome in
the firewall rules (protocol CARP must be
allowed on the interfaces that carry
them), each VHID must be unique per
network segment, and the CARP virtual
address wears the same subnet mask as its
parent interface — the how-to warns that
the tempting `/32` breaks DHCP failover
with the gloriously specific error "peer
holds all free leases".

## What it feels like when it works

The how-to's own test is the one to
reproduce: open an SSH session through
the pair, confirm both state tables show
the same connection, **pull the master's
network cable** — and watch the session
not even freeze. Updating becomes the
same trick played deliberately: update
the backup, put the master into
**persistent CARP maintenance mode** (the
backup takes over, and the mode survives
reboots), verify services on the backup,
update the master, leave maintenance
mode. Two machines, and maintenance stops
being downtime.

## The honest price

- **Two of everything**: hardware, power,
  ports, and patching discipline. The
  pair is one system to administer
  (XMLRPC sync sees to that) but two
  systems to keep identical where it
  counts.
- **New failure modes of its own**: a
  sync link misbehaving, split-brain when
  both machines believe they are master,
  switches whose IGMP snooping or port
  security quietly eat CARP's multicasts —
  the how-to has a known-limitations
  section on switch behaviour for a
  reason.
- **Identical-hardware pressure**: the
  state-sync warning in the how-to is
  stark — different network drivers (a
  physical master, a VM backup) can make
  states unsyncable because interface
  names differ, with a link-aggregation
  workaround as the only exit.

For a business whose internet is its
till, this is the feature that justifies
the whole platform. For a household, it
is usually one firewall more than the
problem needs — chapter 17 says so
plainly — but it is there, in the free
edition, when the problem grows.
