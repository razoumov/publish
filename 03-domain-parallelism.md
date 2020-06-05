<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Single-locale data parallelism](#single-locale-data-parallelism)
- [Multi-locale Chapel setup](#multi-locale-chapel-setup)
- [Simple multi-locale codes](#simple-multi-locale-codes)
- [Data parallelism](#data-parallelism)
  - [Local domains](#local-domains)
  - [Distributed domains](#distributed-domains)
  - [Heat transfer solver on distributed domains](#heat-transfer-solver-on-distributed-domains)
  - [Benchmarking](#benchmarking)
      - [On the training VM:](#on-the-training-vm)
      - [On Graham (faster interconnect):](#on-graham-faster-interconnect)
  - [Final parallel code](#final-parallel-code)
- [I/O](#io)
- [Ideas for future topics](#ideas-for-future-topics)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

* Official lessons at https://hpc-carpentry.github.io/hpc-chapel.
* These notes at https://github.com/razoumov/publish/blob/master/03-domain-parallelism.md

# Single-locale data parallelism

As we mentioned in the previous section, **Data Parallelism** is a style of parallel programming in which
parallelism is driven by *computations over collections of data elements or their indices*. The main tool
for this in Chapel is a `forall` loop -- it'll create an *appropriate* number of threads to execute a
loop, dividing the loop's iterations between them.

```
forall index in iterand   # iterating over all elements of an array or over a range of indices
{instructions}
```

What is the *appropriate* number of tasks?
* on a single core: single task
* on multiple cores on the same nodes: all cores, up to the number of elements or iterations
* on multiple cores on multiple nodes: all cores, up to the problem size, given the data distribution

<!-- Finally, we will mention the `forall` loop which is similar to `coforall`, except that it is a _for_ loop -->
<!-- executed by **all cores** in parallel. The number of tasks created is simply equal to the number of cores -->
<!-- available. -->

Consider a simple code `test.chpl`:

~~~
const n = 1e6: int;
var A: [1..n] real;
forall a in A do
  a += 1;
~~~

In this code we update all elements of the array `A`. The code will run on a single node, lauching as
many threads as the number of available cores.

* if we replace `forall` with `for`, we'll get a serial loop on a sigle core
* if we replace `forall` with `coforall`, we'll create 1e6 threads (definitely an overkill!)

Consider a simple code `forall.chpl` that we'll run inside a 3-core interactive job. We have a range of
indices 1..1000, and they get broken into groups that are processed by different threads:

~~~
var count = 0;
forall i in 1..1000 with (+ reduce count) {   // parallel loop
  count += i;
}
writeln('count = ', count);
~~~

If we have not done so, let's write a script `shared.sh` for submitting single-locale, three-processor
Chapel jobs:

<!-- ~~~ {.bash} -->
<!-- $ module load gcc chapel-single/1.15.0 -->
<!-- $ salloc --time=2:00:0 --ntasks=1 --cpus-per-task=2 --mem-per-cpu=1000 \ -->
<!--          --account=def-razoumov-ws_cpu --reservation=arazoumov-may17 -->
<!-- $ chpl forall.chpl -o forall -->
<!-- $ ./forall -->
<!-- ~~~ -->

~~~ {.bash}
#!/bin/bash
#SBATCH --time=00:05:00   # walltime in d-hh:mm or hh:mm:ss format
#SBATCH --mem-per-cpu=1000   # in MB
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=2
#SBATCH --output=solution.out
./forall
~~~

~~~ {.bash}
$ . /project/shared/startSingleLocale.sh     # on the training VM cluster (Cassiopeia)
$ chpl forall.chpl -o forall
$ sbatch shared.sh
$ cat solution.out
~~~
~~~
count = 500500
~~~

We computed the sum of integers from 1 to 1000 in parallel. How many cores did the code run on? Looking
at the code or its output, **we don't know**. Most likely, on three cores available to us inside the
job. But we can actually check that!

(1) replace `count += i;` with `count = 1;`  
(2) change the last line to `writeln('actual number of threads = ', count);`

~~~ {.bash}
$ chpl forall.chpl -o forall
$ sbatch shared.sh
$ cat solution.out
~~~
~~~
actual number of threads = 3
~~~

> ## Exercise 1
> Using the first version of `forall.chpl` (where we computed the sum of integers 1..1000) as a template,
> write a Chapel code to compute `pi` by calculating the integral (see slides) numerically through
> summation using `forall` parallelism. Implement the number of intervals as `config` variable.
>
> Hint: to get you started, here is a serial version of this code:
> ~~~
> config const n = 1000;
> var h, total: real;
> h = 1.0 / n;    // interval width
> for i in 1..n {
>   var x = h * ( i - 0.5 );
>   total += 4.0 / ( 1.0 + x**2);
> }
> writef('pi is %3.10r\n', total*h);    // C-style formatted write, r stands for real
> ~~~
>
>> ## Solution
>> Change the line
>> ~~~
>> for i in 1..n {
>> ~~~
>> to
>> ~~~
>> forall i in 1..n with (+ reduce total) {
>> ~~~

We finish this section by providing an example of how you can organize a data-parallel, shared-memory
`forall` loop for the 2D heat transfer solver (without writing the full code):

~~~
config const rows = 100, cols = 100;
const rowStride = 34, colStride = 25;    // each block has 34 rows and 25 columns => 3x4 blocks
forall (r,c) in {1..rows,1..cols} by (rowStride,colStride) do {   // nested c-loop inside r-loop
                                                                  // 12 iterations, up to 12 threads
  for i in r..min(r+rowStride-1,rows) do {     // serial i-loop inside each block
    for j in c..min(c+colStride-1,cols) do {   // serial j-loop inside each block
      Tnew[i,j] = 0.25 * (T[i-1,j] + T[i+1,j] + T[i,j-1] + T[i,j+1]);
    }
  }
}
~~~

# Multi-locale Chapel setup

So far we have been working with single-locale Chapel codes that may run on one or many cores on a single
compute node, making use of the shared memory space and accelerating computations by launching parallel
threads on individual cores. Chapel codes can also run on multiple nodes on a compute cluster. In Chapel
this is referred to as *multi-locale* execution.

>> ## Docker side note
>>
>> If you work inside a Chapel Docker container, e.g., chapel/chapel-gasnet, the container environment
>> simulates a multi-locale cluster, so you would compile and launch multi-locale Chapel codes directly by
>> specifying the number of locales with `-nl` flag:
>>
>> ~~~ {.bash}
>> $ chpl --fast mycode.chpl -o mybinary
>> $ ./mybinary -nl 4
>> ~~~
>>
>> Inside the Docker container on multiple locales your code will not run any faster than on a single
>> locale, since you are emulating a virtual cluster, and all tasks run on the same physical node. To
>> achieve actual speedup, you need to run your parallel multi-locale Chapel code on a real physical
>> cluster which we hope you have access to for this session.

On an HPC cluster you would need to submit either an interactive or a batch job asking for several nodes
and then run a multi-locale Chapel code inside that job. In practice, the exact commands to run
multi-locale Chapel codes depend on how Chapel was built on the cluster.

When you compile a Chapel code with the multi-locale Chapel compiler, two binaries will be produced. One
is called `mybinary` and is a launcher binary used to submit the real executable `mybinary_real`. If the
Chapel environment is configured properly with the launcher for the cluster's physical interconnect, then
you would simply compile the code and use the launcher binary `mybinary` to run a multi-locale code.

For the rest of this class we assume that you have a working multi-locale Chapel environment, whether
provided by a Docker container or by multi-locale Chapel on a physical HPC cluster. We will run all
examples on four nodes with three cores per node.

<!-- ~~~ {.bash} -->
<!-- $ chpl mycode.chpl -o mybinary -->
<!-- $ ./mybinary -nl 2 -->
<!-- ~~~ -->
<!-- The exact parameters of the job such as the maximum runtime and the requested memory can be specified -->
<!-- with Chapel environment variables. One drawback of this launching method is that Chapel will have access -->
<!-- to all physical cores on each node participating in the run -- this will present problems if you are -->
<!-- scheduling jobs by-core and not by-node, since part of a node should be allocated to someone else's job. -->
<!-- The Compute Canada clusters Cedar and Graham employ two different physical interconnects, and since we -->
<!-- use exactly the same multi-locale Chapel module on both clusters -->
<!-- ~~~ {.bash} -->
<!-- $ module load gcc chapel-slurm-gasnetrun_ibv/1.15.0 -->
<!-- $ export GASNET_PHYSMEM_MAX=1G      # needed for faster job launch -->
<!-- $ export GASNET_PHYSMEM_NOPROBE=1   # needed for faster job launch -->
<!-- ~~~ -->
<!-- we cannot configure the same single launcher for both. Therefore, we launch multi-locale Chapel codes -->
<!-- using the real executable `mybinary_real`. For example, for an interactive job you would type: -->
<!-- ~~~ {.bash} -->
<!-- $ salloc --time=0:30:0 --nodes=4 --cpus-per-task=2 --mem-per-cpu=1000 --account=def-razoumov-ac -->
<!-- $ echo $SLURM_NODELIST          # print the list of four nodes -->
<!-- $ echo $SLURM_CPUS_PER_TASK     # print the number of cores per node (3) -->
<!-- $ chpl mycode.chpl -o mybinary -->
<!-- $ srun ./mybinary_real -nl 4   # will run on four locales with max 3 cores per locale -->
<!-- ~~~ -->
<!-- Production jobs would be launched with `sbatch` command and a Slurm launch script as usual. -->
<!-- Alternatively, instead of loading the system-wide module, you can configure multi-locale Chapel in your -->
<!-- own directory. Send me an email later, and I'll share the instructions. Here is how you would use it: -->

<!-- On Cedar let's exit our single-node job (Ctrl-D if you are still running it), and then back on the login -->
<!-- node unload `chapel-single` and load `chapel-multi-cedar`, and then start a **4-node** interactive job -->
<!-- with **3 cores per MPI task** (12 cores per job): -->

<!-- ~~~ {.bash} -->
<!-- $ module unload chapel-single -->
<!-- $ module load chapel-multi-cedar/1.16.0 -->
<!-- $ salloc --time=2:00:0 --nodes=4 --cpus-per-task=3 --mem-per-cpu=1000 \ -->
<!--          --account=def-razoumov-ws_cpu --reservation=arazoumov-may17 -->
<!-- $ echo $SLURM_NODELIST          # print the list of nodes (should be four) -->
<!-- $ echo $SLURM_CPUS_PER_TASK     # print the number of cores per node (3) -->
<!-- $ export HFI_NO_CPUAFFINITY=1   # to enable parallelism on each locale with OmniPath drivers -->
<!-- $ export CHPL_RT_NUM_THREADS_PER_LOCALE=$SLURM_CPUS_PER_TASK   # to limit the number of tasks -->
<!-- ~~~ -->

Let's write a job submission script `distributed.sh`:

~~~ {.bash}
#!/bin/bash
#SBATCH --time=00:05:00   # walltime in d-hh:mm or hh:mm:ss format
#SBATCH --mem-per-cpu=1000   # in MB
#SBATCH --nodes=4
#SBATCH --cpus-per-task=2
#SBATCH --output=solution.out
./test -nl 4   # in this case the 'srun' launcher is already configured for our interconnect
~~~

<!-- **Note**: On Graham currently there is no good Chapel installed centrally, so you'll have to load it from -->
<!-- AR's home directory: -->

<!-- ~~~ {.bash} -->
<!-- $ . /home/razoumov/startMultiLocale.sh -->
<!-- $ salloc --time=2:00:0 --nodes=4 --cpus-per-task=3 --mem-per-cpu=1000 \ -->
<!--          --account=def-razoumov-ws_cpu --reservation=arazoumov-may17 -->
<!-- $ echo $SLURM_NODELIST          # print the list of nodes (should be four) -->
<!-- $ echo $SLURM_CPUS_PER_TASK     # print the number of cores per node (3) -->
<!-- ~~~ -->

<!-- Check: without `CHPL_RT_NUM_THREADS_PER_LOCALE`, will 32 threads run on separate 32 cores -->
<!-- or will they run on the 3 cores inside our Slurm job? -->

# Simple multi-locale codes

Let us test our multi-locale Chapel environment by launching the following code:

~~~
writeln(Locales);
~~~
~~~ {.bash}
$ . /project/shared/startMultiLocale.sh     # on the training VM cluster (Cassiopeia)
$ chpl test.chpl -o test
$ sbatch distributed.sh
$ cat solution.out
~~~

This code will print the built-in global array `Locales`. Running it on four locales will produce

~~~
LOCALE0 LOCALE1 LOCALE2 LOCALE3
~~~

We want to run some code on each locale (node). For that, we can cycle through locales:

~~~
for loc in Locales do   // this is still a serial program
  on loc do             // run the next line on locale `loc`
    writeln("this locale is named ", here.name[1..5]);   // `here` is the locale on which the code is running
~~~

This will produce

~~~
this locale is named node1
this locale is named node3
this locale is named node2
this locale is named node4
~~~

Here the built-in variable class `here` refers to the locale on which the code is running, and
`here.name` is its hostname. We started a serial `for` loop cycling through all locales, and on each
locale we printed its name, i.e., the hostname of each node. This program ran in serial starting a task
on each locale only after completing the same task on the previous locale. Note the order in which
locales were listed.

To run this code in parallel, starting four simultaneous tasks, one per locale, we simply need to replace
`for` with `forall`:

~~~
forall loc in Locales do   // now this is a parallel loop
  on loc do
    writeln("this locale is named ", here.name[1..5]);
~~~

This starts four tasks in parallel, and the order in which the print statement is executed depends on the
runtime conditions and can change from run to run:

~~~
this locale is named node1
this locale is named node4
this locale is named node2
this locale is named node3
~~~

We can print few other attributes of each locale. Here it is actually useful to revert to the serial loop
`for` so that the print statements appear in order:

~~~
use Memory;
for loc in Locales do
  on loc {
    writeln("locale #", here.id, "...");
    writeln("  ...is named: ", here.name);
    writeln("  ...has ", here.numPUs(), " processor cores");
    writeln("  ...has ", here.physicalMemory(unit=MemUnits.GB, retType=real), " GB of memory");
    writeln("  ...has ", here.maxTaskPar, " maximum parallelism");
  }
~~~
~~~ {.bash}
$ chpl test.chpl -o test
$ sbatch distributed.sh
$ cat solution.out
~~~
~~~
locale #0...
  ...is named: node1.cassiopeia.westgrid.ca
  ...has 2 processor cores
  ...has 2.77974 GB of memory
  ...has 2 maximum parallelism
locale #1...
  ...is named: node2.cassiopeia.westgrid.ca
  ...has 2 processor cores
  ...has 2.77974 GB of memory
  ...has 2 maximum parallelism
locale #2...
  ...is named: node4.cassiopeia.westgrid.ca
  ...has 2 processor cores
  ...has 2.77974 GB of memory
  ...has 2 maximum parallelism
locale #3...
  ...is named: node3.cassiopeia.westgrid.ca
  ...has 2 processor cores
  ...has 2.77974 GB of memory
  ...has 2 maximum parallelism
~~~

Note that while Chapel correctly determines the number of physical cores on each node and the number of
cores available inside our job on each node (maximum parallelism), it lists the total physical memory on
each node available to all running jobs which is not the same as the total memory per node allocated to
our job.

# Data parallelism

## Local domains

We start this section by recalling the definition of a **range** in Chapel. A range is a 1D set of
integer indices that can be bounded or infinite:

~~~
var oneToTen: range = 1..10; // 1, 2, 3, ..., 10
var a = 1234, b = 5678;
var aToB: range = a..b; // using variables
var twoToTenByTwo: range(stridable=true) = 2..10 by 2; // 2, 4, 6, 8, 10
var oneToInf = 1.. ; // unbounded range
~~~

On the other hand, **domains** are multi-dimensional (including 1D) sets of integer indices that are
always bounded. To stress the difference between domain ranges and domains, domain definitions always
enclose their indices in curly brackets. Ranges can be used to define a specific dimension of a domain:

~~~
var domain1to10: domain(1) = {1..10};        // 1D domain from 1 to 10 defined using the range 1..10
var twoDimensions: domain(2) = {-2..2, 0..2}; // 2D domain over a product of two ranges
var thirdDim: range = 1..16; // a range
var threeDims: domain(3) = {1..10, 5..10, thirdDim}; // 3D domain over a product of three ranges
for idx in twoDimensions do   // cycle through all points in a 2D domain
  write(idx, ', ');
writeln();
for (x,y) in twoDimensions {   // can also cycle using explicit tuples (x,y)
  write('(', x, ', ', y, ')', ', ');
}
writeln();
~~~

Let us define an n^2 domain called `mesh`. It is defined by the single task in our code and is therefore
defined in memory on the same node (locale 0) where this task is running. For each of n^2 mesh points,
let us print out

1. m.locale.id = the ID of the locale holding that mesh point (should be 0)  
1. here.id = the ID of the locale on which the code is running (should be 0)  
1. here.maxTaskPar = the number of available cores (max parallelism with 1 task/core) (should be 3)  

**Note**: We already saw some of these variables/functions: numLocales, Locales, here.id, here.name,
here.numPUs(), here.physicalMemory(), here.maxTaskPar.

~~~
config const n = 8;
const mesh: domain(2) = {1..n, 1..n};  // a 2D domain defined in shared memory on a single locale
forall m in mesh do   // go in parallel through all n^2 mesh points
  writeln(m, ' ', m.locale.id, ' ', here.id, ' ', here.maxTaskPar);
~~~
~~~
((7, 1), 0, 0, 3)
((1, 1), 0, 0, 3)
((7, 2), 0, 0, 3)
((1, 2), 0, 0, 3)
...
((6, 6), 0, 0, 3)
((6, 7), 0, 0, 3)
((6, 8), 0, 0, 3)
~~~

Now we are going to learn two very important properties of Chapel domains. **First**, domains can be used
to **define arrays of variables of any type on top of them**. For example, let us define an n^2 array of
real numbers on top of `mesh`:

~~~
config const n = 8;
const mesh: domain(2) = {1..n, 1..n};   // a 2D domain defined in shared memory on a single locale
var T: [mesh] real;   // a 2D array of reals defined in shared memory on a single locale (mapped onto this domain)
forall t in T do   // go in parallel through all n^2 elements of T
  writeln(t, ' ', t.locale.id);
~~~
~~~ {.bash}
$ chpl test.chpl -o test
$ sbatch distributed.sh
$ cat solution.out
~~~
~~~
(0.0, 0)
(0.0, 0)
(0.0, 0)
(0.0, 0)
...
(0.0, 0)
(0.0, 0)
(0.0, 0)
~~~

By default, all n^2 array elements are set to zero, and all of them are defined on the same locale as the
underlying mesh. We can also cycle through all indices of T by accessing its domain:

~~~
forall idx in T.domain 
  writeln(idx, ' ', T(idx));   // idx is a tuple (i,j); also print the corresponding array element
~~~
~~~
(7, 1) 0.0
(1, 1) 0.0
(7, 2) 0.0
(1, 2) 0.0
...
(6, 6) 0.0
(6, 7) 0.0
(6, 8) 0.0
~~~

Since we use a paralell `forall` loop, the print statements appear in a random runtime order.

We can also define multiple arrays on the same domain:

~~~
const grid = {1..100}; // 1D domain
const alpha = 5; // some number
var A, B, C: [grid] real; // local real-type arrays on this 1D domain
B = 2; C = 3;
forall (a,b,c) in zip(A,B,C) do // parallel loop
  a = b + alpha*c;   // simple example of data parallelism on a single locale
writeln(A);
~~~

The **second important property** of Chapel domains is that they can **span multiple locales** (nodes).

## Distributed domains

Domains are fundamental Chapel concept for distributed-memory data parallelism. 

Let us now define an n^2 distributed (over several locales) domain `distributedMesh` mapped to locales in
blocks. On top of this domain we define a 2D block-distributed array A of strings mapped to locales in
exactly the same pattern as the underlying domain. Let us print out

(1) a.locale.id = the ID of the locale holding the element a of A  
(2) here.name = the name of the locale on which the code is running  
(3) here.maxTaskPar = the number of cores on the locale on which the code is running  

Instead of printing these values to the screen, we will store this output inside each element of A as a string:  
a = int + string + int  
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
  a = a.locale.id:string + '-' + here.name[1..5] + '-' + here.maxTaskPar:string + '  ';
}
writeln(A);
~~~~

The syntax `boundingBox=mesh` tells the compiler that the outer edge of our decomposition coincides
exactly with the outer edge of our domain. Alternatively, the outer decomposition layer could include an
additional perimeter of *ghost points* if we specify

~~~
const mesh: domain(2) = {1..n, 1..n};
const largerMesh: domain(2) dmapped Block(boundingBox=mesh) = {0..n+1,0..n+1};
~~~~

but let us not worry about this for now.

Running our code on four locales with three cores per locale produces the following output:

~~~
0-node1-2   0-node1-2   0-node1-2   0-node1-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2__
0-node1-2   0-node1-2   0-node1-2   0-node1-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2__
0-node1-2   0-node1-2   0-node1-2   0-node1-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2__
0-node1-2   0-node1-2   0-node1-2   0-node1-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2__
2-node4-2   2-node4-2   2-node4-2   2-node4-2   3-node3-2   3-node3-2   3-node3-2   3-node3-2__
2-node4-2   2-node4-2   2-node4-2   2-node4-2   3-node3-2   3-node3-2   3-node3-2   3-node3-2__
2-node4-2   2-node4-2   2-node4-2   2-node4-2   3-node3-2   3-node3-2   3-node3-2   3-node3-2__
2-node4-2   2-node4-2   2-node4-2   2-node4-2   3-node3-2   3-node3-2   3-node3-2   3-node3-2__
~~~

As we see, the domain `distributedMesh` (along with the string array `A` on top of it) was decomposed
into 2x2 blocks stored on the four nodes, respectively. Equally important, for each element `a` of the
array, the line of code filling in that element ran on the same locale where that element was stored. In
other words, this code ran in parallel (`forall` loop) on four nodes, using up to three cores on each
node to fill in the corresponding array elements. Once the parallel loop is finished, the `writeln`
command runs on locale 0 gathering remote elements from other locales and printing them to standard
output.

Now we can print the range of indices for each sub-domain by adding the following to our code:

~~~
for loc in Locales {
  on loc {
    writeln(A.localSubdomain());
  }
}
~~~

On 4 locales we should get:

~~~
{1..4, 1..4}  
{1..4, 5..8}  
{5..8, 1..4}  
{5..8, 5..8}  
~~~

Let us count the number of threads by adding the following to our code:

~~~
var count = 0;
forall a in A with (+ reduce count) { // go in parallel through all n^2 elements
  count = 1;
}
writeln("actual number of threads = ", count);
~~~

If `n=8` in our code is sufficiently large, there are enough array elements per node (8*8/4 = 16 in our
case) to fully utilize all three available cores on each node, so our output should be

~~~ {.bash}
$ chpl test.chpl -o test
$ sbatch distributed.sh
$ cat solution.out
~~~
~~~
actual number of threads = 12
~~~

> ## Exercise 2
> Try reducing the array size `n` to see if that changes the output (fewer threads per locale), e.g.,
> setting n=3. Also try increasing the array size to n=20 and study the output. Does the output make sense?
>
>> Answer: run the code with
>> ~~~ {.bash}
>> $ ./test -nl 4 --n=3
>> $ ./test -nl 4 --n=20
>> ~~~
>> For n=3 we get fewer threads (7 in my case), for n=20 we still get 12 threads (the maximum available
>> number of cores inside our job).

So far we looked at the block distribution `BlockDist`. It will distribute a 2D domain among nodes either
using 1D or 2D decomposition (in our example it was 2D decomposition 2x2), depending on the domain size
and the number of nodes.

Let us take a look at another standard module for domain partitioning onto locales, called
CyclicDist. For each element of the array we will print out again

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
  a = a.locale.id + '-' + here.name[1..5] + '-' + here.maxTaskPar + '  ';
}
writeln(A2);
~~~
~~~ {.bash}
$ chpl -o test test.chpl
$ sbatch distributed.sh
$ cat solution.out
~~~
~~~
0-node1-2   1-node4-2   0-node1-2   1-node4-2   0-node1-2   1-node4-2   0-node1-2   1-node4-2__
2-node2-2   3-node3-2   2-node2-2   3-node3-2   2-node2-2   3-node3-2   2-node2-2   3-node3-2__
0-node1-2   1-node4-2   0-node1-2   1-node4-2   0-node1-2   1-node4-2   0-node1-2   1-node4-2__
2-node2-2   3-node3-2   2-node2-2   3-node3-2   2-node2-2   3-node3-2   2-node2-2   3-node3-2__
0-node1-2   1-node4-2   0-node1-2   1-node4-2   0-node1-2   1-node4-2   0-node1-2   1-node4-2__
2-node2-2   3-node3-2   2-node2-2   3-node3-2   2-node2-2   3-node3-2   2-node2-2   3-node3-2__
0-node1-2   1-node4-2   0-node1-2   1-node4-2   0-node1-2   1-node4-2   0-node1-2   1-node4-2__
2-node2-2   3-node3-2   2-node2-2   3-node3-2   2-node2-2   3-node3-2   2-node2-2   3-node3-2__
~~~

As the name `CyclicDist` suggests, the domain was mapped to locales in a cyclic, round-robin pattern. We
can also print the range of indices for each sub-domain by adding the following to our code:

~~~
for loc in Locales do
  on loc do
    writeln(A2.localSubdomain());
~~~
~~~
{1..7 by 2, 1..7 by 2}  
{1..7 by 2, 2..8 by 2}  
{2..8 by 2, 1..7 by 2}  
{2..8 by 2, 2..8 by 2}  
~~~

In addition to BlockDist and CyclicDist, Chapel has several other predefined distributions: BlockCycDist,
ReplicatedDist, DimensionalDist2D, ReplicatedDim, BlockCycDim -- for details please see
http://chapel.cray.com/docs/1.12/modules/distributions.html.

## Heat transfer solver on distributed domains

Now let us use distributed domains to write a parallel version of our original heat transfer solver
code. We'll start by copying `baseSolver.chpl` into `parallel3.chpl` and making the following
modifications to the latter:

(1) Add

~~~
use BlockDist;
const mesh: domain(2) = {1..rows, 1..cols};   // local 2D domain
~~~

(2) Add a larger (n+2)^2 block-distributed domain `largerMesh` with a layer of *ghost points* on
*perimeter locales*, and define a temperature array T on top of it, by adding the following to our code:

~~~
const largerMesh: domain(2) dmapped Block(boundingBox=mesh) = {0..rows+1, 0..cols+1};
~~~

(3) Change the definitions of T and Tnew (delete those two lines) to

~~~
var T, Tnew: [largerMesh] real;   // block-distributed arrays of temperatures
~~~

<!-- Here we initialized an initial Gaussian temperature peak in the middle of the mesh. As we evolve our -->
<!-- solution in time, this peak should diffuse slowly over the rest of the domain. -->

<!-- > ## Question -->
<!-- > Why do we have   -->
<!-- > forall (i,j) in T.domain[1..n,1..n] {   -->
<!-- > and not   -->
<!-- > forall (i,j) in mesh -->
<!-- >> ## Answer -->
<!-- >> The first one will run on multiple locales in parallel, whereas the -->
<!-- >> second will run in parallel via multiple threads on locale 0 only, since -->
<!-- >> "mesh" is defined on locale 0. -->

<!-- The code above will print the initial temperature distribution: -->

<!-- ~~~ -->
<!-- 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0   -->
<!-- 0.0 2.36954e-17 2.79367e-13 1.44716e-10 3.29371e-09 3.29371e-09 1.44716e-10 2.79367e-13 2.36954e-17 0.0   -->
<!-- 0.0 2.79367e-13 3.29371e-09 1.70619e-06 3.88326e-05 3.88326e-05 1.70619e-06 3.29371e-09 2.79367e-13 0.0   -->
<!-- 0.0 1.44716e-10 1.70619e-06 0.000883826 0.0201158 0.0201158 0.000883826 1.70619e-06 1.44716e-10 0.0   -->
<!-- 0.0 3.29371e-09 3.88326e-05 0.0201158 0.457833 0.457833 0.0201158 3.88326e-05 3.29371e-09 0.0   -->
<!-- 0.0 3.29371e-09 3.88326e-05 0.0201158 0.457833 0.457833 0.0201158 3.88326e-05 3.29371e-09 0.0   -->
<!-- 0.0 1.44716e-10 1.70619e-06 0.000883826 0.0201158 0.0201158 0.000883826 1.70619e-06 1.44716e-10 0.0   -->
<!-- 0.0 2.79367e-13 3.29371e-09 1.70619e-06 3.88326e-05 3.88326e-05 1.70619e-06 3.29371e-09 2.79367e-13 0.0   -->
<!-- 0.0 2.36954e-17 2.79367e-13 1.44716e-10 3.29371e-09 3.29371e-09 1.44716e-10 2.79367e-13 2.36954e-17 0.0   -->
<!-- 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0   -->
<!-- ~~~ -->

Let us define an array of strings `message` with the same distribution over locales as T, by adding the
following to our code:

~~~
var message: [largerMesh] string;
forall m in message do
  m = "%i".format(here.id);   // store ID of the locale on which the code is running
writeln(message);
assert(1>2);    // will halt if the condition is false
~~~
~~~ {.bash}
$ chpl -o parallel3 parallel3.chpl
$ ./parallel3 -nl 4 --rows=8 --cols=8   # run this from inside distributed.sh
~~~

The outer perimeter in the partition below are the *ghost points*, with the inner 8x8 array:

~~~
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
~~~

> ## Exercise 3
> In addition to here.id, also print the ID of the locale holding that value. Is it the same or different
> from `here.id`?
>> ## Solution
>> Something along the lines:
>>   m = "%i".format(here.id) + '-' + m.locale.id;
>> In most cases `m.locale.id` should be the same as `here.id` (computation follows data distribution).

(4) Let's comment out this `message` part, and start working on the parallel solver.

(5) Move the linearly increasing boundary conditions (right/bottom sides) before the `while` loop.

(6) Replace the loop for computing *inner* `Tnew`:

~~~
  for i in 1..rows do {  // do smth for row i
    for j in 1..cols do {   // do smth for row i and column j
      Tnew[i,j] = 0.25 * (T[i-1,j] + T[i+1,j] + T[i,j-1] + T[i,j+1]);
    }
  }
~~~
  
with a parallel `forall` loop (**contains a mistake on purpose!**):

~~~
  forall (i,j) in mesh do
    Tnew[i,j] = 0.25 * (T[i-1,j] + T[i+1,j] + T[i,j-1] + T[i,j+1]);
~~~

> ## Exercise 4
> Can anyone spot a mistake in this loop?
>
>> ## Answer
>> It should be  
>>   **forall (i,j) in largerMesh[1..rows,1..cols] do**   # run on multiple locales in parallel  
>> instead of  
>>   forall (i,j) in mesh do   # run in parallel on locale 0 only  
>> Another possible solution is__
>>   forall (i,j) in Tnew.domain[1..rows,1..cols] do   # run on multiple locales in parallel  
>> Also we cannot have  
>>   forall (i,j) in largerMesh do   # will run in parallel on multiple locales
>> as it will overwrite the boundaries.

(7) Replace

~~~
  delta = 0;
  for i in 1..rows do {
    for j in 1..cols do {
      tmp = abs(Tnew[i,j]-T[i,j]);
      if tmp > delta then delta = tmp;
    }
  }
~~~

with

~~~
  delta = max reduce abs(Tnew[1..rows,1..cols]-T[1..rows,1..cols]);
~~~

(8) Replace

~~~
  T = Tnew;
~~~

with the **inner-only** update

~~~
  T[1..rows,1..cols] = Tnew[1..rows,1..cols];   // uses parallel `forall` underneath
~~~

## Benchmarking

Let's compile both serial and data-parallel versions using the same multi-locale compiler (and we will
need `-nl` flag when running both):

~~~ {.bash}
$ which chpl
~/c3/chapel-1.19.0/bin/linux64-x86_64/chpl
$ chpl --fast baseSolver.chpl -o baseSolver
$ chpl --fast parallel3.chpl -o parallel3
~~~

First, let's try this on a smaller problem. Let's write two job submission scripts:

~~~ {.bash}
#!/bin/bash     # this is baseSolver.sh
#SBATCH --time=00:05:00   # walltime in d-hh:mm or hh:mm:ss format
#SBATCH --mem-per-cpu=1000   # in MB
#SBATCH --output=baseSolver.out
./baseSolver -nl 1 --rows=30 --cols=30 --niter=2000
~~~

~~~ {.bash}
#!/bin/bash     # this is parallel3.sh
#SBATCH --time=00:05:00   # walltime in d-hh:mm or hh:mm:ss format
#SBATCH --mem-per-cpu=1000   # in MB
#SBATCH --nodes=4
#SBATCH --cpus-per-task=2
#SBATCH --output=parallel3.out
./parallel3 -nl 4 --rows=30 --cols=30 --niter=2000
~~~

Let's run them both:

~~~ {.bash}
sbatch baseSolver.sh
sbatch parallel3.sh
~~~

Wait for the jobs to finish and then check the results:

~~~ {.bash}
$ tail -3 baseSolver.out
Final temperature at the desired position [1,30] after 1148 iterations is: 2.58084
The largest temperature difference was 9.9534e-05
The simulation took 0.008524 seconds

$ tail -3 parallel3.out
Final temperature at the desired position [1,30] after 1148 iterations is: 2.58084
The largest temperature difference was 9.9534e-05
The simulation took 193.279 seconds
~~~

As you can see, on the training VM cluster the parallel code on 4 nodes (with 2 cores each) ran ~22,675
times slower than a serial code on a single node ... What is going on here!? Shouldn't the parallel code
run ~8X faster, since we have 8X as many processors?

This is a **_fine-grained_** parallel code that needs lots of communication between tasks, and relatively
little computing. So, we are seeing the **communication overhead**. The training cluster has a very slow
network, so the problem is exponentially worse there ...

If we increase the problem size, there will be more computation (scaling O(n^2)) in between
communications (scaling O(n)), and at some point parallel code should catch up to the serial code and
eventually run faster. Let's try these problem sizes:

~~~
--rows=650 --cols=650 --niter=9500 --tolerance=0.002
Final temperature at the desired position [1,650] after 7750 iterations is: 0.125606
The largest temperature difference was 0.00199985

--rows=2000 --cols=2000 --niter=9500 --tolerance=0.002
Final temperature at the desired position [1,2000] after 9140 iterations is: 0.04301
The largest temperature difference was 0.00199989

--rows=8000 --cols=8000 --niter=9800 --tolerance=0.002
Final temperature at the desired position [1,8000] after 9708 iterations is: 0.0131638
The largest temperature difference was 0.00199974

./baseSolver -nl 1 --rows=16000 --cols=16000 --niter=9900 --tolerance=0.002
Final temperature at the desired position [1,16000] after 9806 iterations is: 0.00818861
The largest temperature difference was 0.00199975
~~~


#### On the training VM:

| | 30^2 | 650^2 | 2,000^2 |
| ----- | ----- | ----- | ----- |
| baseSolver | 0.00852s | 59s | 745s |
| parallel3 --nodes=4 --cpus-per-task=2 | 193s | 2,208s | 5,876s |
| slowdown | ~22,700 | ~38 | ~8 |

#### On Graham (faster interconnect):

| | 30^2 | 650^2 | 2,000^2 | 8,000^2 |
| ----- | ----- | ----- | ----- | ----- |
| baseSolver | 0.0203s | 56s | 565s | 11,140s | 
| parallel3 --nodes=4 --cpus-per-task=2 | 105s | 802s | 1,627s | 13,975s |
| slowdown | ~5,170 | ~14 | ~2.9 | ~1.25 |
| parallel3 --nodes=4 --cpus-per-task=4 | | | | 7,157s |
| parallel3 --nodes=8 --cpus-per-task=4 | | | | 4,096s |

<!-- 16,000^2 on Graham: baseSolver 41,482s; parallel3 --nodes=4 --cpus-per-task=2 61,052s -->

<!-- on Cedar at 650^2 we have ~60X slowdown: 27.5408 seconds and 1658.34 seconds, respectively (bad Chapel build over OmniPath?) -->

<!-- with Chapel 1.17.0 on Graham 650^2 took 21.1198 seconds and 907.352 seconds, respectively -->

<!-- on Cedar 2000^2 baseSolver: The simulation took 469.298 seconds -->

<!-- on Graham 5000^2 took 3697.65 seconds and 6015.98 seconds, respectively -->

## Final parallel code

Here is the final version of the entire code, minus the comments:

~~~
use Time, BlockDist;
config const rows = 100, cols = 100;
config const niter = 500;
config const iout = 1, jout = cols, nout = 20;
config const tolerance = 1e-4: real;
var count = 0: int;
const mesh: domain(2) = {1..rows, 1..cols};
const largerMesh: domain(2) dmapped Block(boundingBox=mesh) = {0..rows+1, 0..cols+1};
var delta: real;
var T, Tnew: [largerMesh] real;   // a block-distributed array of temperatures
T[1..rows,1..cols] = 25;   // the initial temperature
writeln('Working with a matrix ', rows, 'x', cols, ' to ', niter, ' iterations or dT below ', tolerance);
for i in 1..rows do T[i,cols+1] = 80.0*i/rows;   // right-side boundary
for j in 1..cols do T[rows+1,j] = 80.0*j/cols;   // bottom-side boundary
writeln('Temperature at iteration ', 0, ': ', T[iout,jout]);
delta = tolerance*10;   // some safe initial large value
var watch: Timer;
watch.start();
while (count < niter && delta >= tolerance) do {
  count += 1;
  forall (i,j) in largerMesh[1..rows,1..cols] do
    Tnew[i,j] = 0.25 * (T[i-1,j] + T[i+1,j] + T[i,j-1] + T[i,j+1]);
  delta = max reduce abs(Tnew[1..rows,1..cols]-T[1..rows,1..cols]);
  T[1..rows,1..cols] = Tnew[1..rows,1..cols];
  if count%nout == 0 then writeln('Temperature at iteration ', count, ': ', T[iout,jout]);
 }
watch.stop();
writeln('Final temperature at the desired position [', iout,',', jout, '] after ', count, ' iterations is: ', T[iout,jout]);
writeln('The largest temperature difference was ', delta);
writeln('The simulation took ', watch.elapsed(), ' seconds');
~~~

This is the entire multi-locale, data-parallel, hybrid shared-/distributed-memory solver!

> ## Exercise 5
> Add printout to the code to show the total energy on the inner mesh [1..row,1..cols] at each
> iteration. Consider the temperature sum over all mesh points to be the total energy of the system. Is
> the total energy on the mesh conserved?
>
>> ## Solution
>> Just before temperature output (if count%nout == 0), insert the following:
>> ~~~
>>   var total = 0.0;
>>   forall (i,j) in largerMesh[1..rows,1..cols] with (+ reduce total) do
>>     total += T[i,j];
>> ~~~
>> and add total to the temperature output. It is decreasing as energy is leaving the system:
>> ~~~ {.bash}
>> $ chpl --fast parallel3.chpl -o parallel3
>> $ ./parallel3 -nl 1 --rows=30 --cols=30 --niter=2000   # run this from inside distributed.sh
>> Temperature at iteration 0: 25.0
>> Temperature at iteration 20: 3.49566   21496.5
>> Temperature at iteration 40: 2.96535   21052.6
>> ...
>> Temperature at iteration 1100: 2.5809   18609.5
>> Temperature at iteration 1120: 2.58087   18608.6
>> Temperature at iteration 1140: 2.58085   18607.7
>> Final temperature at the desired position [1,30] after 1148 iterations is: 2.58084
>> The largest temperature difference was 9.9534e-05
>> The simulation took 0.114942 seconds
>> ~~~

> ## Exercise 6
> Write a code to print how the finite-difference stencil [i,j], [i-1,j], [i+1,j], [i,j-1], [i,j+1] is
> distributed among nodes, and compare that to the ID of the node where T[i,i] is computed. Use problem
> size 8x8.
>
>> ## Solution
>> Here is one possible solution examining the locality of the finite-difference stencil:
>> ~~~
>> var message: [largerMesh] string = 'empty';
>> ~~~
>> and in the next line after computing Tnew[i,j] put
>> ~~~
>>     message[i,j] = "%i".format(here.id) + message[i,j].locale.id + message[i-1,j].locale.id +
>>       message[i+1,j].locale.id + message[i,j-1].locale.id + message[i,j+1].locale.id + '  ';
>> ~~~
>> and before the end of the `while` loop
>>   writeln(message);
>>   assert(1>2);
>> ~~~
>> Then run it
>> ~~~ {.bash}
>> $ chpl --fast parallel3.chpl -o parallel3
>> $ ./parallel3 -nl 4 --rows=8 --cols=8   # run this from inside distributed.sh
>> ~~~


This produced the following output clearly showing the *ghost points* and the stencil distribution for
each mesh point:

~~~
empty empty empty empty empty empty empty empty empty empty  
empty 000000   000000   000000   000001   111101   111111   111111   111111   empty  
empty 000000   000000   000000   000001   111101   111111   111111   111111   empty  
empty 000000   000000   000000   000001   111101   111111   111111   111111   empty  
empty 000200   000200   000200   000201   111301   111311   111311   111311   empty  
empty 220222   220222   220222   220223   331323   331333   331333   331333   empty  
empty 222222   222222   222222   222223   333323   333333   333333   333333   empty  
empty 222222   222222   222222   222223   333323   333333   333333   333333   empty  
empty 222222   222222   222222   222223   333323   333333   333333   333333   empty  
empty empty empty empty empty empty empty empty empty empty  
~~~

* note that Tnew[i,j] is always computed on the same node where that element is stored
* note remote stencil points at the block boundaries

<!-- ## Periodic boundary conditions -->
<!-- Now let us modify the previous parallel solver to include periodic BCs. At the beginning of each time -->
<!-- step we need to set elements on the *ghost points* to their respective values on the *opposite ends*, by -->
<!-- adding the following to our code: -->
<!-- ~~~ -->
<!--   T[0,1..n] = T[n,1..n]; // periodic boundaries on all four sides; these will run via parallel forall -->
<!--   T[n+1,1..n] = T[1,1..n]; -->
<!--   T[1..n,0] = T[1..n,n]; -->
<!--   T[1..n,n+1] = T[1..n,1]; -->
<!-- ~~~ -->
<!-- Now total energy should be conserved, as nothing leaves the domain. -->

# I/O

Let us write the final solution to disk. There are several caveats:

* we'll write in ASCII
* binary I/O did not work so well for me
<!-- Chapel can also write binary data but nothing can read it (checked: not the endians problem!) -->
* parallel NetCDF/HDF5/etc support is not implemented yet

Let's comment out all lines with `message` and `assert()`, and add the following at the end of our code
to write ASCII:

~~~
var myFile = open('output.dat', iomode.cw);   // open the file for writing
var myWritingChannel = myFile.writer();   // create a writing channel starting at file offset 0
myWritingChannel.write(T);   // write the array
myWritingChannel.close();   // close the channel
~~~
~~~ {.bash}
$ chpl --fast parallel3.chpl -o parallel3
$ ./parallel3 -nl 4 --rows=8 --cols=8   # run this from inside distributed.sh
$ ls -l *dat
-rw-rw-r-- 1 razoumov razoumov 659 Mar  9 18:04 output.dat
~~~

The file *output.dat* should contain the 8x8 temperature array after convergence.

# Ideas for future topics

* binary I/O
  * check https://chapel-lang.org/publications/ParCo-Larrosa.pdf
* write/read NetCDF from Chapel by calling a C/C++ function
* advanced: take a simple non-linear problem, linearize it, implement a parallel multi-locale linear
  solver entirely in Chapel
