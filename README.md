# timeshift-autosnap-patched
Timeshift auto-snapshot script which runs before package upgrade using Pacman hook. **Patched to work correctly with yay and paru, 
taking only one snapshot during an update. Run `autosnap-wrapper` to perform an update with the patch.**

### Features
*  Creates Timeshift snapshots with unique comment.
*  Deletes old snapshots which are created using this script.
*  Auto generates grub if grub-btrfs package is installed.
*  Can be manually executed by running `timeshift-autosnap` command with elevated privileges.
*  Autosnaphot can be temporarily skipped by setting SKIP_AUTOSNAP environment variable (e.g. `sudo SKIP_AUTOSNAP= pacman -Syu`)
*  **Patched to perform only one snapshot when using AUR helpers yay and paru**

### /etc/timeshift-autosnap.conf options
*  `skipAutosnap` - if set to **true** script won't be executed.
*  `deleteSnapshots` - if set to **false** old snapshots won't be deleted.
*  `maxSnapshots` - defines **maximum** number of old snapshots to keep.
*  `updateGrub` - if set to **false** grub entries won't be generated.
*  `snapshotDescription` - defines **value** used to distinguish snapshots created using timeshift-autosnap.

### What was Patched

AUR helpers invoke pacman multiple times during any given system update and the timeshift-autosnap hook dutifully
runs every time pacman is invoked to update packages. This often means that multiple partial-update snapshots are
taken during an upgrade; this makes timeshift-autosnap's function much less useful and adds clutter to our system.

### Our Solution

We need some way to indicate to the various spawned pacman processes that we have (or haven't) made a snapshot during
an update. First we'll setup a location for a lockfile whose presence indicates that we've made a snapshot, save that
in an environment variable and then pass this through to each instance of pacman spawned by our AUR helper.

On first run the autosnap hook sees that our env var is declared but the file doesnt yet exist so we'll touch
the file as the user who called pacman via sudo and then proceed. On subsequent runs the lockfile exists so we
set the `SKIP_AUTOSNAP` variable before running `timeshift-autosnap` and no snapshot is created. On exit, after
our AUR helper is completely done, our wrapper script removes our lockfile as a final cleanup step

**All of these things only ever happen if our AUR helper is launched through `autosnap-wrapper` and '__AUTOSNAP_LOCK'
is defined; otherwise timeshift-autosnap is invoked normally without modification**

### Notes
*  Works both in `BTRFS` and `RSYNC` mode.
*  This script is made with Arch and Arch based distros in mind but if there is any interest it should be easily ported to other distros.
*  You can manually set the AUR helper you are using by writing in your `~/.profile`: `export AUR_HELPER=EXAMPLE`, replacing EXAMPLE with
the one you intend to use (in lowercase).

### Contribution
*  All new ideas and contributons are welcome! I'd appreciate any suggestions which make this patch easier to use.
