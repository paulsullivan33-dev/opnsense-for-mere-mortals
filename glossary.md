# Glossary

**Access point (AP)** — a device that
provides Wi-Fi and bridges it onto the
wired network. In an OPNsense network the
APs are separate from the firewall.

**Alias** — a named list of hosts,
networks or ports, maintained once and
referenced by many firewall rules.

**Bogons** — address ranges that should
never appear as sources on the public
internet (unallocated space, and the
like). The WAN interface blocks them by
default; the list updates monthly in the
factory configuration.

**BSD disklabel** — the FreeBSD partition
table that lives inside a FreeBSD slice
of an MBR disk, naming partitions by
letter (`a` is conventionally the root
filesystem).

**CARP** — Common Address Redundancy
Protocol: two or more machines share a
virtual IP address, with heartbeat
messages deciding which one holds it.

**configd** — OPNsense's backend daemon:
receives orders from the web interface
("restart DNS", "reload the filter") and
carries them out with the necessary
privileges.

**config.xml** — the single XML file
(`/conf/config.xml`) holding the entire
OPNsense configuration.

**Default deny** — the built-in rule at
the end of the rule set: traffic no rule
has allowed is dropped.

**DHCP** — Dynamic Host Configuration
Protocol: hands arriving devices an
address, a gateway and a DNS server.

**Dnsmasq** — a small combined DHCP
server and DNS forwarder; in the factory
configuration it serves DHCP on the LAN.

**Floating rule** — a firewall rule
attached to several interfaces (or none
in particular), processed before group
and interface rules.

**FreeBSD** — the operating system
OPNsense is built on; provider of the
kernel, drivers and the `pf` filter.

**IDS / IPS** — intrusion *detection*
(watch and alert) and intrusion
*prevention* (inspect inline and drop),
here both by Suricata.

**Importer** — the boot-time tool that
loads an existing configuration — from a
disk or a USB stick's `/conf/config.xml`
— into a live system without installing
over it.

**Kea** — ISC's modern DHCP server,
OPNsense's current-generation option.

**LAN / WAN** — the local (trusted) and
wide-area (internet-facing) networks; in
OPNsense, interface roles every
configuration starts with.

**Live environment** — the system booted
from install media before installation:
fully working, read-only, forgetful at
reboot.

**Nano image** — the preinstalled
embedded image meant to be written
straight to a card or stick: the system
is already on it, logs live in memory,
and it grows to fit its media.

**NAT** — network address translation:
rewriting addresses (and ports) as
packets cross the firewall. Outbound /
source NAT shares one public address
inward-out; a port forward (destination
NAT) aims one outside port at one inside
machine; one-to-one NAT and NPTv6 map
addresses individually.

**netmap** — FreeBSD's fast packet path
used by Suricata for inline inspection.

**pf** — the stateful packet filter from
OpenBSD, in FreeBSD's kernel, that
enforces OPNsense's rules and NAT.

**pfsync** — the protocol that copies the
firewall state table between an HA pair,
so failover keeps connections alive.

**Plugin** — an installable extension
package, named `os-*`; support ranges
from core-team (tier 1) to community and
third-party (tier 4).

**Port forward** — see NAT.

**Quick** — a rule property: if this rule
matches, stop processing further rules.
The default; the opposite is "last match
wins".

**Reject vs block** — two ways to say no:
reject tells the sender (fast failure,
polite inside); block drops silently
(correct toward the untrusted outside).

**State** — the firewall's memory of an
allowed conversation, which lets replies
pass without re-asking the rules.

**strongSwan** — the IPsec implementation;
its modern configuration form, used by
OPNsense, is `swanctl` Connections.

**Suricata** — the intrusion detection
and prevention engine (chapter 13).

**UFS** — the Unix File System, FreeBSD's
native filesystem; the nano image's root
is a UFS2 filesystem inside a BSD
disklabel.

**Unbound** — the validating, recursive,
caching DNS resolver that is OPNsense's
standard DNS service.

**VHID** — virtual host ID: the number
identifying a CARP shared address on a
network segment; unique per segment.

**Virtual IP (VIP)** — an address a
machine answers for in addition to its
own; CARP VIPs move between the HA pair.

**Wizard** — the GUI's first-run setup
sequence (general settings, time, WAN,
LAN, admin password, reload), triggered
automatically on first login by a marker
in the factory configuration.

**XMLRPC sync** — the one-way copying of
configuration sections from an HA master
to its backup.
