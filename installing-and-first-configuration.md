# Installing and first configuration

This is the chapter where an image
becomes a firewall. It is taught from
the install manual — the lab could not
boot the system (chapter 1 explains why)
— and it is organised the way the
manual's own flow runs: the recovery
door first, then the live environment,
then the installer, then the first
configuration you will be offered.

## Before the installer: the Importer

Every full image boots the same way: a
few seconds in, it offers **"Press any
key to start the configuration
importer"**. Learn this prompt before
you need it, because the **Importer** is
OPNsense's best recovery trick:

- On a machine with an existing install,
  name the disk holding the
  configuration, and the *new* live
  system boots with *your* settings —
  nothing overwritten. This is how you
  preview a major release against your
  real config, and how you resurrect a
  firewall whose disk died but whose
  config you have.
- On a fresh machine, prepare a second
  USB stick (FAT/FAT32) with a folder
  named **`conf`** containing an
  unencrypted backup renamed to exactly
  **`config.xml`**. Boot with both
  sticks inserted, answer the prompt
  with the second stick's device name,
  and the live system comes up as *your*
  firewall, ready to install itself onto
  the disk with the configuration already
  in place.

Miss the prompt and you simply boot to
the live login; reboot and catch it
next time.

## The live environment

Booted from a full image with no import,
you are in a complete, working firewall
that happens to live on read-only media:
the GUI answers at **https://192.168.1.1**,
SSH answers too, and the console login
is **root** with password **`opnsense`**.
Changes work — and evaporate at reboot.
The manual states it without varnish:
installation media is read-only, and a
live configuration is lost when the
machine restarts. Treat the live system
as the showroom, and the installer as
the purchase.

(For the nano image there is no
showroom: write it, install the card,
boot, and the firewall is simply
*running*, with memory disks for logs
and /tmp per chapter 5. Its remaining
steps are the configuration ones below.)

## The installer

Log in at the console as **`installer`**
(password `opnsense`) — the same login
works over SSH to 192.168.1.1 if the
console is awkward — and the installer
walks its list. The manual's steps, in
order:

1. **Keymap selection** — the default
   suits most.
2. **Install (UFS|ZFS)** — choose the
   filesystem. The manual steers most
   people to **ZFS**: "in most cases the
   best option as it is the most
   reliable" — given a couple of
   gigabytes or more.
3. **Partitioning (ZFS)** — the default
   *stripe* is right for a single disk.
4. **Disk selection** — pick the target
   (`da0`, `nvd0`, …). On appliances
   with card storage, the manual warns
   in terms worth repeating: for an SD
   install choose the card device
   (`mmcsd0`), *not* `da0` — that is the
   USB stick you booted from.
5. **Last Chance!** — the point of no
   return: partitioning destroys the
   disk's contents.
6. **Swap (UFS path)** — accept the
   recommended swap unless the target is
   very small (under 16 GB).
7. **Root password** — set the real one
   here.
8. **Complete Install** — exit, reboot,
   remove the media when told.

An installer can also be pointed at a
running system later (the manual notes
it can clone a live system, nano
included, into a backup you can boot) —
remember that when chapter 8's backups
start to feel abstract; an installable
clone is the heaviest-duty backup of
all.

## First configuration

After installation the system boots and
asks, at the console, about **interface
assignment** — ignore it and the
defaults apply (the interfaces the
factory config expects, LAN on
192.168.1.1). Answer it and you walk the
manual's flow: VLANs first (decline —
they can always be added later), then
the LAN interface by name (`em0`,
`igb0`, whatever your hardware calls
them), then the WAN, then any optional
ones, Enter, confirm. The result is the
welcome banner and a login prompt; the
console will show the addresses it
chose, in the manual's shape:

```
WAN (em1)  ->  v4/DHCP4: …
LAN (em0)  ->  v4: 192.168.1.1/24
```

Log in at the console as **root**
(default password `opnsense` until you
change it) and you meet the **console
menu** — 14 numbered doors; know these
by heart:

| # | Door | You will use it to… |
|---|---|---|
| 0 | Logout | leave |
| 1 | Assign interfaces | recover from new hardware |
| 2 | Set interface IP address | reach the GUI again |
| 3 | Reset the root password | un-forget yourself |
| 4 | Reset to factory defaults | start clean |
| 5 / 6 | Reboot / Halt | obvious, and often |
| 8 | Shell | everything else, including `opnsense-update` |
| 11 | Restart web interface | unstick the GUI |
| 12 | Upgrade from console | chapter 14 |
| 13 | Restore a configuration | chapter 8, at the worst moment |

## The wizard and the dashboard

Now the web interface: browse to
**https://192.168.1.1**, accept the
self-signed certificate warning (the
box's first certificate is its own;
chapter 8's trust settings are where
real ones live), and log in as root.
Because the factory configuration
carries the `<trigger_initial_wizard/>`
marker (chapter 6), the first login
opens the **setup wizard** rather than
the dashboard. Its steps, as documented:
**general information** (hostname,
domain — the factory's `OPNsense` and
`internal` are the values you are
replacing), **time server information**,
**configure WAN** (DHCP for most home
connections — and, if your WAN address
is itself private, as behind another
router, *untick* the block-private
option the defaults set, or the wizard's
own WAN will refuse its neighbours),
**configure LAN** (the 192.168.1.1/24
you may keep or move), **set the admin
password** — the step the whole
ceremony exists for — and finally
**Reload**, which applies it all.

Behind the wizard waits the
**Dashboard**: the first page of every
later login, a grid of widgets — system
information, memory, disk, CPU,
announcements, gateways, interface
statistics, a live firewall-log widget,
a traffic graph, and service states —
resizable and removable, with layouts
saved **per user**. Everything from
here on in this guide is a page off
this dashboard's menus: **System**,
**Interfaces**, **Firewall**,
**Services**, **Reporting**, plus the
**Lobby** the dashboard lives in. The
next chapter starts with the file all
of them are editing.
