# Writing criteria for this guide

Every "for Mere Mortals" guide is written to the same nine-point
standard, so a reader who trusts one can trust the rest. Here it
is, with the verification notes for this particular guide below.

1. **Fix it first.** Reader outcomes lead, not action items: each
   chapter opens by naming the outcome it delivers, shows a check
   the reader can run to confirm they got it, and a reasonable
   reader finishes able to state what was done, what it means,
   and what to try next.
2. **Plain language.** Short sentences, everyday vocabulary,
   technical terms defined on first use. No slang, and no
   idioms that only make sense in one country — the reader may
   be translating as they go.
3. **No dead ends.** A guide never tells the reader to do
   something it cannot show them how to verify. Every claim a
   reader relies on — an install, a command, an output — is
   something this guide demonstrates or clearly labels
   documented-only below.
4. **Live-verified where it matters.** Procedures are run for
   real in the guide's lab wherever the build machine allows it,
   and the transcripts kept in the repository are the evidence.
   What could not be run is listed honestly below, never
   presented as if it had been.
5. **General audiences.** Nothing in this guide — examples,
   fixtures, names, addresses — is modelled on any particular
   reader or author's real life, network, or organisations.
   The lab's example networks are the factory defaults
   (192.168.1.0/24) and the documentation's own worked example
   (172.18.0.0/24 for the CARP chapter).
6. **Standalone.** A reader needs this guide, its lab, and
   nothing else. Where another guide would genuinely help, it
   is pointed to once, by name, and the chapter still completes
   without it.
7. **Honest about limits.** Chapter 17 names what the system
   cannot do: x86-64 only in practice, Wi-Fi that is not its
   job, hardware support bounded by FreeBSD's drivers, offline
   console-first major upgrades, a configuration file that does
   not cross the fork, and a lab that never booted the system.
8. **The safety rules are in the text, early.** A firewall is
   other people's connectivity: chapter 1 puts the warning
   before any procedure (keep a way back — old router, config
   backup, tested console), chapter 5 makes download
   verification a procedure rather than a tip, and chapter 7
   teaches the recovery doors (Importer, console menu) before
   the installer's point of no return.
9. **Builds clean.** The site builds with MkDocs in strict mode
   with no warnings, and a fresh reader following the chapters
   in order reaches the stated outcome of each.

## Verification notes for this guide

The build machine is a shared Ubuntu 24.04 sandbox. **It cannot
boot another operating system** — no virtual machines, no way to
run a FreeBSD kernel — so no OPNsense system was ever started
for this guide, and nothing in it claims otherwise. The core of
the guide (installer behaviour, the setup wizard, the dashboard
and every GUI workflow) is therefore **documented-only**, taught
from the project's official documentation at docs.opnsense.org;
the exact pages read are listed in `opnsense-lab/sources.md`.

What the lab *could* do, it did for real, and the transcripts in
`opnsense-lab/transcripts/` are the evidence:

- **The artefact.** From the official mirror
  (mirror.ams1.nl.leaseweb.net), the current release directory
  `26.7/` — the mirror README in that directory states the
  latest stable release image is 26.7 (July 15, 2026) and the
  next scheduled image release is 27.1 (January 2027). The lab
  downloaded the smallest image, `OPNsense-26.7-nano-amd64.img.bz2`
  (490,849,116 bytes), and the release's checksum and signature
  files (t00).
- **Checksum.** The download's SHA-256
  (`28d5e2f3…c84d0cb9d`) matches `OPNsense-26.7-checksums-amd64.sha256`
  exactly (`sha256sum -c` → OK). The checksums file was also
  fetched from a second mirror (mirror.fra10.de.leaseweb.net)
  and found byte-identical (t00).
- **Signatures.** The release public key `OPNsense-26.7.pub`
  (4096-bit RSA) was compared byte for byte with the key
  printed in the mirror's own README for 26.7: identical. With
  it, following the install manual's procedure (the `.sig`
  files are base64; the image signature is over the
  *uncompressed* image): the checksums file's signature →
  **Verified OK**; the public key file's own signature →
  **Verified OK**; the nano image's signature over the
  decompressed image → **Verified OK** (t01). The decompressed
  image is 3,221,225,472 bytes, SHA-256 `c376b498…ef30d37fcb98`,
  and the signature check is itself the proof the decompression
  was faithful. (One caution the lab earned and chapter 5
  repeats: an early check against a *still-decompressing* file
  failed, as it should — signatures are only meaningful against
  finished files.)
