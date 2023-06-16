Supporting arrow `->` operator on attributes
===

Explainer video: TODO

In python, we can define classes and members within in. To access
the member of the class, we use the dot `.` operator. A simple example
of this is as shown below.

```py
class Person:
    def __init__(self, name):
        self.name = name

p1 = Person("arpit")
print(p1.name)
del(p1.name)
```

What if Python supported accessing member attributes using
the arrow operator `->` just like C.

## Approaching this

We approach this by going through the CPython grammar `Grammar/python.gram`
file and find out a place where the rule is defined.

It could be as simple as looking for the tokrn `'.'` and we find roughly
13 places. Going through each reference, we try to guess what it does.

## Changing the grammar

We find the rule that defines accessing attribute and we just add one
more clause beneath the `'.'` one by replacing `'.'` with `'->'` keeping
everything same.

```
primary[expr_ty]:
    | a=primary '.' b=NAME { _PyAST_Attribute(a, b->v.Name.id, Load, EXTRA) }
    | a=primary '->' b=NAME { _PyAST_Attribute(a, b->v.Name.id, Load, EXTRA) }
    ...
    ...
    ...
```

We find similar places where we create `_PyAST_Attribute` in the grammar file
and add a new clause at every place.

Please refer to the `patch.diff` file to understand what changed.

## Build and Run

To build the changes and the binary from the source we run the following command

```
$ make regen-pegen
$ make
```

Now we run the python and see our changes in action

Create a simple python file with following snippet `class.py`

```py
class Person:
    def __init__(self, name):
        self.name = name

p1 = Person("arpit")
print(p1->name)
del p1->name
```

```sh
$ ./python class.py
arpit
```

Now our version of Python supports accessing class members using an
arrow `->` operator.
