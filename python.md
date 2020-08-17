<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Setup](#setup)
- [Running Jupyter notebooks](#running-jupyter-notebooks)
- [Variables and Assignment](#variables-and-assignment)
- [Data Types and Type Conversion](#data-types-and-type-conversion)
- [Built-in Functions and Help](#built-in-functions-and-help)
- [Conditionals](#conditionals)
- [Lists](#lists)
- [For Loops](#for-loops)
- [While loops](#while-loops)
- [More on lists](#more-on-lists)
- [Advanced topic: list comprehensions](#advanced-topic-list-comprehensions)
- [Advanced topic: dictionaries](#advanced-topic-dictionaries)
- [Writing functions](#writing-functions)
- [Variable scope](#variable-scope)
- [If we have time](#if-we-have-time)
- [Libraries](#libraries)
- [Numpy](#numpy)
  - [Working with mathematical arrays in numpy](#working-with-mathematical-arrays-in-numpy)
  - [Indexing, slicing, and reshaping](#indexing-slicing-and-reshaping)
  - [Vectorized functions on array elements](#vectorized-functions-on-array-elements)
  - [Aggregate functions](#aggregate-functions)
  - [Boolean indexing](#boolean-indexing)
  - [More numpy functionality](#more-numpy-functionality)
  - [External packages built on top of numpy](#external-packages-built-on-top-of-numpy)
- [Pandas dataframes](#pandas-dataframes)
  - [Reading tabular data into dataframes](#reading-tabular-data-into-dataframes)
  - [Subsetting](#subsetting)
  - [Looping over data sets](#looping-over-data-sets)
- [Advanced topic: running Python scripts from the command line](#advanced-topic-running-python-scripts-from-the-command-line)
  - [Very advanced topic: adding standard input support to our scripts](#very-advanced-topic-adding-standard-input-support-to-our-scripts)
- [Plotting](#plotting)
  - [Simple line/scatter plots of gapminder data](#simple-linescatter-plots-of-gapminder-data)
  - [Bar plots](#bar-plots)
  - [Heatmaps](#heatmaps)
  - [Geographical scatterplot](#geographical-scatterplot)
  - [3D topographic elevation](#3d-topographic-elevation)
  - [3D parametric plot](#3d-parametric-plot)
  - [3D scatter plot](#3d-scatter-plot)
  - [3D graph](#3d-graph)
- [Programming Style and Wrap-Up](#programming-style-and-wrap-up)
- [Other advanced Python topics](#other-advanced-python-topics)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

**Course Description**: This six-hour introduction will walk you through the basics of programming in Python. We will
cover the main language features -- variables and data types, conditionals, lists, for/while loops, list comprehensions,
dictionaries, writing functions -- as well as working with external libraries such as pandas (dataframes), numpy
(mathematical arrays), and plotly (basic plotting).

# Setup

These notes started from the official SWC lesson http://swcarpentry.github.io/python-novice-gapminder, but then evolved
quite a bit to include several other topics. You can find these notes at http://bit.ly/pythonmd.

instructor                                    | students
--------------------------------------------- | ----------------------------------------
(1) log in to socrative.com as a teacher      | (1) log in to socrative.com as a student
(2) start a quiz, select 'python quiz 1 or 2' | (2) enter provided room name
(3) select 'teacher paced'                    |
(4) disable student names                     |
(5) start                                     |

# Running Jupyter notebooks

Python pros                                 | Python cons
--------------------------------------------|------------------------
elegant scripting language                  | slow (interpreted language)
powerful, compact constructs for many tasks |
very popular across all fields              |
huge number of external libraries           |

Python programs are plain text files, stored with the .py extension.

Many ways to run Python commands:

* from the inside a Unix shell can start a Python shell and type commands
* running Python scripts saved in *.py files
* from Jupyter (formerly iPython) notebooks - stored as JSON files, displayed as HTML

Today we will use a Jupyter notebook. You have several options:

1. if you have a university computer ID, go to https://syzygy.ca and under Launch select your institution, then fill in your credentials
1. if you have a Google account, go to https://syzygy.ca and under Launch select either Cybera or PIMS, then log in
1. if you have a GitHub account, go to https://westgrid.syzygy.ca
1. if you have Python+Jupyter installed locally on your machine, then you can start it locally from your shell by typing
  `jupyter notebook`; you will need the following Python packages installed on your computer: numpy, networkx, pandas,
  plotly, skimage

This will open a browser page pointing to the Jupyter server (remote except for the last option). Click on New ->
Python 3. Explain: tab completion, annotating code, displaying figures inside the notebook.

* Esc - leave the cell (border becomes blue) to the control mode
* "A" - insert a cell above the current cell
* "B" - insert a cell below the current cell
* "X" - delete the current cell
* "M" - turn the current cell into the markdown cell
* "H" - to display help
* Enter - re-enter the cell (border becomes green) from the control mode
* can enter Latex equations in a markdown cell, e.g. $int_0^\infty f(x)dx$

~~~ {.python}
print(1/2)   # to run all commands in the cell, either use the Run button, or press shift+return
~~~

# Variables and Assignment

* possible names for variables
* don't use built-in function names for variables, e.g. declaring *sum* won't let you use sum(), same for
  *print*
* Python is case-sensitive

~~~ {.python}
age = 100
firstName = 'Jason'
print(firstName, 'is', age, 'years old')
a = 1; b = 2    # can use ; to separate multiple commands in one line
a, b = 1, 2   # assign variables in a tuple notation; same as last line
a = b = 10    #  assign a value to multiple variables at the same time
b = "now I am a string"    # variables can change their type on the fly
~~~

* variables persist between cells
* variables must be defined before use
* variables can be used in calculations

~~~ {.python}
age = age + 3   # another syntax: age += 3
print('age in three years:', age)
~~~

**Quiz 1:** predicting values

Use square brackets to get a substring:
~~~ {.python}
element = 'helium'
print(element[0])   # single character
print(element[0:3])   # a substring
~~~

**Quiz 2:** getting the second digit of a number (not a string!)

* python is case-sensitive
* use meaningful variable names

# Data Types and Type Conversion

~~~ {.python}
print(type(52))
print(type(52.))
print(type('52'))
~~~

~~~ {.python}
print(name+' Smith')   # can add strings
print(name*10)         # can replicate strings by mutliplying by a number
print(len(name))       # strings have lengths
~~~

~~~ {.python}
print(1+'a')           # cannot add strings and numbers
print(str(1)+'a')   # this works
print(1+int('2'))   # this works
~~~

# Built-in Functions and Help

* Python comes with a number of built-in functions
* a function may take zero or more arguments

~~~ {.python}
print('hello')
print()
~~~

~~~ {.python}
print(max(1,2,3,10))
print(min(5,2,10))
print(min('a', 'A', '0'))   # works with characters, the order is (0-9, A-Z, a-z)
print(max(1, 'a'))    # can't compare these
round(3.712)      # to the nearest integer
round(3.712, 1)   # can specify the number of decimal places
help(round)
round?   # Jupyter Notebook's additional syntax
~~~

* every function returns something, whether it is a variable or None
~~~ {.python}
result = print('example')
print('result of print is', result)   # what happened here? Answer: print returns None
~~~

# Conditionals

Python implements conditionals via *if*, *elif* (short for "else if") and *else*. Use an *if* statement to control
whether some block of code is executed or not.

~~~ {.python}
mass = 3.54
if mass > 3.0:
    print(mass, 'is large')
~~~

Let's modify the mass:

~~~ {.python}
mass = 2.07
if mass > 3.0:
    print (mass, 'is large')
~~~

Add an *else* statement:

~~~ {.python}
mass = 2.07
if mass > 3.0:
    print(mass, 'is large')
else:
    print(mass, 'is small')
~~~

Add an *elif* statement:

~~~ {.python}
x = 5
if x > 0:
    print(x, 'is positive')
elif x < 0:
    print(x, 'is negative')
else:
    print(x, 'is zero')
~~~

What is the problem with the following code?

~~~ {.python}
grade = 85
if grade >= 70:
    print('grade is C')
elif grade >= 80:
    print('grade is B')
elif grade >= 90:
    print('grade is A')
~~~

# Lists

A list stores many values in a single structure.

~~~ {.python}
T = [27.3, 27.5, 27.7, 27.5, 27.6]   # array of temperature measurements
print('temperature:', T)
print('length:', len(T))
~~~

~~~ {.python}
print('zeroth item of T is', T[0])
print('fourth item of T is', T[4])
~~~

~~~ {.python}
T[0] = 21.3
print('temperature is now:', T)
~~~

~~~ {.python}
primes = [2, 3, 5]
print('primes is initially', primes)
primes.append(7)   # append at the end
primes.append(11)
print('primes has become', primes)
~~~

~~~ {.python}
print('primes before', primes)
del primes[4]   # remove element #4
print('primes after', primes)
~~~

~~~ {.python}
a = []   # start with an empty list
a.append('Vancouver')
a.append('Toronto')
a.append('Kelowna')
print(a)
~~~

~~~ {.python}
a[99]   # will give an error message (past the end of the array)
a[-1]   # display the last element; what's the other way?
a[:]    # will display all elements
a[1:]   # starting from #1
a[:1]   # ending with but not including #1
~~~

Lists can be heterogeneous and nested:

~~~ {.python}
a = [11, 21, 31]
b = ['Mercury', 'Venus', 'Earth']
c = 'hello'
nestedList = [a, b, c]
print(nestedList)
~~~

You can search inside a list:

~~~ {.python}
'Venus' in b      # returns True
'Mars' in b       # returns False
b.index('Venus')      # returns 1 (position index)
~~~

And you sort lists alphabetically:

~~~ {.python}
b.sort()
b             # returns ['Earth', 'Mercury', 'Venus']
~~~

To delete an item from a list:

~~~ {.python}
b.pop(2)             # you can use its index
b.remove('Earth')       # or you can use its value
~~~

**Exercise:** write a script to find the second largest number in the list [77,9,23,67,73,21].

<!-- **Solution:** -->
<!-- ~~~ {.python} -->
<!-- a = [77, 9, 23, 67, 73, 21] -->
<!-- a.sort(reverse=True) -->
<!-- a[1]         # should print 73 -->
<!-- ~~~ -->

# For Loops

*For* loops are very common in Python and are similar to *for* in other languages, but one nice twist with Python is
that you can iterate over any collection, e.g., a list, a character string, etc.

~~~ {.python}
for number in [2, 3, 5]:    # number is the loop variable; [...] is a collection
    print(number)          # Python uses indentation to show the body of the loop
~~~

This is equivalent to:
~~~ {.python}
print(2)
print(3)
print(5)
~~~

What will this do:
~~~ {.python}
for number in [2, 3, 5]:
    print(number)
print(number)
~~~

* the loop variable could be called anything
* the body of a loop can contain many statements
* use range to iterate over a sequence of numbers

~~~ {.python}
for i in 'hello':
    print(i)
~~~

~~~ {.python}
for i in range(0,3):
    print(i)
~~~
	
Let's sum numbers 1 to 10:

~~~ {.python}
total = 0
for number in range(10):
    total = total + (number + 1)   # what's the other way to sum numbers 1 to 10? how about range(1,11)?
print(total)
~~~

**Quiz 3:** revert a string

<!-- **Solution 1:** -->
<!-- ~~~ {.python} -->
<!-- n = '' -->
<!-- for i in 'computer': -->
<!--     n = i + n -->
<!-- print(n) -->
<!-- ~~~ -->

<!-- **Solution 2:** -->
<!-- ~~~ {.python} -->
<!-- a = list('computer') -->
<!-- a.reverse() -->
<!-- ''.join(a)      # convert the list to a string -->
<!-- help(''.join)   # concatenate all strings in the iterable with the separator from the original string -->
<!-- ~~~ -->

<!-- **Solution 3:** -->
<!-- ~~~ {.python} -->
<!-- 'computer'[::-1] -->
<!-- ~~~ -->

**Exercise:** Print a difference between two lists, e.g., [1, 2, 3, 4] and [1, 2, 5].

<!-- **Solution 1:** -->
<!-- ~~~ {.python} -->
<!-- a1 = [1, 2, 3, 4] -->
<!-- a2 = [1, 2, 5] -->
<!-- for i in a1: -->
<!--     if i not in a2: -->
<!--         print(i) -->
<!-- for i in a2: -->
<!--     if i not in a1: -->
<!--         print(i) -->
<!-- ~~~ -->

**Exercise:** write a script to get the frequency of the elements in a list. You are allowed to google this problem :)

<!-- **Solution 1:** -->
<!-- ~~~ {.python} -->
<!-- a = [77, 9, 23, 67, 73, 21, 23, 9] -->
<!-- a.count(77)        # prints 1 -->
<!-- a.count(9)         # prints 2 -->
<!-- for i in a: -->
<!--     a.count(i)    # counts the frequency of 'i' in list 'a' -->
<!-- ~~~ -->

<!-- **Solution 2:** -->
<!-- ~~~ {.python} -->
<!-- a = [77, 9, 23, 67, 73, 21, 23, 9] -->
<!-- import collections -->
<!-- print(collections.Counter(a)) -->
<!-- ~~~ -->

# While loops

Since we talk about loops, we can also briefly mention *while* loops, e.g.

~~~ {.python}
x = 2
while x > 1.:
    x /= 1.1
    print(x)
~~~

# More on lists

You can also form a *zip* object of tuples from two lists of the same length:

~~~ {.python}
for i, j in zip(a,b):
    print(i,j)
~~~

And you can create an *enumerate* object from a list:

~~~ {.python}
for i, j in enumerate(b):    # creates a list of tuples with an iterator as the first element
    print(i,j)
~~~

<!-- **Exercise:** Write a script to sort a list in increasing order by the last element in each tuple, e.g., -->
<!-- input = [(2, 5), (1, 2), (4, 4), (2, 3), (2, 1)] should result in -->
<!-- [(2, 1), (1, 2), (2, 3), (4, 4), (2, 5)]. -->

# Advanced topic: list comprehensions

It's a compact way to create new lists based on existing lists/collections. Let's list squares of numbers
from 1 to 10:

~~~ {.python}
[x**2 for x in range(1,11)]
~~~

Of these, list only odd squares:

~~~ {.python}
[x**2 for x in range(1,11) if x%2==1]
~~~

You can also use list comprehensions to combine information from two or more lists:

~~~ {.python}
week = ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']
weekend = ['Sat', 'Sun']
print([day for day in week])                         # the entire week
print([day for day in week if day not in weekend])   # only the weekdays
print([day for day in week if day in weekend])       # in both lists
~~~

The syntax is:

~~~ {.python}
[something(i) for i in list1 if i [not] in list2 if i [not] in list3 ...]
~~~

**Quiz 4:** sum up squares of numbers

<!-- **Solution**: one possible answer is sum([x**2 for x in range(1,101)]) -->

**Exercise:** Write a script to build a list of words that are shorter than *n* from a given list of words
['red', 'green', 'white', 'black', 'pink', 'yellow'].

<!-- **Solution**: one possible answer is -->
<!-- ~~~ {.python} -->
<!-- input = ['red', 'green', 'white', 'black', 'pink', 'yellow'] -->
<!-- n = 5 -->
<!-- [x for x in input if len(x) < n] -->
<!-- ~~~ -->

# Advanced topic: dictionaries

**Lists** in Python are ordered sets of objects that you access via their position/index. **Dictionaries** are unordered
sets in which the objects are accessed via their keys. In other words, dictionaries are unordered key-value pairs.

~~~ {.python}
favs = {'mary': 'orange', 'john': 'green', 'eric': 'blue'}
favs
favs['john']      # returns 'green'
favs['mary']      # returns 'orange'
list(favs.values()).index('blue')     # will return the index of the first value 'blue'
~~~

~~~ {.python}
for key in favs:
    print(key)          # will print the names	
for k in favs.keys():
    print(k)            # the same
for key in favs:
    print(favs[key])      # will print the colours
for v in favs.values():
    print(v)                   # the same
for i, j in favs.items():
    print(i,j)           # both the names and the colours
~~~
	
Now let's see how to add items to a dictionary:

~~~ {.python}
concepts = {}
concepts['list'] = 'an ordered collection of values'
concepts['dictionary'] = 'a collection of key-value pairs'
concepts
~~~

Let's modify values:

~~~ {.python}
concepts['list'] = 'simple: ' + concepts['list']
concepts['dictionary'] = 'complex: ' + concepts['dictionary']
concepts
~~~

Deleting dictionary items:

~~~ {.python}
del concepts['list']       # remove the key 'list' and its value
~~~

Values can also be numerical:

~~~ {.python}
grades = {}
grades['mary'] = 5
grades['john'] = 4.5
grades
~~~

And so can be the keys:

~~~ {.python}
grades[1] = 2
grades
~~~

Sorting dictionary items:

~~~ {.python}
favs = {'mary': 'orange', 'john': 'green', 'eric': 'blue', 'jane': 'orange'}
sorted(favs)             # returns the sorted list of keys
sorted(favs.keys())      # the same
sorted(favs.values())         # returns the sorted list of values
for k in sorted(favs):
    print(k, favs[k])          # full dictionary sorted by the key
~~~
	
**Exercise:** Write a script to print the full dictionary sorted by the value.

<!-- **Solution**: many very advanced solutions, but one possible simple solution is -->
<!-- ~~~ {.python} -->
<!-- sorted((v,k) for (k,v) in favs.items())      # works as sorted() acts on the first item in each tuple in the list -->
<!-- ~~~ -->

# Writing functions

* functions encapsulate complexity so that we can treat it as a single thing
* functions enable re-use: write one time, use many times

First define:

~~~ {.python}
def greeting():
    print('Hello!')
~~~

and then we can run it:

~~~ {.python}
greeting()
~~~

~~~ {.python}
def printDate(year, month, day):
    joined = str(year) + '/' + str(month) + '/' + str(day)
    print(joined)
printDate(1871, 3, 19)
~~~

Every function returns something, even if it's None.

~~~ {.python}
a = printDate(1871, 3, 19)
print(a)
~~~

How do we actually return a value from a function?

~~~ {.python}
def average(values):   # the argument is a list
    if len(values) == 0:
        return None
    return sum(values) / len(values)
a = average([1, 3, 4])
print('average of actual values:', a)
~~~

**Quiz 5:** convert from Fahrenheit to Celsius

**Quiz 6:** convert from Celsius to Fahrenheit

**Quiz 7:** convert temperature lists

<!-- **Solution:** -->
<!-- ~~~ {.python} -->
<!-- def celsius(fs): -->
<!--     c = [] -->
<!--     for f in fs: -->
<!--         c.append((f-32.)*5./9.) -->
<!--     return c -->
<!-- ~~~ -->
	
Function arguments in Python can take default values becoming optional:

~~~ {.python}
def addNumber(a, b=1):
    return a+b
print(addNumber(5))
print(addNumber(5,3))
~~~

With several optional arguments it is important to be able to differentiate them:

~~~ {.python}
def modify(a, b=1, coef=1):
    return a*coef + b
print(modify(10))
print(modify(10, 1))   # which argument did we add?
print(modify(10, coef=2))
print(modify(10, coef=2, b=5))
~~~

Any complex python function will have many optional arguments, for example:

~~~ {.python}
?print
~~~

# Variable scope

The scope of a variable is the part of a program that can see that variable.

~~~ {.python}
a = 5
def adjust(b):
	sum = a + b
    return sum
adjust(10)   # what will be the outcome?
~~~

* "a" is the global variable => visible everywhere
* "b" and "sum" are local variables => visible only inside the function

# If we have time

(1) How would you explain the following:

~~~ {.python}
1.01-0.5 == 0.51   # returns True (makes sense!)
1.001-0.5 == 0.501   # returns False -- be aware of this when you use conditionals
print(0.1+0.2)   # returns 0.30000000000000004
~~~

<!-- Explanation: 0.1 in binary will be 0.0001(1001) which we then truncate with round-off, then perform arithmetic, then -->
<!-- convert back to decimal with round-off. -->

(2) More challening: write a code to solve x^3+4x^2-10=0 with a bisection method in the interval
    [1.3, 1.4] with tolerance 1e-8.

# Libraries

Most of the power of a programming language is in its libraries. This is especially true for Python which is a
interpreted language and is therefore very slow (compared to compiled languages). However, the libraries are often
compiled (can be written in compiled languages such as C/C++) and therefore offer much faster performance than native
Python code.

A library is a collection of functions that can be used by other programs. Python's *standard library* includes many
functions we worked with before (print, int, round, ...) and is included with Python. There are many other additional
libraries such as math, numpy, scipy, etc.

~~~ {.python}
print('pi is', pi)
import math
print('pi is', math.pi)
~~~

You can also import math's items directly:

~~~ {.python}
from math import pi, sin
print('pi is', pi)
sin(pi/6)
cos(pi)
help(math)   # help for libraries works just like help for functions
from math import *
~~~

You can also create an alias from the library:

~~~ {.python}
import math as m
print m.pi
~~~

**Quiz 8:** exploring the math library

**Quiz 9:** random numbers

**Quiz 10:** forgot to load the library

**Quiz 11:** degree conversion with math

# Numpy

As you saw before, Python is not statically typed, i.e. variables can change their type on the fly:

~~~
a = 5
a = 'apple'
print(a)
~~~

This makes Python very flexible. Out of these variables you form 1D lists, and these can be inhomogeneous:

~~~
a = [1, 2, 'Vancouver', ['Earth', 'Moon'], {'list': 'an ordered collection of values'}]
a[1] = 'Sun'
a
~~~

Python lists are very general and flexible, which is great for high-level programming, but it comes at a cost. The
Python interpreter can't make any assumptions about what will come next in a list, so it treats everything as a generic
object with its own type. As lists get longer, eventually performance takes a hit.

Python does not have any mechanism for a uniform/homogeneous list, where -- to jump to element #1000 -- you just take
the memory address of the very first element and then increment it by (element size in bytes)*999. **Numpy** library
fills this gap by adding the concept of homogenous collections to python -- `numpy.ndarray`s -- which are multidimensional,
homogeneous arrays of fixed-size items (most commonly numbers).

1. This brings large performance benefits!
  - no reading of extra bits (type, size, reference count)
  - no type checking
  - contiguous allocation in memory
1. numpy lets you work with mathematical arrays.

Lists ans numpy arrays behave very differently:

~~~
a = [1, 2, 3, 4]
b = [5, 6, 7, 8]
a + b              # this will concatenate two lists: [1,2,3,4,5,6,7,8]
~~~

~~~
import numpy as np
na = np.array([1, 2, 3, 4])
nb = np.array([5, 6, 7, 8])
na + nb            # this will sum two vectors element-wise: array([6,8,10,12])
na * nb            # element-wise product
~~~

## Working with mathematical arrays in numpy

Numpy arrays have the following attributes:

- ndim = the number of dimensions
- shape = a tuple giving the sizes of the dimensions
- size = the total number of elements
- dtype = the data type
- itemsize = the size (bytes) of individual elements
- nbytes = the total memory (bytes) occupied by the ndarray
- strides = tuple of bytes to step in each dimension when traversing an array
- data = memory address of the array

~~~
a = np.arange(10)      # 10 integer elements 0..9
a.ndim      # 1
a.shape     # (10,)
a.nbytes    # 80
a.dtype     # dtype('int64')
b = np.arange(10, dtype=np.float)
b.dtype     # dtype('float64')
~~~

In numpy there are many ways to create arrays:

~~~
np.arange(11,20)               # 9 integer elements 11..19
np.linspace(0, 1, 100)         # 100 numbers uniformly spaced between 0 and 1 (inclusive)
np.linspace(0, 1, 100).shape
np.zeros(100, dtype=np.int)    # 1D array of 100 integer zeros
np.zeros((5, 5), dtype=np.float64)     # 2D 5x5 array of floating zeros
np.ones((3,3,4), dtype=np.float64)     # 3D 3x3x4 array of floating ones
np.eye(5)            # 2D 5x5 identity/unit matrix (with ones along the main diagonal)
~~~

You can create random arrays:

~~~
np.random.randint(0, 10, size=(4,5))    # 4x5 array of random integers in the half-open interval [0,10)
np.random.random(size=(4,3))            # 4x3 array of random floats in the half-open interval [0.,1.)
np.random.randn(3, 3)       # 3x3 array drawn from a normal (Gaussian with c=0, sigma=1) distribution
~~~

## Indexing, slicing, and reshaping

For 1D arrays:

~~~
a = np.linspace(0,1,100)
a[0]        # first element
a[-2]       # 2nd to last element
a[5:12]     # values [5..12), also a numpy array
a[5:12:3]   # every 3rd element in [5..12), i.e. elements 5,8,11
a[::-1]     # array reversed
~~~

Similar for multi-dimensional arrays:

~~~
b = np.reshape(np.arange(100),(10,10))      # form a 10x10 array from 1D array
b[0:2,1]      # first two rows, second column
b[:,-1]       # last column
b[5:7,5:7]    # 2x2 block
~~~

~~~
a = np.array([1, 2, 3, 4])
b = np.array([4, 3, 2, 1])
np.vstack((a,b))   # stack them vertically into a 2x4 array (use a,b as rows)
np.hstack((a,b))   # stack them horizontally into a 1x8 array
np.column_stack((a,b))    # use a,b as columns
~~~

## Vectorized functions on array elements

One of the big reasons for using numpy is so you can do fast numerical operations on a large number of elements. The
result is another `ndarray`. In many calculations you can use replace the usual `for`/`while` loops with functions on
numpy elements.

~~~
a = np.reshape(np.arange(100))
a**2          # each element is a square of the corresponding element of a
np.log10(a+1)     # apply this operation to each element
(a**2+a)/(a+1)    # the result should effectively be a floating-version copy of a
~~~

> **Exercise**: Let's verify the equation
> <img src="https://raw.githubusercontent.com/razoumov/publish/master/eq001.png" height="80" />
> using summation of elements of an `ndarray`.
>
> **Hint**: Start with the first ten terms `k = np.arange(1,11)`. Then try the first 50 terms.

<!-- > **Solution**: -->
<!-- > ~~~ -->
<!-- > k = np.arange(1,11)   # let's try the first 10 terms: -->
<!-- > sum(k**2/2**k)        # getting 5.857421875 -->
<!-- > k = np.arange(1,51)   # the first 50 terms -->
<!-- > sum(k**2/2**k)        # getting 5.999999999997597 -->
<!-- > ~~~ -->

## Aggregate functions

Aggregate functions take an ndarray and reduce it along one (or more) axes. E.g., in 1D:

~~~
a = np.linspace(1, 2, 100)
a.mean()     # arithmetic mean
a.max()      # maximum value
a.argmax()   # index of the maximum value
a.sum()      # sum of all values
a.prod()     # product of all values
~~~

Or in 2D:

~~~
b = np.arange(25).reshape(5,5)
>>> b.sum()
300
b.sum(axis=0)   # add rows
b.sum(axis=1)   # add columns
~~~

## Boolean indexing

~~~
a = np.linspace(1, 2, 100)
a < 1.5
a[a<1.5]    # will only return those elements that meet the condition
a[a<1.5].shape
a.shape
~~~

## More numpy functionality

Numpy provides many standard linear algebra algorithms: matrix/vector products, decompositions, eigenvalues, solving
linear equations, e.g.

~~~
a = np.array([[3,1], [1,2]])
b = np.array([9,8])
x = np.linalg.solve(a, b)
x
np.allclose(np.dot(a, x),b)    # check the solution
~~~

## External packages built on top of numpy

A lot of other packages are built on top of numpy. E.g., there is a Python package for analysis and visualization of 3D
multi-resolution volumetric data called [yt](https://yt-project.org) which is based on numpy. Check out [this
visualization](https://raw.githubusercontent.com/razoumov/publish/master/grids.png) produced with yt.

Many image-processing libraries use numpy data structures underneath, e.g.

~~~
import skimage.io        # collection of algorithms for image processing
image = skimage.io.imread(fname="https://raw.githubusercontent.com/razoumov/publish/master/grids.png")
image.shape       # it's a 1024^2 image, with (R,G,B,\alpha) channels
~~~

Let's plot this image using plotly:

~~~
import plotly.offline as py
py.init_notebook_mode(connected=True)   # use the online plotly.js library inside the notebook (smaller file sizes)
import plotly.graph_objs as go
trace = go.Heatmap(z=image[:,:,2])   # plot the blue channel
layout = go.Layout(height=800, width=800)
fig = go.Figure(data=[trace], layout=layout)
py.iplot(fig)
~~~

Using numpy, you can easily manipulate pixels:

~~~
image[:,:,2] = 255 - image[:,:,2]
~~~

and then rerun the previous (plotly) cell.

Another example of a package built on top of numpy is **pandas**.

# Pandas dataframes
## Reading tabular data into dataframes

First, let's download the data. Open a terminal inside your Jupyter dashboard. Inside the terminal, type:

~~~
wget http://bit.ly/pythfiles -O pfiles.zip
unzip pfiles.zip && rm pfiles.zip        # this should unpack into the directory data-python/
~~~

You can now close the terminal panel. Let's switch back to our Python notebook and check our location:

~~~ {.python}
%pwd              # simply run a bash command with a prefix, make sure you see data-python/
~~~

Pandas is a widely-used Python library for working with tabular data, borrows heavily from R's dataframes, built on top
of numpy. We will be reading the data we downloaded a minute ago into a pandas dataframe:

~~~ {.python}
import pandas as pd
data = pd.read_csv('data-python/gapminder_gdp_oceania.csv')
print(data)
data   # this prints out the table nicely in Jupyter Notebook!
~~~

~~~ {.python}
data.shape   # shape is a *member variable inside data*
data.info()    # info is a *member method inside data*
~~~

Use dir(data) to list all member variables and methods. Then call that name without (), and if it's a method it'll tell
you, so you'll need to use ().

Rows are observations, and columns are the observed variables. You can add new observations at any time.

Currently the rows are indexed by number. Let's index by country:

~~~ {.python}
data = pd.read_csv('data-python/gapminder_gdp_oceania.csv', index_col='country')
data
data.shape     # now 12 columns
data.info()    # it's a dataframe! show row/column names, precision, memory usage
print(data.columns)   # will list all the columns
print(data.T)   # this will transpose the dataframe; curously this is a variable
data.describe()   # will print some statistics of numerical columns (very useful for 1000s of rows!)
~~~

Quick question: how to list all country names? (try data.T.columns)

**Quiz 12:** explore Americas

<!-- **Solution:** -->
<!-- ~~~ {.python} -->
<!-- americas = pd.read_csv('data-python/gapminder_gdp_americas.csv', index_col='country') -->
<!-- americas.info() -->
<!-- ~~~ -->

**Quiz 13:** first 3 rows and last 3 columns

<!-- **Solution:** -->
<!-- ~~~ {.python} -->
<!-- americas.head(3) -->
<!-- americas.T.tail(3).T -->
<!-- ~~~ -->

**Quiz 14:** navigating the filesystem from inside Jupyter Notebook

<!-- **Solution:** -->
<!-- ~~~ {.python} -->
<!-- microbes = pd.read_csv('../fieldData/microbes.csv') -->
<!-- ~~~ -->

**Quiz 15:** write a dataframe to disk

<!-- **Solution:** -->
<!-- ~~~ {.python} -->
<!-- help(pd.read_csv)    # this works, displays the help page -->
<!-- help(pd.to_csv)   # produces an error, there is no to_csv ... -->
<!-- help(data.to_csv)      # Ok, this works :) -->
<!-- data.to_csv('data-python/processed.csv') -->
<!-- ~~~ -->

## Subsetting

~~~ {.python}
data = pd.read_csv('data-python/gapminder_gdp_europe.csv', index_col='country')
data.head()
~~~

Let's rename the columns:

~~~
data.rename(columns={'gdpPercap_1952': '1952'})   # this renames only one but does not change `data`
~~~

Let's go through all columns and assign the new names:

~~~
for col in data.columns:
    print(col, col[-4:])
    data = data.rename(columns={col: col[-4:]})

data
~~~

Printing one element:

~~~ {.python}
data.iloc[0,0]   # the very first element by position
data.loc['Albania','1952']    # exactly the same; the very first element by label
~~~

Printing a row:

~~~ {.python}
data.loc['Albania',:]   # usual Python's slicing notation - show all columns in that row
data.loc['Albania']     # exactly the same
data.loc['Albania',]    # exactly the same
~~~

Printing a column:

~~~ {.python}
data.loc[:,'1952']   # show all rows in that column
data['1952']   # exactly the same; single index refers to columns
data.1952      # exactly the same; most compact notation to access columns; does not work with numerical column names ...
~~~

Printing a range:

~~~ {.python}
data.loc['Italy':'Poland','1952':'1967']   # select multiple rows/columns
data.iloc[0:2,0:3]
~~~

Result of slicing can be used in further operations:

~~~ {.python}
data.loc['Italy':'Poland','1952':'1967'].max()   # max for each column
data.loc['Italy':'Poland','1952':'1967'].min()   # min for each column
~~~

Use comparisons to select data based on value:

~~~ {.python}
subset = data.loc['Italy':'Poland', '1962':'1972']
print(subset)
print(subset > 1e4)
~~~

Use a Boolean mask to print values (meeting the condition) or NaN (not meeting the condition):

~~~ {.python}
mask = subset > 1e4
print(mask)
print(subset[mask])   # will print numerical values only if the corresponding elements in mask are True
~~~

NaN's are ignored by statistical operations which is handy:

~~~ {.python}
subset[mask].describe()
subset[mask].max()
~~~

**Quiz 16:** GDP of Serbia

<!-- **Solution:** -->
<!-- ~~~ {.python} -->
<!-- (1) data.loc['Serbia','2007'] -->
<!-- (2) data['2007']['Serbia'] -->
<!-- (3) data.2007['Serbia']     # works only with non-numerical column names ... so not here -->
<!-- ~~~ -->

**Quiz 17:** study the script

<!-- **Solution:** we read data for all countries, select only those in the Americas, remove the Puerto Rico row, remove the -->
<!-- continent column, and save the result to a file result.csv. -->

**Quiz 18:** study the script

<!-- **Solution:** we read the data for all European countries and for each column (=year) print out the name of the poorest -->
<!-- and richest country. -->

How do you create a dataframe from scratch? Many ways; the easiest by defining columns:

~~~ {.python}
col1 = [1,2,3]
col2 = [4,5,6]
pd.DataFrame({'a': col1, 'b': col2})       # dataframe from a dictionary
~~~

Let's index the rows by hand:
~~~ {.python}
pd.DataFrame({'a': col1, 'b': col2}, index=['a1','a2','a3'])
~~~

## Looping over data sets

Let's say we want to read several files in data-python/. We can use **for** to loop through their list:

~~~ {.python}
for filename in ['data-python/gapminder_gdp_africa.csv', 'data-python/gapminder_gdp_asia.csv']:
    data = pd.read_csv(filename, index_col='country')
    print(filename, data.min())   # print min for each column
~~~

If we have many (10s or 100s) files, we want to specify them with a pattern:

~~~ {.python}
from glob import glob
print('all csv files in data directory:', glob('data-python/*.csv'))   # returns a list
print('all text files in data directory:', glob('data-python/*.txt'))   # empty list
list = glob('data-python/*.csv')
len(list)
~~~

~~~ {.python}
for filename in glob('data-python/*.csv'):
    data = pd.read_csv(filename)
    print(filename, data.gdpPercap_1952.min())
~~~

**Quiz 19:** glob's wildmask

<!-- The right answer is A. -->

**Quiz 20:** find the file with fewest records

<!-- **Solution:** -->
<!-- ~~~ {.python} -->
<!-- fewest = 1e6 -->
<!-- for filename in glob('data-python/*.csv'): -->
<!--     fewest = min(fewest, pd.read_csv(filename).shape[0]) -->
<!-- print('smallest file has', fewest, 'records') -->
<!-- ~~~ -->

# Advanced topic: running Python scripts from the command line

In this lesson we'll work with bash command line, instead of the Jupyter notebook. We want to write a python script
'read.py' that takes a set of gapminder_gdp_*.csv files (one/few/many) as an argument and prints out various statistic
(min, max, mean) for each year for all countries in that file:

~~~ {.bash}
$ ./read.py --min data-python/gapminder_gdp_{americas,europe}.csv   # minimum for each year
$ ./read.py --max data-python/gapminder_gdp_europe.csv              # maximum for each year
$ ./read.py --mean data-python/gapminder_gdp_europe.csv data-python/gapminder_gdp_asia.csv
~~~

It would be best to open two shells for the following programming, and keep nano always open in one shell.

Put the following into a file called read.py:

~~~ {.python}
#!/usr/bin/env python
import sys
print('argument is', sys.argv)
~~~

and then run it from bash:

~~~ {.bash}
$ chmod u+x read.py
$ ./read.py     # it'll produce: argument is ['read.py'] (the name of the script is always there)
$ ./read.py one two three     # it'll produce: argument is ['read.py', 'one', 'two', 'three']
~~~

Let's modify our script:

~~~ {.python}
import sys
print(sys.argv[1:])
~~~
~~~ {.bash}
$ python read.py one two three     # it'll produce ['one', 'two', 'three']
~~~

Let's modify our script:

~~~ {.python}
import sys
for f in sys.argv[1:]:
    print(f)
~~~
~~~ {.bash}
$ python read.py one two three     # it'll produce one two three (one per line)
$ python read.py data-python/gapminder_gdp_*
$ python read.py data-python/gapminder_gdp_americas.csv data-python/gapminder_gdp_europe.csv
$ python read.py data-python/gapminder_gdp_{americas,europe}.csv     # same as last line
~~~

Let's actually read the data:

~~~ {.python}
import sys
import pandas as pd
for f in sys.argv[1:]:
    print('\n', f[26:-4].capitalize())
    data = pd.read_csv(f, index_col='country')
    print(data.shape, '\n')
~~~
~~~ {.bash}
$ python read.py data-python/gapminder_gdp_{americas,europe}.csv
$ wc -l !$      # the output should be consistent
~~~

Let's modify our script changing print(data.shape) -> print(data.mean()) to compute average per year for each file:

~~~ {.bash}
$ python read.py data-python/gapminder_gdp_{americas,europe}.csv
~~~

Let's add an *action* flag and calculate statistics based on the flag:

~~~ {.python}
import sys
import pandas as pd
action = sys.argv[1]
for f in sys.argv[2:]:
    print('\n', f[26:-4].capitalize())
    data = pd.read_csv(f, index_col='country')
    if 'continent' in data.columns.tolist():   # if 'continent' column exists
        data = data.drop('continent',1)        #     delete it (1 stands for column, 0 for row)
    if action == '--min':
        values = data.min().tolist()
    elif action == '--mean':
        values = data.mean().tolist()
    elif action == '--max':
        values = data.max().tolist()
    print(values,'\n')
~~~
~~~ {.bash}
$ python read.py --min data-python/gapminder_gdp_{americas,europe}.csv
$ python read.py --max data-python/gapminder_gdp_{americas,europe}.csv
$ python read.py --mean data-python/gapminder_gdp_{americas,europe}.csv
$ python read.py --mean data-python/gapminder_gdp_*.csv   # should print data for all five continents
~~~

Let's add assertion for action (syntax is: assert n > 0.0, 'should be positive') right after the
definition of *action*:

~~~ {.python}
assert action in ['--min', '--mean', '--max'], 'action must be one of: --min --mean --max'
~~~

and try passing some other action.

Now let's package processing of a file (reading + computing + printing) as a function:

~~~ {.python}
import sys
import pandas as pd
action = sys.argv[1]
assert action in ['--min', '--mean', '--max'], 'action must be one of: --min --mean --max'
filenames = sys.argv[2:]
def process(filename, action):
    print('\n', filename[26:-4].capitalize())
    data = pd.read_csv(filename, index_col='country')
    if 'continent' in data.columns.tolist():   # if 'continent' column exists
        data = data.drop('continent',1)        #     delete it (1 stands for column, 0 for row)
    if action == '--min':
        values = data.min().tolist()
    elif action == '--mean':
        values = data.mean().tolist()
    elif action == '--max':
        values = data.max().tolist()
    print(values,'\n')
for f in filenames:
    process(f, action)
~~~

## Very advanced topic: adding standard input support to our scripts

Finally, let's add support for Unix standard input. Delete 'print('\n', f[26:-4].capitalize())' and change the last two
lines to:

~~~ {.python}
if len(filenames) == 0:
    process(sys.stdin,action)       # process standard input
else:
    for f in filenames:             # same as before
        print('\n', f[26:-4].capitalize())
        process(f,action)
~~~
~~~ {.bash}
$ python read.py --mean data-python/gapminder_gdp_europe.csv
$ python read.py --mean < data-python/gapminder_gdp_europe.csv    # anyone knows why this could be useful?
~~~

Here is why: you can do preprocessing in bash before passing the data to your Python script, e.g.

~~~ {.bash}
$ head -5 data-python/gapminder_gdp_europe.csv | python read.py --mean    # process only first five countries
cp data-python/gapminder_gdp_asia.csv a1
cat data-python/gapminder_gdp_europe.csv | sed '1d' >> a1   # add all European data without the header
cat a1 | python read.py --mean          # merge Asian and European data and calculate joint statistics
~~~

Here is our full script:

~~~ {.python}
import sys
import pandas as pd
action = sys.argv[1]
assert action in ['--min', '--mean', '--max'], 'action must be one of: --min --mean --max'
filenames = sys.argv[2:]
def process(filename, action):
    data = pd.read_csv(filename, index_col='country')
    if 'continent' in data.columns.tolist():   # if 'continent' column exists
        data = data.drop('continent',1)        #     delete it (1 stands for column, 0 for row)
    if action == '--min':
        values = data.min().tolist()
    elif action == '--mean':
        values = data.mean().tolist()
    elif action == '--max':
        values = data.max().tolist()
    print(values,'\n')
if len(filenames) == 0:
    process(sys.stdin,action)       # process standard input
else:
    for f in filenames:             # same as before
        print('\n', f[26:-4].capitalize())
        process(f,action)
~~~





# Plotting
## Simple line/scatter plots of gapminder data

One of the most widely used Python plotting libraries is matplotlib. Matplotlib produces static images. In this workshop
we'll use plotly which is another open-source library that produces interactive HTML5 + JavaScript images.

~~~
import plotly.offline as py
py.init_notebook_mode(connected=True)   # use the online plotly.js library inside the notebook (smaller file sizes)
import plotly.graph_objs as go
from numpy import linspace, sin
x1 = linspace(0.01,1,300)
y1 = sin(1/x1)
trace1 = go.Scatter(x=x1, y=y1, mode='lines+markers', name='sin(1/x)')
data = [trace1]
py.iplot(data)   # plot inline in a Jupyter notebook
~~~

Now let's add the layout, replacing the last line with:

~~~
layout = go.Layout(title='An oscillating function', xaxis=dict(title='x'), yaxis=dict(title='f(x)'))
fig = go.Figure(data=data,layout=layout)
py.iplot(fig)
~~~

Let's do a quick gapminder plot with pandas:

~~~
import pandas as pd
import plotly.offline as py
py.init_notebook_mode(connected=True)   # use the online plotly.js library inside the notebook (smaller file sizes)
import plotly.graph_objs as go
data = pd.read_csv('data-python/gapminder_gdp_oceania.csv', index_col='country')
trace1 = go.Scatter(x=data.columns, y=data.loc['Australia'])
layout = go.Layout(yaxis=dict(title='GDP'))
fig = go.Figure(data=[trace1], layout=layout)
py.iplot(fig)
~~~

**Exercise:** add a curve for New Zealand.

<!-- **Solution:** -->
<!-- ~~~ {.python} -->
<!-- ... -->
<!-- trace1 = go.Scatter(x=data.columns, y=data.loc['Australia'], name='Australia') -->
<!-- trace2 = go.Scatter(x=data.columns, y=data.loc['New Zealand'], name='New Zealand') -->
<!-- ... -->
<!-- fig = go.Figure(data=[trace1,trace2], layout=layout) -->
<!-- ... -->
<!-- ~~~ -->

**Exercise:** do a scatter plot of Australia vs. New Zealand.

<!-- **Solution:** -->
<!-- ~~~ {.python} -->
<!-- data = pd.read_csv('data-python/gapminder_gdp_oceania.csv', index_col='country') -->
<!-- trace = go.Scatter(x=data.loc['Australia'], y=data.loc['New Zealand'], mode='markers') -->
<!-- layout = go.Layout(xaxis=dict(title='Australia'), yaxis=dict(title='New Zealand')) -->
<!-- fig = go.Figure(data=[trace], layout=layout) -->
<!-- py.iplot(fig) -->
<!-- ~~~ -->

**Quiz 21 :** (more difficult) plot the average GDP vs. time in each region (each file)

<!-- **Solution:** (omitting the first four lines loading modules) -->
<!-- ~~~ {.python} -->
<!-- curves = [] -->
<!-- for filename in glob('data-python/gapminder_gdp_*.csv'): -->
<!--     data = pd.read_csv(filename) -->
<!--     cols = data.columns -->
<!--     c2 = cols[cols!='continent']       # get rid of continents element if present -->
<!--     c3 = c2[c2!='country']             # get rid of country element if present -->
<!-- 	years = [col[-4:] for col in c3]   # extract last 4 digits from column's names -->
<!--     average = data.loc[:,'gdpPercap_1952':'gdpPercap_2007'].mean() -->
<!--     name = filename[26:-4] -->
<!--     trace = go.Scatter(x=years, y=average, name=name) -->
<!--     curves.append(trace) -->

<!-- layout = go.Layout(yaxis=dict(title='GDP')) -->
<!-- fig = go.Figure(data=curves, layout=layout) -->
<!-- py.iplot(fig) -->
<!-- ~~~ -->

Finally, let's create a plot showing the correlation between GDP and life expectancy in 2007, normalizing marker area by
population, and adding the country name to the mouse-over popup:

~~~ {.python}
data = pd.read_csv('data-python/gapminder_all.csv')
data.columns   # see the columns
from numpy import log10, sqrt   # these versions of log10/sqrt can operate on lists
loggdp = log10(data.gdpPercap_2007.tolist())
trace = go.Scatter(x=loggdp, y=data.lifeExp_2007, mode='markers', text=data.country,
                   marker=dict(sizemode = 'diameter', size = sqrt(data.pop_2007/1e5)))
layout = go.Layout(xaxis=dict(title='log10(GDP)'), yaxis=dict(title='life expectancy'))
fig = go.Figure(data=[trace], layout=layout)
py.iplot(fig)
~~~

## Bar plots

Let's try a Bar plot, constructing `data` directly in one line from the dictionary:

~~~ {.python}
import plotly.offline as py
py.init_notebook_mode(connected=True)
import plotly.graph_objs as go
data = [go.Bar(x=['Vancouver', 'Calgary', 'Toronto', 'Montreal', 'Halifax'],
               y=[2463431, 1392609, 5928040, 4098927, 403131])]
py.iplot(data)
~~~

Let's plot inner city population vs. greater metro area for each city:

~~~ {.python}
import plotly.offline as py
py.init_notebook_mode(connected=True)
import plotly.graph_objs as go
cities = ['Vancouver', 'Calgary', 'Toronto', 'Montreal', 'Halifax']
proper = [631486, 1239220, 2731571, 1704694, 316701]
metro = [2463431, 1392609, 5928040, 4098927, 403131]
bar1 = go.Bar(x=cities, y=proper, name='inner city')
bar2 = go.Bar(x=cities, y=metro, name='greater area')
data = [bar1,bar2]
py.iplot(data)
~~~

Let's now do a stacked plot, with *outer city* population on top of *inner city* population:

~~~ {.python}
import plotly.offline as py
py.init_notebook_mode(connected=True)
import plotly.graph_objs as go
cities = ['Vancouver', 'Calgary', 'Toronto', 'Montreal', 'Halifax']
proper = [631486, 1239220, 2731571, 1704694, 316701]
metro = [2463431, 1392609, 5928040, 4098927, 403131]
outside = [m-p for p,m in zip(proper,metro)]   # need to subtract
bar1 = go.Bar(x=cities, y=proper, name='inner city')
bar2 = go.Bar(x=cities, y=outside, name='outer city')
data = [bar1,bar2]
layout = go.Layout(barmode='stack')         # new element!
fig = go.Figure(data=data, layout=layout)   # new element!
py.iplot(fig)   # we get a stacked bar chart
~~~

What else can we modify in the layout?

~~~ {.python}
import plotly.graph_objs as go
help(go.Layout)
~~~

## Heatmaps

Let's plot a heatmap of monthly temperatures at the South Pole:

~~~ {.python}
import plotly.offline as py
py.init_notebook_mode(connected=True)
import plotly.graph_objs as go
months = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec', 'Year']
recordHigh = [-14.4,-20.6,-26.7,-27.8,-25.1,-28.8,-33.9,-32.8,-29.3,-25.1,-18.9,-12.3,-12.3]
averageHigh = [-26.0,-37.9,-49.6,-53.0,-53.6,-54.5,-55.2,-54.9,-54.4,-48.4,-36.2,-26.3,-45.8]
dailyMean = [-28.4,-40.9,-53.7,-57.8,-58.0,-58.9,-59.8,-59.7,-59.1,-51.6,-38.2,-28.0,-49.5]
averageLow = [-29.6,-43.1,-56.8,-60.9,-61.5,-62.8,-63.4,-63.2,-61.7,-54.3,-40.1,-29.1,-52.2]
recordLow = [-41.1,-58.9,-71.1,-75.0,-78.3,-82.8,-80.6,-79.3,-79.4,-72.0,-55.0,-41.1,-82.8]
trace = go.Heatmap(z=[recordHigh, averageHigh, dailyMean, averageLow, recordLow],
                   x=months,
                   y=['record high', 'aver.high', 'daily mean', 'aver.low', 'record low'])
data = [trace]
py.iplot(data)
~~~

## Geographical scatterplot

Go back to your Python Jupyter Notebook. Now let's do a scatterplot on top of a geographical map:

~~~ {.python}
import plotly.offline as py
py.init_notebook_mode(connected=True)
import plotly.graph_objs as go
import pandas as pd
from math import log10
df = pd.read_csv('data-python/cities.csv')   # lists name,pop,lat,lon for 254 Canadian cities and towns
df['text'] = df['name'] + '<br>Population ' + \
             (df['pop']/1e6).astype(str) +' million' # add new column for mouse-over
largest, smallest = df['pop'].max(), df['pop'].min()
def normalize(x):
    return log10(x/smallest)/log10(largest/smallest)   # x scaled into [0,1]

df['logsize'] = round(df['pop'].apply(normalize)*255)   # new column
cities = go.Scattergeo(
    lon = df['lon'], lat = df['lat'], text = df['text'],
    marker = dict(
        size = df['pop']/5000,
        color = df['logsize'],
        colorscale = 'Viridis',
        showscale = True,   # show the colourbar
        line = dict(width=0.5, color='rgb(40,40,40)'),
        sizemode = 'area'))
layout = go.Layout(title = 'City populations',
                       showlegend = False,   # do not show legend for first plot
                       geo = dict(
                           scope = 'north america',
                           resolution = 50,   # base layer resolution of km/mm
                           lonaxis = dict(range=[-130,-55]), lataxis = dict(range=[44,70]), # plot range
                           showland = True, landcolor = 'rgb(217,217,217)',
                           showrivers = True, rivercolor = 'rgb(153,204,255)',
                           showlakes = True, lakecolor = 'rgb(153,204,255)',
                           subunitwidth = 1, subunitcolor = "rgb(255,255,255)",   # province border
						   countrywidth = 2, countrycolor = "rgb(255,255,255)"))  # country border
fig = go.Figure(data=[cities], layout=layout)
py.iplot(fig)
~~~

**Exercise:** Modify the code to display only 10 largest cities.

## 3D topographic elevation

Let's plot some tabulated topographic elevation data:

~~~ {.python}
import plotly.offline as py
py.init_notebook_mode(connected=True)
import plotly.graph_objs as go
import pandas as pd
table = pd.read_csv('data-python/mt_bruno_elevation.csv')
data = go.Surface(z=table.values)  # use 2D numpy array format
layout = go.Layout(title='Mt Bruno Elevation',
                   width=800, height=800,    # image size
                   margin=dict(l=65, r=10, b=65, t=90))   # margins around the plot
fig = go.Figure(data=[data], layout=layout)
py.iplot(fig)
~~~

## 3D parametric plot

In plotly documentation you can find quite a lot of [different 3D plot types](https://plot.ly/python/3d-charts). Here is
something visually very different, but it still uses `go.Surface(x,y,z)`:

~~~ {.python}
import plotly.offline as py
py.init_notebook_mode(connected=True)
import plotly.graph_objs as go
from numpy import pi, sin, cos, mgrid
dphi, dtheta = pi/250, pi/250    # 0.72 degrees
[phi, theta] = mgrid[0:pi+dphi*1.5:dphi, 0:2*pi+dtheta*1.5:dtheta]
        # define two 2D grids: both phi and theta are (252,502) numpy arrays
r = sin(4*phi)**3 + cos(2*phi)**3 + sin(6*theta)**2 + cos(6*theta)**4
x = r*sin(phi)*cos(theta)   # x is also (252,502)
y = r*cos(phi)              # y is also (252,502)
z = r*sin(phi)*sin(theta)   # z is also (252,502)
surface = go.Surface(x=x, y=y, z=z, colorscale='Viridis')
layout = go.Layout(title='parametric plot')
fig = go.Figure(data=[surface], layout=layout)
py.iplot(fig)
~~~

## 3D scatter plot

Let's take a look at a 3D scatter plot using the `country index` data from http://www.prosperity.com for 142 countries:

~~~ {.python}
import plotly.offline as py
py.init_notebook_mode(connected=True)
import plotly.graph_objs as go
import pandas as pd
df = pd.read_csv('data-python/legatum2015.csv')
spheres = go.Scatter3d(x=df.economy,
                       y=df.entrepreneurshipOpportunity,
                       z=df.governance,
                       text=df.country,
                       mode='markers',
                       marker=dict(
                           sizemode = 'diameter',
                           sizeref = 0.3,   # max(safetySecurity+5.5) / 32
                           size = df.safetySecurity+5.5,
                           color = df.education,
                           colorscale = 'Viridis',
                           colorbar = dict(title = 'Education'),
                           line = dict(color='rgb(140, 140, 170)')))   # sphere edge
layout = go.Layout(height=900, width=900,
                   title='Each sphere is a country sized by safetySecurity',
                   scene = dict(xaxis=dict(title='economy'),
                                yaxis=dict(title='entrepreneurshipOpportunity'),
                                zaxis=dict(title='governance')))
fig = go.Figure(data=[spheres], layout=layout)
py.iplot(fig)
~~~

## 3D graph

We can plot 3D graphs. Consider a Dorogovtsev-Goltsev-Mendes graph: *in each subsequent generation, every edge from the
previous generation yields a new node, and the new graph can be made by connecting together three previous-generation
graphs*.

~~~ {.python}
import plotly.offline as py
py.init_notebook_mode(connected=True)
import plotly.graph_objs as go
import networkx as nx
import sys
generation = 5
H = nx.dorogovtsev_goltsev_mendes_graph(generation)
print(H.number_of_nodes(), 'nodes and', H.number_of_edges(), 'edges')
pos = nx.spectral_layout(H,scale=1,dim=3)
Xn = [pos[i][0] for i in pos]   # x-coordinates of all nodes
Yn = [pos[i][1] for i in pos]   # y-coordinates of all nodes
Zn = [pos[i][2] for i in pos]   # z-coordinates of all nodes
Xe, Ye, Ze = [], [], []
for edge in H.edges():
    Xe += [pos[edge[0]][0], pos[edge[1]][0], None]   # x-coordinates of all edge ends
    Ye += [pos[edge[0]][1], pos[edge[1]][1], None]   # y-coordinates of all edge ends
    Ze += [pos[edge[0]][2], pos[edge[1]][2], None]   # z-coordinates of all edge ends

degree = [deg[1] for deg in H.degree()]   # list of degrees of all nodes
labels = [str(i) for i in range(H.number_of_nodes())]
edges = go.Scatter3d(x=Xe, y=Ye, z=Ze,
                     mode='lines',
                     marker=dict(size=12,line=dict(color='rgba(217, 217, 217, 0.14)',width=0.5)),
                     hoverinfo='none')
nodes = go.Scatter3d(x=Xn, y=Yn, z=Zn,
                     mode='markers',
                     marker=dict(sizemode = 'area',
                                 sizeref = 0.01, size=degree,
                                 color=degree, colorscale='Viridis',
                                 line=dict(color='rgb(50,50,50)', width=0.5)),
                     text=labels, hoverinfo='text')

axis = dict(showline=False, zeroline=False, showgrid=False, showticklabels=False, title='')
layout = go.Layout(
    title = str(generation) + "-generation Dorogovtsev-Goltsev-Mendes graph",
    width=1000, height=1000,
    showlegend=False,
    scene=dict(xaxis=go.layout.scene.XAxis(axis),
               yaxis=go.layout.scene.YAxis(axis),
               zaxis=go.layout.scene.ZAxis(axis)),
    margin=go.layout.Margin(t=100))
fig = go.Figure(data=[edges,nodes], layout=layout)
py.iplot(fig)
~~~

# Programming Style and Wrap-Up

* comment your code as much as possible
* use meaningful variable names
* very good idea to break complex programs into blocks using functions
* change one thing at a time, then test
* use revision control
* use docstrings to provide online help

~~~ {.python}
def average(values):
    "Return average of values, or None if no values are supplied."
    if len(values) > 0:
        return(sum(values)/len(values))

print(average([1,2,3,4]))
print(average([]))
help(average)
~~~

~~~ {.python}
def moreComplexFunction(values):
    """This string spans
       multiple lines.

    Blank lines are allowed."""

help(moreComplexFunction)
~~~

* very good idea to add assertions to your code to check things

~~~ {.python}
assert n > 0., 'Data should only contain positive values'
~~~

is the same as

~~~ {.python}
import sys
if n <= 0.:
    print('Data should only contain positive values')
    sys.exit(1)
~~~

* Python 3 documentation https://docs.python.org/3
* Matplotlib gallery http://matplotlib.org/gallery.html
* NumPy is a scientific computing package http://www.numpy.org
* SciPy is a rich collection of scientific utilities http://www.scipy.org/scipylib
* Python Data Analysis Library http://pandas.pydata.org

# Other advanced Python topics

- list.sort() and list.index(value); heterogeneous lists
- to/from matplotlib, to/from numpy

<!-- https://www.w3resource.com/python-exercises/list -->
<!-- https://www.google.ca/amp/s/zwischenzugs.com/2018/01/06/ten-things-i-wish-id-known-about-bash/amp/#ampshare=https://zwischenzugs.com/2018/01/06/ten-things-i-wish-id-known-about-bash -->
<!-- https://github.com/ComputeCanada/DC-shell_automation -->
