# Effective Python

## Table of contents:

1. [Chapter 1: Pythonic Thinking](#Chapter1)
2. [Chapter 2: Lists and Dictionaries](#Chapter2)
3. [Chapter 3: Functions](#Chapter3)
4. [Chapter 4: Comprehensions and Generators](#Chapter4)
5. [Chapter 5: Classes and Interfaces](#Chapter5)
6. [Chapter 6: Metaclasses and Attributes](#Chapter6)
7. [Chapter 7: Concurrency and Parallelism](#Chapter7)
8. [Chapter 8: Robustness and Performance](#Chapter8)
9. [Chapter 9: Testing and Debugging](#Chapter9)
10. [Chapter 10: Collaboration](#Chapter10)

## Chapter 1: Pythonic Thinking<a name="Chapter1"></a>

### Know Which Version of Python You're Using

You can check the version of python you are using by executing `python --version` or `python3 --version` in case of
python 3.

### Follow the PEP 8 Style Guide

It's worth reading the whole Python Enhancement Proposal #8 (PEP 8)
[guide online](https://www.python.org/dev/peps/pep-0008/). Some of the rules to follow:

Whitespaces:

    * Use spaces instead of tabs for indentation
    * Use four spaces for each level of syntactically significant indenting
    * Lines should be 79 characters in length or less
    * Continuations of long expressions onto additional lines should be indented by four extra spaces from their normal 
      indentation level
    * In a file, functions and classes should be separated by two blank lines
    * In a class, methods should be separated by one blank line
    * In a dictionary, put no whitespace between each key and colon, and put a single space before the corresponding 
      value if it fits on the same line
    * Put only one space before and after the = operator in a variable assignment
    * For type annotations, ensure that there is no separation between the variable name and the colon, and use a space 
      before the type information

Naming:

    * Functions, variables, and attributes should be in lowercase_ underscore format
    * Protected instance attributes should be in _leading_underscore format
    * Private instance attributes should be in __double_leading_ underscore format
    * Classes (including exceptions) should be in CapitalizedWord format
    * Module-level constants should be in ALL_CAPS format
    * Instance methods in classes should use self, which refers to the object, as the name of the first parameter
    * Class methods should use cls, which refers to the class, as the name of the first parameter

Expressions and Statements:

    * Use inline negation (if a is not b) instead of negation of positive expressions (if not a is b)
    * Don't check for empty containers or sequences (like [] or '') by comparing the length to zero 
      (if len(somelist) == 0). Use if not somelist and assume that empty values will implicitly evaluate to False
    * The same thing goes for non-empty containers or sequences (like [1] or 'hi'). The statement if somelist is 
      implicitly True for non-empty values
    * Avoid single-line if statements, for and while loops, and except compound statements. Spread these over multiple 
      lines for clarity
    * If you can't fit an expression on one line, surround it with parentheses and add line breaks and indentation to 
      make it easier to read
    * Prefer surrounding multiline expressions with parentheses over using the \ line continuation character

Imports

    * Always put import statements (including from x import y) at the top of a file.
    * Always use absolute names for modules when importing them, not names relative to the current module's own path
    * If you must do relative imports, use the explicit syntax `from . import foo`
    * Imports should be in sections in the following order: standard library modules, third-party modules, your own 
      modules. Each subsection should  have imports in alphabetical order.

The [Pylint tool](https://www.pylint.org) is a popular static analyzer for Python source code. Pylint provides
automated enforcement of the PEP 8 style guide.

### Know the Differences Between bytes and str

In Python, there are two types that represent sequences of character data: _bytes_ and _str_. Instances of bytes contain
raw, unsigned 8-bit values.

```python
# displayed in the ASCII encoding
a = b'h\x65llo'

# Unicode code points that represent textual characters
b = 'a\u0300 propos'
```

_str_ instances do not have an associated binary encoding, and bytes instances do not have an associated text
encoding. To convert Unicode data to binary data, you must call the `encode` method of _str_. To convert binary data
to Unicode data, you must call the `decode` method of _bytes_.

It's important to do encoding/decoding of Unicode data at the furthest boundary of your interfaces (Unicode sandwich).
The core of your program should use the _str_ type containing Unicode data and should not assume anything about
character encodings.

_bytes_ and _str_ instances are not compatible with each other, so you must be deliberate about the types of character
sequences that you're passing around. By using the + operator, you can only add _bytes_ to _bytes_ and _str_ to _str_.
Comparing _bytes_ and _str_ instances for equality will always evaluate to `False`.

The `%` operator works with format strings for each type, respectively. But you can't pass a _str_ instance to a _bytes_
format string because Python doesn't know what binary text encoding to use.

Operations involving file handles (returned by the open built-in function) default to requiring Unicode strings
instead of raw bytes.

```python
# This fails, use 'wb' instead of 'w' (same with read operations)
with open('data.bin', 'w') as f:
    f.write(b'\xf1\xf2\xf3\xf4\xf5')
```

When a handle is in text mode, it uses the system's default text encoding to interpret binary data using the
`bytes.encode` (for writing) and `str.decode` (for reading) methods. Alternatively, you can pass the expected
encoding to the open function.

### Prefer Interpolated F-Strings Over C-style Format Strings and str.format

Python has four different ways of formatting strings that are built into the language and standard library. The most
common way to format a string in Python is by using the `%` formatting operator:

```python
# Output is 'Binary is 187, hex is 3167'
print('Binary is %d, hex is %d' % ('0b10111011', '0xc5f'))
```

The problems with the above is that if you change the type or order of data values in the tuple on the right side of a
formatting expression, you can get errors due to type conversion incompatibility, it is also difficult to read when
you need to make small modifications to values before formatting them into a string. In addition to this, if you
want to use the same value in a format string multiple times, you have to repeat it in the right side tuple (although
you can use a dictionary and not a tuple like in `'%(key)-10s = %(value).2f' % {'value':value, 'key':key}`), but using
dictionaries in formatting expressions also increases verbosity.

Another way to format strings is to use the `format` Built-in and `str.format` functions:

```python
print(format(1234.5678, ',.2f'))  # prints '1,234.57'
print('*', format('my string', '^20s'), '*')  # prints '*      my string       *'
```

You can use this functionality to format multiple values together by calling the new format method of the _str_ type.
Instead of using C-style format specifiers like `%d`, you can use placeholders with `{}`: `'{} = {}'.format('k','v')`.
Within each placeholder you can optionally provide a colon character followed by format specifiers to customize how
values will be converted into strings: `'{:<10} = {:.2f}'.format('key', 'value')`. Keep in mind that the '{}'
characters needs to be escaped if you want to use them: `'{} replaces {{}}'.format(1.23)`. Within the braces you may
also specify the positional index of an argument passed to the format method to use for replacing the placeholder:
`'{1} = {0}'.format(key, value)`, the same positional index may also be referenced multiple times in the format
string without the need to pass the value to the format method more than once.

Python 3.6 added interpolated format strings (f-strings) to solve these issues. This new language syntax requires
you to prefix format strings with an 'f' character, which is similar to how byte strings are prefixed with a 'b'
character and raw (unescaped) strings are prefixed with an 'r' character. f-strings allows you to reference all names
in the current Python scope as part of a formatting expression:

```python
key = 'my_var'
value = 1.234
formatted = f'{key!r:<10} = {value:.2f}'
print(formatted)
# produces: 
# 'my_var'   = 1.23
```

F-strings also enable you to put a full Python expression within the placeholder braces, or if it's clearer, you
can split an f-string over multiple lines by relying on adjacent-string concatenation:

```python
for i, (item, count) in enumerate(pantry):
    print(f'#{i + 1}: '
          f'{item.title():<10s} = '
          f'{round(count)}')
```

Python expressions may also appear within the format specifier options:

```python
places = 3
number = 1.23456
s  # Prints 'My number is 1.235'
print(f'My number is {number:.{places}f}')
```

### Write Helper Functions Instead of Complex Expressions

Python makes it easy to write single-line expressions that implement a lot of logic. This is could become extremely
hard to read. A new reader of the code would have to spend too much time picking apart the expression to figure out what
it actually does. Even though it's nice to keep things short, it's not worth trying to fit this all on one line. If you
need to reuse some logic repeatedly (even just two or three times) write a helper function. As soon as expressions get
complicated, it's time to consider splitting them into smaller pieces and moving logic into helper functions. What you
gain in readability always outweighs what brevity may have afforded you.

### Prefer Multiple Assignment Unpacking Over Indexing

Python has a built-in tuple type that can be used to create immutable, ordered sequences of values. The values in tuples
can be accessed through numerical indexes `('red','blue')[0] # returns 'red'`. Python also has syntax for unpacking,
which allows for assigning multiple values in a single statement:

```python
colors = ('red', 'blue', 'green')
first_color, second_color, third_color = colors
```

The same pattern matching syntax of unpacking works when assigning to lists, sequences, and multiple levels of arbitrary
iterables within iterables. Unpacking can even be used to swap values in place without the need to create temporary
variables:

```python
def bubble_sort(a):
    for _ in range(len(a)):
        for i in range(1, len(a)):
            if a[i] < a[i - 1]:
                a[i - 1], a[i] = a[i], a[i - 1]  # Swap with no temporary assignment
```

The way the above works is that the right side of the assignment (a[i], a[i-1]) is evaluated first, and its values
are put into a new temporary, unnamed tuple on the first iteration of the loops. Then, the unpacking pattern from
the left side of the assignment `(a[i-1], a[i])` is used to receive that tuple value and assign it to the variable
names contained by `a[i-1]` and `a[i]`, respectively. Finally, the temporary unnamed tuple silently goes away.
Unpacking can be used in for comprehensions and generator expressions:

```python
snacks = [('bacon', 350), ('donut', 240), ('muffin', 190)]
for rank, (name, calories) in enumerate(snacks, 1):  # The second argument is the starting number for the enumerate 
    print(f'#{rank}: {name} has {calories} calories')  # prints #1: bacon has 350 calories...
```

### Prefer enumerate Over range

The range built-in function is useful for loops that iterate over a set of integers, but when you have a data
structure to iterate over, like a list of strings, you can loop directly over the sequence. Often, you'll want to
iterate over a list and also know the index of the current item in the list. Python provides the enumerate built-in
function to address this situation. enumerate wraps any iterator with a lazy generator:

```python
flavor_list = ['vanilla', 'chocolate', 'pecan', 'strawberry']
it = enumerate(flavor_list)
print(next(it))  # prints (0, 'vanilla')
print(next(it))  # prints (1, 'chocolate')
```

### Use zip to Process Iterators in Parallel

Often in Python you find yourself with many lists of related objects, with the items in the derived list are related
to the items in the source list by their indexes. Python provides the zip built-in function. zip wraps two or more
iterators with a lazy generator. The zip generator yields tuples containing the next value from each iterator.

```python
names = ['Cecilia', 'Lise', 'Marie']
counts = [len(n) for n in names]
longest_name = None
max_count = 0
for name, count in zip(names, counts):
    if count > max_count:
        longest_name = name
        max_count = count
```

zip consumes the iterators it wraps one item at a time, which means it can be used with infinitely long inputs without
risk of a program using too much memory and crashing. However, beware of zip's behavior when the input iterators are
of different lengths. _zip_ output is as long as its shortest input. If you don't expect the lengths of the lists passed
to zip to be equal, consider using the _zip\_longest_ function from the itertools built-in module instead:

```python
import itertools

for name, count in itertools.zip_longest(names, counts):
    print(f'{name}: {count}')  # On absent value, this will yield None to match both lists length
```

### Avoid else Blocks After for and while Loops

In Python loops you can put an else block immediately after a loop's repeated interior block:

```python
for i in range(3):
    print('Loop', i)
else:
    print('Else block!')  # runs immediately after the loop finishes.
```

Similarly, else from _try/except/else_ follows this pattern because it means 'Do this if there was no exception to
handle.' One might assume that the _else_ part of _for/else_ means 'Do this if the loop wasn't completed.' But using a
break statement in a loop actually skips the else block. Another surprise is that the _else_ block runs immediately
if you loop over an empty sequence and the _else_ block also runs when while loops are initially _False_.
Given this, the expressivity you gain from the else block doesn't outweigh the burden you put on people who want to
understand your code in the future. Therefore avoid using _else_ blocks after loops entirely.

### Prevent Repetition with Assignment Expressions

An assignment expression (a.k.a. walrus operator) is a new syntax introduced in Python 3.8. Whereas normal assignment
statements are written `a = b` and pronounced "a equals b", these assignments are written `a := b` and pronounced
"a walrus b". Assignment expressions are useful because they enable you to assign variables in places where assignment
statements are disallowed, such as in the conditional expression of an if statement. An assignment expression's value
evaluates to whatever was assigned to the identifier on the left side of the walrus operator. You can replace this:

```python
count = some_list.get('key', 0)
if count:
    some_func(count)
else:
    other_func()
```

with this:

```python
# Assings to count and then evaluates, you can remove the (..)>2 to evaluate on count directly `if count := get(0):`
if (count := some_list.get('key', 0)) > 2:
    some_func(count)
else:
    other_func()
```

This is a lot more readable because it's now clear that count is only relevant to the first block of the if statement.
Another common variation of this repetitive pattern occurs when I need to assign a variable in the enclosing scope
depending on some condition, and then reference that variable shortly afterward in a function call:

```python
if (count := some_list.get('key', 0)) >= 2:
    aux_var = some_logic(count)
try:
    result = compute_sth(aux_var)
except SomeException:
    other_func()
```

Python lacks of a flexible _switch/case_ statement, but the walrus operator provides an elegant solution that can feel
nearly as versatile as dedicated syntax for this:

```python
if (count := some_list.get('key1', 0)) >= 2:
    func1(count)
elif (count := some_list.get('key2', 0)) >= 2:
    func2(count)
elif count := some_list.get('key3', 0):
    func3(count)
else:
    other_func()
```

Python also lacks a _do/while_ loop construct. We can solve this like the code below:

```python
while count := do_count():
    pass  # Do some logic here
```

In general, when you find yourself repeating the same expression or assignment multiple times within a grouping of
lines, consider using assignment expressions in order to improve readability.

## Chapter 2: Lists and Dictionaries<a name="Chapter2"></a>

### Know How to Slice Sequences

Python includes syntax for slicing sequences into pieces.The simplest uses for slicing are the built-in types _list_,
_str_, and _bytes_. Slicing can be extended to any Python class that implements the _\__getitem__ and _\__setitem__
special methods. The basic form of the slicing syntax is `somelist[start:end]`. When slicing from the start of a
list, you should leave out the zero index to reduce visual noise. When slicing to the end of a list, you should leave
out the final index because it's redundant. Using negative numbers for slicing is helpful for doing offsets relative to
the end of a list. Slicing deals properly with start and end indexes that are beyond the boundaries of a list by
silently omitting missing items, but access a missing index causes an Exception to be thrown.

The result of slicing a list is a whole new list. References to the objects from the original list are maintained. When
used in assignments, slices replace the specified range in the original list. Unlike unpacking assignments, the lengths
of slice assignments don't need to be the same. The values before and after the assigned slice will be preserved. Here,
the list shrinks because the replacement list is shorter than the specified slice:

```python
a = ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h']
a[2:7] = [99, 22, 14]  # this is  ['a', 'b', 99, 22, 14, 'h']
```

If you leave out the start and the end indexes when slicing, you end up with a copy of the original list:`b = a[:]`. If
you assign to a slice with no start or end indexes, you replace the entire contents of the list with a copy of what's
referenced instead of allocating a new list:

```python
b = a
a[:] = [101, 102, 103]  # After this, a and b = [101, 102, 103]
assert a is b  # Returns True
```

### Avoid Striding and Slicing in a Single Expression

Python has special syntax for the stride of a slice in the form `somelist[start:end:stride]`. The problem is that the
stride syntax often causes unexpected behavior that can introduce bugs. For example, you can reverse a string with
`thestring[::-1]`, but this does not work when Unicode data is encoded as a UTF-8 byte string.

The stride part of the slicing syntax can be extremely confusing, avoid using a stride along with start and end indexes.
If you must use a stride with start or end indexes, consider using one assignment for striding and another for slicing:

```python
y = x[::2]  # ['a', 'c', 'e', 'g']
z = y[1:-1]  # ['c', 'e']
```

Striding and then slicing creates an extra shallow copy of the data. The first operation should try to reduce the
size of the resulting slice by as much as possible. If your program can't afford the time or memory required for two
steps, consider using the itertools built-in module's `islice` method.

### Prefer Catch-All Unpacking Over Slicing

One limitation of basic unpacking is that you must know the length of the sequences you're unpacking in advance. Python
also supports catch-all unpacking through a starred expression: `first_item, second_item, *others = some_list`.
A starred expression may appear in any position, so you can get the benefits of catch-all unpacking anytime you need
to extract one slice: `first_item, *others, last_item = some_list`. To unpack assignments that contain a starred
expression, you must have at least one required part, or else you'll get a SyntaxError. You can't use a catch-all
expression on its own: `*others = some_list`. You also can't use multiple catch-all expressions in a single-level
unpacking pattern: `first, *middle, *second_middle, last = some_list`. But it is possible to use multiple starred
expressions in an unpacking assignment statement, as long as they're catch-alls for different parts of the multilevel
structure being unpacked:

```python
car_inventory = {'Downtown': ('Silver Shadow', 'Pinto', 'DMC'), 'Airport': ('Skyline', 'Viper', 'Gremlin', 'Nova')}
((loc1, (best1, *rest1)), (loc2, (best2, *rest2))) = car_inventory.items()  # Although not recommended
```

Starred expressions become list instances in all cases. If there are no leftover items from the sequence being unpacked,
the catch-all part will be an empty list. You can also unpack arbitrary iterators with the unpacking syntax, but
keep in mind, however, that because a starred expression is always turned into a list, unpacking an iterator also
risks the potential of a memory error.

### Sort by Complex Criteria Using the key Parameter

The list built-in type provides a sort method for ordering the items in a list instance based on a variety of criteria
(defaults to natural ascending order of items). This natural order is defined in the class special methods (_\__lt___).
The sort method accepts a key parameter that's expected to be a function. The key function is passed a single argument,
which is an item from the list that is being sorted. The return value of the key function should be a comparable
value to use in place of an item for sorting purposes: `my_list.sort(key=lambda x: x.attribute_to_sort)`. If you need to
sort by various criteria (name and age for example), the simplest solution is to use a tuple. Tuples are comparable by
default and have a natural ordering, meaning that they implement all of the special methods. Tuples implement these
special method comparators by iterating over each position in the tuple and comparing the corresponding values one index
at a time. But one limitation of having the key function return a tuple is that the direction of sorting for all
criteria must be the same: either all in ascending order, or all in descending order. For numerical values it's possible
to mix sorting directions by using the unary minus operator in the key function. But if the attribute is not numeric,
you can use the Python's _stable_ sorting algorithm. The sort method of the list type will preserve the order of the
input list when the key function returns values that are equal to each other. This means that I can call sort multiple
times on the same list to combine different criteria together:

```python
my_list.sort(key=lambda x: x.attribute_to_sort2)  # attribute_to_sort ascending
my_list.sort(key=lambda x: x.attribute_to_sort1, reverse=True)  # attribute_to_sort descending
```

Make sure that you execute the sorts in the opposite sequence of what you want the final list to contain, only using
multiple calls to sort if it's absolutely necessary.

### Be Cautious When Relying on dict Insertion Ordering

In Python 3.5 and before, iterating over a dict would return keys in arbitrary order (this applies to all methods
provided by dict that relied on iteration order, including `keys`, `values`, `items`, `popitem`, `**kwargs` passed
to functions and the dict type for class instance dictionaries returned by _\__dict___). This happened because the
dictionary type previously implemented its hash table algorithm with a combination of the hash built-in function and a
random seed that was assigned when the Python interpreter started. Starting with Python 3.6, and officially part of the
Python specification in version 3.7, dictionaries will preserve insertion order.

However, you shouldn't always assume that insertion ordering behavior will be present when you're handling dictionaries
because Python makes it easy for programmers to define their own custom container types that emulate the standard
protocols matching list, dict, and other types. To mitigate this problem you can write code that doesn't rely on
insertion ordering, explicitly check for the dict type at runtime, or require dict values using type annotations and
static analysis.

### Prefer get Over in and KeyError to Handle Missing Dictionary Keys

The three fundamental operations for interacting with dictionaries are accessing, assigning, and deleting keys and
their associated values. Imagine we want to implement the upsert operation for a counter in a dictionary,we have to
check for the key existence, insert it with a default value if it doesn't, and increment the value associated with
the key:

```python
key = 'key1'
if key in counters:
    count = counters[key]
else:
    count = 0
counters[key] = count + 1
```

Another option is to rely on dictionaries raising a _KeyError_ exception when you try to access an absent key:

```python
try:
    count = counters[key]
except KeyError:
    count = 0
counters[key] = count + 1
```

This upsert flow is so common that python provides a built-in function for this: `count = counters.get(key, 0)`,
where the second value is the default value to return if the key doesn't exist. A similar case exists if the
returned value is a collection, for example:

```python
if (names := my_list.get(key)) is None:  # Assigns to names = my_list.get('key') and returns it to compare to None
    my_list[key] = names = []  # Assigns my_list[key] and names the value [], modifying names modifys votes[key] too
names.append('value1')
```

The dict type also provides the _setdefault_ for improved brevity. _setdefault_ tries to fetch the value of a key in the
dictionary and if not present, the method assigns that key to the default value provided. And then the method returns
the value for that key: either the originally present value or the newly inserted default value.

```python
names = my_list.setdefault(key, [])
names.append(who)
```

There's also a big gotcha: The default value passed to _setdefault_ is assigned directly into the dictionary when
the key is missing instead of being copied, so modifying the value also modifies the dict. Make sure that you always
construct a new default value for each key I access with _setdefault_.

### Prefer defaultdict Over setdefault to Handle Missing Items in Internal State

When working with a dictionary that you didn't create, there are a variety of ways to handle missing keys, for some
use cases _setdefault_ appears to be the shortest option:

```python
countries.setdefault('France', set()).add('Paris')  # Short

if (japan := countries.get('Japan')) is None:  # Long
    visits['Japan'] = japan = set()
japan.add('Tokyo')
```

When you control the creation of the dictionary being accessed(like when you're using a dictionary instance to keep
track of the internal state of a class), you might choose to hide the complexity of creating and adding elements to
this state by providing a helper method like in:

```python
class Visits:
    def __init__(self):
        self.data = {}

    def add(self, country, city):
        city_set = self.data.setdefault(country, set())  # A new set is  instantiated in every call to add
        city_set.add(city)
```

the _defaultdict_ class from the collections built-in module simplifies this common use case by automatically storing a
default value when a key doesn't exist. All you have to do is provide a function that will return the default value to
use each time a key is missing:

```python
from collections import defaultdict


class Visits:
    def __init__(self):
        self.data = defaultdict(
            set)  # set here is the function to use to generate the default value, passed as argument

    def add(self, country, city):
        self.data[country].add(city)
```

Using _defaultdict_ is much better than using setdefault for this type of situation, but _get_ is better if you
don't control the dictionary creation.

### Know How to Construct Key-Dependent Default Values with __missing__

There are cases where _defaultdict_ or _setdefault_ is not the best fit for creating default values. For example in
situations where the call to the function to create default values has some effects (like opening a file handle),
given that the function is called even if the key exist in the dictionary. _defaultdict_ might be a choice but the
function passed to this needs to be parameterless (in case of filehandles you need the path to it). Luckily, there
is a solution to this, which is to subclass the dict type and implement the _\__missing___ special method to add custom
logic for handling missing keys:

```python
class Pictures(dict):  # Subclass dict
    def __missing__(self, path):
        value = open_picture(path)
        self[path] = value
        return value

    def open_picture(path):
        try:
            return open(path, 'a+b')
        except OSError:
            print(f'Failed to open path {path}')
            raise


pictures = Pictures()
handle = pictures[path]
handle.seek(0)
image_data = handle.read()
```

When the `pictures[path]` dictionary access finds that the path key isn't present in the dictionary, the _\__missing___
method is called. This method must create the new default value for the key, insert it into the dictionary, and return
it to the caller. Subsequent accesses of the same path will not call _\__missing___ since the corresponding item is
already present.

## Chapter 3: Functions<a name="Chapter3"></a>

### Never Unpack More Than Three Variables When Functions Return Multiple Values

The unpacking syntax allows Python functions to seemingly return more than one value, you should never use more than
three variables when unpacking the multiple return values from a function. These could be individual values from a
three-tuple, two variables and one catch-all starred expression, or anything shorter. If you need to unpack more return
values than that, you're better off defining a lightweight class or namedtuple.

### Prefer Raising Exceptions to Returning None

When writing utility functions, there's a draw for Python programmers to give special meaning to the return value of
_None_. The problem is that in a condition like an _if_ statement, you might accidentally look for any False-equivalent
value to indicate errors instead of only looking for _None_. For this case, you might want to split the return value
into a two-tuple with the first value indicating the success or not of the operation and the second the actual value.
The second approach is to never return None for special cases, and instead, raise an _Exception_ up to the caller and
have the caller deal with it. The caller no longer requires a condition on the return value of the function. Instead, it
can assume that the return value is always valid and use the results immediately in the _else_ block after _try_, but
you have to document the exception-raising behavior and expect callers to rely on that in order to know which Exceptions
they should plan to catch.

### Know How Closures Interact with Variable Scope

As seen before, a common way to sort a list is to pass a helper function as the key argument to a list's sort method.
The helper's return value will be used as the value for sorting each item in the list. This works because:

    * Python supports closures: functions that refer to variables from the scope in which they were defined
    * Functions are first-class objects in Python: You can refer to them directly, assign them to variables, pass 
      them as arguments to other functions, compare them in expressions and if statements, and so on
    * Python has specific rules for comparing sequences (including tuples). It first compares items at index zero, then
      if those are equal, it compares items at index one, and so on

When you reference a variable in an expression, the Python interpreter traverses the scope to resolve the reference in
this order:

    1. The current function's scope
    2. Any enclosing scopes (such as other containing functions)
    3. The scope of the module that contains the code (also called the global scope)
    4. The built-in scope (that contains functions like len and str)

Assigning a value to a variable works differently. If the variable is already defined in the current scope, it will just
take on the new value. If the variable doesn't exist in the current scope, Python treats the assignment as a variable
definition. Critically, the scope of the newly defined variable is the function that contains the assignment.
Python has a special syntax to get data out of a closure: The _nonlocal_ statement is used to indicate that scope
traversal should happen upon assignment for a specific variable name. The only limit is that nonlocal won't traverse up
to the module-level scope (to avoid polluting globals):

```python
def sort_priority(numbers, group):
    found = False

    def helper(x):
        nonlocal found
        if x in group:
            found = True
            return (0, x)
        return (1, x)

    numbers.sort(key=helper)
    return found
```

The nonlocal statement makes it clear when data is being assigned out of a closure and into another scope. It's
complementary to the global statement, which indicates that a variable's assignment should go directly into the module
scope. When your usage of nonlocal starts getting complicated, it's better to wrap your state in a helper class.

### Reduce Visual Noise with Variable Positional Arguments

Accepting a variable number of positional arguments can make a function call clearer and reduce visual noise (varargs).
If I already have a sequence (like a list) and want to call a variadic function like log, I can do this by using the
_*_ operator:

```python
def log(message, *values):
    if not values:
        print(message)
    else:
        print(f'{message}: {", ".join(str(x) for x in values)}')


favorites = ['red', 'blue', 'green']
log('Favorite colors', *favorites)
```

There are two problems with accepting a variable number of positional arguments, one is that these optional positional
arguments are always turned into a tuple before they are passed to a function. This means that if the caller of a
function uses the * operator on a generator, it will be iterated until it's exhausted. The resulting tuple includes
every value from the generator, which could consume a lot of memory and cause the program to crash.

```python
def my_generator():
    for i in range(10):
        yield i


def my_func(*args):
    print(args)


it = my_generator()  # Potential memory issue here
my_func(*it)
```

The second issue with _*args_ is that you can't add new positional arguments to a function in the future without
migrating each caller: if you add a positional argument in front of the argument list, existing callers will break if
not updated.

Functions that accept _*args_ are best for situations where you know the number of inputs in the argument list will be
reasonably small. _*args_ is ideal for function calls that pass many literals or variable names together. It's primarily
for the convenience of the programmer and the readability of the code.

### Provide Optional Behavior with Keyword Arguments

All normal arguments to Python functions can also be passed by keyword, where the name of the argument is used in an
assignment within the parentheses of a function call. The keyword arguments can be passed in any order as long as all of
the required positional arguments are specified. If you already have a dictionary, you can use its contents to
call a function by using the _**_ operator: `my_function(**my_dict)`. You can mix the _**_ operator with positional
arguments or keyword arguments in the function call, as long as no argument is repeated. You can also use the _**_
operator multiple times if you know that the dictionaries don't contain overlapping keys:

```python
my_dict1 = {'some_arg1': 1}
my_dict2 = {'some_arg2': 2}
my_function(**my_dict1, **my_dict2)
```

And if you'd like for a function to receive any named keyword argument, you can use the **kwargs catch-all parameter to
collect those arguments into a dict that you can then process:

```python
def print_parameters(**kwargs):
    for key, value in kwargs.items():
        print(f'{key} = {value}')


print_parameters(alpha=1.5, beta=9, gamma=4)
```

Keyword arguments make the function call clearer to new readers of the code, they can have default values specified in
the function definition and they provide a powerful way to extend a function's parameters while remaining backward
compatible with existing callers. The best practice is to always specify optional arguments using the keyword names and
never pass them as positional arguments.

### Use None and Docstrings to Specify Dynamic Default Arguments

Sometimes you need to use a non-static type as a keyword argument's default value. Something to keep in mind is that a
default argument value is evaluated only once per module load, which usually happens when a program starts up, so avoid
defining default values for timestamp arguments. As with the timestamp example, the convention for achieving the
desired result in Python is to provide a default value of _None_ and to document the actual behavior in the docstring:

```python
def log(message, when=None):
    """Log a message with a timestamp.
    
    Args:
        message: Message to print.
        when: datetime of when the message occurred.
            Defaults to the present time.
    """
    if when is None:
        when = datetime.now()
    print(f'{when}: {message}')
```

Using _None_ for default argument values is especially important when the arguments are mutable:

```python
import json


def decode(data, default={}):
    try:
        return json.loads(data)
    except ValueError:
        return default  # ! WRONG: any subsequent change to this dictionary is propagated
```

The problem here is that the dictionary specified for default will be shared by all calls to decode because default
argument values are evaluated only once (at module load time).

### Enforce Clarity with Keyword-Only and Positional-Only Arguments

To avoid confusion with the keyword arguments passed to a function, you can force the callers of that function to
provide the keywords to the parameters by making them _keyword-only arguments_, the arguments marked like this can
only be passed by keyword but never by position. The _*_ symbol in the argument list indicates the end of positional
arguments and the beginning of keyword-only arguments, like in the function definition below:
`def safe_division(number, divisor, *, ignore_overflow=False, ignore_zero_division=False)`
Note that keyword arguments and their default values will work as expected. This approach has a flaw which is that
the caller may specify the positional required arguments with a mix of positions and keywords, which will break in
case I rename one of my parameters. This is problematic if you never intended for there parameters to be part of an
explicit interface for the function (you just picked convenient parameter names chosen for the implementation). Python
3.8 solves this by providing something called _positional-only arguments_, which forces these arguments to be supplied
only by position and never by keyword, the _/_ symbol in the argument list marks where positional-only arguments ends:
`def safe_division(number, divisor, /, *, ignore_overflow=False, ignore_zero_division=False)`. One notable consequence
of keyword and positional only arguments is that any parameter name between the _/_ and _*_ symbols in the argument
list may be passed either by position or by keyword.

### Define Function Decorators with functools.wraps

Python has special syntax for decorators that can be applied to functions. A decorator has the ability to run additional
code before and after each call to a function it wraps. This means decorators can access and modify input arguments,
return values, and raised exceptions. This functionality can be useful for enforcing semantics, debugging, registering
functions, and more. One issue that can occur with decorators is that it can have side effects if we need to use the
name of the decorated function, or do introspection (such as debuggers), Object serializers break because they can't
determine the location of the original function that was decorated, calling the _help_ built in function will fail
as well...

```python
def trace(func):
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        print(f'{func.__name__}({args!r}, {kwargs!r}) '
              f'-> {result!r}')
        return result

    return wrapper


# Using the _@_ symbol is equivalent to calling the decorator on the function it wraps and assigning the return value
# to the original name in the same scope: `some_func = trace(some_func)`.
@trace
def some_func(n):
    pass


print(some_func)  # Prints <function trace.<locals>.wrapper at 0x108955dc0>
```

The solution is to use the wraps helper function from the _functools_ built-in module. This is a decorator that helps
you write decorators. When you apply it to the wrapper function, it copies all of the important metadata about the
inner function to the outer function:

```python
from functools import wraps


def trace(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        ...

    return wrapper
```

Python functions have many other standard attributes (_\__name___, _\__module___, _\__annotations___) that must be
preserved to maintain the interface of functions in the language. _wraps_ ensures that you'll always get the correct
behavior.

## Chapter 4: Comprehensions and Generators<a name="Chapter4"></a>

### Use Comprehensions Instead of map and filter

Comprehensions is the compact Python's syntax for deriving a new list from another sequence or iterable. An example of
this syntax is `squares = [x**2 for x in a]`. Unless you're applying a single-argument function, list comprehensions are
also clearer than the `map` built-in function for simple cases: `squares = map(lambda x: x ** 2, a)`. Unlike map, list
comprehensions let you easily filter items from the input list, removing corresponding outputs from the result:
`squares = [x**2 for x in a if x % 2 == 0]` instead of `squares = map(lambda x: x**2, filter(lambda x: x % 2 == 0, a))`.
Dictionaries and sets have their own equivalents of list comprehensions (dictionary and set comprehensions):

```python
# The format is {k:v for expr}, note the ':' separating k and v
squares_dict = {x: x ** 2 for x in a if x % 2 == 0}
cubed_set = {x ** 3 for x in a if x % 3 == 0}
# Now with map and filter which is much more verbose
alt_dict = dict(map(lambda x: (x, x ** 2), filter(lambda x: x % 2 == 0, a)))
alt_set = set(map(lambda x: x ** 3, filter(lambda x: x % 3 == 0, a)))
```

### Avoid More Than Two Control Subexpressions in Comprehensions

Comprehensions support multiple levels of looping: `flat_matrix = [x for row in matrix for x in row]`. Or if you
want the output to be another list: `squared_matrix = [[x**2 for x in row] for row in matrix]`. Anything deeper than two
levels of iteration gets visually noisy, and the indentation of the loop alternative makes it even clearer.
Comprehensions support multiple if conditions. Multiple conditions at the same loop level have an implicit and
expression: `[x for x in a if x > 4 if x % 2 == 0]` is equivalent to `[x for x in a if x > 4 and x % 2 == 0]`.
Conditions can be specified at each level of looping after the for subexpression (but avoid using list, dict, or set
comprehensions that look like this): `[[x for x in row if x % 3 == 0] for row in matrix if sum(row) >= 10]`.
The rule of thumb is to avoid using more than two control subexpressions in a comprehension. This could be two
conditions, two loops, or one condition and one loop, use _if_ and _for_ statements otherwise.

### Avoid Repeated Work in Comprehensions by Using Assignment Expressions

A common pattern with comprehensions (list, dict, and set) is the need to reference the same computation in multiple
places: `found = {name: check_item(stock.get(name, 0), 1) for name in order if check_item(stock.get(name, 0), 1)}`. The
`check_item` function is called twice and can lead to errors if there is a change on the parameters in one of the calls.
An easy solution to these problems is to use the walrus operator `:=`:
`found = {name: batches for name in order if (batches := check_item(stock.get(name, 0), 1))}`.

It's valid syntax to define an assignment expression in the value expression for a comprehension. But if you try to
reference the variable it defines in other parts of the comprehension, you might get an exception at runtime because
of the order in which comprehensions are evaluated: `{name: (tenth := count // 10) for name, count in stock.items()}`
this will fail but this won't: `{name: tenth for name, count in stock.items() if (tenth := count // 10) > 0}`.
If a comprehension uses the walrus operator in the value part of the comprehension and doesn't have a condition, it'll
leak the loop variable into the containing scope, but it doesn't happen for the loop variables from comprehensions:

```python
half = [(last := count // 2) for count in stock.values()]
print(f'Last item of {half} is {last}')  # prints Last item of [62, 17, 4, 12] is 12 as 'last' is available here
half = [count // 2 for count in stock.values()]
print(count)  # Exception because loop variable didn't leak
```

It's better not to leak loop variables, so use assignment expressions only in the condition part of a comprehension.
Using an assignment expression also works the same way in generator expressions:

```python
found = ((name, batches) for name in order if (batches := check_item(stock.get(name, 0), 1)))
print(next(found))  # Or some other operation over the iteration 
```

### Consider Generators Instead of Returning Lists

Generators are produced by functions that use yield expressions. When called, a generator function does not actually run
but instead immediately returns an iterator. With each call to the next built-in function, the iterator advances the
generator to its next yield expression. You can easily convert the iterator returned by the generator to a list by
passing it to the list built-in function if necessary: `list(some_function_that_returns_an_iterator(*args))`.
The iterator returned by a generator produces the set of values passed to yield expressions within the generator
function's body.

### Be Defensive When Iterating Over Arguments

When a function takes a list of objects as a parameter, it's often important to iterate over that list multiple times.
In this case using an iterator can be problematic because it produces its results only a single time. If you iterate
an iterator or a generator that has already raised a _StopIteration_ exception, you won't get results calling it again,
but you also won't get errors either. This is because _for_ loops, the _list_ constructor, and many other functions
throughout the Python standard library expect the _StopIteration_ exception to be raised during normal operation. To
solve this, you can explicitly exhaust an input iterator and keep a copy of its entire contents in a list. The
problem with this solution is that the copy of the input iterator's contents could be extremely large, causing the
program to run out of memory and crash. One way around this is to accept a function that returns a new iterator each
time it's called:

```python
def normalize_func(get_iter):
    total = sum(get_iter())  # New iterator
    result = []
    for value in get_iter():  # New iterator
        result.append(100 * value / total)
    return result


# To use normalize_func, I can pass in a lambda expression that calls the generator and produces a new iterator:
percentages = normalize_func(lambda: read_visits('my_numbers.txt'))
```

A better way to achieve the same result is to provide a new container class that implements the iterator protocol. The
iterator protocol is how Python for loops and related expressions traverse the contents of a container type.
When Python sees a statement like `for x in foo`, it actually calls `iter(foo)`. The iter built-in function calls the
`foo.__iter__` special method in turn. The _\__iter___ method must return an iterator object (which itself implements
the _\__next___ special method). Then, the for loop repeatedly calls the next built-in function on the iterator object
until it's exhausted (indicated by raising a _StopIteration_ exception).
You can achieve all of this behavior for your classes by implementing the _\__iter___ method as a generator. so the
above `get_iter()` call can be replaced by something like:

```python
class ReadVisits:
    def __init__(self, data_path):
        self.data_path = data_path

    def __iter__(self):
        with open(self.data_path) as f:
            for line in f:
                yield int(line)
```

And then pass an instance of this class, which would return a new iterator on the `sum(...)` and `for value...` calls.
The protocol states that when an iterator is passed to the iter built-in function, iter returns the iterator itself. In
contrast, when a container type is passed to iter, a new iterator object is returned each time. You can put a condition
to check weather an object is an iterator instead of a container:

```python
if iter(some_numbers) is some_numbers:  # An iterator -- bad!
    raise TypeError('Must supply a container')
```

Also, the `collections.abc` built-in module defines an Iterator class that can be used in an isinstance test this:

```python
from collections.abc import Iterator

if isinstance(some_numbers, Iterator):  # Another way to check
    raise TypeError('Must supply a container')
```

### Consider Generator Expressions for Large List Comprehensions

The problem with list comprehensions is that they may create new list instances containing one item for each value in
input sequences (this can even be a never ending list of values if it is derived from a network socket). To solve this
issue, Python provides generator expressions, which are a generalization of list comprehensions and generators.
Generator expressions evaluate to an iterator that yields one item at a time from the expression, instead of
materializing the whole output sequence when they're run. You create a generator expression by putting
list-comprehension-like syntax between _()_ characters: `the_iterator = (len(x) for x in open('my_file.txt'))`.
Generator expressions can be composed together: `roots = ((x, x**0.5) for x in the_iterator)`. Each time I advance this
iterator, it also advances the interior iterator, creating a domino effect of looping, evaluating conditional
expressions, and passing around inputs and outputs. Generator expressions are a great choice to compose
functionality that's operating on a large stream of input. Be careful not to use these iterators more than once as
they are stateful.

### Compose Multiple Generators with yield from

Generators are so useful that many programs start to look like layers of generators strung together. To avoid
clumsiness on the nesting of generators such as:

```python
def composing_generators():
    for delta in some_generator1(arg1):
        yield delta
    for delta in some_generator2(arg2):
        yield delta
    for delta in some_generator1(arg3):
        yield delta
```

Is to use the yield from expression:

```python
def composing_generators_with_yield():
    yield from some_generator1(arg1)
    yield from some_generator2(arg2)
    yield from some_generator3(arg3)
```

`yield from` causes the Python interpreter to handle the nested _for_ loop and _yield_ expression boilerplate for you,
which results in better performance.

### Avoid Injecting Data into Generators with send

_yield_ expressions provide generator functions with a simple way to produce an iterable series of output values but
there's no immediately obvious way to simultaneously stream data in and out of a generator as it runs. Python
generators support the _send_ method, which upgrades _yield_ expressions into a two-way channel, which can be used to
provide streaming inputs to a generator at the same time it's yielding outputs. Normally, when iterating a generator,
the value of the yield expression is None `returned_value = yield 1 # returned_value is None`, but when I call the
_send_ method instead of iterating the generator with a _for_ loop or the next built-in function, the supplied parameter
becomes the value of the _yield_ expression when the generator is resumed (when the generator first starts, a _yield_
expression has not been encountered yet, so the only valid value for calling _send_ initially is _None_):

```python
def my_generator():
    received = yield 1
    print(f'received = {received}')


it = iter(my_generator())
output = it.send(None)  # Get first generator output
print(f'output = {output}')

try:
    it.send('hello!')  # Send value into the generator
except StopIteration:
    pass

# The output of the above is
# output = 1
# received = hello!
```

To use this in a function you can store the value provided by _send_ by using _yield_ and then act accordingly:

```python
def wave_modulating(steps):
    amplitude = yield  # Receive initial amplitude
    for step in range(steps):
        output = amplitude * math.sin(step * (2 * math.pi / steps))
        amplitude = yield output  # Receive next amplitude
```

But what if we want to chain multiple signals together in sequence? Using the _send_ instead of just yield would  
introduce _None_ values in between the result. When each _yield_ from expression finishes iterating over a nested
generator, it moves on to the next one. Each nested generator starts with a bare _yield_ expression (one without a
value) in order to receive the initial amplitude from a generator _send_ method call. This causes the parent
generator to output a _None_ value when it transitions between child generators. For this reason is better to avoid
the _send_ method entirely and go with a simpler approach. The easiest solution is to pass an iterator to the function
like this:

```python
import math


def wave_cascading(amplitude_it, steps):
    for step in range(steps):
        radians = step * (2 * math.pi / steps)
        fraction = math.sin(radians)
        amplitude = next(amplitude_it)  # Get next input
        yield amplitude * fraction


# Iterators are stateful each of the nested generators picks up where the previous generator left off
def complex_wave_cascading(amplitude_it):
    yield from wave_cascading(amplitude_it, 3)
    yield from wave_cascading(amplitude_it, 4)


def run_cascading():
    amplitudes = [7, 7, 7, 2, 2, 2, 2]
    it = complex_wave_cascading(iter(amplitudes))
    for _ in amplitudes:
        print(next(it))


run_cascading()
```

### Avoid Causing State Transitions in Generators with throw

Another advanced generator feature is the throw method for re-raising Exception instances within generator functions.
When the _throw_ method is called, the next occurrence of a yield expression re-raises the provided Exception instance
after its output is received instead of continuing normally. The throw method can be used to re-raise exceptions within
generators at the position of the most recently executed yield expression.

```python
class MyError(Exception):
    pass


def my_generator():
    yield 1
    yield 2
    yield 3


it = my_generator()
print(next(it))  # Yield 1
print(next(it))  # Yield 2
print(it.throw(NotImplementedError))  # Throws an exception
```

When you call _throw_, the generator function may catch the injected exception with a standard _try/except_ compound
statement that surrounds the last _yield_ expression that was executed.

```python
# The below generator will yield [1,2,'Error!',4]
def my_generator():
    yield 1
    try:
        yield 2
    except NotImplementedError:
        print('Error!')
    else:
        yield 3
    yield 4
```

This functionality provides a two-way communication channel between a generator and its caller that can be useful in
certain situations, but as with _send_, a best approach is to define a stateful closure using an iterable container
object. Often, what you're trying to accomplish by mixing generators and exceptions is better achieved with
asynchronous features (better to avoid this completely).

### Consider itertools for Working with Iterators and Generators

The itertools built-in module contains a large number of functions that are useful for organizing and interacting with
iterators. The following describe the most important functions that you should know in three primary categories:

#### Linking Iterators Together

```python
from itertools import chain, repeat, cycle, tee, zip_longest

chain([1, 2, 3], [4, 5, 6])  # combine multiple iterators into a single sequential iterator
repeat('hello', 3)  # output a single value forever, or use the second parameter to specify the repetitions
[next(cycle([1, 2])) for _ in range(10)]  # repeat an iterator's items forever:  [1, 2, 1, 2, 1, 2, 1, 2, 1, 2]
# split a single iterator into the number of parallel iterators specified by the second parameter
tee(['first', 'second'], 3)  # produces 3 iterators each one with ['first','second']
zip_longest([1, 2, 3], ['one', 'two'], fillvalue='na')  # returns a the fillvalue value when an iterator is exhausted
```

#### Filtering Items from an Iterator

```python
from itertools import islice, takewhile, dropwhile, filterfalse

# Use islice to slice an iterator by numerical indexes without copying. You can specify the end, start and end, or 
# start, end, and step sizes and the behavior is similar to that of standard sequence slicing and striding
islice([1, 2, 3, 4, 5, 6, 7, 8, 9, 10], 5)  # returns [1,2,3,4,5] 
islice([1, 2, 3, 4, 5, 6, 7, 8, 9, 10], 2, 8, 2)  # returns [3,5,7]
takewhile(lambda x: x < 7, [4, 5, 6, 7, 8, 9])  # returns items until a predicate function returns False [4,5,6]
dropwhile(lambda x: x < 7, [4, 5, 6, 7, 8, 9])  # Opposite to takewhile [7,8,9]
filterfalse(lambda x: x < 7, [4, 5, 6, 7, 8, 9])  # opposite of the filter built-in function returns [7,8,9]
```

#### Producing Combinations of Items from Iterators

```python
from itertools import accumulate, product, permutations, combinations, combinations_with_replacement

values = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]


def sum_modulo_20(first, second):
    output = first + second
    return output % 20


# folds an item from the iterator into a running value by applying a function that takes two parameters. Outputs the 
# current accumulated result for each input value
accumulate(values)  # outputs [1, 3, 6, 10, 15, 21, 28, 36, 45, 55]
accumulate(values, sum_modulo_20)  # outputs [1, 3, 6, 10, 15, 1, 8, 16, 5, 15]
# Cartesian product of items from one or more iterators
product([1, 2], repeat=2)  # [(1, 1), (1, 2), (2, 1), (2, 2)]
product([1, 2], ['a', 'b'])  # [(1, 'a'), (1, 'b'), (2, 'a'), (2, 'b')]
permutations([1, 2, 3], 2)  # returns the unique ordered permutations of length N: [(1,2),(1,3),(1,4),(2,1),(2,3)...]
combinations([1, 2, 3], 2)  # returns the unordered combinations of length N with unrepeated items from an iterator
combinations_with_replacement([1, 2, 3], 2)  # like combinations but with repeated values:  [(1,1),(1,2),(1,3),...]
```

## Chapter 5: Classes and Interfaces<a name="Chapter5"></a>

### Compose Classes Instead of Nesting Many Levels of Built-in Types

Python's built-in dictionary type is good for maintaining dynamic internal state over the lifetime of an object, like
situations in which you need to do bookkeeping for an unexpected set of identifiers. But there's a danger of
overextending them to write brittle code. For example if you have a student grade system in place which student names
are not known in advance you might use a map to add and store the student names with the grades. Later on if you want to
itemize the scores by student and subject you might be tempted to add another dictionary under students with the names
of the subjects and the score, but if later the requirements change again and then you see yourself adding yet other
inner dictionaries, it is time to refactor the code to use a hierarchy of classes (you should avoid using sets,
tuples and dicts for more than a level of nesting).

#### Refactoring to Classes

Start moving to classes at the bottom of the dependency tree. Avoid tuples or positional data structures to store
information as it is difficult to maintain in case of a change of requirements. _namedtuple_ is a more appropriate type
as it lets you easily define tiny, immutable data classes (in the case of class grades before):

```python
from collections import namedtuple

Grade = namedtuple('Grade', ('score', 'weight'))
```

These classes can be constructed with positional or keyword arguments. The fields are accessible with named attributes.
Having named attributes makes it easy to move from a _namedtuple_ to a class later if the requirements change again.
Keep in mind that _namedtuple_ doesn't support default values and therefore is not good to store optional properties
and its attribute values are still accessible using numerical indexes and iteration so it can lead to unintentional
usages on external API's. In the case of School grades, the final representation will look like this:

```python
from collections import defaultdict, namedtuple

Grade = namedtuple('Grade', ('score', 'weight'))


class Subject:
    def __init__(self):
        self._grades = []

    def report_grade(self, score, weight):
        self._grades.append(Grade(score, weight))

    def average_grade(self):
        total, total_weight = 0, 0
        for grade in self._grades:
            total += grade.score * grade.weight
            total_weight += grade.weight
        return total / total_weight


class Student:
    def __init__(self):
        self._subjects = defaultdict(Subject)

    def get_subject(self, name):
        return self._subjects[name]

    def average_grade(self):
        total, count = 0, 0
        for subject in self._subjects.values():
            total += subject.average_grade()
            count += 1
        return total / count


class Gradebook:
    def __init__(self):
        self._students = defaultdict(Student)

    def get_student(self, name):
        return self._students[name]
```

### Accept Functions Instead of Classes for Simple Interfaces

Many built-in APIs allow you to customize behavior by passing in a callback function. In other languages, you might
expect hooks to be defined by an abstract class. In Python, many hooks are just stateless functions with well-defined
arguments and return values. Supplying these type of functions makes APIs easy to build and test because it separates
side effects from deterministic behavior. If the function passed needs to maintain state you can use a stateful closure:

```python
from collections import defaultdict


def increment_with_report(current, increments):
    added_count = 0

    def missing():
        nonlocal added_count  # Stateful closure
        added_count += 1
        return 0

    result = defaultdict(missing, current)
    for key, amount in increments:
        result[key] += amount
    return result, added_count
```

This make it easier for simple functions passed to interfaces to add functionality later by hiding state in a closure.
Another approach is to define a small class that encapsulates the state you want to track:

```python
class CountMissing:
    def __init__(self):
        self.added = 0

    def missing(self):
        self.added += 1
        return 0

# And  then  use it like:
# counter = CountMissing()
# result = defaultdict(counter.missing, current)  <-- Method ref
```

The above code still leaves certain bits unclear: Who constructs a _CountMissing_ object or calls the _missing_ method?
Python allows classes to define the _\__call___ special method. _\__call___ allows an object to be called just like a
function. It also causes the callable built-in function to return True for such an instance, just like a normal function
or method. All objects that can be executed in this manner are referred to as callables:

```python
class BetterCountMissing:
    def __init__(self):
        self.added = 0

    def __call__(self):
        self.added += 1
        return 0

# And then use it like:
# counter = BetterCountMissing()
# assert counter() == 0 and callable(counter)
# result = defaultdict(counter, current)  <-- Relies on __call__
```

The _\__call___ method indicates that a class's instances can be used where a function argument is suitable also.

### Use @classmethod Polymorphism to Construct Objects Generically

In Python, not only do objects support polymorphism, but classes do as well. say that I'm writing a MapReduce
implementation, and I want a common class to represent the input data:

```python
class InputData:
    def read(self):
        raise NotImplementedError  # must be defined by subclasses


class PathInputData(InputData):  # Some concrete implementation
    def __init__(self, path):
        super().__init__()
        self.path = path

    def read(self):
        with open(self.path) as f:
            return f.read()
```

Similarly for the workers, we could have the following:

```python
class Worker:
    def __init__(self, input_data):
        self.input_data = input_data
        self.result = None

    def map(self):  # 'Abstract' implementation, needs to be present on the subclasses
        raise NotImplementedError

    def reduce(self, other):
        raise NotImplementedError


class LineCountWorker(Worker):
    def map(self):
        data = self.input_data.read()
        self.result = data.count('\n')

    def reduce(self, other):
        self.result += other.result
```

After we have this implementation in place, how do we connect them? The simplest approach is to manually build and
connect the objects with some helper functions:

```python
import os
from threading import Thread


def generate_inputs(data_dir):  # Lists the contents of the folder passed 
    for name in os.listdir(data_dir):
        yield PathInputData(os.path.join(data_dir, name))


def create_workers(input_list):  # create the LineCountWorker from the InputDatainstances returned by generate_inputs
    workers = []
    for input_data in input_list:
        workers.append(LineCountWorker(input_data))
    return workers


def execute(workers):
    threads = [Thread(target=w.map) for w in workers]
    for thread in threads: thread.start()
    for thread in threads: thread.join()
    first, *rest = workers
    for worker in rest:  # combines the results into one final value
        first.reduce(worker)
    return first.result


def mapreduce(data_dir):  # connect all the pieces together in a function to run each step
    inputs = generate_inputs(data_dir)
    workers = create_workers(inputs)
    return execute(workers)
```

The problem with the above implementation is that the _mapreduce_ function is not generic at all. If I wanted to write
another _InputData_ or _Worker_ subclass, I would also have to rewrite the _generate_inputs_, _create_workers_, and
_mapreduce_ functions to match. In other languages you'd solve this by constructor polymorphism, but Python only allows
for the single constructor method _\__init___. The best way to solve this problem is with *class method* polymorphism.
This is like an instance method polymorphism except that it's for whole classes instead of their constructed objects:

```python
import os


class GenericInputData:
    def read(self):
        raise NotImplementedError

    @classmethod
    # Take a dictionary with a set of configuration parameters that the concrete subclass needs to interpret.
    # Note we are passing 'cls' and not 'self'
    def generate_inputs(cls, config):
        raise NotImplementedError


class PathInputData(GenericInputData):

    @classmethod
    def generate_inputs(cls, config):
        data_dir = config['data_dir']
        for name in os.listdir(data_dir):
            yield cls(os.path.join(data_dir, name))
```

Similarly, I can add the _create\_workers_ helper part and construct instances of the _GenericWorker_ concrete
subclass by using `cls()` as a generic constructor:

```python
class GenericWorker:
    def __init__(self, input_data):
        self.input_data = input_data
        self.result = None

    def map(self):
        raise NotImplementedError

    def reduce(self, other):
        raise NotImplementedError

    @classmethod
    def create_workers(cls, input_class, config):
        workers = []
        for input_data in input_class.generate_inputs(config):
            # calling cls() provides an alternative way to construct GenericWorker objects besides using __init__ 
            workers.append(cls(input_data))
        return workers
```

Then rewrite the _mapreduce_ function to be completely generic:

```python
def mapreduce(worker_class, input_class, config):
    workers = worker_class.create_workers(input_class, config)
    return execute(workers)

# Finally we execute the above passing the classes
# result = mapreduce(LineCountWorker, PathInputData, {'data_dir': tmpdir})
```

### Initialize Parent Classes with super

The old, simple way to initialize a parent class from a child class is to directly call the parent class's _\__init___
method with the child instance, which works fine for basic class hierarchies but breaks in cases such as multiple
inheritance (something to avoid), the _\__init___ call order is not specified across all subclasses. Or diamond
inheritance (inherits from two separate classes that have the same superclass somewhere in the hierarchy) which causes
the common superclass's _\__init___ method to run multiple times causing unexpected behavior.
To solve these problems, Python has the super built-in function and standard method resolution order *(MRO)*. _super_
ensures that common superclasses in diamond hierarchies are run only once, the MRO defines the ordering in which
superclasses are initialized, following an algorithm called *C3 linearization*:

```python
class Times7(BaseClass):
    def __init__(self, value):  # MyBaseClass.__init__, is run only a single time
        super().__init__(value)
        self.value *= 7


class Plus9(BaseClass):
    def __init__(self, value):
        super().__init__(value)
        self.value += 9


class GoodWay(Times7, Plus9):  # The parent classes are run in the order specified in the class statement
    def __init__(self, value):
        super().__init__(value) 
```

You can access the Order of execution by calling the `GoodWay.mro()` method. The above calls `GoodWay.__init__`,
`Plus9.__init__`, then `Times7.__init__` and `SomeBaseClass.__init__`. Once this reaches the top of the diamond, all of
the initialization methods do their work in the opposite order from how their _\__init___ functions were called.
Call to `super().__init__` is much more maintainable than calling `BaseClass.__init__` directly from within subclasses.
The _super_ function can also be called with two parameters: first the type of the class whose MRO parent view you're
trying to access, and then the instance on which to access that view. These parameters are not required for object
instance initialization as Python's compiler automatically provides the correct parameters (_\__class___ and _self_) for
you when super is called with zero arguments within a class definition:

```python
class ExplicitTrisect(BaseClass):
    def __init__(self, value):
        super(ExplicitTrisect, self).__init__(value)
        self.value /= 3
```

### Consider Composing Functionality with Mix-in Classes

Python is an object-oriented language with built-in facilities for making multiple inheritance tractable although you
should consider writing a mix-in (class that defines only a small set of additional methods for its child classes to
provide) instead of using multiple inheritance in order to leverage its convenience and encapsulation. Mix-in classes
don't define their own instance attributes nor require their _\__init___ constructor to be called. Python makes it
trivial to inspect the current state of any object, regardless of its type. _Dynamic inspection_ means you can write
generic functionality just once, in a mix-in, and it can then be applied to many other classes. Mix-ins can be composed
and layered to minimize repetitive code and maximize reuse. An example of this is to a Mix-in to transform a large
number of python objects into a dictionary using:

```python
class ToDictMixin:
    def to_dict(self):
        return self._traverse_dict(self.__dict__)

    def _traverse_dict(self, instance_dict):
        output = {}
        for key, value in instance_dict.items():
            output[key] = self._traverse(key, value)
        return output

    def _traverse(self, key, value):  # Implementation details are straightforward and rely on dynamic attribute access
        if isinstance(value, ToDictMixin):
            return value.to_dict()
        elif isinstance(value, dict):
            return self._traverse_dict(value)
        elif isinstance(value, list):
            return [self._traverse(key, i) for i in value]
        elif hasattr(value, '__dict__'):
            return self._traverse_dict(value.__dict__)
        else:
            return value


# Using the above mixin and the class below together:
class BinaryTree(ToDictMixin):
    def __init__(self, value, left=None, right=None):
        self.value = value
        self.left = left
        self.right = right


tree = BinaryTree(10, left=BinaryTree(7, right=BinaryTree(9)), right=BinaryTree(13, left=BinaryTree(11)))


# Transforming the class above to something readable is as simple as calling print(tree.to_dict())

# You can make their mix-ins generic functionality pluggable so behaviors can be overridden when required. 
class BinaryTreeWithParent(BinaryTree):
    def __init__(self, value, left=None, right=None, parent=None):
        super().__init__(value, left=left, right=right)
        self.parent = parent

    def _traverse(self, key, value):
        if (isinstance(value, BinaryTreeWithParent) and key == 'parent'):
            return value.value  # Prevent cycles
        else:
            return super()._traverse(key, value)
```

In the example above, by defining `BinaryTreeWithParent._traverse`, we've also enabled any class that has an attribute
of type _BinaryTreeWithParent_ to automatically work with the _ToDictMixin_. Mix-ins can also be composed together:

```python
import json


class JsonMixin:
    @classmethod
    def from_json(cls, data):
        kwargs = json.loads(data)
        return cls(**kwargs)

    # the only requirements of a JsonMixin subclass are providing a 
    # to_dict method and taking keyword arguments for the __init__ method 
    def to_json(self):
        return json.dumps(self.to_dict())
```

Avoid using multiple inheritance with instance attributes and _\__init___ if mix-in classes can achieve the same
outcome.

### Prefer Public Attributes Over Private Ones

In Python, there are only two types of visibility for a class's attributes: public and private. Public attributes
can be accessed by anyone using the dot operator on the object and private fields are specified by prefixing an
attribute's name with a double underscore. Private fields can be accessed from methods of the container class but
directly accessing private fields from outside the class raises an exception. Class methods also have access to private
attributes because they are declared within the surrounding class block. A subclass can't access its parent class's
private fields. The private attribute behavior is implemented with a simple transformation of the attribute name. When
the Python compiler sees private attribute access in methods like MyChildObject.get_private_field, it translates the
_\__private\_field__ attribute access to use the name _\_MyChildObject__private\_field_ instead. Accessing the parent's
private attribute from the child class fails simply because the transformed attribute name doesn't exist. Accessing
the parent's private attribute from the child class fails simply because the transformed attribute name doesn't exist.
Check the object's attribute dictionary to see that you get the private attributes stored with the names as they appear
after the transformation: `print(foo.__dict__)`. Fields prefixed by a single underscore (like _protected_field) are
protected by convention, meaning external users of the class should proceed with caution. This means that by choosing
private attributes, you're only making subclass overrides and extensions cumbersome and brittle. Your potential
subclassers will still access the private fields when they absolutely need to do so, but if the class hierarchy
changes beneath you, these classes will break because the private attribute references are no longer valid. As a rule,
document each protected field and explain which fields are internal APIs available to subclasses and which should be
left alone entirely. The only time to seriously consider using private attributes is when you're worried about naming
conflicts with subclasses (primarily a concern with classes that are part of a public API). To reduce the risk of this
issue occurring, you can use a private attribute in the parent class to ensure that there are no attribute names that
overlap with child classes

### Inherit from collections.abc for Custom Container Types

Every Python class is a container of some kind, encapsulating attributes and functionality together. Python also
provides built-in container types for managing data: lists, tuples, sets, and dictionaries. When you're designing
classes for simple use cases like sequences, it's natural to want to subclass Python's built-in list type directly.
Now, imagine that I want to provide an object that feels like a list and allows indexing but isn't a list subclass.
How do you make this class act like a sequence type? Python implements its container behaviors with instance methods
that have special names.

```python
bar = [1, 2, 3]
bar[0]
# it will be interpreted as:
bar.__getitem__(0)
```

To make a class act like a sequence, you can provide a custom implementation of _\__getitem___, but implementing this
method isn't enough to provide all of the sequence semantics you'd expect from a list instance (for example
`len(myclass)` won't work), there are several methods you'll need to implement.

To avoid this difficulty throughout the Python universe, the built-in __collections.abc__ module defines a set of
abstract base classes that provide all the typical methods for each container type. When you subclass from these
abstract base classes and forget to implement required methods, the module tells you something is wrong:
When you do implement all the methods required by an abstract base class from __collections.abc__, it provides all the
additional methods, like __index__ and __count__, for free. Beyond the __collections.abc__ module, Python uses a variety
of special methods for object comparisons and sorting, which may be provided by container classes and non-container
classes alike. 

## Chapter 6: Metaclasses and Attributes<a name="Chapter6"></a>

## Chapter 7: Concurrency and Parallelism<a name="Chapter7"></a>

## Chapter 8: Robustness and Performance<a name="Chapter8"></a>

## Chapter 9: Testing and Debugging<a name="Chapter9"></a>

## Chapter 10: Collaboration<a name="Chapter10"></a>