- **Anatomy.** The image's MBR (one bootable entry, type 0xa5),
  its BSD disklabel (magic verified; 6,291,456 sectors; partition
  `a` = the whole disk, fstype 7/UFS) and the UFS2 superblock
  magic at byte 66,908 (64 KiB + 0x55c, exactly as laid out)
  were parsed and confirmed with the lab's own scripts. 7-Zip
  23.01 cannot open the image ("Cannot open the file as
  archive"), and the lab has no loop device or UFS reader, so
  **no file listing of the image was obtained and none is
  claimed** (t02).
- **The factory configuration.** Shallow clone of
  `opnsense/core` at commit `50105e67ed9ea901d79442804e573bbaf9d9b03b`
  (2026-10-30). `src/etc/config.xml.sample` (172 lines) was read
  entire: every default quoted in chapters 6, 9, 10 and 11 —
  hostname/domain/timezone, WAN by DHCP with blockpriv and
  blockbogons, LAN 192.168.1.1/24 tracking the WAN prefix,
  Dnsmasq on port 53053 serving 192.168.1.100–199, Unbound
  enabled, outbound NAT automatic with reflection disabled, web
  GUI HTTPS with `noroot`, the `<trigger_initial_wizard/>`
  marker, and the two "Default allow LAN to any" rules — comes
  from that file (t03). Also read in the tree: the MVC app
  config naming `configDir => '/conf'`, the config loader's
  last-known-good and factory fallback behaviour, the factory
  reset's removal of `/conf/*`, the 25 MVC subsystem models and
  the 31 configd action files.
- **How images are signed.** In `opnsense/tools` (shallow
  clone), `build/common.sh`'s `sign_image` signs with
  `openssl dgst -sha256 -sign` and base64-encodes the result,
  and `build/nano.sh` shows the nano's construction: `makefs`
  (3 GiB), `gpart create -s BSD`, sign (t03).
- **Plugins.** The `opnsense/plugins` tree (via the GitHub
  API) contains **102 plugin directories** with a Makefile, by
  category: net 27, security 20, sysutils 20, net-mgmt 8,
  www 7, misc 6, dns 4, devel 3, mail 2, and one each of
  benchmarks, databases, emulators, ftp and vendor
  (t03).

Documented only — described from the project's documentation
(docs.opnsense.org, pages listed in `opnsense-lab/sources.md`),
not run, and the text does not claim otherwise:

- The boot process, the configuration Importer in action, the
  live environment, the installer, interface assignment, the
  console menu in use, the setup wizard's screens, the
  dashboard and all GUI pages and workflows.
- The wizard's step list is documented in the project's
  ecosystem (the manual confirms the first login triggers it
  via the marker found in the source; the step names are those
  of the wizard as walked in the project's documentation
  orbit). Menu paths in the text follow the manual's own
  references.
- Firewall rule processing, NAT modes, DHCP/DNS operation,
  VPN setup, Suricata's modes and rulesets, plugin
  installation, firmware updates, and the CARP failover
  behaviour — including the how-to's cable-pull test, which is
  reported as the project's test, not the lab's.
- Hardware tiers and throughput figures are the manual's
  numbers, quoted as such; nothing was benchmarked.

A note on the fork chapter: it follows the project's own
history page (docs.opnsense.org/history/thefork.html) for the
2014–2015 split and its stated motivations, and states
pfSense's continuing position in the same breath. Both
projects' present-day editions and release rhythms are as
their own documentation describes them.

Fixtures and examples are deliberately generic: the factory
default network, documentation example addresses
(203.0.113.0/24-style ranges appear in the manual's own
examples), and the CARP how-to's published worked example. No
real network, organisation or person appears anywhere in this
guide.
