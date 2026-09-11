# sunpart

A disk partition layout planner for SunOS 4.x and Solaris 2.x — does the
cylinder math so you don't have to.

Early Sun machines make you enter disk partitions in **blocks and
cylinders**, and the stock defaults are famously bad: a minuscule `/`, all
the free space dumped on the free-hog slice, and on some early releases
`format` will happily let you build a root partition past the 1 GB boot
limit — which the boot PROM then can't read, leaving you with a machine
that installs fine and won't boot.

sunpart is a single-page, zero-dependency web tool that lets you plan a
layout in megabytes and gives you back numbers you can type straight into
`format`'s partition menu.

## Usage

Open `sunpart.html` in any browser. No build step, no server, nothing to
install. (The only network fetch is Google Fonts; offline it falls back to
system fonts and works fine.)

1. **Describe your disk** the way `format` reports it — paste the label
   line, e.g. `cyl 2733 alt 2 hd 19 sec 80` — or pick a classic Sun disk
   from the preset list (SUN0207 through SUN4.2G). For an unknown drive
   with only a manufacturer's spec sheet, the **spec translator** turns a
   formatted capacity (GB/MB or a guaranteed sector count) into a valid
   Sun label geometry — ZBR means the physical CHS figures in the manual
   don't multiply out, so capacity is the only number that matters.
   Building a ZuluSCSI/BlueSCSI disk instead? The translator's **image
   creator** mode inverts the problem: pick a classic Sun disk to emulate
   (or a target size), and it emits the exact byte count and `truncate`/`dd`
   command to create the image — sized to hold the label's cylinders
   precisely, named for the emulator (`HD3.img` works on ZuluSCSI and
   BlueSCSI v2 alike), with SCSI ID 3 defaulted because Sun expects
   its boot disk `sd0` there.
2. **Lay out slices `a`–`h`** by mount point and size in MB (or cylinders).
   Sizes round up to whole cylinders and slices are packed sequentially on
   cylinder boundaries, so overlaps are impossible by construction. One
   slice can take the remainder of the disk — the free hog, but under your
   control. `c` is locked to the whole disk, as it must be.
3. **Watch the checks**: root on `a` starting at cylinder 0, partition `a`
   inside the 1 GB boot window (toggleable), nothing over-committed or
   unallocated, swap on `b` and big enough for crash dumps, a root/`usr`
   split that will actually hold an install.
4. **Copy the output**: a ready-to-type `partition>` dialogue for SunOS 4.x
   (starting cylinder + block count) or Solaris 2.x (slice numbers, id
   tags, permission flags, sizes in cylinders), plus the exact table
   `p`/`print` should show afterwards so you can verify the label took.

Unused slices are explicitly zeroed in the generated dialogue — stale
entries in old labels are a classic source of install weirdness.

Units: 1 block = 512 bytes; 1 MB = 2048 blocks; 1 cylinder = heads ×
sectors/track blocks.

## License

GPL-3.0-or-later — see [LICENSE](LICENSE).
