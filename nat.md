# NAT and port forwards

If chapter 9's rules are the border's
law, NAT — network address translation —
is its geography: the rewriting of
addresses as packets cross, which is what
makes "one public address, a houseful of
private ones" work at all. This chapter
covers the four kinds OPNsense offers,
the one that is already running before
you touch anything, and the two warnings
that save the most grief.

## Why translation exists

Your LAN uses private addresses —
192.168.1.0/24 in the factory defaults.
Those addresses are meaningless on the
public internet: no router out there will
deliver a reply to 192.168.1.50. So as a
packet leaves, the firewall rewrites its
source to the WAN's public address and
remembers, in the state table, which
internal device the conversation belongs
to. Replies come back to the public
address and are rewritten home. That is
**source NAT** (outbound NAT), and it is
why the whole street can share a handful
of public addresses without knowing it.

## The NAT that is already on

You installed nothing, and outbound NAT
is already correct: the factory
configuration sets outbound mode to
**automatic**, and the manual describes
automatic as the default that "is suitable
in most cases" for a single external
address. OPNsense generates the rewriting
rules itself — every internal network
gets translated to the WAN address as it
leaves.

The other modes, from the same page:

- **Hybrid** — automatic rules plus your
  own manual ones on top. This is where
  most growing networks land.
- **Manual** — no automatic rules; you
  write every translation. Needed when
  outbound traffic must leave from a
  *specific* address — the CARP chapter
  (15) depends on exactly this, so a
  failover pair presents one shared
  address to the world.
- **Disabled** — no source NAT at all;
  the manual's example is a transparent
  bridge, where the firewall filters
  without being an address boundary.

## Port forwards (destination NAT)

Outbound translation has a shadow side:
nobody outside can *start* a conversation
inward, because the firewall has no state
saying where an unsolicited packet for
its public address should go. A **port
forward** is the deliberate exception:
"connections arriving at the WAN address
on port 443 go to the web server at
192.168.1.60". The manual's table for a
forward asks for three addresses —
**source** (who may connect; usually
"any", and narrowing it is the cheapest
security there is), **destination** (the
outside address and port being knocked
on), and **redirect** (the inside machine
and port that should actually answer).

Two couplings to understand:

- **The linked firewall rule.** A forward
  can create its matching pass rule for
  you (the manual's "filter rule
  association"). Set it to *pass* and the
  forward itself waves traffic through —
  recall chapter 9's warning that NAT is
  processed *before* filter rules, so with
  that setting no other rule gets a say.
  Set it to *none* and you write the rule
  yourself, gaining control (logging,
  source restrictions, schedules) at the
  price of a second step. For anything
  beyond a quick test, the manual
  recommends the explicit rule for complex
  setups; this guide recommends it as the
  default habit.
- **Reflection (hairpin NAT).** Inside
  users who knock on the *public* address
  — the name they use from a café — get
  nowhere unless the firewall rewrites
  that too, looping the request back
  inside. That is NAT reflection. The
  manual notes its generated rules cover
  only directly connected networks, and
  the factory configuration is itself
  opinionated here: `disablenatreflection`
  is set to *yes* in the shipped defaults,
  so reflection starts off. The cleaner
  home answer is usually split DNS
  (chapter 11's overrides give the inside
  name the inside address); reflection is
  for the cases that cannot do that.

## The other two kinds

- **One-to-one NAT** maps a whole outside
  address to a whole inside address, in
  both directions — for the organisation
  with a block of public addresses that
  wants each server to live at its own.
  If you have one public address, you
  will not meet it.
- **NPTv6** is the IPv6 counterpart idea:
  prefix-to-prefix translation, strictly
  one-to-one, used to map global IPv6
  prefixes to local ones. (Consumer IPv6
  usually wants the opposite — no
  translation at all, with the LAN
  tracking the WAN's delegated prefix,
  which the factory LAN configuration
  already does via its track-interface
  setting.)

## NAT is not a security feature

The NAT manual opens with a warning worth
quoting in spirit: **NAT should not be
relied upon as a security measure.** The
protection on your border is the stateful
filter and its default deny (chapter 9),
not the translation. Translation happens
to make unsolicited inbound traffic
undeliverable, but the day you add a port
forward — or an IPv6 prefix with no NAT
at all — the filter is what stands
between the internet and your server.
Design as if NAT were invisible, because
to a determined packet, it is.
