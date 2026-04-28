# Description

A wrapper utility that prevents concurrent execution of a command by ensuring
only a single instance runs at any given time. This utility is a from-scratch
implementation designed to be CLI-compatible with the Ubuntu **run-one**
utility by Dustin Kirkland. It can be used as a compatible replacement in
typical scenarios while providing [additional features](ubuntu-run-one.md).

## Main form

**run-one** [options] command [arguments]

Executes the specified command with its arguments. If an identical invocation is already running, **run-one** exits without starting a duplicate.

**run-this-one** [options] command [arguments]

Same as **run-one** except it makes an attempt to terminate the other instance of the running command, if it was previously started with this utility.

## Keep-me-running forms

**keep-running** [options] command [arguments]

Runs the command and restarts it when the command exits. This particular command wrapper does not exist in the original utility.

**run-constantly** is an alias for **keep-running**

**run-until-failure** is equivalent to **keep-running -u f**. If the launched command terminates with zero code it gets restarted.

**run-until-success** is equivalent to **keep-running -u s**. If the launched command terminates with non-zero code it gets restarted.

The following four variants have the same name and semantics as in the original package. This command form is equivalent to running **keep-running**, but in addition it checks whether a duplicate command was previously started with either **run-one** or **keep-one-running** and if is still an active process this invocation simply exists.

**keep-one-running** [options] command [arguments]

**run-one-constantly** is an alias to **keep-one-running**

**run-one-until-failure** is equivalent to **keep-one-running -u f**. If the launched command terminates with zero code it gets restarted.

**run-one-until-success** is equivalent to **keep-one-running -u s**. If the launched command terminates with non-zero code it gets restarted.

When **keep-one-running** is invoked with **-x**, it attempts to terminate all other instances of the command and make sure that only this instance runs instead. This form of command is not present in the original package.


# Examples

Launch this in one terminal:

```
run-one sleep 3600
```
Then run the following in a different terminal:

```
run-one sleep 3600
$ time run-one sleep 3600
Another process is already running

real	0m0.006s
user	0m0.001s
sys	0m0.005s
$ echo $?
122
$
```
When the utility encounters an issue or fails to secure the command instance lock, it has a distinct exit code (default 122) which can be customized via an argument. This new feature allows distinguishing unsuccessful exit codes from the user command and the utility.

You can find more examples in the "man" pages.

# System compatibility

This utility should have no issues running in any GNU Linux distribution.

# Compatibility with run-one

This is an independent, from-scratch implementation that provides
CLI-compatible behavior with the popularized by the Ubuntu
[**run-one**](https://github.com/dustinkirkland/run-one) utility. It is not
affiliated with or derived from the original implementation.

Most usage patterns should translate directly. Bugs and feature requests for
this version should be directed to this project’s issue tracker.
