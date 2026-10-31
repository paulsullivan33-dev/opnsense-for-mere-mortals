# Firewall rules

Everything so far has been preparation.
This chapter is the firewall itself: the
rules that decide, for every packet, pass
or block. The engine underneath is
FreeBSD's `pf`; what OPNsense adds is an
order, a vocabulary and an editor. If you
finish this chapter able to predict what a
given rule set does to a given packet —
and able to find the rule that fired in
the log — you have the core skill of the
whole system.

## The one-paragraph model

A packet arrives on an interface. The
firewall asks three questions in order:
*Have I seen this conversation before?*
(a state). *Does a rule claim this
packet?* (the rule list, in a fixed
processing order). *If no rule claims
it?* — it is dropped: the built-in
posture is **default deny**. Your rules
are therefore mostly a list of *allowed*
things, written against a background of
"no".

## States: why replies just work

Rules are **stateful** by default. When a
packet is allowed, the firewall records
the conversation — addresses, ports,
protocol, and for TCP the sequence
bookkeeping — in a state table. Later
packets belonging to that conversation
match the state, not the rule list, and
pass without re-asking. Two consequences
worth internalising:

- You write rules for the direction a
  conversation *starts*. A LAN user
  browsing the web needs one rule allowing
  the request out; the replies ride the
  state home. There is no "allow answers
  back in" rule, and adding one is a
  classic beginner's hole.
- States are also a security property:
  for TCP, sequence numbers and timestamps
  are checked, so an outsider cannot
  easily inject a packet into someone
  else's conversation.

States cost memory — about 1 kB each, with
a tenth of system memory reserved by
default — and when you change rules,
existing conversations keep their old
treatment until their states expire or
you reset them (the manual notes this
under Firewall ▸ Diagnostics ▸ States;
it is the first thing to check when a new
block "isn't working").

## Anatomy of a rule

Every rule in the shipped configuration —
we will meet a real one in a moment —
answers the same questions:

| Field | Question it answers |
|---|---|
| Action | Pass, **block** (drop silently), or **reject** (drop, but tell the sender) |
| Interface | Which interface's traffic this is about |
| Direction | **In** (arriving on that interface) or out (leaving by it) |
| Protocol | TCP, UDP, ICMP, any… plus, for TCP/UDP, ports |
| Source | Who sent it — an interface's network, an address, an alias, "any" |
| Destination | Who it is for, likewise |
| Quick | Stop at this rule if it matches (the default) — or let later rules still speak |
| Log | Write a line in the firewall log when this rule fires |

Two style points from the manual: use
**block** toward untrusted networks (do
not confirm a scanner's guesses), and
**reject** inside your own networks, so a
misdirected laptop learns "no" at once
instead of waiting out a timeout.

**Direction is the mental leap.** The
default and recommended way is to filter
on **in**, on the interface where traffic
*originates*. "Let the office PCs reach
the printer" is a rule on the LAN
interface, direction in, source LAN net,
destination the printer. Traffic leaving
the firewall itself is accepted by
default. One rule, written where the
conversation starts — the same principle
as states.

## The order rules are heard

Rules are not simply read top to bottom in
one global list. Processing goes by
section, and the manual gives the order
(automatic system rules first of all):

1. **Floating rules** — rules attached to
   several interfaces, or none in
   particular. Your cross-cutting policy
   ("nothing may ever reach the camera
   network") lives here.
2. **Interface group rules** — groups are
   named bundles of interfaces that share
   policy.
3. **Interface rules** — the per-interface
   lists, LAN, WAN and the rest.

Within that, each rule carries a **sort
order** built from a priority group
(floating 200000, groups 300000,
interfaces 400000) plus a sequence number
the GUI manages as you move rules around.
And with **quick** — the default — the
*first* matching rule decides. A non-quick
rule is a standing opinion ("log
everything to the NAS") that lets later
rules still have the final word; the
system's own default-deny is such a rule,
sitting at the end: if nothing else
claimed the packet, last match wins, and
it says no.

**NAT is heard before all of this.**
Chapter 10's warning belongs here too:
address translation is processed before
filter rules, and a port forward created
with its filter association set to "pass"
lets the traffic through with it — no
other rule gets a say. Convenient, and
worth knowing you agreed to.

## Proof by example: the two rules you are born with

The factory configuration contains exactly
two user rules, both on the LAN, read
verbatim from `config.xml.sample` in the
project's source (chapter 6 shows the
file itself):

1. *Default allow LAN to any rule* —
   action pass, quick, interface LAN,
   direction in, IPv4, protocol any,
   source "LAN net", destination any.
2. The same again for IPv6.

That is the whole default policy: **the
LAN may start any conversation; the WAN
may start none** (no WAN rule exists, and
default deny does the rest — reinforced by
the WAN interface's shipped options to
block private addresses and bogon
networks arriving from outside). A home
user can run for years inside those two
rules. Everything beyond them — locking a
games console to one service, keeping
smart bulbs away from the laptops, giving
guests internet but not the file server —
is you, editing this list, one intention
at a time.

## Working habits the manual teaches

- **Aliases** are named lists — hosts,
  networks or ports — maintained in one
  place (Firewall ▸ Aliases) and used in
  many rules, so "the kids' devices" is a
  list you edit, not a dozen rules.
- **Categories** tag rules into folders
  for your own navigation; they change
  the view, never the processing order.
- **The Inspect view** (the rules page's
  inspection mode) reveals the automatic
  rules the system adds for itself, with
  per-rule statistics — states created,
  packets matched. When reality disagrees
  with your reading of the list, Inspect
  plus the live log (Firewall ▸ Log Files)
  is how you find out which rule actually
  fired.
- The generated ruleset itself can be
  inspected on the machine at
  `/tmp/rules.debug` — the plain text `pf`
  will load, for the day the GUI's
  presentation stops making sense.

## A first edit, described

The classic first custom rule — "the
guest network gets the internet, never
the LAN" — is, in this vocabulary: a
block rule on the GUEST interface,
direction in, source GUEST net,
destination LAN net, placed *above* the
pass rule that allows GUEST to any (or a
floating quick rule saying the same for
every internal destination). Save, apply,
reset the affected states, and test from
a guest device: the printer refuses, the
web works. That round trip — intention,
rule, apply, state reset, verify from the
affected device, read the log line — is
the loop every later chapter assumes you
can walk.
