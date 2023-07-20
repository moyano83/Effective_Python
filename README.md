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

## Chapter 4: Comprehensions and Generators<a name="Chapter4"></a>

## Chapter 5: Classes and Interfaces<a name="Chapter5"></a>

## Chapter 6: Metaclasses and Attributes<a name="Chapter6"></a>

## Chapter 7: Concurrency and Parallelism<a name="Chapter7"></a>

## Chapter 8: Robustness and Performance<a name="Chapter8"></a>

## Chapter 9: Testing and Debugging<a name="Chapter9"></a>

## Chapter 10: Collaboration<a name="Chapter10"></a>