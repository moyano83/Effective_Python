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
    amplitude = yield  # Receive initial amplitude
for step in range(steps):


def wave_modulating(steps):
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

### Use Plain Attributes Instead of Setter and Getter Methods

Metaclasses let you intercept Python's class statement and provide special behavior each time a class is defined.
Dynamic attributes enable you to override objects and cause unexpected side effects. Metaclasses can create extremely
bizarre behaviors that are unapproachable to newcomers. Programmers coming to Python from other languages may naturally
try to implement explicit getter and setter methods in their classes. This methods are clumsy but help define the
interface for a class, making it easier to encapsulate functionality, validate usage, and define boundaries.
In Python you never need to implement explicit setter or getter methods, instead, you should always start your
implementations with simple public attributes. Later, if I decide I need special behavior when an attribute is set, I
can migrate to the _@property_ decorator:

```python
class VoltageResistance(Resistor):
    def __init__(self, ohms):
        super().__init__(ohms)
        self._voltage = 0

    @property
    def voltage(self):  # In order for this code to work properly, the names of both the setter and the 
        return self._voltage  # getter methods must match the intended property name

    @voltage.setter
    def voltage(self, voltage):
        self._voltage = voltage
        self.current = self._voltage / self.ohms


r2 = VoltageResistance(1e3)
r2.voltage = 10  # assigning the voltage property will run the voltage setter method
```

Specifying a setter on a property also enables me to perform type checking and validation on values passed to the class.
When you use __@property__ methods to implement setters and getters, be sure that the behavior you implement is not
surprising. For example, don't set other attributes in getter property methods. Be sure to also avoid any other side
effects that the caller may not expect beyond the object, such as importing modules dynamically, running slow helper
functions, doing I/O, or making expensive database queries. The biggest shortcoming of __@property__ is that the
methods for an attribute can only be shared by subclasses. Unrelated classes can't share the same implementation.

### Consider @property Instead of Refactoring Attributes

One advanced but common use of __@property__ is transitioning what was once a simple numerical attribute into an
on-the-fly calculation. This is extremely helpful because it lets you migrate all existing usage of a class to have new
behaviors without requiring any of the call sites to be rewritten (which is especially important if there's calling code
that you don't control). This let you make incremental progress toward better data models by using __@property__, but
consider refactoring a class and all call sites when you find yourself using __@property__ too heavily.

### Use Descriptors for Reusable @property Methods

The big problem with the __@property__ built-in is that the methods it decorates can't be reused for multiple
attributes of the same class. They also can't be reused by unrelated classes. Imagine you want to write a class that
has a validation that is common for several properties of the class, instead of calling the validation check on each
method, you can use a __descriptor__. The descriptor protocol defines how attribute access is interpreted by the
language. A descriptor class can provide _\__get___ and _\__set___ methods that let you reuse the validation behavior
without boilerplate:

```python
class Grade:
    def __get__(self, instance, instance_type):
        pass

    def __set__(self, instance, value):
        pass


class Exam:
    # Class attributes
    math_grade = Grade()
    science_grade = Grade()
    grade = Grade()
```

When I assign a property like `exam.grade = 40`, it is interpreted as `Exam.__dict__['grade'].__set__(exam, 40)`. When
I retrieve a property `exam.grade`, it is interpreted as `Exam.__dict__['grade'].__get__(exam, Exam)`. In short, when an
__Exam__ instance doesn't have an attribute named __grade__, Python falls back to the __Exam__ class's attribute
instead. If this class attribute is an object that has _\__get___ and _\__set___ methods, Python assumes that you want
to follow the descriptor protocol. Combining this with __@property__ and you get something like:

```python
class Grade:
    def __init__(self):
        self._value = 0

    def __get__(self, instance, instance_type):
        return self._value

    def __set__(self, instance, value):
        if not (0 <= value <= 100):
            raise ValueError(
                'Grade must be between 0 and 100')
        self._value = value


class Exam:
    math_grade = Grade()
    writing_grade = Grade()
    science_grade = Grade()
```

But the problem with the above implementation is that a single __Grade__ instance is shared across all __Exam__
instances for the class attributes. The __Grade__ instance for this attribute is constructed once in the program
lifetime, when the __Exam__ class is first defined. You can keep track of the __Grade__ value for each unique __Exam__
instance by saving the per-instance state in a dictionary:

```python
class Grade:
    def __init__(self):
        self._values = {}

    def __get__(self, instance, instance_type):
        if instance is None:
            return self
        return self._values.get(instance, 0)

    def __set__(self, instance, value):
        if not (0 <= value <= 100):
            raise ValueError('Grade must be between 0 and 100')
        self._values[instance] = value
```

The _values dictionary holds a reference to every instance of Exam ever passed to _\__set___ over the lifetime of the
program, which causes instances to never have their reference count go to zero, preventing cleanup by the garbage
collector. To fix this, I can use Python's __weakref__ built-in module. This module provides a special class called
__WeakKeyDictionary__ that can take the place of the simple dictionary used for _values. This class removes instances
from its set of items when the Python runtime knows it's holding the instance's last remaining reference in the program:

```python
from weakref import WeakKeyDictionary


class Grade:
    def __init__(self):
        self._values = WeakKeyDictionary()
```

### Use __getattr__, __getattribute__, and __setattr__ for Lazy Attributes

Imagine that I want to represent the records in a database as Python objects, the code that connects Python objects to
the database doesn't need to explicitly specify the schema of the records, it can be generic. Python makes this dynamic
behavior possible with the _\__getattr___ special method. This is because this method is called every time an attribute
can't be found in an object's instance dictionary (is not called if the attribute is already there). This behavior is
especially helpful for use cases like lazily accessing schemaless data.

```python
class LazyRecord:
    def __init__(self):
        self.exists = 5

    def __getattr__(self, name):
        value = f'Value for {name}'
        setattr(self, name, value)
        return value
```

Python has another object hook called _\__getattribute___. This special method is called every time an attribute is
accessed on an object, even in cases where it does exist in the attribute dictionary (if a property shouldn't exist,
I can raise an __AttributeError__):

```python
class ValidatingRecord:
    def __init__(self):
        self.exists = True

    def __getattribute__(self, name):
        try:
            return super().__getattribute__(name)
        except AttributeError:
            value = f'Value for {name}'
            setattr(self, name, value)
            return value
```

Imagine that I want to lazily push data back to the database when values are assigned to my Python object. I can do
this with _\__setattr___, a similar object hook that lets you intercept arbitrary attribute assignments. The
_\__setattr___ method is always called every time an attribute is assigned on an instance. The problem with
_\__getattribute___ and ___\setattr___ is that they're called on every attribute access for an object, even when you
may not want that to happen. This could cause infinite recursion as an object call to get/set a property calls the
get or set attribute methods which in turns access the property on the object which makes a call to the get or set
attribute method again and again, to avoid this use methods from __super()__ (i.e., the object class) to access
instance attributes.

### Validate Subclasses with __init_subclass__

One of the simplest applications of metaclasses is verifying that a class was defined correctly. When you're building a
complex class hierarchy, you may want to enforce style, require overriding methods, or have strict relationships
between class attributes. Metaclasses enable these use cases by providing a reliable way to run your validation code
each time a new subclass is defined. Using metaclasses for validation can raise errors even before _\__init__ is
called, when the module containing the class is first imported at program startup.

A metaclass is defined by inheriting from type. In the default case, a metaclass receives the contents of associated
class statements in its _\__new___ method:

```python
class ValidatePolygon(type):
    def __new__(meta, name, bases, class_dict):
        if bases:  # Only validate subclasses of the Polygon class
            if class_dict['sides'] < 3:
                raise ValueError('Polygons need 3+ sides')
        return type.__new__(meta, name, bases, class_dict)


class Polygon(metaclass=ValidatePolygon):
    sides = None  # Must be specified by subclasses

    @classmethod
    def interior_angles(cls):
        return (cls.sides - 2) * 180
```

The metaclass has access to the name of the class, the parent classes it inherits from (bases), and all the class
attributes that were defined in the class's body. In the above example, If I try to define a polygon with fewer than
three sides, the validation will cause the class statement to fail immediately after the class statement body (the
program will not even be able to start running when I define such a class). Python 3.6 introduced simplified syntax
to accomplish this: the _\__init_subclass___ special class method:

```python
class APolygon:  # This produces the same effect than the ValidatePoligon Class
    sides = None  # Must be specified by subclasses

    def __init_subclass__(cls):
        super().__init_subclass__()
        if cls.sides < 3:
            raise ValueError('Polygons need 3+ sides')

    @classmethod
    def interior_angles(cls):
        return (cls.sides - 2) * 180
```

One problem with the standard Python metaclass machinery is that you can only specify a single metaclass per class
definition. It's possible to fix this by creating a complex hierarchy of metaclass type definitions to layer validation
(having a validation classes to inherit from each others), but this approach ruins composability, which is often the
purpose of class validation, if you want to apply one of the validations to another set of classes, you'll have to
duplicate the code and make those classes have this new validation. The _\__init_subclass___ can be defined by multiple
levels of a class hierarchy as long as the __super__ built-in function is used to call any parent or sibling
_\__init_subclass___ definitions. It's even compatible with multiple inheritance (you can even use _\__init_subclass___
in complex cases like diamond inheritance):

```python
class Filled:
    color = None  # Must be specified by subclasses

    def __init_subclass__(cls):
        super().__init_subclass__()  # This enables validation in multiple layers of classes and multiple inheritance
        if cls.color not in ('red', 'green', 'blue'):
            raise ValueError('Fills need a valid color')


class RedTriangle(Filled, Polygon):  # The validation is called for each one of the Metaclasses
    color = 'red'
    sides = 3
```

