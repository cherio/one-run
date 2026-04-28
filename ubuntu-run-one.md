### A Word about why I made another version

I admire and respect the author of the original **run-one** utility. I used
that tool and it was good. But it was lacking features and has one design flaw
that troubled me (more about that below).

### Compatibility and design notes

This is a from-scratch implementation that provides CLI-compatible behavior
with the Ubuntu **run-one** utility. Most usage patterns should translate
directly. It is not affiliated with or derived from the original
implementation.

Bugs or feature requests for this implementation should be directed to this
project’s issue tracker.

#### Locking + Respawning dilemma

The original utility's `keep-one-running` variants combine two functionalities
into one: locking and process respawning. My version, while keeping functional
compatibility, keeps these two capabilities separate, and provides an
additional set of runners keeping the naming pattern:
```
keep-running
run-constantly
run-until-failure
run-until-success
```
This version of `keep-one-running`/`run-one-until-*`/`run-one-constantly`
acquires the lock first and then keeps the command running, relaunching when it
terminates. The original utility attempts to acquire the lock every time it
needs to respawn the command.

This is one subtle difference from the original utility's behavior, which can
still be achieved by running separate commands cascading as follows:
```
keep-running + run-one + cmd
```
I believe the chosen here behavior, that aquires the lock and then keeps the
executable running, makes more practical sense than trying to acquire lock it
every time user command respawns.

When using the original utility the situation gets even more complicated, as it
is impossible to tell whether utility fails because flock fails to acquire the
lock or user command is unsiccessful. From this perspective, the original
utility logic of run-one-until-failure/run-one-until-success can lead to
ambiguous outcomes because it doesn't track the failure code of the user
command only, and can have false positives when flock fails.

This utility returns a different code when either flock or its internal logic
triggers an exit. By default, the code is 122 and can be explicitly specified
with the **-E** argument.

#### Killing the others

Another difference is that this utility does not make an attempt to manage
processes launched outside of its scope. It can't stop a user or another
background process from launching a command with identical signature, so that
two identical instances can still run in parallel regardless of the lock
legitimately acquired by run-one. I always questioned  whether the original
utility took too much upon itself by killing off processes that were not
intended to run as singletons.

Bottom line. If a user needs a single instance, they shuold use run-one. If
multiple instances can be spawned elsewhere in the system - do not expect this
utility to control things that can't be controlled.

Process termination is optimized for speed. I experimented trying **lsof** and
**fuser** and found them very slow. They both produce a noticeable pause. This
utility implements a fast **/proc** scan to identify the other processes
holding the semaphore.

### Highlights of the additional features

* Optimized for speed. Whenever possible, the script uses internal shell
computations, built-ins and shell parameter expansion, avoiding external
command execution calls.

* Distinguishes between user command exit codes and `flock` success/failure
exist codes making it possible to tell whether the user command failed or
`flock` was unsuccessful. The default flock failure exit code is 122 and can be
changed via optional parameters.

* Optional command line parameters control and configure certain aspects of the
script behavior, while keeping backward compatibility with the original
utility.

* The utility prefers tmpfs storage to keep lock files, giving the
highest priority to XDG_RUNTIME_DIR mount if present. This way all the
temporary lock files get cleaned up upon reboot.

* The utility maintains GNU Linux compatibility and is expected to run
correctly on most modern Linux distributions. It's not entirely POSIX
compatible, but neither is the original run-one

* Ubuntu version of run-this-one hangs infinitely waiting for the other
processes to terminate. This implementation allows setting optional wait
timeout.

* The original keep-one-running variants combine two functions: locking and
process respawning. This version, while maintaining backward compatibility,
separates the two and provides a set of additional wrappers that can keep a
command running but without locking the semaphore allowing mutiple command
instances to run simultaneously. For instance the following can run from
different directories:

```
# cd /some/directory
keep-running sh -c 'on-new-file "$(inotifywait . -e close_write)"'
```

* Introduced basic logging and status reporting.

* This version of run-this-one does not attempt to kill other instances of the
child process that were not launched through locking mechanism. There's no
point in doing that. Even if run-this-one kills the other instances, they can
be relaunched from the same source that does not run them through run-one. In
other words, user command that was not launched through the locking mechanism
cannot reliably be controlled by this utility and therefore, there is no need
to complicate the utility.

* This variant allows to specify the semaphore unique id not necessarily
  derived from the user command, thus adding flexibulity to run only one
  instance of user command but with different signatures. For instance you can
  run
```
run-this-one -T 'XYZ123' ncat -l 8080
```
but then make a correction and re-run it replacing the original command with additional arguments
```
run-this-one -T 'XYZ123' ncat -l 8080 -k
```
