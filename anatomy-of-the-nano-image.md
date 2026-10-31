# Anatomy of the nano image

Before trusting an image with a network,
the lab took one apart — as far as a
machine that cannot boot it, and cannot
even read its filesystem, honestly can.
This chapter is that dissection, and it
doubles as a field guide: the same
checks work on any FreeBSD appliance
image you will ever meet. Full output is
transcript `t02` in `opnsense-lab/`.

## The outside of the box

`file` on the unpacked image answers:

```
OPNsense-26.7-nano-amd64.img: DOS/MBR boot sector, BSD disklabel
```

Two partition schemes in one breath, and
both are really there:

- **An MBR** (the PC partition table from
  1983) whose four entries are empty
  except the fourth: marked bootable,
  type `0xa5` — the code reserved for a
  FreeBSD slice — beginning at sector 0
  with a nominal length of 50,000 sectors.
  The MBR's job here is only to get a PC
  BIOS/UEFI pointed at the disk; the real
  map is the second scheme.
- **A BSD disklabel** at sector 1, whose
  magic number checks (`0x82564557`), and
  which describes the disk in FreeBSD's
  own terms: 6,291,456 sectors in total —
  exactly the file's 3,221,225,472 bytes —
  in 8 partitions, of which two matter:
  **`a`**, the whole disk, filesystem
  type 7 (UFS), and **`c`**, the
  traditional whole-disk entry, unused.

The build recipe explains the shape
(the project's `opnsense/tools`,
`build/nano.sh`): `makefs` creates a
3 GiB UFS filesystem image, `gpart`
wraps a BSD label and boot code around
it, and the result is signed. One
filesystem, the whole card, root and
everything in it — the nano has no
separate system/data split. When the
manual says the nano "automatically
adapts to the installed media size after
first boot", it is this single `a`
partition that grows.

## The filesystem's fingerprint

UFS2 superblocks carry a magic number,
`0x19540119`, at a fixed offset (0x55c)
within the superblock, which for UFS2
sits 64 KiB into the filesystem. The lab
found the magic at byte **66,908** of
the image — 65,536 + 1,372, to the byte
where the primary superblock must live —
with further copies deeper in, where
UFS keeps its backup superblocks. That
is as certain as identification gets
without reading a single filename: the
file is a UFS2 filesystem, laid out
exactly as FreeBSD lays it out.

## Where the lab had to stop

And there, honestly, the trail ends for
this machine:

- The lab has **no way to mount** the
  image (no loop devices, and Linux's
  own UFS support is not present in its
  tools), and `7z` — which opens a
  surprising range of disk images —
  refuses it outright: *"Cannot open the
  file as archive."*
- So **no file listing of the image
  exists in this guide**, and none is
  claimed. Anyone who tells you what is
  "in" the 26.7 nano image beyond this
  chapter got it from somewhere else.

## The factory configuration, from the source tree

The contents question still has an
honest answer, by a different door: the
configuration a fresh system starts
with ships in the project's source as
`src/etc/config.xml.sample` (172 lines —
the lab read it entire; transcript
`t03`). It is the single most
information-dense file in this guide,
because every "by default" elsewhere
traces back to it. The headline facts:

- **Identity**: hostname `OPNsense`,
  domain `internal`, timezone `Etc/UTC`,
  NTP from the project's own pool
  (`*.opnsense.pool.ntp.org`).
- **A flag to run the setup wizard**:
  the file opens with
  `<trigger_initial_wizard/>` — the
  marker that makes a first GUI login
  become the wizard of chapter 7.
- **WAN**: enabled, address by **DHCP**
  (v4) and DHCP6, and two shields up —
  `blockpriv` (refuse private-address
  sources) and `blockbogons` (refuse
  bogon sources), with the bogon list set
  to refresh monthly.
- **LAN**: enabled, **192.168.1.1/24**;
  its IPv6 *tracks* the WAN's delegated
  prefix.
- **Dnsmasq**: enabled, DHCP range
  **192.168.1.100–192.168.1.199** on the
  LAN (plus an IPv6 range), its DNS side
  parked on port **53053** — because…
- **Unbound**: enabled. The real resolver
  owns port 53 (chapter 11).
- **NAT**: outbound mode **automatic**
  (chapter 10), NAT reflection disabled.
- **The web GUI**: HTTPS, and `noroot`
  set — the GUI process does not run as
  root (chapter 3's security argument,
  visible as one word in the defaults).
- **Users**: group `admins` with full
  page privileges; user `root`. (The
  default password story belongs to
  chapter 7 — the *live* and fresh
  install password is the documented
  `opnsense`, and changing it is the
  wizard's last step for a reason.)
- **SSH**: present in the file with its
  access restricted to the `admins`
  group.
- **Power**: `powerd` in adaptive mode
  (`hadp`) on AC, battery and normal
  profiles.
- **Exactly two firewall rules** — the
  "Default allow LAN to any" pair,
  IPv4 and IPv6, which chapter 9 quotes
  in full.

Read that list again and notice what it
says about the project's judgement: a
fresh OPNsense is *boring on purpose* —
private addresses refused at the WAN,
the LAN trusted wholesale, everything
else shut. Your configuration career is
the art of departing from boring, one
deliberate line of this file at a time.
Chapter 8 takes the file's living form —
backups, history, restore — apart.
