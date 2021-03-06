# SimpleLinuxSandbox
A simple sandbox using Linux namespaces

```
Usage: ./simple_sandbox [OPTIONS] COMMAND

OPTIONS:
    -d         Enable debug messages
    -t T       Terminate program after T milliseconds
    -u uid     Run command with user uid
    -g gid     Run command with group gid
    -p         Mount /proc
    -s         Mount /sys
```

Executes COMMAND in a virtual environment with very limited
access to system resources.

The command is executed with non-root privileges. The uid and gid can be
directly specified with `-u` and `-g` options, otherwise the user running the
sandbox is used. If the sandbox is run as root (e.g. with `sudo`), then the
non-root user must be specified through `-u` and `-g` options.

# Installation:

Compile the program with `make` and install by `sudo make install`.
Note that the sandbox's executable should be a SETUID binary owned by root to
be able to work properly. The Makefile's default target uses `sudo chown root:root ...`
and `sudo chmod +s ...`, so the system will probably ask for your password.

# Sample Usage:

First, you need to find an unprivileged user id and its corresponsing group id.
Most Linux distribution already have an unprivileged user usually named `nobody`.
You can find out the user id of `nobody` on your Linux distribution with the following command:

```
$ cat /etc/passwd | grep nobody
```

On my Ubuntu installation, the above command produces the following output:

```
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
```

To find out the default group name for `nobody`, use:

```
$ groups nobody
```

On my Ubuntu installation, user `nobody` belongs to the group `nogroup`. To find out the group id of `nogroup`, use:

```
$ cat /etc/group | grep nogroup
```

So, using user id of 65534 and group id of 65534 in my case, to run `bash` inside the sandbox:

```
[user@machine ~]$ simple_sandbox -u 65534 -g 65534 -ps /bin/bash
```

If everything works properly you should see an output like this:

```
[user@machine ~]$ ./simple_sandbox -u 65534 -g 65534 -ps /bin/bash
nobody@machine:/$ id
uid=65534(nobody) gid=65534(nogroup) groups=65534(nogroup)
nobody@machine:/$
nobody@machine:/$ ls -l
total 1032
drwxr-xr-x   2 root root    4096 Apr 12 13:05 bin
drwxr-xr-x 165 root root   12288 Aug 17 13:25 etc
drwxr-xr-x  26 root root    4096 May 26 15:02 lib
drwxr-xr-x   2 root root    4096 May 26 15:02 lib32
drwxr-xr-x   2 root root    4096 May 26 15:02 lib64
dr-xr-xr-x 196 root root       0 Aug 17 15:53 proc
-rwxr-xr-x   1 root root 1021112 Oct  7  2014 program
dr-xr-xr-x  13 root root       0 Aug 17 15:53 sys
drwxr-xr-x  12 root root    4096 Apr 14 18:40 usr
nobody@machine:/$
```