### Register Class Existence with __init_subclass__

Another common use of metaclasses is to automatically register types in a program. Registration is useful for doing
reverse lookups, where you need to map a simple identifier back to a corresponding class. Now imagine you want to use a
serialization/deserialization hierarchy of classes using JSON:

```python
import json


class Serializable:
    def __init__(self, *args):
        self.args = args

    def serialize(self):
        return json.dumps({'args': self.args})


class Deserializable(Serializable):
    @classmethod
    def deserialize(cls, json_data):
        params = json.loads(json_data)
        return cls(*params['args'])


class BetterPoint2D(Deserializable):
    pass
```

The problem with the above is that you need to deserialize data like `BetterPoint2D.deserialize(data)` which requires
you to know the intended type of the serialized data ahead of time instead of being able to deserialize any class back
to its corresponding Python object. To solve this you can include the serialized object's class name in the JSON data:

```python
class BetterSerializable:
    def __init__(self, *args):
        self.args = args

    def serialize(self):
        return json.dumps({
            'class': self.__class__.__name__,
            'args': self.args,
        })

    def __repr__(self):
        name = self.__class__.__name__
        args_str = ', '.join(str(x) for x in self.args)
        return f'{name}({args_str})'
```

And then maintain a mapping of class names back to constructors for those objects:

```python
registry = {}


def register_class(target_class):
    registry[target_class.__name__] = target_class


def deserialize(data):
    params = json.loads(data)
    name = params['class']
    target_class = registry[name]
    return target_class(*params['args'])
```

But the problem is that to ensure that deserialize always works properly, you must call _\__register_class__ for every
class I may want to deserialize in the future. This is error prone as you might forget this call, but there is a
way to act on the programmer's intent to use the serialization code and ensure that _\__register_class__ is called in
all cases. Metaclasses enable this by intercepting the class statement when subclasses are defined:

```python
class Meta(type):
    def __new__(meta, name, bases, class_dict):
        cls = type.__new__(meta, name, bases, class_dict)
        register_class(cls)
        return cls


class RegisteredSerializable(BetterSerializable, metaclass=Meta):
    pass
```

Or as seen before, the following can be used too (requires Python 3.6 or above):

```python
class BetterRegisteredSerializable(BetterSerializable):
    def __init_subclass__(cls):
        super().__init_subclass__()
        register_class(cls)
```

### Annotate Class Attributes with __set_name__

One more useful feature enabled by metaclasses is the ability to modify or annotate properties after a class is
defined but before the class is actually used (commonly used with __descriptors__):

```python
class GenericDataBaseField:
    def __init__(self, name):
        self.name = name
        self.internal_name = '_' + self.name

    def __get__(self, instance, instance_type):
        if instance is None:
            return self
        return getattr(instance, self.internal_name, '')

    def __set__(self, instance, value):
        setattr(instance, self.internal_name, value)
```

With the column name stored in the __GenericDataBaseField__ descriptor, I can save all of the per-instance state
directly in the instance dictionary as protected fields by using the __setattr__ built-in function, and later I can load
state with __getattr__. An example of this can be seen below:

```python
class Customer:
    first_name = GenericDataBaseField('first_name')
    last_name = GenericDataBaseField('last_name')
    prefix = GenericDataBaseField('prefix')
    suffix = GenericDataBaseField('suffix')
```

Notice how the left side is redundant with right side why I need to define the attribute name if it is passed in the
GenericDataBaseField constructor? The problem is that the order of operations in the __Customer__ class definition
is the opposite of how it reads from left to right. First, the __GenericDataBaseField__ constructor is called as
`GenericDataBaseField('first_name')`. Then, the return value of that is assigned to `Customer.field_name`. There's no
way for a __GenericDataBaseField__ instance to know upfront which class attribute it will be assigned to. You can
eliminate this redundancy using a metaclass, which let you hook the class statement directly and take action as soon as
a class body is finished:

```python
class Meta(type):
    def __new__(meta, name, bases, class_dict):
        for key, value in class_dict.items():
            if isinstance(value, NewGenericDataBaseField):
                value.name = key
                value.internal_name = '_' + key
        cls = type.__new__(meta, name, bases, class_dict)
        return cls


# All classes representing database rows should inherit from this class to ensure that they use the metaclass
class DatabaseRow(metaclass=Meta):
    pass


class NewGenericDataBaseField:
    def __init__(self):
        # These will be assigned by the metaclass.
        self.name = None
        self.internal_name = None

    def __get__(self, instance, instance_type):
        if instance is None:
            return self
        return getattr(instance, self.internal_name, '')

    def __set__(self, instance, value):
        setattr(instance, self.internal_name, value)


class BetterCustomer(DatabaseRow):
    first_name = Field()
    last_name = Field()
    prefix = Field()
    suffix = Field()
```

As it can be seen, __NewGenericDataBaseField__ no longer requires arguments to be passed to its constructor, eliminating
the redundancy of the field name definition. Instead, its attributes are set by the `Meta.__new__` method. The problem
here is that you need to make sure that your database row representation always inherits from __DatabaseRow__. This
problem can be solved by using the _\__set_name___ special method for descriptors. This method (Python 3.6 and above),
is called on every descriptor instance when its containing class is defined. It receives as parameters the owning
class that contains the descriptor instance and the attribute name to which the descriptor instance was assigned:

```python
class FinalVersionDataBaseField:
    def __init__(self):
        self.name = None
        self.internal_name = None

    def __set_name__(self, owner, name):  # Called on class creation for each descriptor
        self.name = name
        self.internal_name = '_' + name

    def __get__(self, instance, instance_type):
        if instance is None:
            return self
        return getattr(instance, self.internal_name, '')

    def __set__(self, instance, value):
        setattr(instance, self.internal_name, value)


class FixedCustomer:
    first_name = FinalVersionDataBaseField()
    last_name = FinalVersionDataBaseField()
```

With this approach, you can avoid memory leaks and the weakref built-in module by having descriptors store data they
manipulate directly within a class's instance dictionary.

### Prefer Class Decorators Over Metaclasses for Composable Class Extensions

Imagine you want to decorate all of the methods of a class with a helper that prints arguments, return values, and
exceptions raised:

```python
from functools import wraps


def trace_func(func):
    if hasattr(func, 'tracing'):  # Only decorate once
        return func

    @wraps(func)
    def wrapper(*args, **kwargs):
        result = None
        try:
            result = func(*args, **kwargs)
            return result
        except Exception as e:
            result = e
            raise
        finally:
            print(f'{func.__name__}({args!r}, {kwargs!r}) -> '
                  f'{result!r}')

    wrapper.tracing = True
    return wrapper
```

And then use this decorator in a class:

```python
class TraceDict(dict):
    @trace_func
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)

    @trace_func
    def __setitem__(self, *args, **kwargs):
        return super().__setitem__(*args, **kwargs)

    @trace_func
    def __getitem__(self, *args, **kwargs):
        return super().__getitem__(*args, **kwargs)
```

But as it can be seen this is quite cumbersome because you have to redefine all the methods that I wanted to decorate
with __@trace_func__. Even worst, if a method is later added to the dict superclass, it won't be decorated unless I
also define it in __TraceDict__. You can solve this problem is to use a metaclass to automatically decorate all methods
of a class (implemented here by wrapping each function or method in the new type with the __trace_func__ decorator):

```python
import types

trace_types = (
    types.MethodType,
    types.FunctionType,
    types.BuiltinFunctionType,
    types.BuiltinMethodType,
    types.MethodDescriptorType,
    types.ClassMethodDescriptorType)


class TraceMeta(type):
    def __new__(meta, name, bases, class_dict):
        klass = super().__new__(meta, name, bases, class_dict)
        for key in dir(klass):
            value = getattr(klass, key)
            if isinstance(value, trace_types):
                wrapped = trace_func(value)
                setattr(klass, key, wrapped)
        return klass


class TraceDict(dict, metaclass=TraceMeta):
    pass
```

