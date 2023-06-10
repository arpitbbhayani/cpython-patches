How `print` prints
===

Explainer video: https://youtu.be/vuZO4qzjCdQ

To understand how python prints using the `print` statement, let's look for its implementation.

## Approaching this

Finding the implementation of `print` is cumborsome given how common the word `print` is. Even a simple search for `print` throughout the codebase yeilds more than 5400 instances.

Hence, we check the `help` message of the `print` statement and use that to narrow down the files that could potentially hold the implementation.

```py
>>> help(print)
Help on built-in function print in module builtins:

print(...)
    print(value, ..., sep=' ', end='\n', file=sys.stdout, flush=False)
    
    Prints the values to a stream, or to sys.stdout by default.
    Optional keyword arguments:
    file:  a file-like object (stream); defaults to the current sys.stdout.
    sep:   string inserted between values, default a space.
    end:   string appended after the last value, default a newline.
    flush: whether to forcibly flush the stream.
```

We search for the phrase "Prints the values to a" in the codebase and it yields just two files

- `bltinmodule.c`
- `bltinmodule.c.h`

The implementation should be present in the `.c` file and we land on the actual implementation function named `builtin_print_impl`.

```c
static PyObject *
builtin_print_impl(PyObject *module, PyObject *args, PyObject *sep,
                   PyObject *end, PyObject *file, int flush) {

    ...
    ...
    ...
}
```

## Changing the `print` statement

We change the `print` statement and along with printing the intended, we also print

- type of the object that we are printing
- reference count of the object

We pick the above two fields because the code `PyObject` contains these two as its members.

Please refer to the `patch.diff` file to understand what changed.

## Build and Run

To build the changes and the binary from the source we run the following command

```
$ make
```

Now we run the python and see our changes in action

```
$ ./python
Python 3.12.0a7+ (heads/main-dirty:fb8739f0b6, May 15 2023, 01:34:02) [GCC 9.4.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> d = {"a": 1}
>>> print(d)
<dict:3> {'a': 1}
>>> print("hello, world!")
<str:4> hello, world!
```
