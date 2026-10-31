# Troubleshooting

Each entry here is one this guide can
stand behind: it happened in the lab, or
the project's manual names it directly.
Format: the symptom, what it means, the
way out.

## The checksum does not match

**Symptom:** `sha256sum -c` (or the
openssl checksum command in the install
manual) reports FAILED for your image.
**Means:** the file on your disk is not
the file the project published — a
truncated download is the usual cause;
the .bz2 files run to several hundred
megabytes. **Do:** download again, and
download the checksums file again with
it. Verify before writing anything to a
card; the lab's run of exactly this check
is chapter 5. If you want the stronger
proof, verify the signature file that
sits beside the checksums file
(`...-checksums-amd64.sha256.sig`)
against the release public key — the key
printed in the mirror's README for that
release, checked against a second source,
as the install manual insists.

## The installer never appears

**Symptom:** the machine boots its old
system, or sits at firmware.
**Means:** it did not boot your media.
**Do:** enter the boot menu (the install
manual: a function key, Del or ESC in the
first seconds; Deciso appliances use ESC)
and choose the USB device. If the screen
stays dark on a serial-only appliance,
you have the *vga* image where the
*serial* one belongs — the images differ
only in console, and the serial console
runs at 115200 baud (chapter 5).

## The live system forgot everything after reboot

**Symptom:** settings made in the live
environment vanish.
**Means:** that is what "live" means —
the install manual is blunt that live
media is read-only and its configuration
is lost at reboot.
**Do:** run the installer (log in as
**installer**, password **opnsense**)
onto the real disk, or boot with the
Importer and a config on a second stick
if you were only previewing (chapter 7).

## You are locked out of the web interface

**Symptom:** forgotten root password, a
rule that cut off the LAN's access to the
GUI, an address changed and forgotten.
**Do:** go to the console — this is what
it is for. The menu offers, by number:
**1** assign interfaces, **2** set an
interface IP address, **3** reset the
root password, **13** restore a
configuration backup. A locked-out GUI
with a reachable console is an errand,
not a disaster; a locked-out GUI without
one is why chapter 1 insists the console
path be tested before the box goes into
the cupboard.

## A new rule "isn't working"

**Symptom:** you blocked something and it
still flows (or allowed something and it
still fails).
**Checks, in order:**

1. **States.** Existing conversations
   keep their old treatment. Reset them
   (Firewall ▸ Diagnostics ▸ States) or
   wait for expiry before judging the
   rule (chapter 9).
2. **Order and quick.** A broader pass
   above yours matched first. Floating
   before groups before interfaces;
   first match wins for quick rules.
3. **NAT got there first.** A port
   forward associated as "pass" bypasses
   your filter rule entirely (chapters 9
   and 10).
4. **Direction.** The rule is on the
   interface where the conversation
   *arrives*, direction in. Rules on the
   destination's interface, direction in,
   see only unsolicited arrivals.
5. **Inspect.** Turn on inspection in the
   rules view, read the counters; open the
   live log and watch which rule fires.
   The generated ruleset is on the box at
   `/tmp/rules.debug` when all else
   fails.

## The restored firewall asks about interfaces

**Symptom:** after restoring a backup on
different hardware, no automatic reboot;
a prompt to fix interface assignments.
**Means:** the manual's documented
behaviour — the backup's interface names
(em0, igb1…) do not exist here.
**Do:** reassign LAN and WAN to the new
machine's ports at the prompt; everything
else in the file applies as-is. This is
the migration path of chapter 8 working
as designed, not an error.

## DNS stopped after "just enabling" a service

**Symptom:** name resolution dies after
you enabled Dnsmasq's DNS side while
Unbound was running (or the reverse).
**Means:** two services, one port 53.
**Do:** choose one owner for port 53
(chapter 11), move or disable the other,
and read the service log — it names the
conflict in plain words.

## IPS mode is slow, or won't start on an interface

**Checks from the IPS manual:** hardware
offloading on the interface must be
**disabled** for IPS; on VLANs, attach to
the parent interface; if the network card
is not fully netmap-capable, IDS still
works almost everywhere, and the tunable
`dev.netmap.admode` set to `2` forces
emulated netmap as a fallback. Then
measure honestly (chapter 13): inline
inspection costing throughput is physics,
not a fault.

## Both HA firewalls think they are master

**Symptom:** split-brain after adding a
virtual IP alias, or after switch
changes.
**Means:** CARP's heartbeats are not
reaching the peer — the how-to's
known-limitations list points at switch
features (IGMP snooping, MAC/port
security, storm control) eating the
multicasts, and at the specific trap of
adding an alias to a live VHID on one
side first. **Do:** the how-to's
procedure — disable CARP on one machine,
make the change on both, re-enable — and
audit the switch between them before
blaming the firewalls.

## An update fails or finds nothing

**Do:** change the firmware **mirror**
(System ▸ Firmware ▸ Settings); run the
page's **connectivity audit**; confirm
the release type is Production; and for
major upgrades, remember they are
offline, console-watched events
(chapter 14) — a silent web page during
one is the design, not a hang.
