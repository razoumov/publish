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
sbatch ... --reservation=def-guest_cpu_5 --account=def-guest   # batch jobs
salloc ... --reservation=def-guest_cpu_5 --account=def-guest   # interactive jobs
~~~

where the name of the reservation is unique for each day of the school:

--reservation=def-guest_cpu_5  
--reservation=def-guest_cpu_6  
--reservation=def-guest_cpu_7  
--reservation=def-guest_cpu_8  

# Introduction to heat equation

Slide: original 2D diffusion equation, discretized equation with forward Euler (explicit) time stepping.

Slide: assumptions, the final 2D finite-difference equation.

# Chapel base language

Slide: why another language?

Slide: useful links.

many things from learnChapelInYMinutes.chpl

**Exercise**: show the bisection method slide.

# Task parallelism

explain how parallelism and locality are orthogonal things in Chapel => four examples

for  
forall  
coforall  
reduction  
locality.chpl  

~~~
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

**Exercise**: write a task-parallel code to compute pi using the same algorithm as in pi.c (or pi.py) in
the "Introduction to HPC" course. Use a reduction operation.

# Data parallelism

## Domains and single-locale data parallelism

To run single-locale Chapel on Graham:

~~~ {.bash}
. /home/razoumov/startSingleLocale.sh   # run the script under the current shell
mkdir someDirectory && cd someDirectory   # cd ~/chapelCourse/codes
salloc --time=0:30:0 --ntasks=1 --cpus-per-task=3 --mem-per-cpu=1000 --account=def-razoumov-ac # likely def-guest
make test           # chpl test.chpl -o test
./test
~~~

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
this, we need to learn how to run multi-locale Chapel on Graham:

~~~ {.bash}
. /home/razoumov/startMultiLocale.sh   # run the script under the current shell
cd ~/chapelCourse/codes
salloc --time=0:30:0 --nodes=2 --cpus-per-task=3 --mem-per-cpu=1000 --account=def-razoumov-ac # likely def-guest
make test        # chpl test.chpl -o test
srun ./test_real -nl 2   # will run on two locales with max 3 cores per locale
~~~

And let's test this environment by running the following code that should print out information about the
available locales inside your interactive job:

~~~
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

abc http://chapel.cray.com/docs/1.12/technotes/subquery.html
http://chapel.cray.com/docs/1.14/users-guide/locality/localeTypeAndVariables.html






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

**Question**: why do we have  
forall (i,j) in T.domain[1..n,1..n] {  
and not  
forall (i,j) in mesh

**Answer**: the first one will run on multiple locales in parallel, whereas the second will run in
parallel on multiple threads on locale 0 only, since "mesh" is defined on locale 0.

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

**Exercise**: in addition to here.id, also print the ID of the locale holding that value. Is it the same
or different from here.id?

Now we implement the parallel solver, by adding the following to our code (*contains a mistake on
purpose!*):

~~~
var Tnew: [mesh] real;
for step in 1..5 { // time-stepping
  forall (i,j) in mesh do
    Tnew[i,j] = (T[i-1,j] + T[i+1,j] + T[i,j-1] + T[i,j+1]) / 4;
  T[mesh] = Tnew[mesh]; // uses parallel forall underneath
}
~~~

**Exercise**: can anyone see a mistake here?

**Answer**: it should be  
  forall (i,j) in Tnew.domain[1..n,1..n] do  
instead of  
  forall (i,j) in mesh do  
as the last one will run in parallel via threads only on locale 0, whereas the former will run on
multiple locales in parallel.

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

**Exercise**: write a code in which here.id would be different from element.locale.id.

Here is one possible solution in which we examine the locality of the finite-difference stencil:

~~~
var nodeID: [largerMesh] string;
forall (i,j) in nodeID.domain[1..n,1..n] do
  nodeID[i,j] = "%i".format(here.id) + '-' + nodeID[i,j].locale.id + '-' + nodeID[i-1,j].locale.id + '  ';
~~~

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

Now let's modify the previous parallel solver to include periodic BCs.

At the beginning of each time step we need to set elements in the ghost zones to their respective values
on the "opposite ends", by adding the following to our code:

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
* it can also write a binary but it fails to be read anywhere (not the endians problem!)
* would love to write NetCDF and HDF5: probably can do this by calling C/C++ functions from Chapel

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