The problem with this approach is that if you try to use __TraceMeta__ when a superclass already has specified a
metaclass it fails as this new metaclass does not inherit from __TraceMeta__ (and you might not be able to modify it
if you don't own the library where this new metaclass is). This metaclass approach puts too many constraints on the
class that's being modified. To solve this problem, Python supports class decorators. Class decorators work just like
function decorators: They're applied with the __@__ symbol prefixing a function before the class declaration. The func-
tion is expected to modify or re-create the class accordingly and then return it:

```python
def trace(klass):
    for key in dir(klass):
        value = getattr(klass, key)
        if isinstance(value, trace_types):
            wrapped = trace_func(value)
            setattr(klass, key, wrapped)
    return klass


@trace
class MyClass:
    pass
```

A class decorator is a simple function that receives a class instance as a parameter and returns either a new class or a
modified version of the original class, and they also work when the class being decorated already has a metaclass.

## Chapter 7: Concurrency and Parallelism<a name="Chapter7"></a>

### Use subprocess to Manage Child Processes

Python is a feasible alternative when existing shell scripts get complicated, for the sake of readability and
maintainability. Python has many ways to run subprocesses, but the best choice for managing child processes is to use
the subprocess built-in module:

```python
import subprocess

result = subprocess.run(['echo', 'Hello from the child!'], capture_output=True, encoding='utf-8')
result.check_returncode()  # No exception means clean exit
print(result.stdout)  # Prints 'Hello from the child!'
```

Child processes run independently from their parent process(the Python interpreter), which decouples the child process
from the parent frees up the parent process to run many child processes in parallel. If I create a subprocess using the
_Popen_ class instead of the run function, I can poll child process status periodically while Python does other work:

```python
proc = subprocess.Popen(['sleep', '1'])
while proc.poll() is None:
    print('Working...')
```

You can also pipe data from a Python program into a subprocess and retrieve its output.

```python
import os, subprocess


def run_encrypt(data):
    env = os.environ.copy()
    env['password'] = 'zf7ShyBhZOraQDdE/FiZpm/m/8f9X+M1'
    proc = subprocess.Popen(
        ['openssl', 'enc', '-des3', '-pass', 'env:password'],  # sequence of program arguments
        env=env,
        stdin=subprocess.PIPE,
        stdout=subprocess.PIPE)
    proc.stdin.write(data)
    proc.stdin.flush()  # Ensure that the child gets input
    return proc


procs = []
for _ in range(3):
    data = os.urandom(10)
    proc = run_encrypt(data)  # piping random bytes into the encryption function
    procs.append(proc)

for proc in procs:
    out, _ = proc.communicate()  # Sends data to stdin. Read data from stdout and stderr, until end-of-file is reached
    print(out[-10:])
```

It's also possible to create chains of parallel processes, just like UNIX pipelines, connecting the output of one child
process to the input of another:

```python
import subprocess

encrypt_procs = []
hash_procs = []
for _ in range(3):
    data = os.urandom(100)
    encrypt_proc = run_encrypt(data)
    encrypt_procs.append(encrypt_proc)
    hash_proc = run_hash(encrypt_proc.stdout)  # Other function that hashes the given input
    hash_procs.append(hash_proc)
    # Ensure that the child consumes the input stream and the communicate() method doesn't inadvertently steal
    # input from the child. Also lets SIGPIPE propagate to the upstream process if the downstream process dies.
    encrypt_proc.stdout.close()
    encrypt_proc.stdout = None

for proc in encrypt_procs:
    try:
        proc.communicate(timeout=0.1)  # causes an exception if the child process hasn't finished within the time period
    except subprocess.TimeoutExpired:
        proc.terminate()
        proc.wait()
    print('Exit status', proc.poll())
```

### Use Threads for Blocking I/O, Avoid for Parallelism

The standard implementation of Python is called CPython, which runs a Python program in two steps: First, it
parses and compiles the source text into bytecode, which is a low-level representation of the program as 8-bit
instructions and then runs the bytecode using a stack-based interpreter.The bytecode interpreter has state that must be
maintained and coherent while the Python program executes. CPython enforces coherence with a mechanism called the global
interpreter lock (GIL). GIL is a mutual-exclusion lock (mutex) that prevents CPython from being affected by preemptive
multithreading, where one thread takes control of a program by interrupting another thread. The downside of this
mechanism is that GIL causes only one of the multiple available threads of execution to ever make forward progress at a
time.

```python
from threading import Thread


class SomeThread(Thread):
    def __init__(self, arg1):
        super().__init__()
        self.arg1 = arg1

    def run(self):
        self.factors = list(some_function(self.arg1))


threads = []
for arg in [2139079, 1214759, 1516637, 1852285]:
    thread = SomeThread(arg)
    thread.start()
    threads.append(thread)

for thread in threads:
    thread.join()
```

The above code might take longer than just executing the `some_function` function within a for loop in the main script
due to the effects of GIL. There are ways to get CPython to utilize multiple cores, but they don't work with the
standard Thread class, so what's the reason for this class? Because multiple threads make it easy for a program to seem
like it's doing multiple things at the same time and to deal with blocking I/O, which happens when Python does certain
types of system calls. Threads help handle blocking I/O by insulating a program from the time it takes for the operating
system to respond to requests. If you need to do blocking I/O and computation simultaneously, consider moving your
system calls to threads. GIL prevents Python code from running in parallel, but doesn't have an effect on system calls.
This is because Python threads release the GIL just before they make system calls, and they reacquire the GIL as
soon as the system calls are done.

### Use Lock to Prevent Data Races in Threads

The fact that GIL prevents Python threads from running on multiple CPU cores in parallel doesn't mean that it also acts
as a lock for a program's data structures. This is because a thread's operations on data structures can be interrupted
between any two bytecode instructions in the Python interpreter, which is dangerous if you access the same objects from
multiple threads simultaneously. This is specially dangerous on assignments such as `a+=1` due to python dividing this
instruction into: get 'a' value, create intermediate result with `result=a+1` and then assigning this result to 'a'. The
Thread doing this op might be interrupted just after the assignment creating race conditions. To avoid this, you can use
The _Lock_ class from the _threading_ module:

```python
from threading import Lock


class LockingCounter:  # Implementation of a secure Lock counter mechanism
    def __init__(self):
        self.lock = Lock()
        self.count = 0

    def increment(self, offset):
        with self.lock:
            self.count += offset
```

### Use Queue to Coordinate Work Between Threads

One of the most useful arrangements for concurrent work is a pipeline of functions. To create this arrangement, you need
a way to hand off work between the pipeline phases, which can be modeled as a thread-safe producer–consumer queue:

```python
from collections import deque
from threading import Lock


class MyQueue:
    def __init__(self):
        self.items = deque()
        self.lock = Lock()

    def put(self, item):
        with self.lock:
            self.items.append(item)

    def get(self):
        with self.lock:
            return self.items.popleft()
```

To use the above queue with a thread, we need to properly handle the case where the input queue is empty because
the previous phase hasn't completed its work yet. This happens where I catch the IndexError exception below:

```python
def run(self):
    while True:  # Executes forever, there is no way to signal to a worker thread that it's time to exit
        self.polled_count += 1
        try:
            item = self.in_queue.get()
        except IndexError:
            time.sleep(0.01)  # No work to do
        else:
            result = self.func(item)
            self.out_queue.put(result)
            self.work_done += 1
```

When the worker functions vary in their respective speeds, an earlier phase can prevent progress in later phases,
backing up the pipeline. This causes later phases to starve and constantly check their input queues for new work in a
tight loop. This is a waste of CPU time as workers are constantly raising and catching IndexError exceptions. Also
a backup in the pipeline can cause the program to crash arbitrarily due to the fast phase queue constant increase in
size. Fortunately there are better ways to implement this.

#### Queue to the Rescue

_Queue_ eliminates the busy waiting in the worker by making the get method block until new data is available.

```python
from queue import Queue
from threading import Thread

my_queue = Queue()


def consumer():
    my_queue.get()


thread = Thread(target=consumer)
thread.start()

# Even though the thread is running first, it won't finish until an item is put on the Queue instance and the get 
# method has something to return
my_queue.put(object())  # Runs before get() above
thread.join()
```

The _Queue_ class lets you specify the maximum amount of pending work to allow between two phases, which causes calls to
put to block when the queue is already full. The _Queue_ class can also track the progress of work using the `task_done`
method. This lets you wait for a phase's input queue to drain and removes the need to poll the last phase of a pipeline.

```python
def consumer():
    work = in_queue.get()  # Runs Second 
    # Doing work
    in_queue.task_done()  # Runs Third


thread = Thread(target=consumer)
thread.start()

in_queue.put(object())  # Runs first
in_queue.join()  # Runs fourth
thread.join()
```

Putting all of these together into a Queue subclass that also tells the worker thread when it should stop processing:

```python
from threading import Thread
from queue import Queue


class ClosableQueue(Queue):
    SENTINEL = object()

    def close(self):
        self.put(self.SENTINEL)

    def __iter__(self):
        while True:
            item = self.get()
            try:
                if item is self.SENTINEL:
                    return  # Causes the thread to exit
                yield item
            finally:
                self.task_done()  # To track progress on the work queue


class StoppableWorker(Thread):
    def __init__(self, func, in_queue, out_queue):
        super().__init__()
        self.func = func
        self.in_queue = in_queue
        self.out_queue = out_queue

    def run(self):  # The thread will exit when the for loop is exhausted
        for item in self.in_queue:
            result = self.func(item)
            self.out_queue.put(result)


def input_function():
    pass


def transform_function():
    pass


input_queue = ClosableQueue()
transform_queue = ClosableQueue()
done_queue = ClosableQueue()
threads = [
    StoppableWorker(input_function, input_queue, transform_queue),
    StoppableWorker(transform_function, transform_queue, done_queue)
]

for thread in threads:
    thread.start()

for _ in range(1000):
    input_queue.put(object())
# Send the stop signal after all the input work has been injected by closing the input queue of the first phase
input_queue.close()
# Wait for the work to finish by joining the queues that con- nect the phases.
input_queue.join()
# signal the next phase to stop by closing its input queue
transform_queue.close()
transform_queue.join()

for thread in threads:
    thread.join()
```

### Know How to Recognize When Concurrency Is Necessary

The hardest type of change to handle is moving from a single-threaded program to one that needs multiple concurrent
lines of execution. This is usually the case when there are I/O operations involved. The process of spawning a
concurrent line of execution for each unit of work is called fan-out and waiting for all of those concurrent units
of work to finish before moving on to the next phase in a coordinated process is called fan-in. Python provides many
built-in tools for achieving fan-out and fan-in with various trade-offs.

### Avoid Creating New Thread Instances for On-demand Fan-out

Threads are the natural first tool to reach for in order to do parallel I/O in Python, but they have significant
downsides when you try to use them for fanning out to many concurrent lines of execution:

    * Thread instances require special tools to coordinate with each other safely, making it harder to read and reason
    * Threads require a lot of memory (8 MB per executing thread)
    * Starting a thread is costly, and threads have a negative performance impact when they run due to context switching

Also threads make it more difficult to deal with errors and exceptions, the _Thread_ class will independently catch any
exceptions that are raised by the target function and then write their traceback to `sys.stderr`, leaving the code that
created the Thread and called join unaffected. Given all of these issues, threads are not the solution if you need to
constantly create and finish new concurrent functions.

### Understand How Using Queue for Concurrency Requires Refactoring

The next approach to try adding concurrent execution is to implement a threaded pipeline using the Queue class from the
queue built-in module, instead of creating one thread per work unit, you can create a fixed number of worker threads
upfront and have them do parallelized I/O as needed. But this takes a significant amount of work to refactor existing
code to use _Queue_, especially when multiple stages of a pipeline are required. Using Queue fundamentally limits the
total amount of I/O parallelism a program can leverage compared to alternative approaches provided by other built-ins.

### Consider ThreadPoolExecutor When Threads Are Necessary for Concurrency

Python includes the concurrent.futures built-in module, which provides the _ThreadPoolExecutor_ class. You can fan out
work items by submitting a function to an executor that will be run in a separate thread. Later, I can wait for the
result of all tasks in order to fan in:

```python
from concurrent.futures import ThreadPoolExecutor


def submit_work(pool, items):
    futures = []
    with ThreadPoolExecutor(max_workers=10) as pool:
        for i in items:
            future = pool.submit(some_func, *i)  # Fan out
            futures.append(future)
        for future in futures:
            future.result()  # Fan in
```

The threads used for the executor can be allocated in advance, which means I don't have to pay the startup cost on each
execution. _ThreadPoolExecutor_ automatically propagates exceptions back to the caller when the result method is
called on the Future instance returned by the submit method.

### Achieve Highly Concurrent I/O with Coroutines

Python addresses the need for highly concurrent I/O with coroutines. Coroutines let you have a very large number of
seemingly simultaneous functions in your Python programs. They're implemented using the async and await keywords along
with the same infrastructure that powers generators. The cost of starting a coroutine is a function call. Once a
coroutine is active, it uses less than 1 KB of memory until it's exhausted. Coroutines pause at each await
expression and resume executing an async function after the pending awaitable is resolved:

```python
async def some_logic(state, args):
    ...
    # Do some input/output in here:
    data = await my_socket.read(50)
    ...


import asyncio


async def execute(items, args):
    tasks = []
    for x in items:
        task = some_logic(x, *args)  # Fan out 
        tasks.append(task)
    await asyncio.gather(*tasks)  # Fan in
    return some_result


# Call the above with
asyncio.run(execute(some_items, args))  # Run the event loop
```

Calling an async function doesn't immediately run that function. Instead, it returns a coroutine instance that can be
used with an await expression at a later time (similar to yield in generators). The gather function from the asyncio
built-in library causes fan-in. The await expression on gather instructs the event loop to run the coroutines
concurrently and resume execution of the enclosing coroutine when all of them have been completed. No locks are required
as all execution occurs within a single thread. The I/O is parallelized as part of the event loop provided by asyncio.
If my requirements change and I also need to do I/O somewhere else, I can easily accomplish this by adding _async_
and _await_ keywords to the existing functions and call sites instead of having to restructure everything as I would
have had to do if I were using Thread or Queue instances.

### Know How to Port Threaded I/O to asyncio

[Asyncio library](https://docs.python.org/3/library/asyncio.html) has great features. Python provides asynchronous
versions of for loops, with statements, generators, comprehensions, and library helper functions that can be used as
drop-in replacements in coroutines. The asyncio built-in module makes it straightforward to port existing code that
uses threads and blocking I/O over to coroutines and asynchronous I/O.

### Mix Threads and Coroutines to Ease the Transition to asyncio

To port a large program to asyncio, you usually do this sequentially which results in a mix of threads and coroutines.
Asyncio provides built-in facilities to do this. To convert threaded code to coroutines there are two approaches,
top-down or bottom-up. Top-down is useful when you maintain a lot of common modules that you use across many different
programs. The concrete steps are:

    1. Change a top function to use async def instead of def
    2. Wrap all of its calls that do I/O—potentially blocking the event loop—to use asyncio.run_in_executor instead
    3. Ensure that the resources or callbacks used by run_in_executor invocations are properly synchronized 
       (using Lock or the asyncio.run_coroutine_threadsafe function)
    4. Try to eliminate get_event_loop and run_in_executor calls by moving downward through the call hierarchy and 
       converting intermediate functions and methods to coroutines following the first three steps

A snippet on how to use this function is shown below:

```python
import asyncio


def some_function_to_run_in_the_even_loop():
    pass


async def run_mix_threads_and_async(handles, some_file):  # Step 1 
    loop = asyncio.get_event_loop()
    with open(some_file, 'wb') as output:
        async def write_async(data):
            output.write(data)

        def write(data):
            coro = write_async(data)
            future = asyncio.run_coroutine_threadsafe(coro, loop)  # eliminates the need for the Lock, Step 3
            future.result()

        tasks = []
        for handle in handles:
            task = loop.run_in_executor(None, some_function_to_run_in_the_even_loop, handle, write)  # Step 2
            tasks.append(task)
        await asyncio.gather(*tasks)
```

The first parameter to `run_in_executor` is the _ThreadPoolExecutor_ to run the `some_function_to_run_in_the_even_loop`
in, _None_ means to run it using the default one.

The bottom-up approach has four steps too:

    1. Create a new asynchronous coroutine version of each leaf function that you're trying to port
    2. Change the existing synchronous functions so they call the coroutine versions and run the event loop
    3. Move up a level of the call hierarchy, make another layer of coroutines, and replace existing calls to 
       synchronous functions with calls to the coroutines defined in step 1
    4. Delete synchronous wrappers around coroutines created in step 2 as you stop requiring them

To run that coroutine until it finishes, I need to create an event loop for each worker thread and then call its
`run_until_complete` method. This method will block the current thread and drive the event loop until the worker
coroutine exits:

```python
def some_function(handle, interval, write_func):
    loop = asyncio.new_event_loop()
    asyncio.set_event_loop(loop)

    async def write_async(data):
        write_func(data)

    coro = other_async_function(handle, interval, write_async)
    loop.run_until_complete(coro)
```

The `run_until_complete` method of the asyncio event loop enables synchronous code to run a coroutine until it finishes.
The `asyncio.run_coroutine_threadsafe` function provides the same functionality across thread boundaries. Together these
help with bottom-up migrations to asyncio.

### Avoid Blocking the asyncio Event Loop to Maximize Responsiveness

In the example above, the open, close, and write calls for the output file handle happen in the main event loop.
These operations all require making system calls to the program's host operating system, which may block the event
loop for significant amounts of time and prevent other coroutines from making progress. Yo should migrate this
operations to a thread safe version of them like the one below:

```python
from threading import Thread
import asyncio


class WriteThread(Thread):
    def __init__(self, output_path):
        super().__init__()
        self.output_path = output_path
        self.output = None
        self.loop = asyncio.new_event_loop()

    def run(self):
        asyncio.set_event_loop(self.loop)
        with open(self.output_path, 'wb') as self.output:
            self.loop.run_forever()
        # Run one final round of callbacks so the await on stop() in another event loop will be resolved.
        self.loop.run_until_complete(asyncio.sleep(0))

    async def real_write(self, data):
        self.output.write(data)

    async def write(self, data):
        coro = self.real_write(data)
        future = asyncio.run_coroutine_threadsafe(coro, self.loop)
        await asyncio.wrap_future(future)

    # The following two functions allows this class to be run in with statements
    async def __aenter__(self):
        loop = asyncio.get_event_loop()
        await loop.run_in_executor(None, self.start)
        return self

    async def __aexit__(self, *_):
        await self.stop()
```

The above class can be used like this:

```python
async def run_fully_async(handles, interval, output_path):
    async with WriteThread(output_path) as output:
        tasks = []
        for handle in handles:
            task = asyncio.create_task(coro)  # coro is some coroutine
            tasks.append(task)
        await asyncio.gather(*tasks)
```

### Consider concurrent.futures for True Parallelism

The multiprocessing built-in module enables Python to utilize multiple CPU cores in parallel by running additional
interpreters as child processes (each one with its own GIL). Each child has a link to the main process where it receives
instructions to do computation and returns results:

```python
from concurrent.futures import ThreadPoolExecutor


def main():
    pool = ThreadPoolExecutor(max_workers=2)
    results = list(pool.map(some_function, arguments_to_the_function))


if __name__ == '__main__':
    main()
```

In this code, _ProcessPoolExecutor_ does the following:

    1. It takes each item from the arguments to map.
    2. It serializes the item into binary data by using the pickle module
    3. copies the serialized data from the main interpreter process to a child interpreter process over a local socket
    4. It deserializes the data back into Python objects, using pickle in the child process
    5. It runs the some_function on the input data in parallel with other child processes
    6. It serializes the result back into binary data
    7. It copies that binary data back through the socket
    8. It deserializes the binary data back into Python objects in the parent process
    9. It merges the results from multiple children into a single list to return

The overhead of using multiprocessing via _ProcessPoolExecutor_ is high because of all the serialization and
deserialization that must happen between the parent and child processes. So use it with tasks that doesn't share state
and only a small amount of data must be transferred between the parent and child processes

## Chapter 8: Robustness and Performance<a name="Chapter8"></a>

Python includes many of the algorithms and data structures you need to achieve high performance.

### Take Advantage of Each Block in try/except /else/finally

There are four distinct times when you might want to take action during exception handling in Python. These are captured
in the functionality of _try_, _except_, _else_, and _finally_ blocks.

#### finally Blocks

Use _try/finally_ when you want exceptions to propagate up but also want to run cleanup code even when exceptions occur.
For example to close file handles, although you must call open before the try block because exceptions that occur when
opening the file (like OSError if the file does not exist) should skip the _finally_ block entirely.

#### else Blocks

Use _try/except/else_ to make it clear which exceptions will be handled by your code and which exceptions will propagate
up. When the try block doesn't raise an exception, the _else_ block runs. The _else_ block helps you minimize the
amount of code in the _try_ block, which is good for isolating potential exception causes and improves readability.

#### Everything Together

Use _try/except/else/finally_ when you want to do it all in one compound statement. _try_ to perform the read
operations and initialization, _else_ for the normal business case, _except_ to handle specific failure scenarios
and _finally_ to perform any cleanup or closing operation.

### Consider contextlib and with Statements for Reusable try/finally Behavior

The with statement in Python is used to indicate when code is running in a special context:

```python
# Mutual exclusion lock example 
from threading import Lock

lock = Lock()

with lock:
    # Do something while maintaining an invariant
    pass
```

The example above is equivalent to this _try/finally_ construction because the _Lock_ class properly enables the _with_
statementL:

```python
from threading import Lock

lock = Lock()

lock.acquire()
try:
    pass
finally:
    lock.release()
```

The with statement version of this is clearly better. It's easy to make your objects and functions work in with
statements by using the _contextlib_ built-in module. This module contains the _contextmanager_ decorator, which lets a
simple function be used in with statements. For example:

```python
import logging
from contextlib import contextmanager


def my_function():
    logging.debug('Some debug data')
    logging.error('Error log here')
    logging.debug('More debug data')


# I can elevate the log level of this function temporarily by defining a context manager. This helper function boosts 
# the logging severity level before running the code in the with block and reduces the logging severity level afterward

@contextmanager
def debug_logging(level):
    logger = logging.getLogger()
    old_level = logger.getEffectiveLevel()
    logger.setLevel(level)
    try:
        yield  # This is the point at which the with block's contents will execute, exceptions here will be re-raised
    finally:
        logger.setLevel(old_level)


with debug_logging(logging.DEBUG):
    print('* Inside:')
    my_function()

my_function()  # The same function running outside the with block won't print debug messages:
```

#### Using with Targets

The context manager passed to a with statement may also return an object, which is assigned to a local variable in the
as part of the compound statement. This gives the code running in the with block the ability to directly interact with
its context. To enable your own functions to supply values for as targets, all you need to do is yield a value from your
context manager:

```python
import logging
from contextlib import contextmanager


@contextmanager
def log_level(level, name):
    logger = logging.getLogger(name)
    old_level = logger.getEffectiveLevel()
    logger.setLevel(level)
    try:
        yield logger
    finally:
        logger.setLevel(old_level)  # This restores the log level as soon as the with block exits


# The value yielded by context managers is supplied to the as part of the with statement
with log_level(logging.DEBUG, 'my-log') as logger:
    logger.debug(f'This is a message for {logger.name}!')  # Logger is the one returned by log_level function
    logging.debug('This will not print')  # The default logging level is still WARNING
```

### Use datetime Instead of time for Local Clocks

If your program handles time, you'll probably find yourself converting time between UTC and local clocks for the sake of
human understanding. Python provides two ways of accomplishing time zone conversions.

#### The time Module

The localtime function from the time built-in module lets you convert a UNIX timestamp to a local time that matches the
host computer's time zone.

```python
import time

local_tuple = time.localtime(1552774475)
time_format = '%Y-%m-%d %H:%M:%S'
time_str = time.strftime(time_format, local_tuple)
print(time_str)  # Prints 2019-03-16 15:14:35

time_tuple = time.strptime(time_str, time_format)  # In case we do the inverse operation
utc_now = time.mktime(time_tuple)
print(utc_now)  # prints 1552774475.0
```

To convert between timezones is a bit more complicated, Python lets you use these time zones through the time module if
your platform supports it. _time_ module is platform-dependent and its behavior is determined by how the underlying C
functions work with the host operating system. This makes the functionality of the time module unreliable in Python.

#### The datetime Module

_datetime_ can be used to convert from the current time in UTC to local time:

```python
from datetime import datetime, timezone

now = datetime(2019, 3, 16, 22, 14, 35)
now_utc = now.replace(tzinfo=timezone.utc)
now_local = now_utc.astimezone()
print(now_local)  # 2019-03-16 15:14:35-07:00

time_str = '2019-03-16 15:14:35'
now = datetime.strptime(time_str, time_format)
time_tuple = now.timetuple()
utc_now = time.mktime(time_tuple)
print(utc_now)  # prints 1552774475.0
```

Unlike the time module, the _datetime_ module has facilities for reliably converting from one local time to another
local time. However, _datetime_ only provides the logic for time zone operations with its _tzinfo_ class and methods
(The Python default installation is missing time zone definitions besides UTC). To cover this gap, use the _pytz_ module
which contains a full database of every time zone definition you might need.

```python
import pytz

arrival_nyc = '2019-03-16 23:33:24'
nyc_dt_naive = datetime.strptime(arrival_nyc, time_format)
eastern = pytz.timezone('US/Eastern')
nyc_dt = eastern.localize(nyc_dt_naive)
utc_dt = pytz.utc.normalize(nyc_dt.astimezone(pytz.utc))  # always convert local times to UTC first
print(utc_dt)  # prints 2019-03-17 03:33:24+00:00

pacific = pytz.timezone('US/Pacific')
sf_dt = pacific.normalize(utc_dt.astimezone(pacific))  # Then, convert to local times as a final step
print(sf_dt)  # prints 2019-03-17 09:18:24+05:45
```

With _datetime_ and _pytz_, conversions are consistent across all environments, regardless of what operating system.

### Make pickle Reliable with copyreg

The pickle built-in module can serialize Python objects into a stream of bytes and deserialize bytes back into objects.
Pickled byte streams shouldn't be used to communicate between untrusted parties, use it to pass Python objects between
programs that you control over binary channels.

```python
import pickle

program_state_path = 'game_state.bin'
with open(program_state_path, 'wb') as f:
    pickle.dump(program_state_path, f)

with open(program_state_path, 'rb') as f:
    program_state_after = pickle.load(f)
print(program_state_after.__dict__)  # prints the contents of the object loaded
```

The problem with the previous approach is that if the class definition loaded changes, previous stored versions of
the class that doesn't contain some attributes would skip them. You can fix this problems using the _copyreg_ module,
which lets you register the functions responsible for serializing and deserializing Python objects.

#### Default Attribute Values

In the simplest case, you can use a constructor with default arguments and use that constructor with pickling:

```python
import copyreg


class SomeState:
    def __init__(self, attr1=0, attr2=4):
        self.attr1 = attr1
        self.attr2 = attr2


def pickle_some_state(some_state):
    kwargs = some_state.__dict__
    # This turns some_state into a tuple of parameters for the copyreg module
    # The returned tuple contains the function for unpickling and the parameters to pass to the unpickling function  
    return unpickle_some_state, (kwargs,)


def unpickle_some_state(kwargs):
    return SomeState(**kwargs)


# Now, I register these functions with the copyreg built-in module
copyreg.pickle(SomeState, pickle_some_state)
```

With the above registration done, deserializing an old _SomeState_ object will result in valid game data instead of
missing attributes. This works because `unpickle_some_state` calls the _SomeState_ constructor directly instead of using
the pickle module's default behavior of saving and restoring only the attributes that belong to an object.

#### Versioning Classes

Sometimes you need to make backward-incompatible changes to your Python objects by removing fields. You can control
this behaviour by adding a version parameter to the functions supplied to copyreg.

```python
def pickle_game_state(game_state):
    kwargs = game_state.__dict__
    kwargs['version'] = 2
    return unpickle_some_state, (kwargs,)


def unpickle_game_state(kwargs):
    version = kwargs.pop('version', 1)
    if version == 1:
        del kwargs['attr1']  # explicitly removing the attribute from the dic
    return SomeState(**kwargs)
```

#### Stable Import Paths

Other issue you may encounter with pickle is breakage from renaming a class. But trying to deserialize an old class into
the new definition of the class throwa an exception because the import path of the serialized object's class is
encoded in the pickled data. I can specify a stable identifier for the function to use for unpickling an object, which
allows me to transition pickled data to different classes with different names when it's deserialized:

```python
import copyreg


class NewSomeState:  # renaming the SomeState class to NewSomeState and remove the old class from the program entirely
    def __init__(self, attr1=0, attr2=0, attr3=5):
        self.attr1 = attr1
        self.attr2 = attr2
        self.attr3 = attr3


copyreg.pickle(NewSomeState, pickle_some_state)
state = NewSomeState()
serialized = pickle.dumps(state)
```

With this approach, the import path to `unpickle_some_state` is encoded in the serialized data instead of
_NewSomeState_. Once I serialize data with a function, it must remain available on that import path for deserialization.

### Use decimal When Precision Is Paramount

The integer type in python produces incorrect results in certain situations due to the way the conversion between
binary and decimal is computed. In these cases where it is important, use the Decimal class from the decimal
built-in module instead.

```python
from decimal import Decimal

rate = Decimal('1.45')
seconds = Decimal(3 * 60 + 42)  # Represents 3 minutes 42 seconds
cost = rate * seconds / Decimal(60)
print(cost)  # prints 5.365
```

_Decimal_ instances can be given either a _str_ containing the number, or directly passing a _float_ or an _int_
instance to the constructor. The _Decimal_ class has a built-in function for rounding to exactly the decimal place
needed with the desired rounding behavior:

```python
from decimal import ROUND_UP

rounded = cost.quantize(Decimal('0.01'), rounding=ROUND_UP)
print(f'Rounded {cost} to {rounded}')
```

For representing rational numbers with no limit to precision, consider using the Fraction class from the fractions
built-in module.

### Profile Before Optimizing

You shouldn't assumne Python's behaviors in its runtime performance, slowdowns in a Python program can be obscure.
Instead, measure the performance of a program before you try to optimize it. Python provides a built-in _profiler_ for
this. Python provides two built-in profilers: one that is pure Python (profile) and another that is a C-extension
module (cProfile). The cProfile built-in module is better because of its minimal impact on the performance of your
program while it's being profiled. The pure Python alternative imposes a high overhead that skews the results.

```python
from random import randint
from cProfile import Profile
from pstats import Stats

max_size = 10 ** 4
data = [randint(0, max_size) for _ in range(max_size)]
test = lambda: some_function_to_test(data)

profiler = Profile()
profiler.runcall(test)

stats = Stats(profiler)  # Stats extracts statistics about the performance
stats.strip_dirs()
stats.sort_stats('cumulative')
stats.print_stats()
```

Some of the information returned includes:

    * ncalls: number of calls to the function during the profiling period
    * tottime: number of seconds spent executing the function, excluding time spent executing other functions it calls
    * tottime percall: average number of seconds spent in the function each time it is called, excluding time spent 
      executing other functions it calls. This is tottime divided by ncalls
    * cumtime: cumulative number of seconds spent executing the function, including other function calls
    * cumtime percall: average number of seconds spent in the function each time it is called, including time spent in 
      all other functions it calls. This is cumtime divided by ncalls

Sometimes you might find that a common utility function is responsible for the majority of execution time, but the
default output from the profiler makes such a situation difficult to understand because it doesn't show that the utility
function is called by many different parts of your program. Python profiler provides the `print_callers` method to show
which callers contributed to the profiling information of each function.

### Prefer deque for Producer–Consumer Queues

A FIFO queue, also known as Producer-Consumer queue, is often implemented with a _List_ in python and using the `append`
and `pop(0)` methods of the list. The problem with this approach is that the performance of the list decreases as
the element stored on it increases. The reason for this is that the pop method needs to move every item in the list back
an index, effectively reassigning the entire list's contents. Python provides the _deque_ class from the collections
built-in module to solve this problem, which provides constant time operations for inserting or removing items from its
beginning or end. Use the methods `deque.append(x)` and `deque.popleft()` to implement the FIFO queue, and get an
instance of deque with `collections.deque`.

### Consider Searching Sorted Sequences with bisect

It's common to find yourself with a large amount of data in memory as a sorted list that you then want to search.
Regardless of the data your specific program needs to process, searching for a specific value in a list takes linear
time proportional to the list's length when you call the index method. If you're not sure whether the exact value you're
searching for is in the list, then you may want to search for the closest index that is equal to or exceeds your goal
value. You can use the bisect_left function from the bisect module to do an efficient binary search through any
sequence of sorted items. The index it returns will either be where the item is already present in the list or where
you'd want to insert the item in the list to keep it in sorted order:

```python
from bisect import bisect_left

data = [....]
index = bisect_left(data, 91234)  # Exact match
assert index == 91234
index = bisect_left(data, 91234.56)  # Closest match
assert index == 91235
```

_bisect_ has a logaritmic performance and can be used with any python object that acts like a sequence.

### Know How to Use heapq for Priority Queues

Contrary to the FIFO queues, a priority queue orders the elements on the collection in order of relative importance.
Using a list for this purpose is not ideal because there is a cost on sorting the whole list each time a new element
is added. On top of this, a linear scan is needed each time an element needs to be removed as subsequent elements in
the list needs to be reordered. Python has the built-in _heapq_ module that solves this problem:

```python
from heapq import heappush

heappush(some_queue, the_element)  # Adds an element similar to append in a list
```

The heapq module requires items in the priority queue to be comparable and have a natural sort order (otherwise it
raises an error). You can quickly give your class to be ordered this behavior by using the `total_ordering` class
decorator from the _functools_ built-in module and implementing the `__lt__` special method. To create a _heap_ I
can use the `sort` method in a list, or use the `heapq.heapify(unordered_list)`. To get the first element of a _heap_,
use the `heapq.heappop(the_heap)` function.

### Consider memoryview and bytearray for Zero-Copy Interactions with bytes

Although Python isn't able to parallelize CPU-bound computation without extra effort, it is able to support
high-throughput, parallel I/O in a variety of ways. Slicing a _bytes_ instance causes the underlying data to be copied,
which takes CPU time. You can avoid this by using the Python's built-in _memoryview_ type, which exposes CPython's
high-performance buffer protocol to programs:

```python
data = b'Some binary data to be sliced'
view = memoryview(data)
chunk = view[12:19]  # This does not make a copy of the underlying data
```

Imagine that you need to insert a chunk of binary data into a binary collection, you can use the `join` method to
insert the data but it is very costly. Instead use _bytearray_ type in conjunction with _memoryview_. One limitation
with bytes instances is that they are read-only and don't allow for individual indexes to be updated:

```python
my_bytes = b'hello'
my_bytes[0] = b'\x79'  # This raises an Error
# bytearray is like a mutable version of bytes that allows for arbitrary positions to be overwritten,it uses 
# integers for its values instead of bytes
my_array = bytearray(b'hello')
my_array[0] = 0x79  # This works
```

A _memoryview_ can also be used to wrap a _bytearray_. When you slice such a _memoryview_, the resulting object can be
used to assign data to a particular portion of the underlying buffer:

```python
my_array = bytearray(b'Some binary data')
my_memory = memoryview(my_array)
slice_memory = my_memory[3:13]
```

## Chapter 9: Testing and Debugging<a name="Chapter9"></a>

### Use repr Strings for Debugging Output

The human-readable string for a value doesn't make it clear what the actual type and its specific composition are. When
debugging a program, you almost always want the _repr_ version of an object. The _repr_ built-in function returns the
printable representation of an object: `print(repr('6'))` prints `'6'`. This is equivalent to using the _'%r'_ format
string with the _%_ operator or an f-string with the _!r_ type conversion. `eval(repr(something))` should equal
`something`. However, the default implementation of repr for object subclasses isn't especially helpful, but if you
have control of the class, you can define your own `__repr__` special method that returns a string containing the Python
expression that re-creates the object:

```python
class SomeClass:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __repr__(self):
        return f'SomeClass({self.x!r}, {self.y!r})'
```

When you don't have control over the class definition, you can reach into the object's instance dictionary, which is
stored in the `__dict__` attribute.

### Verify Related Behaviors in TestCase Subclasses

The canonical way to write tests in Python is to use the unittest built-in module. To define tests, you usually create
a file named like the python file you want to test but appended '_test', and then import and extend the test classes
as follows:

```python
# my_tests.py
from unittest import TestCase, main  # Import test machinery
from the_class_i_want_to_test import function_to_test  # Some class to test


class UtilsTestCase(TestCase):
    def test_case1(self):
        self.assertEqual('Some Result', function_to_test(b'Some Result'))  # Some test cases
    # Other test cases to follow


if __name__ == '__main__':
    main()  # function that triggers the testing
```

Tests are organized into _TestCase_ subclasses. Each test case is a method beginning with the word 'test'.If one test
fails, the _TestCase_ subclass continues running the other test methods so you can get a full picture of the results.
You can also run a single test with `python3 <python file> <Main class name>.<test name>`.
The _TestCase_ class provides helper methods such as `assertEqual` or `assertTrue` which are better than the built-in
`assert` statement because they print out the inputs and outputs to help you understand the failure reasons. There's
also an `assertRaises` helper method for verifying exceptions that can be used as a context manager in with statements:

```python
def test_exception(self):
    with self.assertRaises(SomeError):
        method_that_raises_the_exception('some argument to the method')
```

You can define other helper methods in the same test class as long as their name doesn't start with 'test'. Use the
fail method to clarify which assumption or invariant wasn't met.
The _TestCase_ class also provides a `subTest` helper method that enables you to avoid boilerplate by defining multiple
tests within a single test method:

```python
class DataDrivenTestCase(TestCase):
    def test_good(self):
        good_cases = [
            (b'my bytes', 'my bytes'),
            ('no error', b'no error'),  # This one will fail
            ('other str', 'other str'),
            ...
        ]
        for value, expected in good_cases:
            with self.subTest(value):
                self.assertEqual(expected, to_str(value))
```

### Isolate Tests from Each Other with setUp, tearDown, setUpModule, and tearDownModule

Override the `setUp` and `tearDown` methods of a _TestCase_ subclass to have the test environment set up before test
methods run (a.k.a. test harness):

```python
# environment_test.py
from pathlib import Path
from tempfile import TemporaryDirectory
from unittest import TestCase, main


class EnvironmentTest(TestCase):
    def setUp(self):
        self.test_dir = TemporaryDirectory()
        self.test_path = Path(self.test_dir.name)

    def tearDown(self):
        self.test_dir.cleanup()

    def test_modify_file(self):
        with open(self.test_path / 'data.bin', 'w') as f:
            pass


if __name__ == '__main__':
    main()
```

For cases in which the test preparation can't be done for each test due to latency (for example database setup), the
_unittest_ module also supports module-level test harness initialization. To use this define `setUpModule` and
`tearDownModule` functions within the module containing the _TestCase_ classes:

```python
# integration_test.py
from unittest import TestCase, main


def setUpModule():
    print('* Module setup')  # Called once before tests starts


def tearDownModule():
    print('* Module clean-up')  # Called once after tests ends


class IntegrationTest(TestCase):
    def setUp(self):
        print('* Test setup')  # Called before each test

    def tearDown(self):
        print('* Test clean-up')  # called after each test

    def test_end_to_end1(self):
        pass


if __name__ == '__main__':
    main()
```

### Use Mocks to Test Code with Complex Dependencies

Use mocked functions and classes to simulate behaviors when it's too difficult or slow to use the real thing.Python has
the `unittest.mock` built-in module for creating mocks and using them in tests. Using the spec parameter to Mock is
especially useful when mocking classes because it ensures that the code under test doesn't call a misspelled method name
by accident.

```python
from unittest.mock import Mock

mock = Mock(spec=get_friends)

expected = [('Victor', 1), ('David', 2), ('John', 3)]
mock.return_value = expected


def get_friends(some_database, lookup_key):
    pass


database = object()
result = mock(database, 'Patrick')
assert result == expected
```

The _Mock_ class creates a mock function. The return_value attribute of the mock is the value to return when it is
called. The _spec_ argument indicates that the mock should act like the given object (function in this case).
This verifies that the mock responded correctly, to verify that the code called the mock provided with the correct
carguments use `assert_called_once_with` method, which verifies that a single call with exactly the given parameters was
made `mock.assert_called_once_with(database, 'Patrick')`.If I supply the wrong parameters, an exception is raised,
and any _TestCase_ that the assertion is used in fails. I can indicate that any value is okay for an argument by using
the `unittest.mock.ANY` constant:

```python
from unittest.mock import ANY

mock = Mock(spec=get_friends)
# Some setup here
mock.assert_called_with(ANY, 'Patrick')
```

The Mock class also makes it easy to mock exceptions being raised:

```python
class MyError(Exception):
    pass


mock = Mock(spec=get_friends)
mock.side_effect = MyError('Whoops! Big problem')
result = mock(database, 'Patrick')
```

How do I get the codebase to use the mocks inside functions? You can pass keyword argument functions to the method
and inject the mocks there (very verbose), or you can use `unittest.mock.patch`. This family of functions temporarily
reassigns an attribute of a module or class:

```python
from unittest.mock import patch

...
with patch('__main__.get_friends'):
    print(f'Here {get_friends} would refer to the mock version of the method, not the real one')
```

_patch_ works for many modules, classes, and attributes. It can be used in with statements, as a function decorator, or
in the setUp and tearDown methods of _TestCase_ classes. However, _patch_ doesn't work in all cases. Python won't let
me modify classes defined in a C-extension module for example. For this cases, you can create a helper function calling
the intended function or class, and mock this helper function instead or extract this function call into a keyed
argument:

```python
from unittest.mock import DEFAULT  # This value indicates that I want a standard Mock instance created for each name

with patch.multiple('__main__',
                    autospec=True,  # generated mocks will adhere to the specification of the objects simulated
                    get_time=DEFAULT,
                    get_friends=DEFAULT):
    now_func = Mock(spec=datetime.utcnow)
    now_func.return_value = datetime(2019, 6, 5, 15, 45)
    get_time.return_value = timedelta(hours=3)
    get_friends.return_value = [('Victor', 1), ('David', 2), ('John', 3)]
```

### Encapsulate Dependencies to Facilitate Mocking and Testing

One way to improve test readability is to use a wrapper object to encapsulate the underlying object interface
instead of passing a real object that needs to be mocked. For example create a class with the method names of the
database where some info is retrieved, and call that object's function instead.

### Consider Interactive Debugging with pdb

When you need to track specific cases that cause trouble, consider using Python's built-in interactive debugger. The
easiest way to use the debugger is by modifying your program to directly initiate the debugger just before you think
you'll have an issue worth investigating, and call the `breakpoint` function:

```python
def compute_smth(observed, ideal):
    # implementation
    breakpoint()  # Calling python3 <my_file.py> would execute the logic and pause the program here
    # continue implementation
```

At the (Pdb) prompt, you can type in the names of local variables to see their values printed out, display a list of all
local variables by calling the `locals` built-in function, import modules, inspect global state, construct new objects,
run the help built-in function, and even modify parts of the running program. In addition, the debugger has a variety of
special commands to control and understand program execution:

    * where: Print the current execution call stack
    * up: Move your scope up the execution call stack to the caller of the current function
    * down: Move your scope back down the execution call stack one level

You can use these five debugger commands to control the program's execution in different ways:

    * step:  Run until the next line of execution in the program, and then return control back to the debugger
    * next:  Run until the next line of execution in the current function, and then return control back to the debugger
    * return:  Run until the current function returns, and then return control back to the debugger
    * continue: Continue running until the next breakpoint is hit
    * quit: Exit the debugger and end the program

You can also call the debugger prompt by using post-mortem debugging. This enables you to debug a program after it 's
already raised an exception and crashed. You can use the command line `python3 -m pdb -c continue <program path>` to run
the program controlled by the _pdb_ module. The `continue` command tells _pdb_ to get the program started immediately.
When the program hits a problem it automatically enters the interactive debugger and I can inspect the program state.
You can also use post-mortem debugging after hitting an uncaught exception in the interactive Python interpreter by
calling the `pm` function of the _pdb_ module (often in a single line like `import pdb; pdb.pm()`).

### Use tracemalloc to Understand Memory Usage and Leaks

Memory management in the default implementation of Python, CPython, uses reference counting. This ensures that as soon
as all references to an object have expired, the referenced object is also cleared from memory, freeing up that space
for other data. CPython also has a built-in cycle detector to ensure that self-referencing objects are eventually
garbage collected. Most of the time you won't have to care about allocating or deallocating memory, but when you do,
the first way to debug memory usage is to ask the gc built-in module to list every object currently known by the
garbage collector:

```python
import gc

found_objects = gc.get_objects()  # returns a class with information of the objects held in memory

import my_wasteful_program

hold_reference = my_wasteful_program.run()  # run exists in my_wasteful_program and executes the program itself
found_objects = gc.get_objects()
```

The `get_objects` method does not tell you anything about how the objects were allocated. Python 3.4 introduced a new
_tracemalloc_ built-in module for solving this problem. _tracemalloc_ makes it possible to connect an object back to
where it was allocated. You use it by taking before and after snapshots of memory usage and comparing them to see what's
changed.

```python
# top_n.py
import tracemalloc
import my_wasteful_program

tracemalloc.start(10)  # Set stack depth
time1 = tracemalloc.take_snapshot()

x = my_wasteful_program.run()
time2 = tracemalloc.take_snapshot()
stats = time2.compare_to(time1, 'lineno')  # Compare snapshots  

print('Biggest offender is:\n'.join(stats[0].traceback.format()))  # print out the full stack trace of each allocation
```

## Chapter 10: Collaboration<a name="Chapter10"></a>

### Know Where to Find Community-Built Modules

Python has a central [repository of modules](https://pypi.org) that you can install and use in your programs. Using
_pip_ to install a new module is simple: `python3 -m pip install <package name>`. _pip_ is best used together with the
built-in module _venv_ to consistently track sets of packages to install for your projects.

### Use Virtual Environments for Isolated and Reproducible Dependencies

_pip_ installs new packages in a global location. That causes all Python programs on your system to be affected by
these installed modules, which can cause problems with transitive dependencies (check transitive dependencies on
_pip_ with `python3 -m pip show <package_name>`). The solution to this problem is using a tool called _venv_, which
provides virtual environments which allows you to create isolated versions of the Python environment.

#### Using venv on the Command Line

Each virtual environment must live in its own unique directory. Create an environment with `python3 -m venv <project>`.
Activate a virtual environment by navigating to the environment folder and running `bin/activate`, which causes all of
my environment variables to match the virtual environment and updates the command-line prompt to include the virtual
environment name. The virtual environment I created with venv starts with no packages installed except for pip and
_setuptools_. Install packages with the same command shown in the previous point. Use `bin/deactivate` to go back to
the default system.

#### Reproducing Dependencies

To reproduce the development environment in another machine, use `python3 -m pip freeze` to save all of my explicit
package dependencies into a file (by convention named _requirements.txt_). You can install all the dependencies in a
different folder using `python3 -m pip install -r <path to the folder>  <requirements file>`. It's important to note
that the specific version of Python you're using is not included in the _requirements.txt_ file.

### Write Docstrings for Every Function, Class, and Module

Python provides built-in support for attaching documentation to blocks of code, you can add documentation by providing a
docstring immediately after the def statement of a function:

```python
def my_function(self):
    """Attach the documentation for your function after the 3 double quotes"""
    pass
```

You can retrieve the docstring from within the Python program by accessing the function's __doc__ special attribute.
You can also use the built-in pydoc module from the command line to run a local web server that hosts all the Python
documentation that's accessible to your interpreter `python3 -m pydoc -p <port number>`. Docstrings can be attached to
functions, classes, and modules.

#### Documenting Modules

Each module should have a top-level docstring: A string literal that is the first statement in a source file
describing the module purpose. The module docstring is also a jumping-off point where you can highlight important
classes and functions found in the module.

#### Documenting Classes

Each class should have a class-level docstring. The first line is the single-sentence purpose of the class. Paragraphs
that follow discuss important details of the class's operation.Important public attributes and methods of the class
should be highlighted as well.

#### Documenting Functions

Each public function and method should have a docstring. Any return values should be mentioned. Any exceptions that
callers must handle as part of the function's interface should be explained. If a function accepts a variable number
of arguments, use `*args` and `**kwargs` in the documented list of arguments to describe their purpose. You should
also document default values, if it is a generator describe what it yields and in case of an asynchronous coroutine
explain when it will stop execution.

#### Using Docstrings and Type Annotations

Python now supports type annotations for a variety of purposes, making the docstrings redundant in some cases. Omit
the information that's already present in type annotations from docstrings.

### Use Packages to Organize Modules and Provide Stable APIs

Reorganize a project structure and refactor the code might be necessary as the project size grows. You find yourself
with so many modules that you need another layer in your program to make it understandable. Python provides _packages_
for this purpose. Packages are modules that contain other modules. In most cases, packages are defined by putting an
empty file named `__init__.py` into a directory. Once `__init__.py` is present, any other Python files in that directory
will be available for import, using a path relative to the directory. The functionality provided by packages has two
primary purposes in Python programs.

#### Namespaces

Packages help divide your modules into separate namespaces. You can have many modules with the same filename but
different absolute paths that are unique, but this approach breaks when the functions, classes, or submodules
defined in packages have the same names. Use the _as_ clause of the import statement to rename whatever you've imported
for the current scope or access names by their highest unique module name (i.e. use `foo.bar.function(...)`).

#### Stable APIs

Packages provides strict, stable APIs for external consumers. You can provide stable functionality that doesn't change
between releases by hiding your internal code organization from external users. This way, you can refactor and improve
your package's internal modules without breaking existing users. Python can limit the surface area exposed to API
consumers by using the `__all__` special attribute of a module or package. The value of `__all__` is a list of every
name to export from the module as part of its public API. When consuming code executes `from foo import *`, only the
attributes in `foo.__all__` will be imported from `foo`. If `__all__` isn't present in `foo`, then only public
attributes (those without a leading underscore) are imported. You should avoid wildcard imports.

### Consider Module-Scoped Code to Configure Deployment Environments

A deployment environment is a configuration in which a program runs. Every program has at least one deployment
environment: _the production environment_. Production environments often require many external assumptions that
are hard to reproduce in development environments. The best way to work around such issues is to override parts of a
program at startup time to provide different functionality depending on the deployment environment.

### Define a Root Exception to Insulate Callers from APIs

When you're defining a module's API, the exceptions you raise are just as much a part of your interface as the
functions and classes you define. Python has a built-in hierarchy of exceptions for the language and standard library,
but sometimes it is worth to define your own hierarchy of exceptions. I can do this by providing a root _Exception_
in your module and having all other exceptions raised by that module inherit from the root exception. Having a root
exception in a module makes it easy for consumers of an API to catch all the exceptions that were raised deliberately.
Using root exceptions helps find bugs in an API module's code. If my code only deliberately raises exceptions that I
define within my module's hierarchy, then all other types of exceptions raised by my module must be the ones that I
didn't intend to raise. Over time, I might want to expand my API to provide more specific exceptions in certain
situations, I can take API future-proofing further by providing a broader set of exceptions directly below the root
exception.

### Know How to Break Circular Dependencies

Inevitably, while you're collaborating with others, you'll find a mutual interdependence between modules. This can lead
to circular dependencies. When a module is imported, here's what Python actually does, in depth-first order:

    * Searches for a module in locations from sys.path
    * Loads the code from the module and ensures that it compiles 
    * Creates a corresponding empty module object
    * Inserts the module into sys.modules
    * Runs the code in the module object to define its contents

The problem with a circular dependency is that the attributes of a module aren't defined until the code for those
attributes has executed (after step 5). But the module can be loaded with the import statement immediately after it's
inserted into sys.modules (after step 4). There are three ways to break circular dependencies:

#### Reordering Imports

Change the order of imports, if I import the dependant module toward the bottom of my app, after the app module's
other contents have run, the problem is solved. But this goes against the PEP 8 style guide of putting all imports
at the beginning of your file. This approach is not recommended.

#### Import, Configure, Run

Have modules minimize side effects at import time. I can have my modules only define functions, classes, and
constants and avoid actually running any functions at import time. Then, I have each module provide a configure function
that I call once all other modules have finished importing. The purpose of configure is to prepare each module's state
by accessing the attributes of other modules. This works well in many situations and enables patterns like dependency
injection. But sometimes it can be difficult to structure your code so that an explicit configure step is possible.

#### Dynamic Import

Use an import statement within a function or method. This is called a dynamic import because the module import
happens while the program is running, not while the program is first starting up and initializing its modules:

```python
def my_func():
    import other_module  # Dynamic import
    self.prop1 = other_module.prefs.get('prop1')
```

The difference with the previous approach is that it requires no structural changes to the way the modules are defined
and imported. In general, it's good to avoid dynamic imports like this. The cost of the import statement is not
negligible and can be especially bad in tight loops.

### Consider warnings to Refactor and Migrate Usage

When a codebase grows, you might need a way to notify and encourage the people that you collaborate with to refactor
their code and migrate their API usage to the latest forms. Python provides the built-in warnings, which is a
programmatic way to inform other programmers that their code needs to be modified due to a change to an underlying
library that they depend on.

```python
if some_condition is None:
    warnings.warn('Some Warning message', MyCustomWarning)
```

Warning would appear in the system error output. The `warnings.warn` function supports the stacklevel parameter,
which makes it possible to report the correct place in the stack as the cause of the warning. It also lets me configure
what should happen when a warning is encountered. One option is to make all warnings become errors with
`warnings.simplefilter('error')` (useful in test). Use the `-W` error command-line argument to the Python
interpreter or the _PYTHONWARNINGS_ environment variable to apply this policy. It might be useful to redirect
warnings to the logs instead, for example in a production environment:

```python
import logging

fake_stderr = io.StringIO()
handler = logging.StreamHandler(fake_stderr)
formatter = logging.Formatter('%(asctime)-15s WARNING] %(message)s')
handler.setFormatter(formatter)
logging.captureWarnings(True)
logger = logging.getLogger('py.warnings')
logger.addHandler(handler)
logger.setLevel(logging.DEBUG)
warnings.resetwarnings()
warnings.simplefilter('default')
warnings.warn('This will go to the logs output')
```

You can test warnings as well like this:

```python
with warnings.catch_warnings(record=True) as found_warnings:
    found = require('my_arg', None, 'Warning message')
    expected = 'Warning message'
    assert found == expected
```

### Consider Static Analysis via typing to Obviate Bugs

Python has introduced special syntax and the built-in [typing](https://docs.python.org/3.8/library/typing) module,
which allow you to annotate variables, class fields, functions, and methods with type information. _typing_ allows to
run static analysis tools to ingest a program's source code and identify where bugs are most likely to occur. there are
multiple implementations of static analysis tools for Python that use _typing_. The most popular tools are
[mypy](https://github.com/python/mypy), [pytype](https://github.com/google/pytype),
[pyright](https://github.com/microsoft/pyright), and [pyre](https://pyre-check.org). An example of usage using mypy:
`python3 -m mypy --strict example.py`. Parameter and variable type annotations are delineated with a colon. Return value
types are specified with `->` type following the argument list:

```python
def subtract(a: int, b: int) -> int:  # Function annotation
    return a - b
```

Type annotations can also be applied to classes and to ensure the proper collection type:

```python
from typing import Callable, List, TypeVar

Value = TypeVar('Value')
Func = Callable[[Value, Value], Value]  # akind to type function with same type for the 2 input parameters and return 


def combine(func: Func[Value], values: List[Value]) -> Value:
    assert len(values) > 0
    result = values[0]
    for next_value in values[1:]:
        result = func(result, next_value)
    return result


Real = TypeVar('Real', int, float)


def add(x: Real, y: Real) -> Real:
    return x + y


inputs = [1, 2, 3, 4j]  # Oops: included a complex number
result = combine(add, inputs)
assert result == 10
```

The typing module supports _option_ types, which ensure that programs only interact with values after proper null checks
have been performed.

```python
from typing import Optional


def get_or_default(value: Optional[int],
                   default: int) -> int:
    if value is not None:
        return value
    return value  # should have returned "default"
```

Exceptions are not included because Exceptions are not considered part of an interface's definition. _typing_ does
not work particularly well with forward references:

```python
class FirstClass:
    def __init__(self, value: SecondClass) -> None:  # Breaks cause second class hasn't been defined yet
        self.value = value


class SecondClass:
    def __init__(self, value: int) -> None:
        self.value = value
```

One workaround supported by these static analysis tools is to use a string as the type annotation that contains the
forward reference. The string value is later parsed and evaluated to extract the type information to check:

```python
class FirstClass:
    def __init__(self, value: 'SecondClass') -> None:  # OK
        self.value = value
```

A better approach is to use `from __future__ import annotations`, which instructs the Python interpreter to completely
ignore the values supplied in type annotations when the program runs. some of the best practices to keep in mind:

    * It's going to slow you down if you try to use type annotations from the start when writing a new piece of code.
      First write a version without annotations, then write tests, and then add type information where valuable.
    * Type hints are most important at the boundaries of a codebase, such as an API
    * Apply type hints to the most complex and error-prone parts of your codebase that aren't part of an API
    * Include static analysis as part of your automated build and test system
    * Run the type checker as you go to avoid a big backlog of errors 
    * For small prgrams, ad-hoc code, legacy codebases, and prototypes, type hints might not be worth