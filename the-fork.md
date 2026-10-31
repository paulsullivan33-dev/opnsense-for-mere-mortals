# The fork: OPNsense and pfSense

Sooner or later every OPNsense reader asks
the obvious question: isn't this pfSense?
The honest answer is "it was, in 2015 —
and the two have been separate projects,
with separate code, ever since". This
chapter tells the story once, fairly, so
the rest of the guide can get on with the
software. Both projects are real, both are
still developed, and people choose each
for defensible reasons.

## The family tree

- **m0n0wall** (2003) pioneered the shape:
  a small FreeBSD firewall configured
  entirely from a web page, with its whole
  configuration in one XML file. If that
  shape sounds familiar from chapter 2, it
  should — it is the ancestor of both
  systems below.
- **pfSense** (2004) began as a fork of
  m0n0wall aimed at fuller hardware than
  the embedded boards m0n0wall targeted.
  It grew into the best-known open source
  firewall, and was later acquired by
  **Netgate**, the company that now
  develops it.
- **OPNsense** (2015) is a fork of
  pfSense, made by **Deciso**, a Dutch
  company that had sold and supported
  pfSense in Europe for years.

## Why it happened, in the project's own words

Deciso published its reasons, and they are
worth meeting directly rather than in
rumour form. The project's history page
lists four:

- **Technical** — dissatisfaction with
  code quality and scattered development;
  a wish for structure, a roadmap and
  regular releases.
- **Security** — the web interface should
  not itself perform tasks needing root;
  logic should be separated so problems
  are caught before they become
  vulnerabilities. (Chapter 2's configd
  exists because of this paragraph.)
- **Quality** — new work to be built on a
  proper framework with a
  model-view-controller shape, migrating
  old code gradually rather than in one
  risky rewrite.
- **Transparency** — discomfort with the
  direction of pfSense under its new
  owner: development tools removed from
  public view without warning, brand used
  against competitors, and a licence
  change "for no apparent reason".

The same page states the outcome plainly:
today **less than 10% of the original
legacy code base remains**. Whatever
OPNsense was at the fork, it is now its
own platform — its own GUI framework, its
own API, its own release train — that
happens to share an ancestor and, still,
much of its vocabulary with pfSense.

To be fair in the other direction: the
licence and governance arguments were the
fork's view, argued at the time; Netgate
has continued to develop pfSense
throughout, publishes its own roadmap,
and pfSense remains a capable, widely
deployed system. The fork was a genuine
disagreement about how an open project
should be run, not the rescue of an
abandoned one.

## What is actually different today

For a reader choosing between them, the
differences that survive contact with
daily use:

- **Licence and editions.** OPNsense is
  under the two-clause BSD licence, with a
  free Community Edition and a paid
  Business Edition that mainly changes the
  update track and support. pfSense comes
  as the free pfSense CE and the commercial
  pfSense Plus, which is what Netgate's
  own appliances ship and where some of
  its newer features land first.
- **Release rhythm.** OPNsense publishes
  two major releases a year (January and
  July) with smaller updates about every
  two weeks, and says so in its manual.
  pfSense releases on Netgate's schedule,
  historically less frequent.
- **Interface and API.** OPNsense's GUI is
  the newer framework with a documented
  REST API covering most of the system.
  pfSense's GUI is the older lineage;
  its coverage and feel differ page by
  page.
- **Configuration.** Both still keep the
  m0n0wall inheritance — one XML file —
  and both can import each other's files
  only in the loosest sense: expect a
  migration to be a rebuild with the old
  file as a reference, not a restore.
- **Hardware.** Both run on ordinary
  x86-64 hardware; Netgate and Deciso each
  sell appliances with their system
  preinstalled, and buying from either
  funds the project behind it.

## So which one?

This guide obviously teaches OPNsense,
but the fair summary is: the two systems
do the same job, share a dialect, and
differ in licence, release cadence,
interface, and the company behind them.
Someone with a working pfSense and no
grievance has little reason to move;
someone starting fresh, or who values the
API, the faster update train and the
permissive licence, lands where this guide
lives. Anyone migrating either way should
plan it as a fresh installation with the
old configuration open beside them —
chapter 8's backup machinery makes that
painless to *prepare*, even though the
file itself is not portable.
