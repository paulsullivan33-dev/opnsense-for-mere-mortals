# Getting the images

OPNsense arrives as a disk image, and
choosing the right one — then *proving*
it is the file the project published —
is the first real procedure of the
guide. The lab performed the download
and the full verification chain for
real; this chapter teaches both.

## The four shapes

Releases live on the project's mirrors,
one directory per series (`26.7/`,
`26.1/`, …), each carrying the same four
image types for amd64. The install
manual's table, condensed:

| Type | What it is | Shape |
|---|---|---|
| **dvd** | ISO for CD/USB, boots a live system, VGA console | Full installer |
| **vga** | USB image, live system, VGA console, UEFI | Full installer |
| **serial** | USB image, live system, serial console only (115200 baud), UEFI | Full installer |
| **nano** | A *preinstalled* system on a 3 GB image, serial console with secondary VGA | Embedded |

"Full" images boot into a **live
environment** — a working firewall
running from the stick — from which you
run the installer onto the real disk
(chapter 7). The **nano** image skips
installation entirely: the system is
already on it, tailored to reduce write
cycles on flash cards (its logs and
/tmp live on memory disks), and it grows
to fill its card or stick after first
boot. The manual's sizing floor for it
is a 4 GB card or larger.

Two scheduling facts, from the mirror's
own README for the current release: the
latest stable release image is **26.7
(July 15, 2026)**, the next scheduled
image release is 27.1 (January 2027) —
and the README reminds you to *update
to the latest version immediately after
installation*, because images are only
rebuilt on the January/July rhythm while
updates flow fortnightly (chapter 14).

## Which one do you want?

- A machine with a monitor (or a VM):
  **vga** (or **dvd** if you think in
  ISOs).
- A headless appliance with a serial
  port: **serial**.
- An appliance or board that will boot
  its system straight from a card, or
  the classic "write it and it is
  installed" appliance route: **nano**.
- In every case: the newest series
  directory on the mirror, and the
  architecture is amd64 — there are no
  other official builds to choose
  between.

## Verification: the part not to skip

You are downloading the software that
will see every packet your network
sends. The project provides two layers
of proof, and the lab ran both, end to
end, on the nano image (transcript
`t00` and `t01` in `opnsense-lab/`):

**Layer 1 — the checksum.** Beside the
images sits `OPNsense-26.7-checksums-amd64.sha256`,
listing a SHA-256 per image. The lab's
download of the nano image:

```
$ sha256sum OPNsense-26.7-nano-amd64.img.bz2
28d5e2f37e40d87468a924e3006ef10e2ddc6de485b85333d9e3958c84d0cb9d
$ sha256sum -c   # the nano line
OPNsense-26.7-nano-amd64.img.bz2: OK
```

That proves the file matches the list.
It does not prove the *list*.

**Layer 2 — the signature on the list.**
The mirror also carries
`OPNsense-26.7-checksums-amd64.sha256.sig`
and the release public key,
`OPNsense-26.7.pub`. The install manual
insists — in bold terms — that you not
simply trust the key file from the same
mirror: compare it against the key
printed in an independent place (the
release announcements, the forum, the
changelog repository). The lab compared
it byte for byte with the key printed
in the mirror's `README`: identical.
Then, exactly as the manual prescribes
(the `.sig` files are base64 text;
decode first):

```
$ openssl base64 -d -in OPNsense-26.7-checksums-amd64.sha256.sig -out checksums.sig
$ openssl dgst -sha256 -verify OPNsense-26.7.pub \
    -signature checksums.sig OPNsense-26.7-checksums-amd64.sha256
Verified OK
```

**And the image carries its own
signature too.** Since 24.1 the project
signs images *before* compression (the
build tools show it directly:
`openssl dgst -sha256 -sign … | openssl base64`),
so the per-image `.sig` is verified
against the *unpacked* file:

```
$ bunzip2 OPNsense-26.7-nano-amd64.img.bz2
$ openssl base64 -d -in OPNsense-26.7-nano-amd64.img.sig -out image.sig
$ openssl dgst -sha256 -verify OPNsense-26.7.pub \
    -signature image.sig OPNsense-26.7-nano-amd64.img
Verified OK
```

For the record, the unpacked image the
lab dissected in chapter 6 is
3,221,225,472 bytes, SHA-256
`c376b498338e9e0ddd785ac447f933e940f7d556810779475eecef30d37fcb98`.
That the image's own signature verifies
over those decompressed bytes is itself
the proof that the decompression was
faithful — a single wrong byte anywhere
in 3.2 GB would have failed it.

One caution the lab earned: a mismatch
at any step means *download again* —
and a signature that fails against a
file you are still copying or
decompressing means nothing at all.
Verify against finished files only.

## Writing it to media

The manual's media instructions, in
brief: write the unpacked image (or the
ISO) to a USB stick or card with a raw
image writer — `dd` on Unix-likes,
Rufus or balenaEtcher on Windows — the
operation overwrites the whole stick.
For nano, that write *is* the
installation; for the full images, it
makes the installer you boot in
chapter 7.
