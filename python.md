<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

    - [Setup](#setup)
- [Part 1](#part-1)
    - [Running and Quitting](#running-and-quitting)
    - [Variables and Assignment](#variables-and-assignment)
    - [Data Types and Type Conversion](#data-types-and-type-conversion)
    - [Built-in Functions and Help](#built-in-functions-and-help)
    - [Conditionals](#conditionals)
    - [Lists](#lists)
    - [For Loops](#for-loops)
    - [List comprehensions](#list-comprehensions)
    - [Writing Functions](#writing-functions)
    - [Variable Scope](#variable-scope)
    - [If we have time](#if-we-have-time)
- [Part 2](#part-2)
    - [Libraries](#libraries)
    - [Reading Tabular Data into Data Frames](#reading-tabular-data-into-data-frames)
    - [Pandas Data Frames](#pandas-data-frames)
    - [Plotting](#plotting)
    - [Looping Over Data Sets](#looping-over-data-sets)
    - [Programming Style and Wrap-Up](#programming-style-and-wrap-up)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

### Setup

official lesson http://swcarpentry.github.io/python-novice-gapminder

me                                            | students
--------------------------------------------- | ----------------------------------------
(1) log in to socrative.com as a teacher      | (1) log in to socrative.com as a student
(2) start a quiz, select 'python quiz 1 or 2' | (2) enter provided room name
(3) select 'teacher paced'                    |
(4) disable student names                     |
(5) start                                     |

* note to self: type "carpentry"
* personal lesson notes (this file) http://bit.ly/pythonmd

# Part 1

### Running and Quitting

Python pros:
* elegant scripting language
* powerful, compact constructs for many tasks
* very popular across all fields
* huge number of external libraries

Python cons:
* slow (interpreted language)

Python programs are plain text files, stored with the .py extension.

Many ways to run Python commands:
* from the inside a Unix shell can start a Python shell and type commands
* running Python scripts saved in *.py files
* from Jupyter (formerly iPython) notebooks - stored as JSON files, displayed as HTML

Start a Jupyter notebook from bash
~~~ {.bash}
jupyter notebook
~~~

This will open a browser page pointing to the local server. Click on New -> Python 3. The server runs on
your local machine (no need for Internet). Tab completion. Can annotate code. Can display figures next to
code.

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

### Variables and Assignment

* possible names for variables
* don't use built-in function names for variables, e.g. declaring *sum* won't let you use sum(), same for
  *print*
* Python is case-sensitive

~~~ {.python}
age = 100
firstName = 'Jason'
print(firstName, 'is', age, 'years old')
~~~

* variables persist between cells
* variables must be defined before use
* variables can be used in calculations

~~~ {.python}
age = age + 3   # another syntax: age += 3
print('age in three years:', age)
~~~

***Quiz 1.1:*** predicting values

Use square brackets to get a substring:
~~~ {.python}
element = 'helium'
print(element[0])   # single character
print(element[0:3])   # a substring
~~~

***Quiz 1.2:*** getting the second digit of a number (not a string!)

* python is case-sensitive
* use meaningful variable names

### Data Types and Type Conversion

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

### Built-in Functions and Help

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
round(3.712)
round(3.712, 1)   # can specify the number of decimal places
help(round)
round?   # Jupyter Notebook's additional syntax
~~~

* every function returns something, whether it is a variable or None
~~~ {.python}
result = print('example')
print('result of print is', result)   # what happened here? Answer: print returns None
~~~

### Conditionals

Use an *if* statement to control whether some block of code is executed or not.

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

### Lists

A list stores many values in a single structure.

~~~ {.python}
densities = [0.273, 0.275, 0.277, 0.275, 0.276]
print('densities:', densities)
print('length:', len(densities))
~~~

~~~ {.python}
print('zeroth item of densities', densities[0])
print('fourth item of densities', densities[4])
~~~

~~~ {.python}
densities[0] = 0.265
print('densities is now:', densities)
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

### For Loops

A *for* loop tells Python to execute some statements once for each value in a list, a character string,
or some other collection

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

***Quiz 1.3:*** revert a string

***Answer 1:***
~~~ {.python}
n = ''
for i in 'computer':
    n = i + n
print(n)
~~~

***Answer 2:***
~~~ {.python}
'computer'[::-1]
~~~

### List comprehensions

It's a compact way to create new lists based on existing lists/collections. Let's list squares of numbers
from 1 to 10:

~~~ {.python}
[x**2 for x in range(1,11)]
~~~

Of these, list only odd squares:

~~~ {.python}
[x**2 for x in range(1,11) if x%2==1]
~~~

~~~ {.python}
week = ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']
weekend = ['Sat', 'Sun']
~~~

This will show the entire week::

~~~ {.python}
print([day for day in week])
~~~

This will show only the weekdays:
~~~ {.python}
print([day for day in week if day not in weekend])
~~~

***Quiz 1.4:*** sum up squares of numbers

***Answer***: one possible answer is sum([x**2 for x in range(1,101)])

### Writing Functions

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

***Quiz 1.5:*** convert from Fahrenheit to Celsius

***Quiz 1.6:*** convert from Celsius to Fahrenheit

***Quiz 1.7:*** convert temperature lists

***Answer:***
~~~ {.python}
def celcius(fs):
    c = []
    for f in fs:
        c.append((f-32.)*5./9.)
    return c
~~~
	
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

### Variable Scope

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

### If we have time

(1) How would you explain the following:

~~~ {.python}
1.01-0.5 == 0.51   # returns True (makes sense!)
1.001-0.5 == 0.501   # returns False -- be aware of this when you use conditionals
print(0.1+0.2)   # returns 0.30000000000000004
~~~

Explanation: 0.1 in binary will be 0.0001(1001) which we then truncate with round-off, then perform
arithmetic, then convert back to decimal with round-off.

(2) More challening: write a code to solve x^3+4x^2-10=0 with a bisection method in the interval
    [1.3, 1.4] with tolerance 1e-8.

# Part 2

### Libraries

Most of the power of a programming language is in its libraries. This is especially true for Python which
is a interpreted language and is therefore very slow (compared to compiled languages). However, the
libraries are often compiled and therefore offer much faster performance than native Python code.

A library is a collection of functions that can be used by other programs. Python's standard library
includes many functions we worked with before (print, int, round, ...) and is included with Python. There
are many other additional libraries such as math, numpy, scipy, etc.

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

***Quiz 2.1:*** exploring the math library

***Quiz 2.2:*** random numbers

***Quiz 2.3:*** forgot to load the library

***Quiz 2.4:*** degree conversion with math

### Reading Tabular Data into Data Frames

* open http://bit.ly/pythfiles in your browser, it'll download the file pfiles.zip
* unpack pfiles.zip to your Desktop; you should see ~/Desktop/data-python

Pandas is a widely-used Python library for working with tabular data, borrows heavily from R's data
frames.

We will be reading data from the directory into which you unpack it. Note your current directory and
proceed accordingly. To change and list directories in Jupyter Notebook, you can use bash commands with %
prefix:

~~~ {.python}
%cd directoryName
%pwd
~~~

~~~ {.python}
import pandas
data = pandas.read_csv('data-python/gapminder_gdp_oceania.csv')
print(data)
data   # this prints out the table in Jupyter Notebook!
~~~

~~~ {.python}
data.shape   # shape is a *member variable inside data*
data.info()    # info is a *member method inside data*
~~~

Use dir(data) to list all member variables and methods. Then call that name without (), and if it's a
method it'll tell you, so you'll need to use ().

Rows are observations, and columns are the observed variables. You can add new observations at any time.

Currently the rows are indexed by number. Let's index by country:

~~~ {.python}
data = pandas.read_csv('data-python/gapminder_gdp_oceania.csv', index_col='country')
data
data.shape     # now 12 columns
data.info()    # it's a data frame! show row/column names, precision, memory usage
print(data.columns)   # will list all the columns
print(data.T)   # this will transpose the data frame; curously this is a variable
data.describe()   # will print some statistics of numerical columns (very useful for 1000s of rows!)
~~~

Quick question: how to list all country names? (try data.T.columns)

***Quiz 2.5:*** explore Americas

***Answer:***
~~~ {.python}
americas = pandas.read_csv('data-python/gapminder_gdp_americas.csv', index_col='country')
americas.info()
~~~

***Quiz 2.6:*** first 3 rows and last 3 columns

***Answer:***
~~~ {.python}
americas.head(3)
americas.T.tail(3).T
~~~

***Quiz 2.7:*** navigating the filesystem from inside Jupyter Notebook

***Answer:***
~~~ {.python}
microbes = pandas.read_csv('../fieldData/microbes.csv')
~~~

***Quiz 2.8:*** write data frame to disk

***Answer:***
~~~ {.python}
help(pandas.read_csv)    # this works, displays the help page
help(pandas.to_csv)   # produces an error, there is no to_csv ...
help(data.to_csv)      # Ok, this works :)
data.to_csv('data-python/processed.csv')
~~~

### Pandas Data Frames

~~~ {.python}
data = pandas.read_csv('data-python/gapminder_gdp_europe.csv', index_col='country')
data.head()
~~~

Printing one element:

~~~ {.python}
data.iloc[0,0]   # the very first element by position
data.loc['Albania','gdpPercap_1952']    # exactly the same; the very first element by label
~~~

Printing a row:

~~~ {.python}
data.loc['Albania',:]   # usual Python's slicing notation - show all columns in that row
data.loc['Albania']     # exactly the same
data.loc['Albania',]    # exactly the same
~~~

Printing a column:

~~~ {.python}
data.loc[:,'gdpPercap_1952']   # show all rows in that column
data['gdpPercap_1952']   # exactly the same; single index refers to columns
data.gdpPercap_1952      # exactly the same; most compact notation to access columns
~~~

Printing a range:

~~~ {.python}
data.loc['Italy':'Poland','gdpPercap_1952':'gdpPercap_1967']   # select multiple rows/columns
data.iloc[0,0:3]
~~~

Result of slicing can be used in further operations:
~~~ {.python}
data.loc['Italy':'Poland','gdpPercap_1952':'gdpPercap_1967'].max()   # max for each column
data.loc['Italy':'Poland','gdpPercap_1952':'gdpPercap_1967'].min()   # min for each column
~~~

Use comparisons to select data based on value:
~~~ {.python}
subset = data.loc['Italy':'Poland', 'gdpPercap_1962':'gdpPercap_1972']
print(subset)
print(subset > 1e4)
~~~

Use a Boolean mask to print values or NaN:

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

***Quiz 2.9:*** GDP of Serbia

***Answer:***
~~~ {.python}
(1) df.loc['Serbia','gdpPercap_2007']
(2) df.gdpPercap_2007['Serbia']
~~~

***Quiz 2.10:*** study the script

***Answer:*** we read data for all countries, select only those in the Americas, remove the Puerto Rico
row, remove the continent column, and save the result to a file result.csv.

***Quiz 2.11:*** study the script

***Answer:*** we read the data for all European countries and for each column (=year) print out the name
of the poorest and richest country.

How do you create a data frame from scratch? Many ways; the easiest by defining columns:

~~~ {.python}
col1 = [1,2,3]
col2 = [4,5,6]
pandas.DataFrame({'a': col1, 'b': col2})
~~~

Let's index the rows by hand:
~~~ {.python}
pandas.DataFrame({'a': col1, 'b': col2}, index=['a1','a2','a3'])
~~~

### Plotting

One of the most widely used Python plotting libraries is matplotlib.

~~~ {.python}
%matplotlib inline
import matplotlib.pyplot as plt
~~~

~~~ {.python}
x = [1, 2, 3, 4, 5]
y = [2, 4, 6, 7, 5]
plt.plot(x, y)
plt.xlabel('x')
plt.ylabel('y')
~~~

~~~ {.python}
help(plt.plot)   # very useful!
~~~

We can also plot directly from a Pandas data frame; underneath it still uses matplotlib.pyplot:

~~~ {.python}
import pandas
data = pandas.read_csv('data-python/gapminder_gdp_oceania.csv', index_col='country')
data.loc['Australia'].plot()   # plot a single row
plt.xticks(rotation=20)
~~~

The syntax dataframe.plot() produces one line for each column:

~~~ {.python}
data.plot()   # plot for each year -- looks weird: one line for each column (=year)
data.T.plot()   # now one line for each country
plt.ylabel('GDP per capita')
plt.xticks(rotation=20)
plt.style.use('ggplot')   # use ggplot style
data.T.plot(kind='bar')   # bar for each data point
help(data.plot)   # learn all the options
~~~

Can also convert data frame to lists:

~~~ {.python}
years = []
for col in data.columns:
    years.append(col[-4:])   # extract last 4 digits from column's names
print(years)
gdpA = data.loc['Australia'].tolist()
plt.plot(years, gdpA, 'bo')   # plot as blue dots
plt.plot(years, gdpA, 'bo', label='Australia')   # plot as blue dots
~~~

~~~ {.python}
gdpNZ = data.loc['New Zealand'].tolist()
plt.plot(years, gdpNZ, 'g-', label='New Zealand')
plt.legend(loc='upper left')
plt.xlabel('Year')
plt.ylabel('GDP per capita ($)')
~~~

Can do a scatter plot of Australia vs New Zealand:

~~~ {.python}
plt.scatter(gdpA,gdpNZ)
plt.xlabel('Australia')
plt.ylabel('New Zealand')
~~~

Let's create a plot showing the correlation between GDP and life expectancy for 2007, normalizing marker
size by population:

~~~ {.python}
data = pandas.read_csv('data-python/gapminder_all.csv')
data.columns   # see the columns
plt.figure(figsize=(10,8))
plt.scatter(data.gdpPercap_2007,data.lifeExp_2007,s=data.pop_2007/1e6)   # first attempt
~~~

Let's change to log scale for the x-axis and make circle area proportional to the population:

~~~ {.python}
plt.figure(figsize=(10,8))
from numpy import log10, sqrt   # these versions of log10/sqrt can operate on lists
loggdp = log10(data.gdpPercap_2007.tolist())
plt.scatter(loggdp,data.lifeExp_2007,s=sqrt(data.pop_2007/1e3))
plt.xlabel('log10(GDP)')
plt.ylabel('life expectancy')
~~~

### Looping Over Data Sets

Let's say we want to read several files in data-python/. We can use **for** to loop through their list:

~~~ {.python}
for filename in ['data-python/gapminder_gdp_africa.csv', 'data-python/gapminder_gdp_asia.csv']:
    data = pandas.read_csv(filename, index_col='country')
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
    data = pandas.read_csv(filename)
    print(filename, data.gdpPercap_1952.min())
~~~
	
***Quiz 2.12:*** glob's wildmask

The right answer is A.

***Quiz 2.13:*** find the file with fewest records

***Answer:***
~~~ {.python}
fewest = 1e6
for filename in glob('data-python/*.csv'):
    fewest = min(fewest, pandas.read_csv(filename).shape[0])
print('smallest file has', fewest, 'records')
~~~

***Quiz 2.14 :*** (more difficult) plot the average GDP vs. time in each region

***Answer:***
~~~ {.python}
%matplotlib inline
import matplotlib.pyplot as plt
plt.figure(figsize=(10,8))
for filename in glob('data-python/gapminder_gdp_*.csv'):
    data = pandas.read_csv(filename)
    years=[]
    cols = data.columns
    c2 = cols[cols!='continent']  # get rid of continents element if present
    c3 = c2[c2!='country']   # get rid of country element if present
    for col in c3:
        years.append(col[-4:])   # extract last 4 digits from column's names
    average = data.loc[:,'gdpPercap_1952':'gdpPercap_2007'].mean().tolist()
    plt.plot(years, average, label=filename[19:], linewidth=3)
plt.legend(loc='upper left')
~~~

### Programming Style and Wrap-Up

* comment your code as much as possible
* use meaningful variable names
* very good idea to break complex programs into blocks using functions
* change one thing at a time, then test
* use revision control
* use docstrings to provide online help

~~~ {.python}
def average(values):
    "Return average of values, or None if no values are supplied."
    if len(values)>0:
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
