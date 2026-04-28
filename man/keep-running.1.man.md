% keep-running(1) Version 1.0 | User Commands
% Yuri Cheromushnikov
% April 22, 2026

# NAME

**keep-running** - a wrapper utility that launches a command and relaunches it when it terminates

# SYNOPSIS

**keep-running** [*OPTIONS*] ... COMMAND [*ARGS*] ...

**run-constantly** [*OPTIONS*] ... COMMAND [*ARGS*] ...

**run-until-failure** [*OPTIONS*] ... COMMAND [*ARGS*] ...

**run-until-success** [*OPTIONS*] ... COMMAND [*ARGS*] ...

# DESCRIPTION

The base form of utility simply keeps a program running, relaunching it whenever it exits.

**run-constantly** is an alias for **keep-running**.

**run-until-failure** is equivalent to / an alias for **keep-running -u f**. It respawns user command until it returns a non-zero exit code.

**run-until-success** is equivalent to / an alias for **keep-running -u s**. It responds user command until it returns with the exit code of zero.

# OPTIONS

**-u [s|f]**
: This parameter defines respawn loop exit condition. **'s'** orders to stop relaunching when the command exit code is zero (same as running **run-until-success**), while **'f'** exit the loop when user command fails (same as running **run-until-failure**).

**-r NUMBER**
: limits the number of times the supplied command can be launched. By default it keeps respawning the command indefinitely.

**-R BURST:RATE**
: This parameter controls the relaunch frequency. It assumes that, under normal conditions, the command will not terminate often. BURST - The number of times the command can relaunch immediately before throttling begins. RATE - The minimum required interval (in ms) between relaunches once the burst is exhausted. If BURST is 0 or 1, throttling engages immediately, ensuring the command does not restart more than once per RATE interval.

` `
: In other words, if a command exits immediately, it is allowed to relaunch without delay for a total of BURST attempts. Once this limit is reached, the utility enforces a mandatory RATE (in milliseconds) sleep between subsequent restarts to throttle the execution speed.

**-E EXIT_CODE**
: custom error exit code; enables the user to differentiate between the command failure and the utility exit error (see EXIT STATUS section for details)

**-D PATH**
: debug/log file path.

# EXAMPLES

**keep-running -R 10:2000 -r 2000 my-favorite-watchdog arg1**
: keeps relaunching the user process allowing up to 2000 respawns; applies rate limiting for frequent command restarts allowing 10 relaunches in one single initial burst (when a process exists very frequently/instantly and gets relaunched immediately) and then limiting the frequency of respawns to minimum 2000 milliseconds between launches, pausing/sleeping in between.


# EXIT STATUS

**keep-running** and variants exit with the following status:
```
0       upon success
123     when this utility encounters a problem; customizable with -E
```
Exit code 123 is returned when the utility itself encounters an abnormal situation, e.g. when a parameter is invalid of respawn count is exceeded. This code is customizable with -E option.

Exit code from **run-until-failure** is never zero. It is either 123 (customizable) or it is the exit code returned by the user command.

# SEE ALSO

**keep-one-running**(1), **run-one**(1), **run-one-constantly**(1), **run-one-until-failure**(1), **run-one-until-success**(1), **run-this-one**(1)
