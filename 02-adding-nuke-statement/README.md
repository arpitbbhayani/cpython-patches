Adding a new `nuke` statement
===

To understand the CPython codebase better, let's make a significant change and add the `nuke` command that just exits the program abruptly with an exit code of `45`.

```
>>> nuke
```

## Approaching this

One command that has similar (not same) semantics as `nuke` is `break`. The `break` statement also does not take any addititional information and just exits the loop. Instead existing the loop we just need to abruptly exit the program.

The approach we take here is to find our all referneces of `break` statement and add `nuke` specific changes next to it.

## Changing grammar

For us to support the new statement, we have to update the grammar and add `nuke` support to it. Python grammar can be found in `Grammar/python.gram` file. We find reference of `break` and add `nuke` next to it.

Once we make the changes to the grammar, we have to regenerate the parser by running the command below.

```
$ make regen-pegen
```

## Updating Source

We recursively go through the references of new functions, types, added and keep updating the source adding similar things for "Nuke".

Please refer to the `patch.diff` file to understand what changed.

## Build and Run

To build the changes and the binary from the source we run the following command

```
$ make
```

We not start the shell and fire `nuke` and we see that the runtime abruptly exits with an exit code of 45.

```
$ ./python
Python 3.12.0a7+ (heads/main-dirty:fb8739f0b6, May 15 2023, 01:34:02) [GCC 9.4.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> nuke
$ echo $?
45
```
