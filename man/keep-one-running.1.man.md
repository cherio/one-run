% keep-one-running(1) Version 1.0 | User Commands
% Yuri Cheromushnikov
% April 22, 2026

# NAME

**keep-one-running** - a wrapper utility that launches a single instance of the supplied command and relaunches it when it terminates

# SYNOPSIS

**keep-one-running** [*OPTIONS*] ... COMMAND [*ARGS*] ...

**run-one-constantly** [*OPTIONS*] ... COMMAND [*ARGS*] ...

**run-one-until-failure** [*OPTIONS*] ... COMMAND [*ARGS*] ...

**run-one-until-success** [*OPTIONS*] ... COMMAND [*ARGS*] ...

# DESCRIPTION

**keep-one-running** is a wrapper that both insures the command respawns should it exit and only one instance of the command runs at a time.

Internally it wraps **run-one** and **keep-running** and inherits all command options from each.

This form is for the most part compatible with the original Ubintu **run-one** utility. The internal design differs in one way that it acquires the lock first and then keeps the command running, relaunching when it terminates.

The original run-one utility acquires the lock every time it needs to respawn the command. This creates ambiguity with **run-until-failure**/**run-until-success** variants as there is no way of telling whether a failure comes from the utility itself, from locking the file with flock or from the user command. This implementation solves the ambiguity by locking prior to launching the respawn loop and allowing a different/custom exit code for the utility and flock.

To achieve the behavior exactly compatible with the original utility, construct the command as follows:

	keep-running run-one command [args ...]

For details regarding how the lock is acquired and managed, see man for **run-one**.

for details regarding the logic of relaunching the command, see man for **keep-running**.

**run-one-constantly** is an alias for **keep-one-running**.

**run-one-until-failure** is equivalent to running the following:

	keep-one-running -u f

**run-one-until-success** is equivalent to running the following:

	keep-one-running -u s

With this utility, it is also possible to force running only this command instance and terminate the other by specifying **-x** flag:

	keep-one-running -x command args ...


# OPTIONS

Available options are the superset of options in **run-one** and **keep-running**.

**-x**
: terminates other commands with the identical command ID (or command signature) previously launched by **run-one** or **run-this-one**, locks the semaphore and runs this command; equivalent to running **run-this-one**

**-w SECONDS**
: (**run-this-one**) maximum time (in seconds) to wait for the lock to become available before giving up on acquiring the lock; only whole numbers are understood, decimals are not allowed

**-k SECONDS**
: (**run-this-one**) send `SIGKILL` to the other process instance that holds the lock holder after this many seconds of waiting for it to terminate gracefully with `SIGTERM`; only whole numbers are understood, decimals are not allowed

**-a**
: run this invocation anyway, even if another process did not give up the lock in the given wait time. This invalidates the lock acquired by another process and creates a new lock file

**-T TOKEN**
: use this token for the lock file name instead of calculating user command hash. This option is also used internally when running **keep-one-running cmd**, as this command variant internally runs as **run-one + keep-running + cmd** but needs a hash token for **cmd** and not **keep-running + cmd**

**-E EXIT_CODE**
: custom error exit code; enables the user to differentiate between the command failure and the utility exit error (see EXIT STATUS section for details)

**-D PATH**
: debug/log file path

**-u [s|f]**
: This parameter defines respawn loop exit condition. **'s'** orders to stop relaunching when the command exit code is zero (same as running **run-until-success**), while **'f'** exit the loop when user command fails (same as running **run-until-failure**).

**-r NUMBER**
: limits the number of times the supplied command can be launched. By default it keeps respawning the command indefinitely.

**-R BURST:RATE**
: This parameter controls the relaunch frequency. It assumes that, under normal conditions, the command will not terminate often. BURST - The number of times the command can relaunch immediately before throttling begins. RATE - The minimum required interval (in ms) between relaunches once the burst is exhausted. If BURST is 0 or 1, throttling engages immediately, ensuring the command does not restart more than once per RATE interval.

` `
: In other words, if a command exits immediately, it is allowed to relaunch without delay for a total of BURST attempts. Once this limit is reached, the utility enforces a mandatory RATE (in milliseconds) sleep between subsequent restarts to throttle the execution speed.

**-h | --help**
: Print help information

# EXAMPLES

**run-one-until-success -R 1:20000 ncat -z -w5 my-host 443**
: make sure only one instance of ncat is running at a time; keep running it until it succeeds connecting to port 443 of my-host; limit the frequency of connectivity checks to 20 seconds

**keep-one-running -u s -R 1:20000 -x -w 120 -k 60 -E 111 -D /tmp/debug.msg -- ncat -z -w5 my-host 443**
: same as the previous one, but in addition does this: terminates other instances of this command that held the lock waiting 2 min total for the other process to terminate and additionally sends SIGKILL to it after 1 min of waiting; uses exit code 111 in case the utility encounters an internal error; dumps some debug/status information in /tmp/debug.msg.

# EXIT STATUS

**keep-one-running** and variants exit with the following status:

```
0       upon success
123     when this utility encounters a problem; customizable with -E
```
Exit code 123 is returned the utility itself encounters an abnormal situation, e.g. when a parameter is invalid of flock fails. This code is customizable with -E option.

Exit code from **run-one-until-failure** is never zero. It is either 123 (customizable) or it is the exit code returned from the user command.

# SEE ALSO

**keep-running**(1), **run-constantly**(1), **run-one**(1), **run-this-one**(1), **run-until-failure**(1), **run-until-success**(1)
