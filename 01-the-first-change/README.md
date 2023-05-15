Making the first change
===

Explainer video: https://youtu.be/UmF57P7TerI

Working with open source is an enriching experience, and you feel really great when you understand a complex piece of code.
It sounds really interesting, but most people get stuck on step 1 - finding out where does the code start?

Let me walk you through the process I follow to navigate a huge open source codebase and find the starting point and making
our first change.

## Setup

1. Clone the repository: https://github.com/python/cpython
2. Follow the setup instructions present in the `README` file

```
./configure
make
sudo make install
```

3. Run the following command to start the Python shell built from your source

```
$ ./python
```

## Altering the code

Make the first change in the source code to understand

1. your setup is correct
2. changes you will make will be picked up
3. where it all begins

Given that the CPython builds Python in C language, it should have the `main()` function as its entry point. Let's find that. You can use the regular `Ctrl + F` method to find the substring `main(` in the codebase.

While doing so, you will stumble upon a bunch of files. Going through them will familiarize you bit with the codebase. But eventually you will stumble upon the file

```
Modules/main.c
```

the file contains the function `Py_RunMain` which looks like something the Pytthon interpreter should start. Check its reference we uncover a function `Py_Main`, taking us closer.

We now search `Py_Main(` in our codebase and finally we find the file

```
Programs/python.c
```

which contains the following code snippet

```c
/* Minimal main program -- everything is loaded from the library */

#include "Python.h"

#ifdef MS_WINDOWS
int
wmain(int argc, wchar_t **argv)
{
    return Py_Main(argc, argv);
}
#else
int
main(int argc, char **argv)
{
    return Py_BytesMain(argc, argv);
}
#endif
```

The above code snippet is clearly the entry point to our codebase as it contains the classic `int main(int argc, char **argv)`.

## Understanding Conditional Compilation

### Two `main` functions

The above code snippet is a macro that defines two different functions, `wmain()` and `main()`. The `wmain()` function is used on Windows systems, while the `main()` function is used on other systems.

C language has a different entry point for the Windows platform. The entry point for the Windows platform is `wmain()`.

The reason for the difference is that Windows uses a different character encoding than other platforms. Windows uses the Unicode character encoding, while other platforms typically use the ASCII character encoding. The `wmain()` function is designed to work with Unicode strings, while the `main()` function is designed to work with ASCII strings.

If we are writing a C program that will be compiled for Windows, we should use the `wmain()` function. If we are writing a C program that will be compiled for other platforms, you should use the `main()` function.

### Conditional Compilation

The `#ifdef` directive is used to conditionally compile code. The `MS_WINDOWS` macro is defined on Windows systems, so the `wmain()` function will be compiled if the code is being compiled for Windows. Otherwise, the `main()` function will be compiled.

The `#else` directive is used to specify code that should be compiled if the condition specified by the `#ifdef` directive is not met. In this case, the `main()` function will be compiled if the code is not being compiled for Windows.

The `#endif` directive marks the end of the conditional compilation block.

## Making our first change

Our first change can be pretty straight forward, where we can simply add `printf` statement in `main` function. But which?

1. add to both of them
2. add to the one that will be used for your underlying platform

### Close observation

Both `main` functions invokes one function each and it is either

1. `Py_Main`, or
2. `Py_BytesMain`

Observing both of them reveals that they both invoke the same function `pymain_main` which means we can simply add our `printf` statement to this function and now no matter which platform our code compiles on, we would see our `printf` getting executed.

```c
int
Py_Main(int argc, wchar_t **argv)
{
    ...
    return pymain_main(&args);
}


int
Py_BytesMain(int argc, char **argv)
{
    ...
    return pymain_main(&args);
}
```

## Updating `pymain_main`

We will add a simple `printf` in the beginning of the function and then compile the source.

```c
static int
pymain_main(_PyArgv *args)
{
	printf("Arpit's version of Python\n");
    ...
}
```

Compiling the source is as simple as issuing `make` and then running our shell from the python executable generated from our source code should render the following output.

```
$ make
$ ./python
Arpit's Version of Python
Python 3.12.0a7+ (heads/main-dirty:fb8739f0b6, May 15 2023, 01:34:02) [GCC 9.4.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>>
```
