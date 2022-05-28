<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Setup](#setup)
- [Introduction to heat equation](#introduction-to-heat-equation)
- [Chapel base language](#chapel-base-language)
- [Task parallelism](#task-parallelism)
- [Data parallelism](#data-parallelism)
  - [Domains and single-locale data parallelism](#domains-and-single-locale-data-parallelism)
  - [Distributed domains](#distributed-domains)
  - [Diffusion solver on distributed domains](#diffusion-solver-on-distributed-domains)
  - [Periodic boundary conditions](#periodic-boundary-conditions)
- [I/O](#io)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Setup

You can find a copy of these notes at http://bit.ly/chapelnotes.

For submitting jobs on Graham we'll be using

~~~
sbatch ... --reservation=def-guest_cpu_6 --account=def-guest   # batch jobs
salloc ... --reservation=def-guest_cpu_6 --account=def-guest   # interactive jobs
~~~

where the name of the reservation is unique for each day of the school:

Monday: --reservation=def-guest_cpu_5  
Tuesday: --reservation=def-guest_cpu_6  
Wednesday: --reservation=def-guest_cpu_7  
Thursday: --reservation=def-guest_cpu_8  

# Introduction to heat equation

Slide: original 2D diffusion equation, discretized equation with forward Euler (explicit) time stepping.

Slide: assumptions, the final 2D finite-difference equation.

# Chapel base language

Slide: why another language?

Slide: useful links.

Note: many useful things in learnChapelInYMinutes.chpl.

> ## Exercise 1
> See the bisection method slide.
>> ## Solution
>> ~~~
>> proc f(x: real) : real {
>>   return x**5 + 8*x**3 - 2*x**2 + 5*x - 1.2;
>> }
>> var a = -1.0, b = 1.0;
>> var c = (a+b)/2.0;
>> var fa = f(a), fb = f(b), fc: real;
>> while (max((c-a),(b-c)) >= 1.e-8) {
>>   fc = f(c);
>>   if (fa*fc < 0.0) then
>>     b = c;
>>   else
>>     a = c;
>>   c = (a+b)/2.0;
>>   writef("%.8n  %.12n\n", c, max((c-a),(b-c)));
>> }
>> ~~~

# Task parallelism

Let's review some of the basic concepts we learned in the "task parallelism" section.

Explain how parallelism and locality are orthogonal things in Chapel => four examples.

- *for* is a serial loop to iterate over a range (forall i in 1..100 { some statements; })
- *coforall* creates a new task for each iteration (run in quasi-random order!), the execution of the
  parent task will not continue until all the children sync up
- *forall* creates only a "reasonable" number of tasks, i.e. one per available core, unless overwritten
  by "--dataParTasksPerLocale=..." at runtime; again, the execution of the parent task will not continue
  until all the children sync up

Reduction operation in Chapel can be specified by a variable declared outside a loop and can be used with
'forall' task parallelism:

~~~
var counter = 0;
forall a in 1..100 with (+ reduce counter) { // parallel loop, one task per core
  counter = 1;
}
writeln("actual number of threads = ", counter);
~~~

On a standalone computer (or a dedicated server) such as your laptop with single-locale Chapel installed,
you would run a Chapel code with something like this:

~~~
make test
./test
./test --dataParTasksPerLocale=10
~~~

This will try to use all available CPU cores on the computer. Obviously, it would not be a good practice
to run this on a cluster's login node, as it might try to use all cores on the node making it very slow
for all users. Instead, we'll be submitting our compiled Chapel code to the scheduler. To run
single-locale Chapel on Graham, use --reservation=def-guest_cpu_6 --account=def-guest below:

~~~ {.bash}
. /home/razoumov/startSingleLocale.sh   # run the script under the current shell
mkdir someDirectory && cd someDirectory   # cd ~/chapelCourse/codes
salloc --time=0:30:0 --ntasks=1 --cpus-per-task=3 --mem-per-cpu=1000 \
	--reservation=def-guest_cpu_6 --account=def-guest
make test           # chpl test.chpl -o test
./test
./test --dataParTasksPerLocale=10
~~~

Try changing 'forall' to 'coforall' in the code above -- how does the reported number of threads change?

Other concepts in locality.chpl.

There is a number of built-in variables to access locality information, illustrated by the following
code:

~~~
use Memory;
writeln("there are ", numLocales, " locales");
for loc in Locales do   // this is still a serial program; Locales array
                        // contains an entry for each locale
  on loc {   // simply move the lines inside to locale loc
    writeln("locale #", here.id, "...");
    writeln("  ...is named: ", here.name);
    writeln("  ...has ", here.numPUs(), " processor cores");
    writeln("  ...has ", here.physicalMemory(unit=MemUnits.GB, retType=real), " GB of memory");
    writeln("  ...has ", here.maxTaskPar, " maximum parallelism");
  }
~~~

> ## Exercise 2
> Write a task-parallel code to compute pi using the same algorithm as in pi.c (or pi.py) in the
> "Introduction to HPC" course. Use a reduction operation. Try timing this with different number of
> threads, and then changing forall to coforall.
>> ## Solution
>> ~~~
>> const pi = 3.14159265358979323846, n = 1000000;
>> var i: int, h: real, sum: real = 0;
>> h = 1.0 / n;
>> forall i in 1..n with (+ reduce sum) {
>>   var x = h * (i-0.5);
>>   sum += 4.0 / (1.0+x**2);
>> }
>> sum *= h;
>> writef("%i  %.12n  %.6n\n", n, sum, abs(sum-pi));
>> ~~~

Here is another version of the task-parallel diffusion solver (without time
stepping) on a single locale:

~~~
const n = 100, stride = 20;
var T: [0..n+1, 0..n+1] real;
var Tnew: [1..n,1..n] real;
var x, y: real;
for (i,j) in {1..n,1..n} { // serial iteration
  x = ((i:real)-0.5)/n;
  y = ((j:real)-0.5)/n;
  T[i,j] = exp(-((x-0.5)**2 + (y-0.5)**2)/0.01); // narrow gaussian peak
}
coforall (i,j) in {1..n,1..n} by (stride,stride) { // 5x5 decomposition into 20x20 blocks => 25 tasks
  for k in i..i+stride-1 { // serial loop inside each block
    for l in j..j+stride-1 do {
      Tnew[i,j] = (T[i-1,j] + T[i+1,j] + T[i,j-1] + T[i,j+1]) / 4;
    }
  }
}
~~~

# Data parallelism

## Domains and single-locale data parallelism

Recall: ranges are 1D sets of integer indices.

~~~
var range1to10: range = 1..10; // 1, 2, 3, ..., 10
var a = 1234, b = 5678;
var rangeatob: range = a..b; // using variables
var range2to10by2: range(stridable=true) = 2..10 by 2; // 2, 4, 6, 8, 10
var range1toInf = 1.. ; // unbounded range
~~~

Recall: domains are bounded multi-dimensional sets of integer indices.

~~~
var domain1to10: domain(1) = {1..10};        // 1D domain from 1 to 10
var twoDimensions: domain(2) = {-2..2,0..2}; // 2D domain over a product of ranges
var thirdDim: range = 1..16;
var threeDims: domain(3) = {thirdDim, 1..10, 5..10}; // 3D domain using a range variable
for idx in twoDimensions do
  write(idx, ", ");
writeln();
for (x,y) in twoDimensions {
  write("(", x, ", ", y, ")", ", "); // these tuples can also be deconstructed
}
~~~

Let's define an n^2 domain and print out

(1) t.locale.id = the ID of the locale holding the element t of T (should be 0)  
(2) here.id = the ID of the locale on which the code is running (should be 0)  
(3) here.maxTaskPar = the number of cores (max parallelism with 1 task/core) (should be 3)  

**Note**: We already saw some of these variables/functions: numLocales, Locales, here.id here.name,
here.numPUs(), here.physicalMemory(), here.maxTaskPar. There is also LocaleSpace which returns a
numerical index of locales (whereas Locales array contains an entry for each locale).

~~~
config const n = 8;
const mesh: domain(2) = {1..n, 1..n};  // a 2D domain defined in shared memory on a single locale
var T: [mesh] real; // a 2D array of reals defined in shared memory on a single locale (mapped onto this domain)
forall t in T { // go in parallel through all n^2 elements of T
  writeln((t.locale.id,here.id,here.maxTaskPar));
}
~~~

By the way, how do we access indices of T?

~~~
forall t in T.domain {
  writeln(t);   // t is a tuple (i,j)
}
~~~

How can we define multiple arrays on the same mesh?

~~~
const grid = {1..100}; // 1D domain
const alpha = 5; // some number
var A, B, C: [grid] real; // local arrays on this 1D domain
B = 2; C = 3;
forall (a,b,c) in zip(A,B,C) do // parallel loop
  a = b + alpha*c;   // simple example of data parallelism on a single locale
writeln(A);
~~~

## Distributed domains

Domains are fundamental Chapel concept for distributed-memory data parallelism. But before we jump into
this, we need to learn how to run multi-locale Chapel on Graham. Use the lines below with
--reservation=def-guest_cpu_6 --account=def-guest:

~~~ {.bash}
. /home/razoumov/startMultiLocale.sh   # run the script under the current shell
cd ~/chapelCourse/codes
salloc --time=0:30:0 --nodes=2 --cpus-per-task=3 --mem-per-cpu=1000 \
	--reservation=def-guest_cpu_6 --account=def-guest
make test        # chpl test.chpl -o test
srun ./test_real -nl 2   # will run on two locales with max 3 cores per locale
~~~

And let's test this environment by running the following code that should print out information about the
available locales inside your interactive job:

~~~
use Memory;
writeln("there are ", numLocales, " locales");
for loc in Locales do   // this is still a serial program
  on loc {   // simply move the lines inside to locale loc
    writeln("locale #", here.id, "...");
    writeln("  ...is named: ", here.name);
    writeln("  ...has ", here.numPUs(), " processor cores");
    writeln("  ...has ", here.physicalMemory(unit=MemUnits.GB, retType=real), " GB of memory");
    writeln("  ...has ", here.maxTaskPar, " maximum parallelism");
  }
~~~

Let's now define an n^2 distributed (over several locales) domain distributedMesh mapped to locales in
blocks. On top of this domain we define a 2D block-distributed array A of strings mapped to locales in
exactly the same pattern as the underlying domain. Let's print out

(1) a.locale.id = the ID of the locale holding the element a of A  
(2) here.name = the name of the locale on which the code is running  
(3) here.maxTaskPar = the number of cores on the locale on which the code is running  

We'll package output into each element of A as a string:  
a = "%i".format(int) + string + int  
is a shortcut for  
a = "%i".format(int) + string + "%i".format(int)  

Slide: distributed domains.

~~~
use BlockDist; // use standard block distribution module to partition the domain into blocks
config const n = 8;
const mesh: domain(2) = {1..n, 1..n};
const distributedMesh: domain(2) dmapped Block(boundingBox=mesh) = mesh;
var A: [distributedMesh] string; // block-distributed array mapped to locales
forall a in A { // go in parallel through all n^2 elements in A
  // assign each array element on the locale that stores that index/element
  a = "%i".format(a.locale.id) + '-' + here.name + '-' + here.maxTaskPar + '  ';
}
writeln(A);
~~~~

Let's try to run this on 1, 2, 4 locales. Here is an example on 4 locales with 1 core/local:

0-gra796-1   0-gra796-1   0-gra796-1   0-gra796-1   1-gra797-1   1-gra797-1   1-gra797-1   1-gra797-1  
0-gra796-1   0-gra796-1   0-gra796-1   0-gra796-1   1-gra797-1   1-gra797-1   1-gra797-1   1-gra797-1  
0-gra796-1   0-gra796-1   0-gra796-1   0-gra796-1   1-gra797-1   1-gra797-1   1-gra797-1   1-gra797-1  
0-gra796-1   0-gra796-1   0-gra796-1   0-gra796-1   1-gra797-1   1-gra797-1   1-gra797-1   1-gra797-1  
2-gra798-1   2-gra798-1   2-gra798-1   2-gra798-1   3-gra799-1   3-gra799-1   3-gra799-1   3-gra799-1  
2-gra798-1   2-gra798-1   2-gra798-1   2-gra798-1   3-gra799-1   3-gra799-1   3-gra799-1   3-gra799-1  
2-gra798-1   2-gra798-1   2-gra798-1   2-gra798-1   3-gra799-1   3-gra799-1   3-gra799-1   3-gra799-1  
2-gra798-1   2-gra798-1   2-gra798-1   2-gra798-1   3-gra799-1   3-gra799-1   3-gra799-1   3-gra799-1  

Now print the range of indices for each sub-domain by adding the following to our code:

~~~
for loc in Locales {
  on loc {
    writeln(A.localSubdomain());
  }
}
~~~

On 4 locales we should get:

{1..4, 1..4}  
{1..4, 5..8}  
{5..8, 1..4}  
{5..8, 5..8}  

The following checks if the index set owned by a locale can be represented by a single domain:

~~~
for loc in Locales {
  on loc {
    writeln(A.hasSingleLocalSubdomain());
  }
}
~~~

In our case the answer should be yes.

Let's count the number of threads by adding the following to our code:

~~~
var counter = 0;
forall a in A with (+ reduce counter) { // go in parallel through all n^2 elements
  counter = 1;
}
writeln("actual number of threads = ", counter);
~~~

Does the number make sense? If running on a large number of cores, the result will depend heavily depends
on n (the size of the domain): for large n=30 will maximize the number of threads on each locale, for
small n=5 will run a few threads per locale.

So far we looked at block distribution. Let's take a look at another standard module to partition the
domain among locales, called CyclicDist. For each element of the array we will print out again

(1) a.locale.id = the ID of the locale holding the element a of A  
(2) here.name = the name of the locale on which the code is running  
(3) here.maxTaskPar = the number of cores on the locale on which the code is running  

~~~
use CyclicDist; // elements are sent to locales in a round-robin pattern
config const n = 8;
const mesh: domain(2) = {1..n, 1..n};  // a 2D domain defined in shared memory on a single locale
const m2: domain(2) dmapped Cyclic(startIdx=mesh.low) = mesh; // mesh.low is the first index (1,1)
var A2: [m2] string;
forall a in A2 {
  a = "%i".format(a.locale.id) + '-' + here.name + '-' + here.maxTaskPar + '  ';
}
writeln(A2);
~~~

0-gra796-1   1-gra797-1   0-gra796-1   1-gra797-1   0-gra796-1   1-gra797-1   0-gra796-1   1-gra797-1  
2-gra798-1   3-gra799-1   2-gra798-1   3-gra799-1   2-gra798-1   3-gra799-1   2-gra798-1   3-gra799-1  
0-gra796-1   1-gra797-1   0-gra796-1   1-gra797-1   0-gra796-1   1-gra797-1   0-gra796-1   1-gra797-1  
2-gra798-1   3-gra799-1   2-gra798-1   3-gra799-1   2-gra798-1   3-gra799-1   2-gra798-1   3-gra799-1  
0-gra796-1   1-gra797-1   0-gra796-1   1-gra797-1   0-gra796-1   1-gra797-1   0-gra796-1   1-gra797-1  
2-gra798-1   3-gra799-1   2-gra798-1   3-gra799-1   2-gra798-1   3-gra799-1   2-gra798-1   3-gra799-1  
0-gra796-1   1-gra797-1   0-gra796-1   1-gra797-1   0-gra796-1   1-gra797-1   0-gra796-1   1-gra797-1  
2-gra798-1   3-gra799-1   2-gra798-1   3-gra799-1   2-gra798-1   3-gra799-1   2-gra798-1   3-gra799-1  

Again let's print the range of indices for each sub-domain by adding the following to our code:

~~~
for loc in Locales {
  on loc {
    writeln(A2.localSubdomain());
  }
}
~~~

{1..7 by 2, 1..7 by 2}  
{1..7 by 2, 2..8 by 2}  
{2..8 by 2, 1..7 by 2}  
{2..8 by 2, 2..8 by 2}  

There are quite a few predefined distributions: BlockDist, CyclicDist, BlockCycDist, ReplicatedDist,
DimensionalDist2D, ReplicatedDim, BlockCycDim - for details see
http://chapel.cray.com/docs/1.12/modules/distributions.html

## Diffusion solver on distributed domains

Now let's return to our original diffusion solver problem -- show the slide.

~~~
use BlockDist;
config const n = 8;
const mesh: domain(2) = {1..n, 1..n};  // local 2D n^2 domain
~~~

We will add a larger (n+2)^2 block-distributed domain largerMesh with one-cell-wide "ghost zones" on "end
locales", and define a temperature array T on top of it, by adding the following to our code:

~~~
const largerMesh: domain(2) dmapped Block(boundingBox=mesh) = {0..n+1, 0..n+1};
var T: [largerMesh] real; // a block-distributed array of temperatures
forall (i,j) in T.domain[1..n,1..n] {
  var x = ((i:real)-0.5)/(n:real); // x, y are local to each task
  var y = ((j:real)-0.5)/(n:real);
  T[i,j] = exp(-((x-0.5)**2 + (y-0.5)**2) / 0.01); // narrow gaussian peak
}
writeln(T);
~~~

> ## Question
> Why do we have  
> forall (i,j) in T.domain[1..n,1..n] {  
> and not  
> forall (i,j) in mesh
>> ## Answer
>> The first one will run on multiple locales in parallel, whereas the
>> second will run in parallel on multiple threads on locale 0 only, since
>> "mesh" is defined on locale 0.

The code above will produce something like this:

 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0  
 0.0 2.36954e-17 2.79367e-13 1.44716e-10 3.29371e-09 3.29371e-09 1.44716e-10 2.79367e-13 2.36954e-17 0.0  
 0.0 2.79367e-13 3.29371e-09 1.70619e-06 3.88326e-05 3.88326e-05 1.70619e-06 3.29371e-09 2.79367e-13 0.0  
 0.0 1.44716e-10 1.70619e-06 0.000883826 0.0201158 0.0201158 0.000883826 1.70619e-06 1.44716e-10 0.0  
 0.0 3.29371e-09 3.88326e-05 0.0201158 0.457833 0.457833 0.0201158 3.88326e-05 3.29371e-09 0.0  
 0.0 3.29371e-09 3.88326e-05 0.0201158 0.457833 0.457833 0.0201158 3.88326e-05 3.29371e-09 0.0  
 0.0 1.44716e-10 1.70619e-06 0.000883826 0.0201158 0.0201158 0.000883826 1.70619e-06 1.44716e-10 0.0  
 0.0 2.79367e-13 3.29371e-09 1.70619e-06 3.88326e-05 3.88326e-05 1.70619e-06 3.29371e-09 2.79367e-13 0.0  
 0.0 2.36954e-17 2.79367e-13 1.44716e-10 3.29371e-09 3.29371e-09 1.44716e-10 2.79367e-13 2.36954e-17 0.0  
 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0  

Lets define an array of strings nodeID with the same distribution over locales as T, by adding the
following to our code:

~~~
var nodeID: [largerMesh] string;
forall m in nodeID do
  m = "%i".format(here.id);
writeln(nodeID);
~~~

The outer perimeter in the partition below should be "ghost zones":

0 0 0 0 0 1 1 1 1 1  
0 0 0 0 0 1 1 1 1 1  
0 0 0 0 0 1 1 1 1 1  
0 0 0 0 0 1 1 1 1 1  
0 0 0 0 0 1 1 1 1 1  
2 2 2 2 2 3 3 3 3 3  
2 2 2 2 2 3 3 3 3 3  
2 2 2 2 2 3 3 3 3 3  
2 2 2 2 2 3 3 3 3 3  
2 2 2 2 2 3 3 3 3 3  

> ## Exercise 3
> In addition to here.id, also print the ID of the locale holding that value. Is it the same or different
> from here.id?
>> ## Solution
>> Something along the lines:
>>   m = "%i".format(here.id) + '-' + m.locale.id

Now we implement the parallel solver, by adding the following to our code (*contains a mistake on
purpose!*):

~~~
var Tnew: [largerMesh] real;
for step in 1..5 { // time-stepping
  forall (i,j) in mesh do
    Tnew[i,j] = (T[i-1,j] + T[i+1,j] + T[i,j-1] + T[i,j+1]) / 4;
  T[mesh] = Tnew[mesh]; // uses parallel forall underneath
}
~~~

> ## Exercise 4
> Can anyone see a mistake in the last code?
>> ## Solution
>> It should be  
>>   forall (i,j) in Tnew.domain[1..n,1..n] do  
>> instead of  
>>   forall (i,j) in mesh do  
>> as the last one will likely run in parallel via threads only on locale 0,
>> whereas the former will run on multiple locales in parallel.

Here is the final version of the entire code:

~~~
use BlockDist;
config const n = 8;
const mesh: domain(2) = {1..n,1..n};
const largerMesh: domain(2) dmapped Block(boundingBox=mesh) = {0..n+1,0..n+1};
var T, Tnew: [largerMesh] real;
forall (i,j) in T.domain[1..n,1..n] {
  var x = ((i:real)-0.5)/(n:real);
  var y = ((j:real)-0.5)/(n:real);
  T[i,j] = exp(-((x-0.5)**2 + (y-0.5)**2) / 0.01);
}
for step in 1..5 {
  forall (i,j) in Tnew.domain[1..n,1..n] {
    Tnew[i,j] = (T[i-1,j]+T[i+1,j]+T[i,j-1]+T[i,j+1])/4.0;
  }
  T = Tnew;
  writeln((step,T[n/2,n/2],T[1,1]));
}
~~~

This is the entire parallel solver! Note that we implemented an open boundary: T in "ghost zones" is
always 0. Let's add some printout and also compute the total energy on the mesh, by adding the following
to our code:

~~~
  writeln((step, T[n/2,n/2], T[2,2]));
  var total: real = 0;
  forall (i,j) in mesh with (+ reduce total) do
    total += T[i,j];
  writeln("total = ", total);
~~~

Notice how the total energy decreases in time with the open BCs: energy is leaving the system.

> ## Exercise 5
> Write a code in which here.id would be different from element.locale.id.
>> ## Solution
>> Here is one possible solution in which we examine the locality of the finite-difference stencil:
>> ~~~
>> var nodeID: [largerMesh] string;
>> forall (i,j) in nodeID.domain[1..n,1..n] do
>>   nodeID[i,j] = "%i".format(here.id) + '-' + nodeID[i,j].locale.id + '-' + nodeID[i-1,j].locale.id + '  ';
>> ~~~

This produced the following output for me:

 0-0-0   0-0-0   0-0-0   0-0-0   1-1-1   1-1-1   1-1-1   1-1-1  
 0-0-0   0-0-0   0-0-0   0-0-0   1-1-1   1-1-1   1-1-1   1-1-1  
 0-0-0   0-0-0   0-0-0   0-0-0   1-1-1   1-1-1   1-1-1   1-1-1  
 0-0-0   0-0-0   0-0-0   0-0-0   1-1-1   1-1-1   1-1-1   1-1-1  
 2-2-0   2-2-0   2-2-0   2-2-0   3-3-1   3-3-1   3-3-1   3-3-1  
 2-2-2   2-2-2   2-2-2   2-2-2   3-3-3   3-3-3   3-3-3   3-3-3  
 2-2-2   2-2-2   2-2-2   2-2-2   3-3-3   3-3-3   3-3-3   3-3-3  
 2-2-2   2-2-2   2-2-2   2-2-2   3-3-3   3-3-3   3-3-3   3-3-3  

## Periodic boundary conditions

Now let's modify the previous parallel solver to include periodic BCs. At the beginning of each time step
we need to set elements in the ghost zones to their respective values on the "opposite ends", by adding
the following to our code:

~~~
  T[0,1..n] = T[n,1..n]; // periodic boundaries on all four sides; these will run via parallel forall
  T[n+1,1..n] = T[1,1..n];
  T[1..n,0] = T[1..n,n];
  T[1..n,n+1] = T[1..n,1];
~~~

Now total energy should be conserved, as nothing leaves the domain.

# I/O

Let's write the final solution to disk. There are several caveats:

* works only with ASCII
* Chapel can also write binary data but nothing can read it (checked: not the endians problem!)
* would love to write NetCDF and HDF5, probably can do this by calling C/C++ functions from Chapel

We'll add the following to our code to write ASCII:

~~~
var myFile = open("output.dat", iomode.cw); // open the file for writing
var myWritingChannel = myFile.writer(); // create a writing channel starting at file offset 0
myWritingChannel.write(T); // write the array
myWritingChannel.close(); // close the channel
~~~

Run the code and check the file *output.dat*: it should contain the array T after 5 steps in ASCII.

<!-- Domain types http://chapel.cray.com/tutorials/ACCU2017/03-DataPar.pdf -->

<!-- http://chapel.cray.com/docs/1.14/primers/primers/distributions.html -->
<!-- http://chapel.cray.com/tutorials/ACCU2017/03-DataPar.pdf -->
<!-- builtin Locales variable -->
<!-- - for loc in Locales {} followed by on loc {} -->
<!-- - do something on Locales[1] -->
<!-- become very proficient with regular domains -->

<!-- # Advanced language features -->
