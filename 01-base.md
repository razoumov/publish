<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Chapel: basic language features](#chapel-basic-language-features)
    - [Running Chapel codes](#running-chapel-codes)
    - [Case study: solving the **_Heat transfer_** problem](#case-study-solving-the-_heat-transfer_-problem)
  - [Variables](#variables)
  - [Ranges and Arrays](#ranges-and-arrays)
  - [Conditional statements](#conditional-statements)
  - [Structured iterations with for-loops](#structured-iterations-with-for-loops)
  - [Using command line arguments](#using-command-line-arguments)
  - [Timing the execution of code in Chapel](#timing-the-execution-of-code-in-chapel)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

* Official lessons at https://hpc-carpentry.github.io/hpc-chapel.
* These notes at https://github.com/razoumov/publish/blob/master/01-base.md.

<!-- as productive as Python -->
<!-- as fast as Fortran -->
<!-- as portable as C -->
<!-- as scalabale as MPI -->
<!-- as fun as your favourite programming language -->

<!-- - lower-level task parallelism: create one task to do this, another task to do this -->
<!-- - higher-level data parallelism: for all elements in my array, distribute them this way -->

<!-- parallelism and locality are completely separate concepts -->

<!-- - library of standard domain maps provided by chapel -->
<!-- - users can write their own domain maps -->

# Chapel: basic language features

**_Chapel_** is
- a modern, open-source programming language developed at _Cray Inc._
- simplicity and readability of scripting languages such as Python or Matlab
- speed and performance of Fortran and C
- supports high-level abstractions for data distribution/parallelism, and for task parallelism
  - allow users to express parallel computations in a natural, almost intuitive, manner
  - can achieve anything you can do with MPI and OpenMP
- designed around a _multi-resolution_ philosophy: users can incrementally add more detail to their
  original code, to bring it as close to the machine as required
- compiled language
- source code stored in text files with the extension **_.chpl_**

### Running Chapel codes

On Compute Canada clusters Cedar and Graham we have two versions of Chapel, one is a single-locale
(single-node) Chapel, and the other is a multi-locale (multi-node) Chapel. For now, we will start with
single-locale Chapel. If you are logged into Cedar or Graham, you'll need to load the single-locale
Chapel module:

~~~ {.bash}
$ module load gcc chapel-single/1.15.0
~~~

On Cedar, let's write a simple Chapel code, compile and run it.

~~~ {.bash}
$ cd ~/tmp
$ nano test.chpl
    writeln('If you can see this, everything works!');
$ chpl test.chpl -o test
$ ./test
~~~

You can optionally pass the flag `--fast` to the compiler to optimize the binary to run as fast as
possible in the given architecture.

<!-- Chapel was designed from scratch as a new programming language. It is an imperative language with its own -->
<!-- syntax (with elements similar to C) that we must know before introducing the parallel programming -->
<!-- concepts. -->

<!-- In this lesson we will learn the basic elements and syntax of the language; then we will study **_task -->
<!-- parallelism_**, the first level of parallelism in Chapel, and finally we will use parallel data -->
<!-- structures and **_data parallelism_**, which is the higher level of abstraction, in parallel programming, -->
<!-- offered by Chapel. -->

Depending on the code, it might utilize one / several / all cores on the current node. The command above
implies that you are allowed to utilize all cores. This might not be the case on an HPC cluster, where a
login node is shared by many people at the same time, and where it might not be a good idea to occupy all
cores on a login node with CPU-intensive tasks. Therefore, we'll be running test Chapel codes inside
submitted jobs on compute nodes. We'll start by submitting a single-core interactive job:

~~~ {.bash}
$ salloc --time=0:30:0 --mem-per-cpu=1000 --account=def-razoumov-ac
~~~

and then inside that job compile and run the test code

~~~ {.bash}
$ chpl test.chpl -o test
$ ./test
~~~

### Case study: solving the **_Heat transfer_** problem

- have a square metallic plate with some initial temperature distribution (**_initial conditions_**)
- its border is in contact with a different temperature distribution (**_boundary conditions_**)
- want to simulate the evolution of the temperature across the plate

To solve the 2nd-order hear diffusion equation, we need to **_discretize_** it, i.e., to consider the
plate as a grid of points, and to evaluate the temperature on each point at each iteration, according to
the following **_finite difference equation_**:

```
Tnew[i,j] = 0.25 * (T[i-1,j] + T[i+1,j] + T[i,j-1] + T[i,j+1])
```

- Tnew = new temperature computed at the current iteration
- T = temperature calculated at the past iteration (or the initial conditions at the first iteration)
- the indices (i,j) indicate the grid point located at the i-th row and the j-th column

So, our objective is to:

1. Write a code to implement the difference equation above.The code should have the following
   requirements: (a) it should work for any given number of rows and columns in the grid, (b) it should
   run for a given number of iterations, or until the difference between T and Tp is smaller than a given
   tolerance value, and (c) it should output the temperature at a desired position on the grid every
   given number of iterations.
1. Use task parallelism to improve the performance of the code and run it in the cluster
1. Use data parallelism to improve the performance of the code and run it in the cluster.

## Variables

A variable has three elements: a **_name_**, a **_type_**, and a **_value_**. When we store a value in a
variable for the first time, we say that we **_initialized_** it. Further changes to the value of a
variable are called **_assignments_**, in general, `x=a` means that we assign the value *a* to the
variable *x*.

Variables in Chapel are declared with the `var` or `const` keywords. When a variable declared as const is
initialized, its value cannot be modified anymore during the execution of the program.

In Chapel, to declare a variable we must specify the type of the variable, or initialize it in place with
some value. The common variable types in Chapel are:

* integer `int`, 
* floating point number `real`, 
* boolean `bool`, or 
* string `string`

If a variable is declared without a type, Chapel will infer it from the given initial value, for example
(let's store this in file `baseSolver.chpl`)

~~~
const rows = 100, cols = 100;      // number of rows and columns in a matrix
const niter = 500;     // number of iterations
const iout = 50, jout = 50;     // some row and column numbers
~~~

All these constant variables will be created as integers, and no other values can be assigned to these
variables during the execution of the program.

On the other hand, if a variable is declared without an initial value, Chapel will initialize it with a
default value depending on the declared type. The following variables will be created as real floating
point numbers equal to 0.0.

~~~
var delta: real;    // the greatest temperature difference from one iteration to next
var tmp: real;      // for temporary results when computing the temperatures
~~~

Of course, we can use both, the initial value and the type, when declaring a varible as follows:

~~~
const tolerance = 0.0001: real;   // temperature difference tolerance
var count = 0: int;                // the iteration counter
const nout = 20: int;             // the temperature at (iout,jout) will be printed every nout interations
~~~

Lets print out our configuration after we set all parameters:

~~~
writeln('Working with a matrix ', rows, 'x', cols, ' to ', niter, ' iterations or dT below ', tolerance);
~~~

## Ranges and Arrays

A series of integers (1,2,3,4,5, for example), is called a **_range_** in Chapel. Ranges are generated
with the `..` operator, and are useful, among other things, to declare **_arrays_** of variables. For
example, the following variables

~~~
var T: [0..rows+1,0..cols+1] real;      // current temperatures
var Tnew: [0..rows+1,0..cols+1] real;   // newly computed temperatures
~~~

are 2D arrays (matrices) with (`rows + 2`) rows and (`cols + 2`) columns of real numbers, all initialized
as 0.0. The ranges `0..rows+1` and `0..cols+1` used here, not only define the size and shape of the
array, they stand for the indices with which we could access particular elements of the array using the
`[ , ]` notation. For example, `T[0,0]` is the real variable located at the first row and first column of
the array `T`, while `T[3,7]` is the one at the 4th row and 8th column; `T[2,3..15]` access columns 4th
to 16th of the 3th row of `T`, and `T[0..3,4]` corresponds to the first 4 rows on the 5th column of
`T`. Similarly, with

~~~
T[1..rows,1..cols] = 25;     // set the initial temperature
~~~

we assign an initial temperature of 25 degrees across all points of our metal plate.

We must now be ready to start coding our simulations ... here is what we are going to do:

- this simulation will consider a matrix of _rows_ by _cols_ elements
- it will run up to _niter_ iterations, or until the largest difference in temperature between iterations
  is less than _tolerance_
- at each iteration print out the temperature at the position (iout,jout)

## Conditional statements

Chapel, as most *high level programming languages*, has different staments to control the flow of the
program or code.  The conditional statements are: the **_if statement_**, and the **_while statement_**.

The general syntax of a while statement is: 

```
while condition do 
{instructions}
```

The code flows as follows: first, the condition is evaluated, and then, if it is satisfied, all the
instructions within the curly brackets are executed one by one. This will be repeated over and over again
until the condition does not hold anymore.

The main loop in our simulation can be programmed using a while statement like this

~~~
delta = tolerance;   // safe initial bet; could also be a large number
while (count < niter && delta >= tolerance) do {
  // specify boundary conditions for T
  count += 1;      // increase the iteration counter by one
  Tnew = T;    // will be replaced: calculate Tnew from T
  // update delta, the greatest difference between Tnew and T
  T = Tnew;    // update T once all elements of Tnew are calculated
  // print the temperature at [iout,jout] if the iteration is multiple of nout
}
~~~

<!-- Essentially, what we want is to repeat all the code inside the curly brackets until the number of -->
<!-- iterations is gerater than or equal to `niter`, or the difference of temperature between iterations is -->
<!-- less than `tolerance`. (Note that in our case, as `delta` was not initialized when declared -and thus -->
<!-- Chapel assigned it the default real value 0.0-, we need to assign it a value greater than or equal -->
<!-- to 0.001, or otherwise the condition of the while statemnt will never be satisfied. A good starting point -->
<!-- is to simple say that `delta` is equal to `tolerance`). -->

Let's focus, first, on printing the temperature every 20 interations. To achieve this, we only need to
check whether `count` is a multiple of 20, and in that case, to print the temperature at the desired
position. This is the type of control that an **_if statement_** give us. The general syntax is:

```
if condition then 
  {instructions A} 
else 
  {instructions B}
```

In our case we print the temperature at the desired position if the iteration is multiple of niter:

~~~
if count%nout == 0 then writeln('Temperature at iteration ', count, ': ', T[iout,jout]);
~~~

To print 0th iteration, we can insert just before the time loop

~~~
writeln('Temperature at iteration ', count, ': ', T[iout,jout]);
~~~

Note that when only one instruction will be executed, there is no need to use the curly brackets. `%`
returns the remainder after the division (i.e. it returns zero when `count` is multiple of 20).

Let's compile and execute our code to see what we get until now

~~~ {.bash}
$ chpl baseSolver.chpl -o baseSolver
$ ./baseSolver
~~~

~~~
Temperature at iteration 0: 25.0
Temperature at iteration 20: 25.0
Temperature at iteration 40: 25.0
...
Temperature at iteration 500: 25.0
~~~

Of course the temperature is always 25.0, as we haven't done any computation yet.

## Structured iterations with for-loops

To compute the new temperature `Tnew` at any point, we need to add temperatures `T` at all the surronding
points, and divide the result by 4. And, esentially, we need to repeat this process for all the elements
of `Tnew`, or, in other words, we need to *iterate* over the elements of `Tnew`. When it comes to
iterating over a given number of elements, the **_for-loop_** is what we want to use. The for-loop has
the following general syntax:

```
for index in iterand do
  {instructions}
``` 

The *iterand* is a statement that expresses an iteration; it could be the range 1..15, for
example. *index* is a variable that exists only in the context of the for-loop, and that will be taking
the different values yielded by the iterand. The code flows as follows: index takes the first value
yielded by the iterand, and keeps it until all the instructions inside the curly brackets are executed
one by one; then, index takes the second value yielded by the iterand, and keeps it until all the
instructions are executed again. This pattern is repeated until index takes all the different values
exressed by the iterand.

We need to iterate both over all rows and all columns in order to access every single element of
`Tnew`. This can be done with nested _for_ loops like this

~~~
for i in 1..rows do {   // process row i
  for j in 1..cols do {   // process column j, row i
    Tnew[i,j] = (T[i-1,j] + T[i+1,j] + T[i,j-1] + T[i,j+1])/4;
  }
}
~~~

Now let's compile and execute our code again:

~~~ {.bash}
$ chpl baseSolver.chpl -o baseSolver
$ ./baseSolver
~~~

~~~
Temperature at iteration 0: 25.0
...
Temperature at iteration 200: 25.0
Temperature at iteration 220: 24.9999
Temperature at iteration 240: 24.9996
...
Temperature at iteration 480: 24.8883
Temperature at iteration 500: 24.8595
~~~

As we can see, the temperature in the middle of the plate (position 50,50) is slowly decreasing as the
plate is cooling down.

> ## Exercise 1
> What would be the temperature at the top right corner (row 1, column `cols`) of the plate? The border
> of the plate is in contact with the boundary conditions, which are set to zero (default boundary values
> for T), so we expect the temperature at these points to decrease faster. Modify the code to see the
> temperature at the top right corner.
>> ## Solution
>> To see the evolution of the temperature at the top right corner of the plate, we just need to modify
>> `iout` and `jout`. This corner correspond to the first row (`iout=1`) and the last column (`jout=cols`) of the
>> plate.
>> ~~~ {.bash}
>> $ chpl baseSolver.chpl -o baseSolver
>> $ ./baseSolver
>> ~~~
>> ~~~
>> Temperature at iteration 0: 25.0
>> Temperature at iteration 20: 1.48171
>> Temperature at iteration 40: 0.767179
>> ...
>> Temperature at iteration 460: 0.068973
>> Temperature at iteration 480: 0.0661081
>> Temperature at iteration 500: 0.0634717
>> ~~~

> ## Exercise 2
> Now let's have some more interesting boundary conditions. Suppose that the plate is heated by a source
> of 80 degrees located at the bottom right corner (row `rows`, column `cols`), and that the temperature
> on the rest of the border on adjacent sides (to this bottom right corner) decreases linearly to zero as
> one gets farther from that corner. Utilize `for` loops to setup the described boundary
> conditions. Compile and run your code to see how the temperature is changing now.
>> ## Solution
>> To get the linear distribution, the 80 degrees must be divided by the number of rows or columns in our
>> plate. So, the following couple of for loops at the start of time iteration will give us what we want:
>> ~~~
>> // boundary conditions
>> for i in 1..rows do
>>   T[i,cols+1] = i*80.0/rows;
>> for j in 1..cols do
>>   T[rows+1,j] = j*80.0/cols;
>> ~~~
>> Note that 80 degrees is written as a real
>> number 80.0. The division of integers in Chapel returns an integer, then, as `rows` and `cols` are
>> integers, we must have 80 as real so that the result is not truncated.
>> ~~~ {.bash}
>> $ chpl baseSolver.chpl -o baseSolver
>> $ ./baseSolver
>> ~~~
>> ~~~
>> Temperature at iteration 0: 25.0
>> Temperature at iteration 20: 2.0859
>> Temperature at iteration 40: 1.42663
>> ...
>> Temperature at iteration 460: 0.826941
>> Temperature at iteration 480: 0.824959
>> Temperature at iteration 500: 0.823152
>> ~~~

> ## Exercise 3
> So far, `delta` has been always equal to `tolerance`, which means that our main while loop will always
> run the 500 iterations. So let's update `delta` after each iteration. Use what we have studied so far
> to write the required piece of code.
>> ## Solution
>> The idea is simple, after each iteration of the while loop, we must compare all elements of `Tnew` and
>> `T`, find the greatest difference, and update `delta` with that value. The next nested for
>> loops do the job:
>> ~~~
>> // update delta, the greatest difference between Tnew and T
>> delta = 0;
>> for i in 1..rows do {
>>   for j in 1..cols do {
>>     tmp = abs(Tnew[i,j] - T[i,j]);
>>     if tmp > delta then delta = tmp;
>>   }
>> }
>> ~~~
>> Clearly there is no need to keep the difference at every single position in the array, we just need to
>> update `delta` if we find a greater one.
>> ~~~ {.bash}
>> $ chpl baseSolver.chpl -o baseSolver
>> $ ./baseSolver
>> ~~~
>> ~~~
>> Temperature at iteration 0: 25.0
>> Temperature at iteration 20: 2.0859
>> Temperature at iteration 40: 1.42663
>> ...
>> Temperature at iteration 460: 0.826941
>> Temperature at iteration 480: 0.824959
>> Temperature at iteration 500: 0.823152
>> ~~~

Now, after Exercise 3 we should have a working program to simulate our heat transfer equation. Let's
just print some additional useful information:

~~~
writeln('Final temperature at the desired position [', iout, ',', jout, '] after ', count, ' iterations is: ', T[iout,jout]);
writeln('The largest temperature difference between the last two iterations was: ', delta);
~~~

and compile and execute our final code

~~~ {.bash}
$ chpl baseSolver.chpl -o baseSolver
$ ./baseSolver
~~~
~~~
Temperature at iteration 0: 25.0
Temperature at iteration 20: 2.0859
...
Temperature at iteration 500: 0.823152
Final temperature at the desired position [1,100] after 500 iterations is: 0.823152
The largest temperature difference between the last two iterations was: 0.0258874
~~~

## Using command line arguments 

From the last run of our code, we can see that 500 iterations is not enough to get to a _steady state_ (a
state where the difference in temperature does not vary too much, i.e. `delta`<`tolerance`). Now, if we
want to change the number of iterations we would need to modify `niter` in the code, and compile it
again. What if we want to change the number of rows and columns in our grid to have more precision, or if
we want to see the evolution of the temperature at a different point (iout,jout)? The answer would be the
same, modify the code and compile it again!

No need to say that this would be very tedious and inefficient. A better scenario would be if we can pass
the desired configuration values to our binary when it is called at the command line. The Chapel
mechanism for this is the use of **_config_** variables. When a variable is declared with the `config`
keyword, in addition to `var` or `const`, like this:

~~~
config const niter = 500;    //number of iterations
~~~
~~~ {.bash}
$ chpl baseSolver.chpl -o baseSolver       # using the default value 500
$ ./baseSolver --niter=3000                # passing another value from the command line
~~~
~~~
Temperature at iteration 0: 25.0
Temperature at iteration 20: 2.0859
...
Temperature at iteration 2980: 0.793969
Temperature at iteration 3000: 0.793947
Final temperature at the desired position after 3000 iterations is: 0.793947
The greatest difference in temperatures between the last two iterations was: 0.00142546
~~~

> ## Exercise 4
> Make `rows`, `cols`, `nout`, `iout`, `jout`, `tolerance`, `rows` and `cols` configurable variables, and
> test the code simulating different configurations. What can you conclude about the performance of the
> code.
>
>> ## Solution
>> For example, lets use a 650 x 650 grid and observe the evolution of the temperature at the position (200,300) for 10000 iterations or until the difference of temperature between iterations is less than 0.002; also, let's print the temperature every 1000 iterations.
>> ~~~ {.bash}
>> $ chpl --fast baseSolver.chpl -o baseSolver
>> $ ./baseSolver --rows=650 --cols=650 --iout=200 --jout=300 --niter=10000 --tolerance=0.002 --nout=1000
>> ~~~
>> ~~~
>> Temperature at iteration 0: 25.0
>> Temperature at iteration 1000: 25.0
>> Temperature at iteration 2000: 25.0
>> Temperature at iteration 3000: 25.0
>> Temperature at iteration 4000: 24.9998
>> Temperature at iteration 5000: 24.9984
>> Temperature at iteration 6000: 24.9935
>> Temperature at iteration 7000: 24.9819
>> Final temperature at the desired position after 7750 iterations is: 24.9671
>> The greatest difference in temperatures between the last two iterations was: 0.00199985
>> ~~~

## Timing the execution of code in Chapel

The code generated after Exercise 4 is the basic implementation of our simulation. We will be using it
as a benchmark, to see how much we can improve the performance when introducing the parallel programming
features of the language in the following lessons.

But first, we need a quantitative way to measure the performance of our code. Maybe the easiest way to do
it, is to see how much it takes to finish a simulation. The UNIX command `time` could be used to this
effect

~~~ {.bash}
$ time ./baseSolver --rows=650 --cols=650 --x=200 --y=300 --niter=10000 --tolerance=0.002 --n=1000
~~~
~~~
Working with a matrix 650x650 to 10000 iterations or dT below 0.002
Temperature at iteration 0: 25.0
Temperature at iteration 1000: 25.0
Temperature at iteration 2000: 25.0
Temperature at iteration 3000: 25.0
Temperature at iteration 4000: 24.9998
Temperature at iteration 5000: 24.9984
Temperature at iteration 6000: 24.9935
Temperature at iteration 7000: 24.9819
Final temperature at the desired position after 7750 iterations is: 24.9671
The greatest difference in temperatures between the last two iterations was: 0.00199985

real   0m9.206s
user   0m9.122s
sys    0m0.040s
~~~

The real time is what interest us. Our code is taking around 9.2 seconds from the moment it is called at
the command line until it returns. Sometimes, however, it could be useful to take the execution time of
specific parts of the code. This can be achieved by modifying the code to output the information that we
need. This process is called **_instrumentation of the code_**.

An easy way to instrument our code with Chapel is by using the module `Time`. **_Modules_** in Chapel are
libraries of useful functions and methods that can be used in our code once the module is loaded. To load
a module we use the keyword `use` followed by the name of the module. Once the Time module is loaded we
can create a variable of the type `Timer`, and use the methods `start`, `stop`and `elapsed` to instrument
our code.

~~~
use Time;
var watch: Timer;
watch.start();
while (count < niter && delta >= tolerance) do {
  ...
}
watch.stop();
writeln('The simulation took ', watch.elapsed(), ' seconds');
~~~
~~~ {.bash}
$ chpl --fast baseSolver.chpl -o baseSolver
$ ./baseSolver --rows=650 --cols=650 --x=200 --y=300 --niter=10000 --tolerance=0.002 --n=1000
~~~
~~~
Working with a matrix 650x650 to 10000 iterations or dT below 0.002
Temperature at iteration 0: 25.0
Temperature at iteration 1000: 25.0
Temperature at iteration 2000: 25.0
Temperature at iteration 3000: 25.0
Temperature at iteration 4000: 24.9998
Temperature at iteration 5000: 24.9984
Temperature at iteration 6000: 24.9935
Temperature at iteration 7000: 24.9819
The simulation took 8.03206 seconds
Final temperature at the desired position after 7750 iterations is: 24.9671
The greatest difference in temperatures between the last two iterations was: 0.00199985
~~~

> ## Exercise 4.5
> Try recompiling without `--fast` and see how it affects the execution time. If it becomes too slow,
> try reducing the problem size.
>> ## Solution
>> Without `--fast` the calculation will become slower by ~95X.
