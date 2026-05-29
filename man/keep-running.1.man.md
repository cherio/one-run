---
title: keep-running
section: 1
header: User Commands
date: May 28, 2026
footer: Version 1.0.6
---

# NAME

**keep-running** - a wrapper utility that launches a command and relaunches it when it terminates

# SYNOPSIS

**keep-running** [*OPTIONS*].\.. COMMAND [*ARGS*].\..

**run-constantly** [*OPTIONS*].\.. COMMAND [*ARGS*].\..

**run-until-failure** [*OPTIONS*].\.. COMMAND [*ARGS*].\..

**run-until-success** [*OPTIONS*].\.. COMMAND [*ARGS*].\..

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
: This parameter controls the relaunch frequency. It assumes that, under normal conditions, the command will not terminate often. BURST - The number of times the command can relaunch immediately before throttling begins. RATE - The minimum required interval (in ms) between relaunches once the burst is exhausted. If BURST is 0 or 1, throttling engages immediately, ensuring the command does not restart more than once per RATE interval. The default is 16:1000 (a initial burst of 16 with subsequent throttling to launching a new instance not more frequently than once per second)

` `
: In other words, if a command exits immediately, it is allowed to relaunch without delay for a total of BURST attempts. Once this limit is reached, the utility enforces a mandatory RATE (in milliseconds) sleep between subsequent restarts to throttle the execution speed.

**-E EXIT_CODE**
: custom error exit code; enables the user to differentiate between the command failure and the utility exit error (see EXIT STATUS section for details)

**-D PATH**
: debug/log file path.

**-h | -\-help**
: Print help information

# EXAMPLES

**keep-running -R 10:2000 -r 2000 my-favorite-watchdog arg1**
: keeps relaunching the user process allowing up to 2000 respawns; applies rate limiting for frequent command restarts allowing 10 relaunches in one single initial burst (when a process exists very frequently/instantly and gets relaunched immediately) and then limiting the frequency of respawns to minimum 2000 milliseconds between launches, pausing/sleeping in between.

**run-constantly tail -f /path/to/log-file**
: this one is useful if log file is rotated by the utility that writes the log (some versions of tail lose track of the file descriptor when it is rotated)

# EXIT STATUS

**keep-running** and variants exit with the following status:
```
0       upon success
123     when this utility encounters a problem; customizable with -E
```
Exit code 123 is returned when the utility itself encounters an abnormal situation, e.g. when a parameter is invalid of respawn count is exceeded. This code is customizable with -E option.

Exit code from **run-until-failure** is never zero. It is either 123 (customizable) or it is the exit code returned by the user command.

# AUTHORS
Yuri Cherio

# COPYRIGHT
Copyright (C) 2026 The Utility Project Contributors\
SPDX-License-Identifier: GPL-3.0-or-later

# SEE ALSO

**keep-one-running**(1), **run-one**(1), **run-one-constantly**(1), **run-one-until-failure**(1), **run-one-until-success**(1), **run-this-one**(1)
