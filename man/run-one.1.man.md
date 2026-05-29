---
title: run-one
section: 1
header: User Commands
date: May 28, 2026
footer: Version 1.0.6
---

# NAME
**run-one** - a wrapper utility that prevents concurrent execution of a command by ensuring only a single instance runs at any given time


# SYNOPSIS

**run-one** [*OPTIONS*].\.. COMMAND [*ARGS*].\..

**run-this-one** [*OPTIONS*].\.. COMMAND [*ARGS*].\..


# DESCRIPTION

This utility prevents multiple instances of the same command from running concurrently. It can either block a new instance from starting if one is already active (**run-one**), or terminate the existing process and let the new one take over (the **run-this-one** variant).

To ensure only one instance runs at a time, this utility uses a flock-based semaphore. The unique ID for the lock is either specified on the command line using the **-i** parameter or generated automatically by hashing the user command line. The semaphore file is stored in one of the following directories in this order of preference:

```
$XDG_RUNTIME_DIR/one-run
/run/user/$UID/one-run
/dev/shm/one-run_$UID
/tmp/one-run_$UID
```
Most of these directories are on **tmpfs** file systems (/tmp is often a **tmpfs** mount in many recent Linux distributions), and all of them get cleaned up upon reboot.

The user supplied command line is then launched with a flock. If the corresponding lock file is available, then the command gets launched. If the lock on the file was already acquired by another process then depending on the flavor of command (**run-one**/**run-this-one**) or use of options **-t**|**-k**, the utility either exits or attempts to terminate other command instances, waiting for the lock to be released so that the user command can be launched.

Lock aquisition wait time can be limited with **-w**. If the lock is not aquired withing that time, the utility can be told to send `SIGTERM` signal for **-t** number of seconds while probing the lock. If the lock is still inavailable, the utility can send `SIGKILL` for **-k** number of seconds. If the process that holds the lock is still active and the lock is not released, the utility can forcefully invalidate that lock with **-a** and launch the command regardless.

# OPTIONS

**-w SECONDS**
: maximum time (in seconds) to wait for the lock to become available before giving up on acquiring the lock or start sending `TERM`/`KILL` signals; only whole numbers are understood, decimals are not allowed

**-t SECONDS**
: after the wait period **-w** expires, keep sending `SIGTERM` signal for this number of seconds, while checking whether the semaphore lock is released. Launches the command if the lock becomes available.

` `

Running **run-this-one** is equivalent to running **run-one -t 600**

**-k SECONDS**
: after both the **-w** and **-t** expire, keep sending `SIGKILL` signal for this number of seconds, while checking whether the semaphore lock is released. Launches the command if the lock becomes available.

**-a**
: run this invocation anyway, even if another process does not give up the lock in the time given by **-w** or **-t** or **-k**. This invalidates the lock acquired by another process and creates a new lock file

**-i TOKEN**
: use this token for the lock file name instead of calculating user command hash. This option is also used internally when running **keep-one-running cmd**, as this command variant internally runs as **run-one + keep-running + cmd** as **run-one** needs the token for **cmd** and not for **keep-running + cmd**

**-T**
: does not run the command but locates other instances of the command and prints the process IDs of the processes holding the lock

**-\-**
: double-dash can (optionally) be used to separate the options from the user command.

**-E EXIT_CODE**
: custom error exit code; enables the user to differentiate between the command failure and the utility exit error (see EXIT STATUS section for details)

**-D PATH**
: debug/log file path

**-h | -\-help**
: Print help information

# EXAMPLES

```
run-this-one inotifywait -m ~/tmp -e close_write | \
	run-until-failure sh -c 'read -r event && echo "$event"'
```

Sets a watcher on ~/tmp directory and keeps printing events that occur whenever a new file is written in it. If another identical instance of such watcher was already running, it gets terminated prior to running this instance.

```
run-one -w 60 -t 30 -k 10 -a -i "Watch my DIR" -- inotifywait -m ~/tmp -e close_write | \
	keep-running -u f -r 100 -- sh -c 'read -r event && echo "$event"'
```
Similar to the previous example but has more control. First **-w 60** it allows 1 min for the other process to finish, then sends `SIGTERM` to another instance for 30 sec **-t 30** asking it to wrap it up, and then attempts to shoot it with SIGKILL for 10 sec **-k 10**. It searches the other instance by "Watch my DIR" token (the other instance must be launched with **run-one -i "Watch my DIR"**). The **keep-running** pipe runs until either the user command fails or until it quits 100 times.

# EXIT STATUS

**run-one** and **run-this-one** exit with one of the following status:
```
0     upon success
122   when this utility encounters a problem; customizable with -E
```
Exit code 122 is returned the utility itself encounters an abnormal situation, e.g. when a parameter is invalid of flock fails. This code is customizable with -E option.

Any other exit code indicates the status returned from the user command.

# AUTHORS
Yuri Cherio

# COPYRIGHT
Copyright (C) 2026 The Utility Project Contributors\
SPDX-License-Identifier: GPL-3.0-or-later

# SEE ALSO

**flock**(1), **kill**(1), **sha256sum**(1), **keep-one-running**(1), **keep-running**(1), **run-constantly**(1), **run-one-constantly**(1), **run-one-until-failure**(1), **run-one-until-success**(1), **run-until-failure**(1), **run-until-success**(1)

Full documentation and source code available at [https://github.com/user/project](https://github.com/user/project)
