# Python

Python basic notes

Other good notes and cheatsheets I found:

## PYTHON PROJECT STRUCTURE

[Helpful doc](https://realpython.com/python-application-layouts/)
```bash
```bash
python-basic/
├── bin
│   └── some_binary_file
├── data
│   └── input.csv
├── docs
│   └── handbook.md
├── python_basic
│   ├── main
│   │   ├── __init__.py
│   │   └── main.py
│   ├── package_1
│   │   ├── __init__.py
│   │   └── module_A.py
│   ├── package_2
│   │   └── __init__.py
│   ├── __init__.py
│   ├── __main__.py
│   └── runner.py
├── scripts
│   └── some_task_script.py
├── tests
│   ├── functional_tests
│   │   └── __init__.py
│   ├── integration_tests
│   │   ├── __init__.py
│   │   └── integration_test_1.py
│   ├── unit_tests
│   │   ├── main
│   │   │   ├── __init__.py
│   │   │   └── main_tests.py
│   │   ├── package_1
│   │   │   ├── __init__.py
│   │   │   └── module_A_tests.py
│   │   ├── package_2
│   │   │   └── __init__.py
│   │   └── __init__.py
│   ├── __init__.py
│   ├── README.md
│   └── runtests.py
├── exterior_runner.py
├── LICENSE
├── README.md
├── requirements.txt
└── setup.py
```

## CLASSES - METHODS AND ATTRIBUTES

```python
class MyClass:
    class_variable_1 = ''
    class_variable_2 = ''

    def __init__(self, var1, var2):
        self.instance_variable_1 = var1
        self.instance_variable_2 = var2

    def __str__(self):
        return str(self.instance_variable_1)

    @classmethod
    def public_class_method(cls):
        #accessing class variable
        cls.class_variable_1
        #accessing class method
        cls.public_class_method()

    @staticmethod
    def public_static_mehotd():
        #No class varriable or method should be accessed from a static mehod
        #But if it is needed. you can:
        #accessing class variable
        MyClass.class_variable_1
         #accessing class method
        MyClass.public_class_method()

    def public_instance_method(self):
        # accessing class variable to read/write (best option)
        MyClass.class_variable_1
        
        # accessing class variable to read/write (other options)
        type(self).class_variable_1
        self.__class__.class_variable_1

        # accessing ONLY for reading (other option)
        self.class_variable_1

        # Doing this, class_variable wiil be overrided, 
        # by a new instance variable with the same name
        self.class_variable_1 = 1

        # accesssing instance variable
        self.instance_variable_1

        # calling class or static method
        MyClass.public_class_method()
        self.public_class_method()
        self.__class__.public_class_method()

        # calling instance method
        self._private_instance_mehod()
    
    def _private_instance_method(self):
        self.instance_variable_1
```

## METHODS ORDER IN CLASSES

Methods order:
1. `__init__()`
2. public methods
3. private methods
4. magic methods

```python
class SomeClass(object):

    def __init__(self):
        "__init__() always the first"

    ########################
    #    Public Methods    #
    ########################
    def public_method(self):
        "public methods, in order of importance"

    ########################
    #   Private Methods    #
    ########################
    def _private_method(self):
        "then private methods, grouped by importance or related function"

    ########################
    #     Magic Methods    #
    ########################
    def __magic_method__(self):
        "magic methods last"

```


## DECORATORS

Decorator template to build a custom decorator:
```python
import functools

def my_decorator(func):
    @functools.wraps(func)
    def wrapper_decorator(*args, **kwargs):
        # Do something before
        value = func(*args, **kwargs)
        # Do something after
        return value
    return wrapper_decorator
```

You will use this decorator in this way:
```python
@my_decorator
def my_function():
    pass
```

If you apply multiple decorators, the decorators will be executed in the order they are listed.
```python
@decor1
@decor2
@decor3
def my_function():
    pass

# This is the same
def my_function():
    pass

my_function = decor1(decor2(decor3(my_function)))
```


## GENERATORS AND ITERATORS

A generator is a function that returns a iterable object of the class 'generator' (every function that uses `yield` returns a iterable object, not a variable). Iterator is a class that implements the two magic methods required to be iterable:`__iter__()` and `__next__()`

```python
# GENERATOR: function that uses yield
def generator_example(n):
    for x in range(11):
        yield x**n


# ITERATOR: class that implements __next__ and __iter__
class IteratorExample:
    def __init__(self, n):
        self.n = n

    def __iter__(self):
        self.a = -1
        return self

    def __next__(self):
        self.a += 1
        if self.a > 10:
            raise StopIteration
        return self.a**self.n


# USES
a = generator_exapmle(2)
for x in a:
    print(x)
# Output: 0 1 4 9 16 25 36 49 64 81 100

b = IteratorExample(2)
for x in b:
    print(x)
# Output: 0 1 4 9 16 25 36 49 64 81 100

b = IteratorExample(2)
iterator = iter(b)
print(next(iterator))
print(next(iterator))
# Output: 0  1

a = generator_exapmle(2)
print(list(a))
# Output: [0, 1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
```

## DEBUGGING
Build-in function: `breakpoint()`

`python3 -m pdb app.py arg1 arg2`

* n: Continue execution until the next line in the current function is reached or it returns.
* s: Execute the current line and stop at the first possible occasion (either in a function that is called or in the current function).
* c: Continue execution and only stop when a breakpoint is encountered.
* h: See a list of all available commands.

## KEYWORDS

### with

```python 
with <context manager> as <var>:
    <statements>
```

`with` statement can be used in exception handling to make the code cleaner and much more readable. `Try-except` is more common though.

```python
# file handling
 
# 1) without handling exceptions
file = open('file_path', 'w')
file.write('hello world !')
file.close()
 
# 2) without using with statement
file = open('file_path', 'w')
try:
    file.write('hello world')
finally:
    file.close()

# 3) using with statement
with open('file_path', 'w') as file:
    file.write('hello world !')
```

To use `with` statement in user defined objects you only need to add the methods `__enter__()` and `__exit__()` in the object methods:

```python
# a simple file writer object
 
class MessageWriter(object):
    def __init__(self, file_name):
        self.file_name = file_name
     
    def __enter__(self):
        self.file = open(self.file_name, 'w')
        return self.file
 
    def __exit__(self, *args):
        self.file.close()
 
# using with statement with MessageWriter
 
with MessageWriter('my_file.txt') as xfile:
    xfile.write('hello world')
```

`with` is normally used in this 2 contexts (not common in other uses):
1. When working with unmanaged resources (like file streams, database CRUD processes, etc)
2. When testing with 'pytest' library, to ckeck if a exception was raised

```python
# 1  resources (sqlite db):
def get_all_songs():
    with sqlite3.connect('db/songs.db') as connection:
        cursor = connection.cursor()
        cursor.execute("SELECT * FROM songs")
        all_songs = cursor.fetchall()
        return all_songs

# 2 pytest
def test_1():
    with pytest.raises(ValueError):
        object.method_1('Value that generates an ValueError exception)
```


### in, not in

1- Checking if an element is in a sequence
```python
>>> 'g' in 'ghost'
True
>>> 'G' in 'ghost'
False
>>> 10 in [10000,1000,100,10]
True
>>> dict1={1:'apple',2:'banana',3:'pineapple'}
>>> 3 in dict1
True
>>> 'Banana' in ['Apple', 'Banana','Pineapple']
True
>>> 'Strawberry' in ['Apple', 'Banana','Pineapple']
False
>>> 'Strawberry' not in ['Apple', 'Banana','Pineapple']
True
```

2- Iterating through a sequence in a for loop
```python
>>> list1 = [1, 2, 3, 4, 5]
>>> for item in list1:
>>>     print(item)
1 2 3 4 5
```


### is, is not

 * `is` evaluates to True if the variables on either side of the operator point to the same object and false otherwise. Used to check `True`, `False` or `None`
 * `is not` evaluates `True` if both variables are not the same object.

```python
>>> a = [1, 2, 3]
>>> b = [1, 2, 3]
>>> a is b
False
>>> a is not b
True
>>> a == b
True
>>> a is False
False
>>> a is True
False
>>> bool(a) is True
True
>>> a is None
False
```

### is, is not, ==, !=

`is` is used in Python for testing object identity. While the `==` operator is used to test if two variables are equal or not, `is` is used to test if the two variables refer to the same object.

```python 
>>> [] == []
True
>>> [] is []
False
>>> [] is  not []
True
>>> {} == {}
True
>>> {} is {}
False
>>> {} is not{}
True

>>> a = 55555
>>> b = 55555
>>> a is b
False
>>> a == b
True

>>> a = (2, 3, 'a')
>>> b = (2, 3, 'a')
>>> a is b
False
>>> a == b
True

>>> class A:
...     def __init__(self, x):
...             self.x = x
...     def __eq__(self, other):
...             return self.x == other.x
... 
>>> obj1 = A(3)
>>> obj2 = A(3)
>>> obj1 is obj2
False
>>> obj1 is not obj2
True
>>> obj1 == obj2
True
>>> obj1 != obj2
False
```

Use `is` and `is not` with `True`, `False` and `None` to avoid hidden mistakes:

```python
>>> d = True
>>> d is True
True
>>> d = 1
>>> d is True
False
>>> d == True
True
>>> d = 2
>>> d is True
False
>>> d == True
False
>>> d = None
>>> d is None
True
```

## CONTROL FLOW

```python
if <expr1>:
    <statements>
elif <expr2>:
    <statements>
elif <expr3>:
    <statements>
else:
    <statements>


for <element> in <container>:
    <statements> | break | continue
else:
    <statements>  #if break not called


while <expr>:
    <statements> | break | continue
else:
    <statements> #if break not called


try:
    <statements>
except <exception1>:
    <statements>
except <exception2>:
    <statements>
else:
    <statements> #if there is no exception
finally:
    <statements> #always executed

```

Examples:

```python
if a < 0:
    pass
elif a = 0:
    pass
elif a > 0 and a < 5:
    pass
else:
    pass


for x in range(0, 10, 2):
    if x == 4:
        continue
    if x > 6:
        break
    print(x)
else:
    print('break was NOT called to exit the loop early')
# Output: 0, 2, 6


while True:
    x = input()
    if x = 2:
        break
else:
    print('break was NOT called')


try:
    print('Here we try statements that can raise exceptions')
except Error1:
    print('Error1 raised')
else:
    print('There was no raised exception')
finally:
    print('This is always executed')
```


## BUILT-IN DATA STRUCTURES

### list, str, tuple

Duplicated elements, indexed elements

* list: mutable array
* tuple: immutable array
* str: immutable array of characters


| **list**                      | **str**           | **tuple** |
|-------------------------------|-------------------|-----------|
| append(elem)                  |                   |           |
| clear()                       |                   |           |
| copy()->list                  |                   |           |
| count(elem)->int              | count()           | count()   |
| extend(iterable)              |                   |           |
| index(e, [start], [end])->int | index() or find() | index()   |
| insert(index, elem)           |                   |           |
| pop([index])                  |                   |           |
| remove(elem)                  |                   |           |
| reverse()                     |                   |           |
| sort([reverse], [key])        |                   |           |



### set

mutable, not-duplicated elements, not-indexed elements

| **set**                              | **clarification**         |
|--------------------------------------|---------------------------|
| add(elem)                            |                           |
| clear()                              |                           |
| copy()-> set                         |                           |
| pop()-> elem                         | *remove random element    |
| remove(elem) or discard(elem)        | *remove() raises an error |
| update(iterable)                     |                           |
|                                      |                           |
| difference(set)-> set                | A - B                     |
| difference_update(set)               | A = A - B                 |
| intersection(set, [set], ...)-> set  | A ∩ B    ∩ C...           |
| intersection_update(set, [set], ...) | A = A ∩ B    ∩ C...       |
| isdisjoint(set)                      | A ∩ B == ∅ ?              |
| issubset(set)                        | A ⊆ B ?                   |
| issuperset(set)                      | A ⊇ B ?                   |
| symetric_difference()                | (A ∪ B) - (A ∩ B)         |
| symetric_difference_update()         | A = (A ∪ B) - (A ∩ B)     |
| union(set, [set], ...)-> set         | A ∪ B    ∪ C...           |



### dict

mutable, not-duplicated keys, indexed elements (insertion order)


| **dict**                         | **clarification**                   |
|----------------------------------|-------------------------------------|
| clear()                          |                                     |
| copy()->dict                     |                                     |
| fromkeys(keys:set,[value])->dict | *create new dict                    |
| get(key, [default])-> value      | *if key not exits, returns default  |
| items()-> list(tuple(key,value)) |                                     |
| keys()-> list                    |                                     |
| pop(key, [default])-> value      | *if key not exits, returns default  |
| popitem()-> tuple(key, value)    | *removes the last (>=python3.7)     |
| setdefault(key,[default])->value | *if key not exists, inserts default |
| update(dict or iterable)         | *iterable with key-value pairs      |
| values()-> list                  |                                     |


### str

| **str**                        | **clarification**            |
|--------------------------------|------------------------------|
| capitalize()-> str             | "apple" -> "Apple"           |
| casefold()-> str               | "AÑЖß"->"añжss" (+agressive) |
| lower()-> str                  | "AVÑЖß" -> "avñжß"           |
| format(val1, val2, ...)-> str  | "a {val1} b" -> "a value b"  |
| join(iterable)-> str           | "-".["a","b","c"] -> "a-b-c" |
| replace(old,new,[count])-> str | "TaaZ".("aa","BB") -> "TBBZ" |
| split([sep],[max])-> list      | "aa bb d" -> ["aa","bb","d"] |
| strip([str])-> str             | "  aa bb   " -> "aa bb"      |
| translate(dict)-> str          | replaces chars(+maketrans()) |
| maketrans(st1,st2,[st3])->dict | *to use with translate()     |
| upper()-> str                  | "avñжh"-> "AVÑЖH"            |
| isalnum(),isalpha(),isascii(), | "adf876".isalnum() -> True   |



### format() and f-strings

`format()` method:
```python
greeting = "Hello {name}! Welcome to {place}."
print(greeting.format(name="Trey", place="Mongolia"))

greeting = "Hello {0}! Welcome to {1}."
print(greeting.format("Trey", "Mongolia"))

greeting = "Hello {}! Welcome to {}."
print(greeting.format("Trey", "Mongolia"))
```

`f-strings` is an evolution of `str.format()` (From python3.6):
```python
# format() method
greeting = "Hello {name}! Welcome to {place}."
print(greeting.format(name="Trey", place="Mongolia"))

# f-string
name = "Trey"
place = "Mongolia"
print(f"Hello {name}! Welcome to {place}.")
```



## COMPREHENSION (LIST and others)

* List comprehension
* Dictionary comprehension
* Set comprehension
* Generator comprehension


`new_list = [expression for item in iterable (if conditional)]`


```python
# List comprehension

sentence = "I work as a waiter in city center"
vowels = [i for i in sentence if i in 'aeiou']
>>> ['o', 'a', 'a', 'a', 'i', 'e', 'i', 'i', 'e', 'e']

fruits = ["banana", "orange", "apple", "banana"]
newlist = [x if x != "banana" else "orange" for x in fruits]
>>> ['orange', 'orange', 'apple', 'orange']

num_list = [y for y in range(100) if y % 2 == 0 if y % 5 == 0]
>>> [0, 10, 20, 30, 40, 50, 60, 70, 80, 90]

# Dictionary comprehension
d = {num: num**2 for num in range(1, 7)}
>>> {1: 1, 2: 4, 3: 9, 4: 16, 5: 25, 6: 36}

# Set comprehension
vlans = [10, '30', 30, 10, '56']
unique_vlans = {int(vlan) for vlan in vlans}
>>> {56, 10, 30}

# Generator comprehension
generator = (i*2 for i in range(4) if i%2 == 0)
print(list(generator))
>>> [0, 4, 8, 12, 16]
```

## LAMBDA FUNCTIONS

Lambda function = Anonymous function

Syntax: `lambda p1, p2: expression`

Examples:
```python
a = lambda x: x + 1
>>> a(5) -> 6

a = lambda x, y, z: x + y + z
>>> a(1, 2, 3) -> 6

a = lambda a, b : a if(a > b) else b
>>> a(3, 8) -> 8

a = lambda x: x * 10 if x > 10 else (x * 5 if x < 5 else x)
>>> a(3) -> 15
>>> a(8) -> 8
>>> a(12) -> 120
```

Normal uses:
* In built-in functions that have a function as a parameter: `map()`, `filter()`, `sorted()`, `reduce()`, `max()` and `min()`
* With `sort()` method, within lists,
* (not usual) In any other function that has a function as a parameter, when we don't need to define a normal function to be called in other parts of the code and if that function is not complex or can be written in one line.
* With UI frameworks as Tkinter

In any other case, you should use normal functions.


## BUILT-IN FUNCTIONS FOR ITERABLES

map, filter, sorted, *reduce, max, min, sum, any, all


`map(function, iterable, [*iterables]) -> iterator`:

map() applies the function on each element of the iterable.


`filter(function, iterable) -> iterator`: 

`filter()` calls the function on each element of the iterable, then returns only the elements whose function's result was `True`. (function must return a boolean)


`sorted(iterable, [key:function], [reverse]) -> list`:

`sorted()` sorts iterable's elements. If `key` function is given, it applies that function to every element before starting to order. `key` is used to specify any detailed sorting criteria, and must have just one parameter.


`functools.reduce(function, iterable, [initializer]) ->  value`
* `reduce()` is not buit-in, we need to import `functools`
* `function` must accept two parameters
* performs the following steps:
  1. Apply a function (or callable) to the first two items in an iterable and generate a partial result.
  2. Use that partial result, together with the third item in the iterable, to generate another partial result.
  3. Repeat the process until the iterable is exhausted and then return a single cumulative value.

So `reduce(lambda x,y: x*y, list1)` == `(((list1[0]*list1[1])*list1[2])*list1[3])`


`max(iterable, [key:function], [default]) -> elem`

`max()` returns max value. If key if given, applies it to every element


`min(iterable, [key:function], [default]) -> elem`

`min()` returns min value. If key if given, applies it to every element


`sum(iterable, [start]) -> int`

`min()` returns sum of all values. If start if given, it will be the initial sum


`all(iterable) -> bool`

`True` if all elements are `True`. If iterable is empty, returns `True`


`any(iterable) -> bool`

`True` if any of the elements is `True`, `False` otherwise. If iterable is empty, returns `False`