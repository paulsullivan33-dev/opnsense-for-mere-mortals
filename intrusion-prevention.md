# Intrusion prevention

Rules decide who may talk. **Intrusion
detection** reads what is said: it
inspects passing traffic against a large
database of known-bad patterns — exploit
attempts, malware phoning home, scanners'
fingerprints — and raises an alert when
something matches. **Intrusion
prevention** takes the one step further
that changes the name: it sits *in* the
path and can drop the packet. OPNsense
builds both on **Suricata**, and this
chapter explains the machinery and then
spends equal time on the honest costs,
because IPS is the feature most often
enabled in enthusiasm and disabled in
experience.

## How it fits in the path

Suricata on OPNsense can capture three
ways (the settings page's Capture mode):

- **PCAP live (IDS)** — a copy of the
  traffic, inspected aside. Alerts only;
  nothing is ever dropped, and the
  forwarding path never waits for
  inspection. The safe first step.
- **Netmap (IPS)** — inspection inline
  through FreeBSD's netmap, which hands
  packets to Suricata at high speed; a
  rule whose action is *drop* discards
  the packet.
- **Divert (IPS)** — firewall rules with a
  *divert-to* setting (chapter 9) hand
  selected traffic to Suricata and take
  the verdict back — finer-grained: only
  the flows you choose pay the inspection
  cost. The manual adds the sharp edge:
  if the listening service stops, the
  diverting rule drops its packets.

IPS mode needs the hardware offloading
features on the interface **disabled** —
the manual is explicit — and, for VLANs,
runs on the parent interface.

## Which interface, and why the manual argues

Choose **LAN, not WAN** — the manual
spends real effort here, and the argument
is worth following. Outside the WAN, NAT
has already rewritten every address to
the firewall's own: an alert would blame
the firewall for everything, and the
rules' notion of a "home network"
($HOME_NET, usually the RFC1918 space) is
defined in terms of the addresses only
visible inside. Besides, traffic the
firewall would drop anyway gains nothing
from being inspected on its way to being
dropped. Inside, on the LAN, Suricata
sees the real machines and can protect
them from each other and catch the
already-infected device phoning out —
which is where home and small-office
value actually lives.

## Rulesets and policies

The engine ships empty; the teeth are
downloaded **rulesets**:

- **ET Open** (Emerging Threats, from
  Proofpoint) — free, BSD-licensed, and
  integrated into OPNsense directly. The
  manual is candid that it "is not a full
  coverage ruleset" and may not satisfy
  regulated environments alone.
- **ET Pro** — the commercial sibling,
  unlocked with a subscription code via
  the `os-intrusion-detection-content-et-pro`
  plugin.
- **abuse.ch** lists — community
  blacklists with narrow, high-signal
  jobs: SSL certificate blacklist, Feodo
  botnet trackers, the URLHaus malware
  URL list, and app-detection rules.

Downloaded rules default to *alert*.
**Policies** (the mechanism since 21.1)
decide in bulk which rulesets, categories
or matched metadata become *drop*,
stay *alert*, or are disabled — plus
per-rule adjustments for the exceptions.
The Alerts tab shows what fired, with the
rule and the packet details, and alerts
can be shipped out as syslog in
Suricata's EVE format for anyone running
a log collector.

## The honest costs

- **Speed.** Inline inspection is CPU
  work on every packet. The pattern
  matcher matters (the manual recommends
  Hyperscan where the platform offers it,
  else the Ken Steele Aho–Corasick
  variant), and chapter 4's throughput
  table quietly assumes you are *not*
  doing this. Expect a real reduction in
  top-end throughput; measure yours after
  enabling, not before.
- **Attention.** A ruleset is a feed, not
  a fact: patterns misfire on innocent
  traffic, and every misfire in *drop*
  mode is a self-inflicted outage with an
  innocent cause ("the printer firmware
  update looked like an exploit"). The
  working rhythm is IDS first, read the
  alerts for some weeks, promote to drop
  only the rulesets whose alerts proved
  true.
- **Encrypted traffic is opaque.** Most
  of what crosses a modern network is
  TLS. Suricata reads handshakes, server
  names and certificate fingerprints —
  enough for the abuse.ch SSL blacklist
  and some real detections — but not
  content. IPS is a tripwire, not an
  X-ray.
- **Disk and care.** The alert database
  is why chapter 4's minimum tier excludes
  IPS; rulesets also update on a schedule
  you now own.

None of this argues against Suricata. It
argues for enabling it the way the manual
quietly steers you: detection first, on
the LAN, with rulesets whose names you
can explain.
