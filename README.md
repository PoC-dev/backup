### What is it?
Backup utilizes `dump(8)` for doing per-partition backups of ext2/ext3/ext4 filesystems to a destination.

Valid destinations can be
- local tape device
- remote (`rmt`) tape device
- local file system
   - NFS is a local file system from this particular viewpoint
- remote file system
   - accessed via `rsh` (unencrypted, more CPU efficient)
   - accessed via `ssh` (encrypted, when transferring over the internet)

`Backup` looks in */etc/fstab* for backupable partitions. These must be of type ext2, ext3 or or ext4 and the dump-flag be set to >0.

### Usage
The following command line arguments apply:
```
backup [-n] [-l] level
```

- `-n` prohibits ejecting of tape media.
- `-l` creates a "lock" file for signalling external programs to not use that file (e. g. for writing to a tape drive).
- `level` is the increment level, dump should use.

If no level is submitted, it will be calculated from the weekday-number, which is incremented by one (so a full-backup has to be explicitly specified).

Other settings are pulled from a *preferences-file*. Backup looks for this file in the following order, searching for the most specific file first:
- `~/.backuprc-${HOSTNAME}-${LEVEL}`
- `~/.backuprc-${HOSTNAME}`
- `~/.backuprc-${LEVEL}`
- `~/.backuprc`

This enables using different settings depending on the hostname and/or increment level as well as one common backup user within a NIS/NFS environment. Backup tells you at startup which file it is using. See example file for the variables.

The variables are commented in dot-backuprc-sample and reasonably self-explanatory.

If `TAPEDEV` is set, backup assumes you're dumping to a tape device and takes appropriate measures for setting 64k blocks and unlimited filesize. Doing online compression is considered bad habit, especially on slow CPUs. Using ssh in LAN environments for feeding tape drives is also considered "please avoid" — it creates unneccessary load on the CPU and overhead over the network, potentially resulting on unneccessary stop-and-go of the tape drive.

Else if `TOPATH` is set, a directory structure will be created in `${TOPATH}`. This is a workaround, since `rmt` cannot create files itself). You may use `cleanup-oldbackups` to remove outdated backups: Everything which is older than the newest level-0 backup.

`Backup` uses the ext2 partition label (see `e2label(8)`) for creating the pathes. It is a good advice to check if these are consistent with your intended use.

If `backup` finds a file named `.backup-todo` in the mountpoint which is to be dumped, it will be executed with parameters pre and post. This is especially useful for backing up a mysql-database (which should reside on a partition for itself then) which needs to be writelocked as long as the backup runs.

----

- 2011-09-11, PoC (initial version)
- 2024-04-21, poc@pocnet.net (released to GitHub)
