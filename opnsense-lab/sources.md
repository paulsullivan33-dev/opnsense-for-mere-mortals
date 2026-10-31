# Sources for the OPNsense lab

Everything concrete in this guide traces to one of these.

## Artefacts downloaded and verified (2026-10-30)

- Mirror directory: https://mirror.ams1.nl.leaseweb.net/opnsense/releases/26.7/
  - `OPNsense-26.7-nano-amd64.img.bz2` — 490,849,116 bytes;
    SHA-256 `28d5e2f37e40d87468a924e3006ef10e2ddc6de485b85333d9e3958c84d0cb9d`,
    matching `OPNsense-26.7-checksums-amd64.sha256`.
  - Uncompressed `OPNsense-26.7-nano-amd64.img` — 3,221,225,472 bytes;
    SHA-256 `c376b498338e9e0ddd785ac447f933e940f7d556810779475eecef30d37fcb98`.
  - Signatures verified with the release public key `OPNsense-26.7.pub`
    (4096-bit RSA; byte-identical to the key printed in the same
    directory's `README`): the checksums file's own `.sig` and the
    nano image's `.sig` both return "Verified OK".
  - The checksums file was also fetched from a second mirror
    (mirror.fra10.de.leaseweb.net) and found identical.
  - The `README` in that directory states: latest stable release
    image is 26.7 (July 15, 2026); next scheduled image release
    27.1 (January 2027).
- Release series directories on the mirror at the time: 23.1 …
  26.7, amd64 only.

## Source trees read

- `opnsense/core` (github.com/opnsense/core), shallow clone at
  commit `50105e67ed9ea901d79442804e573bbaf9d9b03b` (2026-10-30).
  Key files: `src/etc/config.xml.sample` (the factory
  configuration, 172 lines), `src/etc/inc/`,
  `src/opnsense/mvc/app/models/OPNsense/` (25 models),
  `src/opnsense/mvc/app/config/config.php` (`configDir => '/conf'`),
  `src/opnsense/service/conf/actions.d/` (31 configd action files).
- `opnsense/tools` (github.com/opnsense/tools), shallow clone:
  `build/common.sh` (`sign_image`: `openssl dgst -sha256 -sign`,
  output base64-encoded) and `build/nano.sh` (`makefs` 3G image,
  `gpart create -s BSD`, then sign).
- `opnsense/plugins` (github.com/opnsense/plugins): directory
  tree read via the GitHub API — 102 plugin directories with a
  Makefile, counted by category.

## Official documentation read (docs.opnsense.org)

- Manual index: https://docs.opnsense.org/
- Install: https://docs.opnsense.org/manual/install.html
- Hardware: https://docs.opnsense.org/manual/hardware.html
- Firewall rules: https://docs.opnsense.org/manual/firewall.html
- NAT: https://docs.opnsense.org/manual/nat.html
- Backups/configuration: https://docs.opnsense.org/manual/backups.html
- Cloud backup (Nextcloud):
  https://docs.opnsense.org/manual/how-tos/cloud_backup.html
- CARP how-to: https://docs.opnsense.org/manual/how-tos/carp.html
- IPS: https://docs.opnsense.org/manual/ips.html
- Updates: https://docs.opnsense.org/manual/updates.html
- Dashboard: https://docs.opnsense.org/manual/dashboard.html
- Unbound DNS: https://docs.opnsense.org/manual/unbound.html
- VPN: https://docs.opnsense.org/manual/vpnet.html
- About the fork: https://docs.opnsense.org/history/thefork.html
- Support tiers: https://docs.opnsense.org/support.html

The image itself was deleted from the lab after dissection, as
it is large; the checksums and transcripts in `transcripts/`
are the durable record.
