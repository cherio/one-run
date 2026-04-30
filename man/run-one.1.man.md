% run-one(1) Version 1.0 | User Commands
% Yuri Cheromushnikov
% April 22, 2026

# NAME
**run-one** - a wrapper utility that prevents concurrent execution of a command by ensuring only a single instance runs at any given time


# SYNOPSIS

**run-one** [_OPTIONS_] ... COMMAND [*ARGS*] ...

**run-this-one** [_OPTIONS_] ... COMMAND [*ARGS*] ...


# DESCRIPTION

This utility prevents multiple instances of the same command from running concurrently. It can either block a new instance from starting if one is already active (**run-one**), or terminate the existing process and let the new one take over (the **run-this-one** variant).

To ensure only one instance runs at a time, this utility uses a flock-based semaphore. The unique ID for the lock is either specified on the command line using the **-T** parameter or generated automatically by hashing the user command line. The semaphore file is stored in one of the following directories in this order of preference:

```
$XDG_RUNTIME_DIR/one-run
/run/user/$UID/one-run
/dev/shm/one-run_$UID
/tmp/one-run_$UID
```
Most of these directories are on **tmpfs** file systems (/tmp is often a **tmpfs** mount in many recent Linux distributions), and all of them get cleaned up upon reboot.

The user supplied command line is then launched with a flock. If the corresponding lock file is available, then the command gets launched. If the lock on the file was already acquired by another process then depending on the flavor of command (**run-one**/**run-this-one**) or use of option **-x**, the utility either exits or attempts to terminate other command instances, waiting for the lock to be released so that the user command can be launched.

Wait time can be limited with **-w**. In addition the other process can be terminated with `SIGKILL` if it doesn't respond to `SIGTERM` in under the number of seconds specified with **-k**.

# OPTIONS

**-x**
: terminates other commands with the identical command ID (or command signature) previously launched by **run-one** or **run-this-one**, locks the semaphore and runs this command; equivalent to running **run-this-one**

**-w SECONDS**
: (**run-this-one**) maximum time (in seconds) to wait for the lock to become available before giving up on acquiring the lock; only whole numbers are understood, decimals are not allowed

**-k SECONDS**
: (**run-this-one**) send `SIGKILL` to the other process instance that holds the lock holder after this many seconds of waiting for it to terminate gracefully with `SIGTERM`; only whole numbers are understood, decimals are not allowed

**-T TOKEN**
: use this token for the lock file name instead of calculating user command hash. This option is also used internally when running **keep-one-running cmd**, as this command variant internally runs as **run-one + keep-running + cmd** but needs a hash token for **cmd** and not **keep-running + cmd**

**-E EXIT_CODE**
: custom error exit code; enables the user to differentiate between the command failure and the utility exit error (see EXIT STATUS section for details)

**-D PATH**
: debug/log file path

**-h | --help**
: Print help information

# EXAMPLES

```
run-this-one inotifywait -m ~/tmp -e close_write | \
	run-until-failure -- sh -c 'read -r event && echo "$event"'
```

Sets a watcher on ~/tmp directory and keeps printing events that occur whenever a new file is written in it. If another identical instance of such watcher was already running, it gets terminated prior to running this instance.

```
run-one -x -T -w 60 -k 30 "Watch my DIR" inotifywait -m ~/tmp -e close_write | \
	keep-running -u f -r 100 -- sh -c 'read -r event && echo "$event"'
```
This does the same as the previous example, except it matches the command by the provided ID/token "Watch my DIR", instead of command signature. This enables replacing this instance with an updated command later e.g. if you decide to watch **/tmp** instead of **~/tmp**. It also limits the number of printed events to 100; it sends SIGKILL to another process that holds the lock after 30 sec of waiting and quits waiting altogether after 1 min.

# EXIT STATUS

**run-one** and **run-this-one** exit with one of the following status:
```
0     upon success
122   when this utility encounters a problem; customizable with -E
```
Exit code 122 is returned the utility itself encounters an abnormal situation, e.g. when a parameter is invalid of flock fails. This code is customizable with -E option.

Any other exit code indicates the status returned from the user command.


# SEE ALSO

**flock**(1), **kill**(1), **sha256sum**(1), **keep-one-running**(1), **keep-running**(1), **run-constantly**(1), **run-one-constantly**(1), **run-one-until-failure**(1), **run-one-until-success**(1), **run-until-failure**(1), **run-until-success**(1)
