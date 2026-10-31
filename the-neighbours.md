# The neighbours

OPNsense is one answer to "what should
guard the border?" This chapter lines up
the others honestly — what each does that
OPNsense doesn't, and vice versa — so the
choice is a comparison, not a reflex.

## pfSense

The sibling (chapter 3 tells the story).
Same m0n0wall inheritance, same job,
same one-file configuration idea.
pfSense does it with a longer commercial
history, Netgate's appliance line and
pfSense Plus tier, and an enormous
accumulated corpus of forum answers for
every error message you will ever meet.
OPNsense answers with the permissive
licence, the faster public update train,
the framework GUI and the API. Neither
embarrasses the other at the actual work
of filtering packets.

## OpenWrt

The series' *OpenWrt for Mere Mortals*
guide covers the other great open router
project. OpenWrt is Linux firmware that
runs *on the router you already own* —
hundreds of consumer devices — and turns
it into a proper, scriptable router with
its UCI configuration system. Its centre
of gravity is lower and wider than
OPNsense's: smaller hardware, real Wi-Fi
support (it is the Wi-Fi firmware of
choice), one box doing routing and radio
together; its firewall and services are
capable but artisanal next to OPNsense's
GUI, models and reporting. Rule of thumb:
one consumer box, Wi-Fi included →
OpenWrt. A dedicated x86 box, services
and scrutiny → OPNsense. Many networks
run both: OpenWrt access points behind an
OPNsense border.

## Your provider's router, and consumer mesh

The default answer, and not a stupid one:
it is free, supported by someone else,
and its Wi-Fi is usually excellent. What
it does not give you is this guide's
subject matter at all — rules you can
read, logs you can search, updates on a
schedule you can see, a VPN that ends in
your house under your keys, or a
configuration you own as a file. People
leave it behind at the moment one of
those becomes a need rather than a taste.

## Linux firewalls you assemble yourself

nftables on Debian, a hand-rolled
`iptables` script, or a distribution like
IPFire or VyOS pre-assembles the same
idea on Linux. The assembled-yourself
route wins on familiarity (your existing
Linux skills transfer whole) and loses on
exactly what OPNsense is *for*: the
integrated model — one config file, a
GUI and API generated from it, configd
applying changes, updates as a train
rather than a hobby. Sophos's and
Fortinet's free home tiers sit at the
commercial end of this shelf: polished,
and tenants of someone else's licence.

## The cloud-managed stacks

UniFi and its kin move the border into a
vendor's controller and cloud account:
beautiful dashboards, one pane for
switches and access points and gateway
together, zero interest in showing you a
rule as text. For networks that are
mostly *kit* to be adopted and watched,
that is genuinely attractive — and it is
the opposite trade from OPNsense's: your
configuration as their data, on their
update schedule, under their account
recovery process.

## The one-line summary

Choose OPNsense when you want the border
run as *infrastructure you own*: open
source, inspectable, scriptable,
redundant if you buy it twice. Choose
otherwise when the border is an appliance
problem (mesh, UniFi), a radio problem
(OpenWrt), or somebody else's problem
(provider router) — and revisit when it
becomes an ownership problem again.
