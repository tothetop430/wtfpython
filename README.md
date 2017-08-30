# What the f*ck Python!

[![WTFPL 2.0][license-image]][license-url]

> A collection of tricky Python examples

Python being an awesome higher level language, provides us many functionalities for programmer's comfort. But sometimes, the outcomes may not seem obvious to a normal Python user at the first sight.

Here's an attempt to collect such classic and tricky examples of unexpected behaviors in Python and see what exactly is happening under the hood! Anyways, I find it a nice way to learn internals of a language and I think you'll like them as well!

- If you're an beginner to intermdediate level Python programmer, I'd personally recommend you to go through all of the examples below, as being aware about such pitfalls may be able to save a lot of debugging time in your future.
- If you're an experienced Python programmer, you might be familiar with most of these examples, and I might be able to revive some nice old memories of yours being bitten by these gotchas.

So, here ya go...

# Table of Contents

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

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
    - [Explanation](#explanation-5)
  - [The disappearing variable from outer scope](#the-disappearing-variable-from-outer-scope)
    - [Explanation](#explanation-6)
  - [Return in both `try` and `finally` clauses](#return-in-both-try-and-finally-clauses)
    - [Explanation](#explanation-7)
  - [When True is actually False](#when-true-is-actually-false)
    - [Explanation](#explanation-8)
  - [The GIL messes it up (Multithreading vs Mutliprogramming example)](#the-gil-messes-it-up-multithreading-vs-mutliprogramming-example)
  - [Be careful with chained comparisons](#be-careful-with-chained-comparisons)
    - [Explanation](#explanation-9)
  - [a += b doesn't behave the same way as a = a + b](#a--b-doesnt-behave-the-same-way-as-a--a--b)
    - [Explanation](#explanation-10)
  - [Backslashes at the end of string](#backslashes-at-the-end-of-string)
    - [Explaination](#explaination-1)
  - [Editing a dictionary while iterating over it](#editing-a-dictionary-while-iterating-over-it)
    - [Explaination:](#explaination)
  - [Minor ones](#minor-ones)
  - ["Needle in a Haystack" bugs](#needle-in-a-haystack-bugs)
    - [Initializing a tuple containing single element](#initializing-a-tuple-containing-single-element)
      - [Explanation](#explanation-11)
- [Contributing](#contributing)
- [Acknowledgements](#acknowledgements)
    - [Some nice Links!](#some-nice-links)
- [🎓 License](#-license)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# 👀 Examples

**Environment:** All the examples mentioned below are run on Python 3.5.2 interactive interpreter unless explicitly specified.

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
>>> another_tuple[2] += [99, 999]
TypeError: 'tuple' object does not support item assignment
>>> another_tuple
([1, 2], [3, 4], [5, 6, 1000, 99, 999])
```

### Explanation

* Quoting from https://docs.python.org/2/reference/datamodel.html

> Immutable sequences
    An object of an immutable sequence type cannot change once it is created. (If the object contains references to other objects, these other objects may be mutable and may be changed; however, the collection of objects directly referenced by an immutable object cannot change.)

* `+=` operator changes the list in-place. The item assignment doesn't work, but when the exception occurs, the item has already been changed in place.

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
* When you make an assignment to a variable in a scope, it becomes local to that scope. So `a` becomes local to the scope of `another_func` but it has not been initialized previously in the same scope which throws an error.
* Read [this](http://sebastianraschka.com/Articles/2014_python_scope_and_namespaces.html) short but awesome guide to learn more about how namespaces and scope resolution works in Python.
* To actually modify the outer scope variable `a` in `another_func`, use `global` keyword.
  ```py
  def another_func()
      global a
      a += 1
      return a
  ```
  **Output:**
  ```py
  >>> another_func()
  2
  ```


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

## Evaluation time disperancy

```py
array = [1, 8, 15]
g = (x for x in array if array.count(x) > 0)
array = [2, 8, 22]
```

**Output:**
```py
>>> print(list(g))
[8]
```

### Explainiation

- In a generator expression, the `in` clause is evaluated at declaration time, but the conditional clause is evaluated at run time.
- So before run time, `array` is re-assigned to the list `[2, 8, 22]`, and since out of `1`, `8` and `15`, only the count of `8` is greater than `0`, the generator only yields `8`.


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

1.
```py
a = [1, 2, 3, 4]
b = a
a = a + [5, 6, 7, 8]
```

**Output:**
```py
>>> a
[1, 2, 3, 4, 5, 6, 7, 8]
>>> b
[1, 2, 3, 4]
```

2.
```py
a = [1, 2, 3, 4]
b = a
a += [5, 6, 7, 8]
```

**Output:**
```py
>>> a
[1, 2, 3, 4, 5, 6, 7, 8]
>>> b
[1, 2, 3, 4, 5, 6, 7, 8]
```

### Explanation

* The expression `a = a + [5,6,7,8]` generates a new object and sets `a`'s reference to that new object, leaving `b` unchanged.

* The expression `a + =[5,6,7,8]` is actually mapped to an "extend" function that operates on the object such that `a` and `b` still point to the same object that has been modified in-place.

## Backslashes at the end of string

```
>>> print("\\ some string \\")
>>> print(r"\ some string")
>>> print(r"\ some string \")

    File "<stdin>", line 1
      print(r"\ some string \")
                             ^
SyntaxError: EOL while scanning string literal
```

### Explaination

A raw string literal, where the backslash doesn't have the special meaning, as indicated by the prefix r. What it actually does, though, is simply change the behavior of backslashes so they pass themselves and the following character through. That's why backslashes don't work at the end of a raw string.

## Editing a dictionary while iterating over it

```py
x = {0: None}

for i in x:
    del x[i]
    x[i+1] = None
    print(i)
```

**Output:**

```
0
1
2
3
4
5
6
7
```

Yes, it runs for exactly 8 times and stops.

### Explaination:

* Iteration over a dictionary that you edit at the same time is not supported.
* It runs 8 times because that's the point at which the dictionary resizes to hold more keys (we have 8 deletion entries so a resize is needed). This is actually an implementation detail.
* Refer to this StackOverflow [thread](https://stackoverflow.com/questions/44763802/bug-in-python-dict) explaining a similar example.

## `is not ...` is different from `is (not ...)`

```py
>>> 'something' is not None
True
>>> 'something' is (not None)
False
```

### Explaination

- `is not` is a single binary operator, and has behavior different than using `is` and `not` separated.
- `is not` evaluates to `False` if the variables on either side of the operator point to the same object and `True` otherwise.

## Time for some hash brownies!

```py
some_dict = {}
some_dict[5.5] = "Ruby"
some_dict[5.0] = "JavaScript"
some_dict[5] = "Python"
```

**Output:**
```py
>>> some_dict[5.5]
"Ruby"
>>> some_dict[5.0]
"Python"
>>> some_dict[5]
"Python"
```

### Explaination

* `5` (an `int` type) is implicitly converted to `5.0` (a `float` type) before calculating the hash in Python.
  ```py
  >>> hash(5) == hash(5.0)
  True
  ```
* This StackOverflow [answer](https://stackoverflow.com/a/32211042/4354153) explains beautifully the rationale behind it.

## Identical looking names

```py
>>> value = 11
>>> valuе = 32
>>> value
11
```

Wut?

### Explaination

Some Unicode characters look identical to ASCII ones, but are considered distinct by the interpreter.

```py
>>> value = 42 #ascii e
>>> valuе = 23 #cyrillic e, Python 2.x interpreter would raise a `SyntaxError` here
>>> print(value)
```

## Name resolution ignoring class scope

1.
```py
x = 5
class SomeClass:
    x = 17
    y = (x for i in range(10))
```

**Output:**
```py
>>> list(SomeClass.y)[0]
5
```

2.
```py
x = 5
class SomeClass:
    x = 17
    y = [x for i in range(10)]
```

**Output (Python 2.x):**
```py
>>> SomeClass.y[0]
17
```

**Output (Python 3.x):**
```py
>>> SomeClass.y[0]
5
```

### Explaination
- Scopes nested inside class definition ignore names bound at the class level.
- A generator expression has its own scope.
- Starting in 3.X, list comprehensions also have their own scope.

## In-place update functions of mutable object types

```py
some_list = [1, 2, 3]
some_dict = {
  "key_1": 1,
  "key_2": 2,
  "key_3": 3
}

some_list = some_list.append(4)
some_dict = some_dict.update({"key_4": 4})
```

**Output:**
```py
>>> print(some_list)
None
>>> print(some_dict)
None
```

### Explaination

Most methods that modiy the items of sequence/mapping objects like `list.append`, `dict.update`, `list.sort`, etc modify the objects in-place and return `None`. The rationale behind this is to improve performance by avoiding making a copy of the object if the operation can be done in-place (Referred from [here](http://docs.python.org/2/faq/design.html#why-doesn-t-list-sort-return-the-sorted-list))

## Deleting a list item while iterating over it

```py
list_1 = [1, 2, 3, 4]
list_2 = [1, 2, 3, 4]
list_3 = [1, 2, 3, 4]
list_4 = [1, 2, 3, 4]

for idx, item in enumerate(list_1):
    del item

for idx, item in enumerate(list_2):
    list_2.remove(item)

for idx, item in enumerate(list_3[:]):
    list_3.remove(item)

for idx, item in enumerate(list_4):
    list_4.pop(idx)
```

**Output:**
```py
>>> list_1
[1, 2, 3, 4]
>>> list_2
[2, 4]
>>> list_3
[]
>>> list_4
[2, 4]
```

### Explanation

* It's never a good idea to change the object you're iterating over. The correct way to do so is to iterate over a copy of the object instead, and `list_3[:]` does just that.

 ```py
 >>> some_list = [1, 2, 3, 4]
 >>> id(some_list)
 139798789457608
 >>> id(some_list[:]) # Notice that python creates new object for sliced list.
 139798779601192
 ```


**Difference between `del`, `remove`, and `pop`:**
* `remove` removes the first matching value, not a specific index, raises `ValueError` if value is not found.
* `del` removes a specific index (That's why first `list_1` was unaffected), raises `IndexError` if invalid index is specified.
* `pop` removes element at specific index and returns it, raises `IndexError` if invalid index is specified.

* **Why the output is `[2, 4]`?** The list iteration is done index by index, and when we remove `1` from `list_2` or `list_4`, the contents of the lists are now `[2, 3, 4]`. The remaining elements are shifted down i.e. `2` is at index 0 and `3` is at index 1. Since the next iteration is going to look at index 1 (which is the `3`), the `2` gets skipped entirely. Similar thing will happen with every alternate element in the list sequence.

* See this StackOverflow [thread](https://stackoverflow.com/questions/45877614/how-to-change-all-the-dictionary-keys-in-a-for-loop-with-d-items) for a similar example related to dictionaries in Python.

## Explicit typecast of strings

```py
a = float('inf')
b = float('nan')
c = float('-iNf')  #These strings are case-insensitive
d = float('nan')
```

**Output:**
```py
>>> a
inf
>>> b
nan
>>> c
-inf
>>> float('some_other_string')
ValueError: could not convert string to float: some_other_string
>>> a == -c #inf==inf
True
>>> b == d #but nan!=nan
False
>>> 50/a
0
>>> a/a
nan
>>> 23 + b
nan
```

### Explanation

`'inf'` and `'nan'` are special strings (case-insensitieve), which when explicitly type casted to `float` type, are used to represent mathematical "infinity" and "not a number" respectively.


## Well, something is fishy...

```py
def square(x):
    sum_so_far = 0
    for counter in range(x):
        sum_so_far = sum_so_far + x
  return sum_so_far

print(square(10))
```

**Output (Python 2.x):**

(After pasting the above snippet in the interactive Python interpreter)
```py
10
```

**Output (Python 3.x):**
```py
TabError: inconsistent use of tabs and spaces in indentation
```

**Note:** If you're not able to reproduce this, try running the file [mixed_tabs_and_spaces.py](/mixed_tabs_and_spaces.py) via the shell.

### Explaination

* **Don't mix tabs and spaces!** The character just preceding return is a "tab" and the code is indented by multiple of "4 spaces" elsewhere in the example.
* This is how Python handles tabs:
  > First, tabs are replaced (from left to right) by one to eight spaces such that the total number of characters up to and including the replacement is a multiple of eight <...>
* So the "tab" at the last line of `square` function is replace with 8 spaces and it gets into the loop.

## Catching the Exceptions!

```py
some_list = [1, 2, 3]
try:
    # This should raise an ``IndexError``
    print(some_list[4])
except IndexError, ValueError:
    print("Caught!")

try:
    # This should raise a ``ValueError``
    some_list.remove(4)
except IndexError, ValueError:
    print("Caught again!")
```

**Output (Python 2.x):**
```py
Caught!

ValueError: list.remove(x): x not in list
```

**Output (Python 3.x):**
```py
  File "<input>", line 3
    except IndexError, ValueError:
                     ^
SyntaxError: invalid syntax
```

### Explaination

* To add multiple Exceptions to the except clause, you need to pass them as parenthesized tuple as the first argument. The second argument is an optional name, which when supplied will bind the Exception instance that has been raised. Example,
  ```py
  some_list = [1, 2, 3]
  try:
     # This should raise a ``ValueError``
     some_list.remove(4)
  except (IndexError, ValueError), e:
     print("Caught again!")
     print(e)
  ```
  **Output (Python 2.x):**
  ```
  Caught again!
  list.remove(x): x not in list
  ```
  **Output (Python 3.x):**
  ```py
    File "<input>", line 4
      except (IndexError, ValueError), e:
                                       ^
  IndentationError: unindent does not match any outer indentation level
  ```

* Separating the exception from the variable with a comma is deprecated and does not work in Python 3; the correct way is to use `as`. Example,
  ```py
  some_list = [1, 2, 3]
  try:
      some_list.remove(4)

  except (IndexError, ValueError) as e:
      print("Caught again!")
      print(e)
  ```
  **Output:**
  ```
  Caught again!
  list.remove(x): x not in list
  ```


## Minor Ones

- `join()` is a string operation instead of list operation. (sort of counter-intuitive at first usage)
  **Explanation:**
  If `join()` is a method on a string then it can operate on any iterable (list, tuple, iterators). If it were a method on a list it'd have to be implemented separately by every type. Also, it doesn't make much sense to put a string-specific method on a generic list.

  Also, it's string specific, and it sounds wrong to put a string-specific method on a generic list.
- `[] = ()` is a semantically correct statement (unpacking an empty `tuple` into an empty `list`)
- Python uses 2 bytes for local variable storage in functions. In theory this means that only 65536 variables can be defined in a function. However, python has a handy solution built in that can be used to store more than 2^16 variable names. The following code demonstrates what happens in the stack when more than 65536 local variables are defined (Warning: This code prints around 2^18 lines of text, so be prepared!):
 ```py
 import dis
 exec("""
 def f():
     """ + """
     """.join(["X"+str(x)+"=" + str(x) for x in range(65539)]))

 f()

 print(dis.dis(f))
 ```
- **Explicit type cast of string**
  ```py
  >>> float('inf')
  inf
  >>> float('nan') #case-insensitive
  nan
  >>> float('some_other_string')
  ValueError: could not convert string to float: some_other_string
  ```

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
* https://www.youtube.com/watch?v=sH4XF6pKKmk
* https://www.reddit.com/r/Python/comments/3cu6ej/what_are_some_wtf_things_about_python
* https://sopython.com/wiki/Common_Gotchas_In_Python
* https://stackoverflow.com/questions/530530/python-2-x-gotchas-and-landmines

# 🎓 License

[![CC 4.0][license-image]][license-url]

&copy; [Satwik Kansal](https://satwikkansal.xyz)

[license-url]: http://www.wtfpl.net
[license-image]: https://img.shields.io/badge/License-WTFPL%202.0-lightgrey.svg?style=flat-square

