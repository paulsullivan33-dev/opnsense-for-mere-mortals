# VPNs: tunnels home and between sites

A VPN — virtual private network — is an
encrypted tunnel across an untrusted
network, treated by both ends as if it
were a cable. OPNsense is a natural place
to end tunnels: it already sits on the
border, holds the keys, and knows the
routes. This chapter surveys the three
systems in the box, the two shapes of
tunnel people build, and what the manuals
say about each. For the WireGuard
protocol itself, the series' *WireGuard
for Mere Mortals* guide goes deep; here
the subject is OPNsense as the endpoint.

## The two shapes

- **Road warrior (remote access).** One
  laptop or phone, out in the world,
  tunnels back to the firewall and becomes
  — as far as the LAN can tell — a device
  at home. For reaching the file server
  from a hotel, and for not trusting hotel
  Wi-Fi with anything at all.
- **Site to site.** Two firewalls, two
  networks, one permanent tunnel: the
  branch office's 192.168.2.0/24 and
  headquarters' 192.168.1.0/24 become one
  routable whole.

Every system below can build both; they
differ in ceremony.

## IPsec: the enterprise classic

OPNsense's IPsec is **strongSwan**,
configured in its modern form:
**Connections**, written underneath as a
`swanctl.conf` file. The vocabulary has
been deliberately aligned with
strongSwan's own:

- A **connection** (the old "phase 1")
  sets the endpoints, authentication
  (certificates or pre-shared keys, kept
  in their own Pre-Shared Keys and Key
  Pairs stores), and the encryption
  proposals — the manual's example uses
  `aes256-sha256-modp2048`.
- **Children** (the old "phase 2") name
  the networks each end will send through
  the tunnel: `local_ts` and `remote_ts`
  traffic selectors.
- Two implementation styles: **policy
  based**, where matching traffic is
  trapped into the tunnel by policy (no
  routes to manage — the easy default),
  and **route based (VTI)**, where the
  tunnel is a virtual interface and normal
  routing applies (more flexible, more to
  understand; the manual is careful to
  note that filtering then happens on the
  `IPsec` interface group, not the VTI
  interface itself).
- **Dead peer detection** exists but is
  off until you set a DPD delay above
  zero — a tunnel to a site that vanishes
  will otherwise be mourned slowly.

IPsec remains the lingua franca for
talking to other vendors' firewalls and
to cloud providers, which is its real
recommendation: when the other end is not
yours to choose, IPsec is usually what it
speaks.

## OpenVPN: the certificate workhorse

OpenVPN — the SSL VPN — is the traditional
road-warrior answer: a server on the
firewall, a certificate per user (created
and revoked in OPNsense's own trust
store), and a client configuration export
utility the VPN manual singles out as
making "client configuration a breeze".
Users authenticate with certificates,
optionally plus their firewall account,
and land on a tunnel network you assign.
It is more machinery than WireGuard and
more forgiving of odd networks — it can
run over TCP port 443 and cross proxies
that strangle UDP tunnels.

## WireGuard: the small modern one

WireGuard is in the core — it has its own
configuration model (`Wireguard`) in the
source tree, not merely a plugin. Its
shape will be familiar from its guide:
each side has a key pair; an **instance**
on the firewall holds the firewall's key
and a listening port; each **peer** is a
public key plus the list of addresses
that peer may claim (its AllowedIPs),
which is simultaneously its route and its
identity check. No certificates, no
negotiation, a configuration that fits on
an index card — at the price of no
built-in user accounts: a peer is a key,
and revoking a person means removing
their key.

For a personal tunnel home, this is the
usual modern recommendation. For a
fleet of employees, the certificate
systems' revocation lists may argue the
other way. (ZeroTier, the overlay network
that finds its own paths, is available as
a plugin — chapter 14.)

## Whichever system: the firewall still applies

A tunnel ending on the firewall is not
yet permission to go anywhere. Tunnelled
traffic surfaces on an interface — the
WireGuard instance's interface, the
`IPsec` group, an OpenVPN interface —
and chapter 9's rules decide what it may
touch. The discipline is the same as for
any network: give the tunnel the smallest
reach that does the job. "Road warriors
may reach the file server and the printer,
nothing else" is three rules, and it is
the difference between a lost laptop being
an inconvenience and being a skeleton key.

Also budget the CPU: encryption is per
packet work. On the recommended hardware
tier (chapter 4) a household's tunnels
are nothing; a site-to-site link expected
to run at full gigabit wants AES-NI and a
second thought.
