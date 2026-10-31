# Plugins and updates

Two kinds of change keep a firewall
alive: **plugins** add jobs to it, and
**updates** keep the jobs it has from
going stale. This chapter covers both —
the plugin system and its support tiers,
and the release train you are signing up
for when you install the Community
Edition.

## Plugins: the firewall as a platform

A plugin is a FreeBSD package — every one
is named `os-<something>` — installed
from **System ▸ Firmware ▸ Plugins** (or
with `pkg` at the shell). Installing one
adds its pages to the GUI, its model to
the configuration, and its service to
configd's repertoire, in one move. The
official plugins repository is a public
tree; this guide's lab counted **102
plugins** there, in fourteen categories —
the largest being networking (27) and
security and system utilities (20 each).
A fair sample of what people actually add:

- **os-zerotier** and **os-tailscale** —
  overlay VPN clients (chapter 12's
  alternatives), so the firewall itself
  joins those networks.
- **os-sunnyvalley** — the vendor
  repository for Zenarmor, a third-party
  traffic-inspection and filtering layer.
- **os-haproxy**, **os-nginx**,
  **os-acme-client** — reverse proxies and
  automatic Let's Encrypt certificates, for
  publishing internal services properly.
- **os-dyndns**-class helpers,
  **os-netdata**-style reporting,
  **os-qemu-guest-agent** for VM installs,
  and the **os-firewall-legacy** plugin
  carrying the old rule pages for those
  migrating from them.

### The tiers, stated before you depend on one

The project's support page is unusually
plain about what a plugin promises:

- **Tier 1 (critical)** — the core team
  develops and supports it; failures block
  releases. The standard install and
  officially supported plugins live here.
- **Community tiers** — valuable, widely
  used, and *not* the core team's
  responsibility ("you may safely assume
  the core team has no (or very limited)
  involvement", says the same page, of
  the community tier). Popular plugins can
  be promoted when their quality,
  maintainability and documentation earn
  it.
- **Tier 4** — development versions and
  any third-party repository you add
  yourself: your risk, entirely.

Read that as a design rule: the border of
your network should run tier-1 machinery
where it can, and a community plugin
should be a reasoned adoption — read its
page, check its pulse, know whose name is
on it — not a reflex.

## Updates: the train schedule

The updates manual states the rhythm in
its first paragraph: **two major releases
a year**, with **minor updates about
every two weeks**. Versions are named for
their moment — 26.7 is the July 2026
major release; the fortnightly minors add
a third number (26.7.3 is the third minor
update to it). Images for a new install
are published on that January/July
schedule; an installed system walks
forward from any image by updating, and
the project advises updating to the
latest version immediately after
installing.

### Doing an update

- **From the GUI:** System ▸ Firmware ▸
  Status, **Check for updates**, then
  install what is offered. The settings
  page beside it chooses the **mirror**
  and the **release type** — Production
  (the tested fortnightly line) or
  Development (the newest, not fully
  tested code; the manual asks you
  plainly to leave it on Production
  unless you fully understand the
  implications). The same page offers
  one-click **audits** — connectivity,
  health, security — worth meeting before
  you need them.
- **From the console:** menu option **12**
  upgrades from the console; the
  `opnsense-update` tool underneath it
  has a man page and scriptable flags.
- **Major upgrades are offline events.**
  The manual warns: during a major
  upgrade no web or SSH server runs for
  you to watch; do it at the console
  (VGA or serial), and have a snapshot or
  a config backup (chapter 8) as your way
  back if it fails. The updater downloads
  everything first, reboots, installs,
  and reboots again onto the new release.

### The update discipline this implies

A firewall that updates fortnightly is a
firewall that is *maintained* — that is
the Community Edition's side of the
bargain, and the reason the Business
Edition's slower, more conservative track
exists for those who would rather wait.
Practical rhythm that works for homes and
small offices: take the minor updates
when offered (they carry the security
fixes the fork chapter mentioned), read
the release notes for majors, back up the
configuration first, and do majors at the
console on a day when an hour of downtime
is an inconvenience rather than an
incident. On a CARP pair, chapter 15
gives the rolling procedure that makes
even majors nearly lossless.
