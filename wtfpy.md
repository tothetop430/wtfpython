# What the f*ck Python!

[![WTFPL 2.0][license-image]][license-url]

> A collection of tricky Python examples

Python being an awesomoe higher level language, provides us many functionalities for our comfort. But sometimes, the outcomes may not seem obvious to a normal Python user at the first sight. Here's an attempt to collect such classic examples of unexpected behaviors in Python and see what exactly is happening under the hood! I find it a nice way to learn internals of a language and I think you'll like them as well!

# Table of Contents

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Checklist](#checklist)
- [👀 Examples](#-examples)
  - [Example heading](#example-heading)
    - [Explanation:](#explanation)
  - [`datetime.time` object is considered to be false if it represented midnight in UTC](#datetimetime-object-is-considered-to-be-false-if-it-represented-midnight-in-utc)
    - [Explanation](#explanation)
  - [`is` is not what it is!](#is-is-not-what-it-is)
    - [💡 Explanation:](#-explanation)
  - [The function inside loop magic](#the-function-inside-loop-magic)
    - [Explaination](#explaination)
  - [Loop variables leaking out of local scope!](#loop-variables-leaking-out-of-local-scope)
    - [Explanation](#explanation-1)
  - [A tic-tac-toe where X wins in first attempt!](#a-tic-tac-toe-where-x-wins-in-first-attempt)
    - [Explanation](#explanation-2)
  - [Beware of default mutable arguments!](#beware-of-default-mutable-arguments)
    - [Explanation](#explanation-3)
  - [You can't change the values contained in tuples because they're immutable.. Oh really?](#you-cant-change-the-values-contained-in-tuples-because-theyre-immutable-oh-really)
    - [Explanation](#explanation-4)
  - [Using a varibale not defined in scope](#using-a-varibale-not-defined-in-scope)
- [Contributing](#contributing)
- [Acknowledgements](#acknowledgements)
- [🎓 License](#-license)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# 👀 Examples

## Example heading

One line of what's happening:

```py
setting up
```

```py
>>> triggering_statement
weird output
```

### Explanation:

* Better to give outside links
* or just explain again in brief

## `datetime.time` object is considered to be false if it represented midnight in UTC

```py
from datetime import datetime

midnight = datetime(2018, 1, 1, 0, 0)
midnight_time = midnight.time()

noon = datetime(2018, 1, 1, 12, 0)
noon_time = noon.time()

if midnight_time:
    print("Time at midnight is", midnight_time)

if noon_time:
    print("Time at noon is", noon_time)
```

**Output:**
```sh
('Time at noon is', datetime.time(12, 0))
```

### Explanation

Before Python 3.5, a datetime.time object was considered to be false if it represented midnight in UTC. It is error-prone when using the `if obj:` syntax to check if the `obj` is null or some equivalent of "empty".


## `is` is not what it is!


```py
>>> a = 256
>>> b = 256
>>> a is b
True

>>> a = 257
>>> b = 257
>>> a is b
False

>>> a = 257; b = 257
>>> a is b
True
```


### 💡 Explanation:

**The difference between `is` and `==`**

* `is` operator checks if both the operands refer to the same object (i.e. it checks if the identity of the operands matches or not).
* `==` operator compares the values of both the operands and checks if they are the same.
* So if the `is` operator returns `True` then the equality is definitely `True`, but the opposite may or may not be True.


**`256` is an existing object but `257` isn't**

When you start up python the numbers from `-5` to `256` will be allocated. These numbers are used a lot, so it makes sense to just have them ready.

Quoting from https://docs.python.org/3/c-api/long.html
> The current implementation keeps an array of integer objects for all integers between -5 and 256, when you create an int in that range you actually just get back a reference to the existing object. So it should be possible to change the value of 1. I suspect the behaviour of Python in this case is undefined. :-)

```py
>>> id(256)
10922528
>>> a = 256
>>> b = 256
>>> id(a)
10922528
>>> id(b)
10922528
>>> id(257)
140084850247312
>>> x = 257
>>> y = 257
>>> id(x)
140084850247440
>>> id(y)
140084850247344
```

Here the integer isn't smart enough while executing `y = 257` to recongnize that we've already created an integer of the value `257` and so it goes on to create another object in the memory.


**Both `a` and `b` refer to same object, when initialized with same value in same line**

* When a and b are set to `257` in the same line, the Python interpretor creates new object, then references the second variable at the same time. If you do it in separate lines, it doesn't "know" that there's already `257` as an object.
* It's a compiler optimization and specifically applies to interactive environment. When you do two lines in a live interpreter, they're compiled separately, therefore optimized separately. If you were to try this example in a `.py` file, you would not see the same behavior, because the file is compiled all at once.

```py
>>> a, b = 257, 257
>>> id(a)
140640774013296
>>> id(b)
140640774013296
>>> a = 257
>>> b = 257
>>> id(a)
140640774013392
>>> id(b)
140640774013488
```


## The function inside loop magic

```py
funcs = []
results = []
for x in range(7):
    def some_func():
        return x
    funcs.append(some_func)
    results.append(some_func())

funcs_results = [func() for func in funcs]
```

**Output:**
```py
>>> results
[0, 1, 2, 3, 4, 5, 6]
>>> funcs_results
[6, 6, 6, 6, 6, 6, 6]
```

//OR

```py
>>> powers_of_x = [lambda x: x**i for i in range(10)]
>>> [f(2) for f in powers_of_x]
[512, 512, 512, 512, 512, 512, 512, 512, 512, 512]
```

### Explaination

When defining a function inside a loop that uses the loop variable in its body, the loop function's closure is bound to the variable, not its value. So all of the functions use the latest value assigned to the variable for computation.

To get the desired behavior you can pass in the loop variable as a named varibable to the function which will define the variable again within the function's scope.

```py
funcs = []
for x in range(7):
    def some_func(x=x):
        return x
    funcs.append(some_func)
```

**Output:**
```py
>>> funcs_results = [func() for func in funcs]
>>> funcs_results
[0, 1, 2, 3, 4, 5, 6]
```


## Loop variables leaking out of local scope!

1.
```py
for x in range(7):
    if x == 6:
        print(x, ': for x inside loop')
print(x, ': x in global')
```

**Output:**
```py
6 : for x inside loop
6 : x in global
```

But `x` was never defined ourtside the scope of for loop...

2.
```py
# This time let's initialize x first
x = -1
for x in range(7):
    if x == 6:
        print(x, ': for x inside loop')
print(x, ': x in global')
```

**Output:**
```py
6 : for x inside loop
6 : x in global
```

3.
```
x = 1
print([x for x in range(5)])
print(x, ': x in global')
```

**Output (on Python 2.x):**
```
[0, 1, 2, 3, 4]
(4, ': x in global')
```

**Output (on Python 3.x):**
```
[0, 1, 2, 3, 4]
1 : x in global
```

### Explanation

In Python for-loops use the scope they exist in and leave their defined loop-variable behind. This also applies if we explicitly defined the for-loop variable in the global namespace before. In this case it will rebind the existing variable.

The differences in the output of Python 2.x and Python 3.x interpreters for list comprehension example can be explained by following change documented in [What’s New In Python 3.0](https://docs.python.org/3/whatsnew/3.0.html) documentation:

> "List comprehensions no longer support the syntactic form `[... for var in item1, item2, ...]`. Use `[... for var in (item1, item2, ...)]` instead. Also note that list comprehensions have different semantics: they are closer to syntactic sugar for a generator expression inside a `list()` constructor, and in particular the loop control variables are no longer leaked into the surrounding scope."


## A tic-tac-toe where X wins in first attempt!

```py
# Let's initialize a row
row = [""]*3 #row i['', '', '']
# Let's make a bord
board = [row]*3
```

**Output:**
```py
>>> board
[['', '', ''], ['', '', ''], ['', '', '']]
>>> board[0]
['', '', '']
>>> board[0][0]
''
>>> board[0][0] = "X"
>>> board
[['X', '', ''], ['X', '', ''], ['X', '', '']]
```

### Explanation

When we initialize `row` varaible, this visualization explains what happens in the memory

![image](/images/tic-tac-toe/after_row_initialized.png)

And when the `board` is initialized by multiplying the `row`, this is what happens inside the memory (each of the elements board[0], board[1] and board[2] is a reference to the same list referred by `row`)

![image](/images/tic-tac-toe/after_board_initialized.png)

## Beware of default mutable arguments!

```py
def some_func(default_arg=[]):
    default_arg.append("some_string")
    return default_arg
```

**Output:**
```py
>>> some_func()
['some_string']
>>> some_func()
['some_string', 'some_string']
>>> some_func([])
['some_string']
>>> some_func()
['some_string', 'some_string', 'some_string']
```

### Explanation

The default mutable arguments of functions in Python aren't really initialized every time you call the function. Instead, the recently assigned value to them is used as the default value. When we explicitly passed `[]` to `some_func` as the argument, the default value of the `default_arg` variable was not used, so the function returned as expected.

```py
def some_func(default_arg=[]):
    default_arg.append("some_string")
    return default_arg
```

```py
>>> some_func.__defaults__ #This will show the default argument values for the function
([],)
>>> some_func()
>>> some_func.__defaults__
(['some_string'],)
>>> some)func()
>>> some_func.__defaults__
(['some_string', 'some_string'],)
>>> some_func([])
>>> some_func.__defaults__
(['some_string', 'some_string'],)
```

A common practice to avoid bugs due to mutable arguments is to assign `None` as the default value and later check if any value is passed to the function corresponding to that argument. Examlple:

```py
def some_func(default_arg=None):
    if not default_arg:
        default_arg = []
    default_arg.append("some_string")
    return default_arg
```


## You can't change the values contained in tuples because they're immutable.. Oh really?

This might be obvious for most of you guys, but it took me a lot of time to realize it.

```py
some_tuple = ("A", "tuple", "with", "values")
another_tuple = ([1, 2], [3, 4], [5, 6])
```

**Output:**
```py
>>> some_tuple[2] = "change this"
TypeError: 'tuple' object does not support item assignment
>>> another_tuple[2].append(1000) #This throws no error
>>> another_tuple
([1, 2], [3, 4], [5, 6, 1000])
```

### Explanation

Quoting from https://docs.python.org/2/reference/datamodel.html

> Immutable sequences
    An object of an immutable sequence type cannot change once it is created. (If the object contains references to other objects, these other objects may be mutable and may be changed; however, the collection of objects directly referenced by an immutable object cannot change.)

## Using a varibale not defined in scope

```py
a = 1
def some_func():
    return a

def another_func():
    a += 1
    return a
```

**Output:**
```py
>>> some_func()
1
>>> another_func()
UnboundLocalError: local variable 'a' referenced before assignment
```

### Explanation
When you make an assignment to a variable in a scope, it becomes local to that scope. So `a` becomes local to the scope of `another_func` but it has not been initialized previously in the same scope which throws an error. Read [this](http://sebastianraschka.com/Articles/2014_python_scope_and_namespaces.html) short but awesome guide to learn more about how namespaces and scope resolution works in Python.


## The disappearing variable from outer scope

```py
e = 7
try:
    raise Exception()
except Exception as e:
    pass
```

**Output (Python 2.x):**
```py
>>> print(e)
# prints nothing
```

**Output (Python 3.x):**
```py
>>> print(e)
NameError: name 'e' is not defined
```

### Explanation

* Source: https://docs.python.org/3/reference/compound_stmts.html#except

  When an exception has been assigned using as target, it is cleared at the end of the except clause. This is as if

  ```py
  except E as N:
      foo
  ```

  was translated to

  ```py
  except E as N:
      try:
          foo
      finally:
          del N
  ```

  This means the exception must be assigned to a different name to be able to   refer to it after the except clause. Exceptions are cleared because with the   traceback attached to them, they form a reference cycle with the stack frame, keeping all locals in that frame alive until the next garbage collection occurs.

* The clauses are not scoped in Python. Everything in the example is present in same scope and the variable `e` got removed due to the execution of the `except` clause. The same is not the case with functions which have their separate inner-scopes. The example below illustrates this:

 ```py
 def f(x):
     del(x)
     print(x)

 x = 5
 y = [5, 4, 3]
 ```

 **Output:**
 ```py
 >>>f(x)
 UnboundLocalError: local variable 'x' referenced before assignment
 >>>f(y)
 UnboundLocalError: local variable 'x' referenced before assignment
 >>> x
 5
 >>> y
 [5, 4, 3]
 ```

* In Python 2.x the variable name `e` gets assigned to `Exception()` instance, so when you try to print, it prints nothing.

**Output (Python 2.x):**
```py
>>> e
Exception()
>>> print e
# Nothing is printed!
```


## Return in both `try` and `finally` clauses

```py
def some_func():
    try:
        return 'from_try'
    finally:
        return 'from_finally'
```

**Output:**
```py
>>> some_func()
'from_finally'
```

### Explanation

When a `return`, `break` or `continue` statement is executed in the `try` suite of a "try…finally" statement, the `finally` clause is also executed ‘on the way out. The return value of a function is determined by the last `return` statement executed. Since the `finally` clause always executes, a `return` statement executed in the `finally` clause will always be the last one executed.

## When True is actually False

```py
True == False
if True == False:
    print("I've lost faith in truth!")
```

**Output:**
```
I've lost faith in truth!
```

### Explanation

Initially, Python used to have no `bool` type (people used 0 for false and non-zero value like 1 for true). Then they added `True`, `False`, and a `bool` type, but, for backwards compatibility, they couldn't make `True` and `False` constants- they just were built-in variables.
Python 3 was backwards-incompatible, so it was now finally possible to fix that, and so this example wont't work with Python 3.x.


## The GIL messes it up (Multithreading vs Mutliprogramming example)

## Be careful with chained comparisons

```py
>>> True is False == False
False
>>> False is False is False
True
>>> 1 > 0 < 1
True
>>> (1 > 0) < 1
False
>>> 1 > (0 < 1)
False
```

### Explanation

As per https://docs.python.org/2/reference/expressions.html#not-in

> Formally, if a, b, c, ..., y, z are expressions and op1, op2, ..., opN are comparison operators, then a op1 b op2 c ... y opN z is equivalent to a op1 b and b op2 c and ... y opN z, except that each expression is evaluated at most once.

* `False is False is False` is equivalent to `(False is False) and (False is False)`
* `True is False == False` is equivalent to `True is False and False == False` and since the first part of the statement (`True is False`) evaluates to `False`, the overall expression evaluates to `False`.
* `1 > 0 < 1` is equivalent to `1 > 0 and 0 < 1` which evaluates to `True`.
* The expression `(1 > 0) < 1` is equivalent to `True < 1` and
  ```py
  >>> int(True)
  1
  >>> True + 1 #not relevant for this example, but just for fun
  2
  ```
  So, `1 < 1` evaluates to `False`


## a += b doesn't behave the same way as a = a + b

```
>>> a=[1,2,3,4]
>>> b=a
>>> a=a+[5,6,7,8]
>>> a
[1, 2, 3, 4, 5, 6, 7, 8]
>>> b
[1, 2, 3, 4]


>>> a=[1,2,3,4]
>>> b=a
>>> a+=[5,6,7,8]
>>> a
[1, 2, 3, 4, 5, 6, 7, 8]
>>> b
[1, 2, 3, 4, 5, 6, 7, 8]
```

### Explanation

The expression a=a+[5,6,7,8] generates a new object and sets the A reference to that new object, leaving b to the old object unchanged.

The expression a+=[5,6,7,8] is actually mapped to an "extend" function that operates on the object in place such that a and b still point to the same object that has been modified in place



## That "is" on the same non-static method of the class instance returns False. It was the first and the last time I tried prettify my code with "is".

## Some title
```
x = {0: None}

for i in x:
    del x[i]
    x[i+1] = None
    print i
```

## Minor ones

- `join()` is a string operation instead of list operation. (sort of counterintuitive)
  **Explanation:**
  If `join()` is a method on a string then it can operate on any iterable (list, tuple, iterators). If it were a method on a list it'd have to be implemented separately by every type. Also, it doesn't make much sense to put a string-specific method on a generic list.

  Also, it's string specific, and it sounds wrong to put a string-specific method on a generic list.
- `[] = ()` is a semantically correct statement (unpacking an empty `tuple` into an empty `list`)
- No multicore support yet

## "Needle in a Haystack" bugs

This contains some of the potential bugs in you code that are very common but hard to detect.

### Initializing a tuple containing single element

```py
t = ('one', 'two')
for i in t:
    print(i)

t = ('one')
for i in t:
    print(i)

t = ()
print(t)
```

**Output:**
```py
one
two
o
n
e
tuple()
```

#### Explanation
* The correct statement for expected behavior is `t = ('one',)` or `t = 'one',` (missing comma) otherwise the interpreter considers `t` to be a `str` and iterates over it character by character.
* `()` is a special token and denotes empty `tuple`.


# Contributing

All patches are Welcome! Filing an issue first before submitting a patch will be appreciated :)

# Acknowledgements

The idea and design for this list is inspired from Denys Dovhan's awesome project [wtfjs](https://github.com/denysdovhan/wtfjs).

### Some nice Links!
* https://www.reddit.com/r/Python/comments/3cu6ej/what_are_some_wtf_things_about_python
* https://www.youtube.com/watch?v=sH4XF6pKKmk

# 🎓 License

[![CC 4.0][license-image]][license-url]

&copy; [Satwik Kansal](https://satwikkansal.xyz)

[license-url]: http://www.wtfpl.net
[license-image]: https://img.shields.io/badge/License-WTFPL%202.0-lightgrey.svg?style=flat-square

