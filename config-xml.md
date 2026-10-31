# config.xml: the whole firewall in one file

Chapter 2 promised that the GUI is an
editor and one file is the firewall.
This chapter keeps the promise: where
the file lives, what a change to it
looks like over time, and the four
things you can do with it — back it up,
watch its history, restore it, and
carry it to new hardware. Master this
file's lifecycle and an OPNsense
installation becomes very hard to lose.

## Where it lives, and who guards it

The running configuration is
**`/conf/config.xml`**. The directory is
not a convention but a literal in the
source: the web framework's own
configuration sets `configDir => '/conf'`
and opens `config.xml` inside it. The
loader's behaviour, readable in the same
code, is a small essay in caution:

- If the file parses, it rules.
- If it is missing or invalid, the system
  logs the fact and **attempts a restore
  of the last known config** — the
  previous version, kept beside it.
- If that fails too, it **restores the
  factory configuration** — the sample of
  chapter 6 — rather than boot a firewall
  with no law at all.
- A **factory reset** (console menu 4, or
  System ▸ Configuration ▸ Defaults) is
  mechanically just this: the source's
  reset path removes `/conf/*` (and logs
  and shell history) and lets the
  defaults take over at next boot.

Every GUI page, API call and wizard step
in this guide is, at the bottom, an edit
to this file followed by configd
applying the affected pieces.

## Backup: the whole job, one download

**System ▸ Configuration ▸ Backups**
downloads the configuration as a single
XML file. Options the manual lists:

- **Password-protect the export** — do;
  the file contains password hashes,
  VPN keys and certificates. The manual's
  note is blunt about treating backups as
  sensitive.
- **Include RRD statistics** — the
  reporting history can ride along in the
  same export, if you want graphs to
  survive a migration too.
- **Restore, whole or partial** — a
  backup can be restored completely
  (the safe default; the manual warns
  that components depend on each other)
  or in selected sections (supported,
  with an explicit warning that partial
  restores can behave unexpectedly).
  Two mercies are built in: console
  settings are excluded from a restore
  by default — so a backup from a serial
  appliance cannot lock a VGA machine
  out of its console — and a restore
  onto hardware with different interface
  names does not force a reboot; it
  prompts you to fix the assignments
  (chapter 18).

## History: the file remembers

Every change keeps its predecessor.
**System ▸ Configuration ▸ History**
lists the preserved versions; pick any
two and the page shows their difference
as a **unified diff** — the exact lines
a misclick changed, in either direction.
From the same screen you can download a
chosen version or **revert** to it
(with the manual's caveat that reverting
restarts nothing by itself — a reboot
gives you the clean state). The number
of versions kept is configurable in the
backups settings. For a system whose
whole law is one file, this turns
"what changed last Tuesday?" from
archaeology into a dropdown.

The project also documents two
traceability companions: backups **via
secure copy (SFTP)**, and tracking
configuration changes **with Git** —
the file's changes committed to a
repository, so its history outlives the
box.

## Automatic off-box backup

A backup you must remember to take is a
backup you will not have. The
configuration section can push the file
off the machine on its own: the
documented remote-backup feature
supports **SFTP** and **Nextcloud**, and
its rhythm is the sensible one — a first
backup as soon as it is set up, then,
whenever the configuration has changed,
a fresh one **once a day, early in the
morning** (more often, if you add your
own cron jobs). The Nextcloud how-to's
shape generalises to both: create a
dedicated user and an app password on
the storage side, enter the URL and
credentials in the backup settings, save,
and verify the first uploaded file
appears in its directory before trusting
the arrangement. (Earlier releases also
offered a Google Drive backup; current
documentation presents SFTP and
Nextcloud as the routes — one more
reason chapter 5 says to read the
manual for *your* release.)

## Carrying it to new hardware

Put chapters 7 and 8 together and
migration stops being an event:

1. On the old box: **Backups ▸ Download**
   (password-protected), or let the
   automatic backup do it.
2. On the new box: boot the installer
   media with a second stick carrying
   `/conf/config.xml`, answer the
   **Importer** prompt, and the new
   machine *is* the old firewall —
   reassign interfaces if the names
   differ, install, done.
3. Or, after a plain install, restore
   from the Backups page instead.

Either way, the organisation's network
memory — rules, leases, overrides,
certificates, users, a decade of small
decisions — moves house in one file.
That is the deepest reason for the
design chapter 2 described, and the
reason this guide keeps returning to
it: **back up config.xml and you have
backed up the firewall.** Everything
else is reinstallable.
