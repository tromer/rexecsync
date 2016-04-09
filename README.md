rexecsync: a script for efficiently saving the output of a remote command into a local file
================================================================================

This script runs a command remotely and saves its output to a local file. The data transfer is done using an rsync-like difference-based, algorithm to reduce communication on subsequent updates.

--------------------------------------------------------------------------------
Usage
--------------------------------------------------------------------------------
    rexecsync [-b] [-c] [-v] [-a ...] [-r ...] 'rshell_cmd' 'remote_cmd' file
where `rshell_cmd` is the prefix command used to execute a remote shell commands,
`remote_cmd` is the remote command whose output is to be saved (must be a legal bash command with no single quotes), and
`file` is the local file to be written,


Options:
* `-b`: create backup of old file with "~" suffix; overwrite if already exists.
* `-c`: check whether the exit value of remote_cmd is zero; abort otherwise.
* `-v`: verbose output and progress indicators (progress percentage is estimated according to the original file size).
* `-a` arg: add arg as command-line arguments to the local rdiff (e.g., "rexecsync -a '--block-size=16384 --sum-size=4' ...").
* `-r` cmd: use cmd instead of "$REMOTE_RDIFF" for invoking rdiff on the remote machine (must be a legal bash command with no single-quotes).
* `-d`: do a dumb transfer of the whole data, without rdiff (for debugging, or in absence of a remote rdiff).

--------------------------------------------------------------------------------
Examples
--------------------------------------------------------------------------------

* Full list of files on remote host:

```bash
rexecsync -v 'ssh user@remote.host' 'ls -lR ~/' file-list
```

* Backup of directory on remote host into a local `tar.gz` file:

```bash
rexecsync -v 'ssh root@remote.host' 'tar czf - /home/remote-dir/' backup.tar.gz
```

* Same, but more efficient by using an difference-friendly compression:

```bash
rexecsync -v 'ssh root@remote.host' 'tar cf - /home/remote-dir/ | gzip -c --rsyncable' backup.tar.gz
```

* Full backup of root filesystem on remote host:

```bash
rexecsync -v 'ssh root@remote.host' 'tar cf - --atime-preserve / --exclude=/proc --exclude=/sys --exclude=/dev --exclude=/tmp | gzip -c --rsyncable' backup.tar.gz
```


--------------------------------------------------------------------------------
Notes
--------------------------------------------------------------------------------

Dependencies: `rdiff` (from [`librsync`](https://github.com/librsync/librsync)) and `pv`.

The script blindly invokes complex commands on arbitrary remote systems, which may cause disasterous results. Always test first in non-critical setting. <i>Patches to improve error checking and handling are very welcome.</i>

Large files (greater than 4GB) are supported, assuming `librsync` is version 0.9.7 (relesed in 2004). Earlier versions of `librsync` will silently corrupt files larger than 4GB.
