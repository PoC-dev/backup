### What is it?
Backup utilizes `dump(8)` for doing per-partition backups of ext2/ext3/ext4 filesystems to a given destination.

Valid destinations can be
- local tape device
- remote (`rmt`) tape device
- local file system
   - NFS is a local file system from this particular viewpoint
- remote file system
   - accessed via `rsh` (unencrypted, more CPU efficient)
   - accessed via `ssh` (encrypted, when transferring over the internet)

`Backup` looks in */etc/fstab* for backupable partitions/file systems. These must be of type ext2, ext3 or or ext4, and the dump-flag be set to >0.

The dump-flag is used to decide which kind of backup to allow:
- 0 means no automatic backup.
- 1 mans level 0 and higher dumps are done automatically.
- 2 mans level 1 and higher dumps are done automatically.
- 3 mans level 2 and higher dumps are done automatically.
- ...

This is primarily meant for very large file systems with only few changes over prolonged time, such as archives. Doing level 0 dumps of those consumes a lot of time. If the dump-flag is set to 2 after an initial level 0 dump (with dump-flag=1), only small and quick differential backups to the last level 0 dump are done, while automatic but explicit level 0 dumps are omitted.

### Usage
The following command line arguments apply:
```
backup [-d] [-l] level
```

- `-d` creates a "done" file for signalling external programs that a certain file is finished and can be post-processed, e. g. written to tape.
- `-n` prohibits ejecting of tape media.
- `level` is the increment level, dump should use.

If no level is submitted, it will be calculated from the weekday's number, which is incremented by one (so a full-backup has to be explicitly specified).

Other settings are pulled from a *preferences-file*. Backup looks for this file in the following order, searching for the most specific file first:
- `~/.backuprc-${HOSTNAME}-${LEVEL}`
- `~/.backuprc-${HOSTNAME}`
- `~/.backuprc-${LEVEL}`
- `~/.backuprc`

This enables using different settings depending on the hostname and/or increment level as well as one common backup user within a NIS/NFS environment. Backup tells you at startup which file it is using. See example file for the variables.

The variables are commented in dot-backuprc-sample and reasonably self-explanatory.

If `TAPEDEV` is set, backup assumes you're dumping to a tape device and takes appropriate measures for setting 64k blocks and unlimited filesize. Doing online compression is considered bad habit, especially on slow CPUs. Using ssh in LAN environments for feeding tape drives is also considered "please avoid" — it creates unneccessary load on the CPU and overhead over the network, potentially resulting on unneccessary stop-and-go of the tape drive.

Else if `TOPATH` is set, a directory structure will be created in `${TOPATH}`. This is a workaround, since `rmt` cannot create files itself. You may use `cleanup-oldbackups` to remove outdated backups: Everything which is older than the newest level-0 backup.

`Backup` uses the ext2 partition label (see `e2label(8)`) for creating the pathes. It is a good advice to check if these are consistent with your intended use. Also, `e2label` needs to access not only the device file, but also the mount path. If the user running this script is not allowed to read the mountpoint, `e2label` fails and the backup file name will be derived by the mountpoint name.

If `backup` finds a file named `.backup-todo` in the mountpoint which is to be dumped, it will be executed with parameters pre and post. This is especially useful for backing up a mysql-database (which should reside on a partition for itself then) which needs to be writelocked as long as the backup runs. Also this can be used for hibernating Qemu VMs.

----

- 2011-09-11, PoC (initial version)
- 2024-04-27, poc@pocnet.net (released to GitHub)
