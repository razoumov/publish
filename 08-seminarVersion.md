<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [A taste of parallel programming with Chapel (in one hour)](#a-taste-of-parallel-programming-with-chapel-in-one-hour)
  - [Task parallelism on a single locale](#task-parallelism-on-a-single-locale)
  - [Data parallelism on a single locale](#data-parallelism-on-a-single-locale)
  - [Running Chapel code on multiple locales](#running-chapel-code-on-multiple-locales)
  - [Data parallelism on multiple locales](#data-parallelism-on-multiple-locales)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

* These notes at http://bit.ly/ch1hr

# A taste of parallel programming with Chapel (in one hour)

**Abstract**: Chapel is a relatively new high-level programming language for shared- and
distributed-memory machines. It combines the ease-of-use of Python and the speed of C++ and is the
perfect language to learn the basics of parallel programming, whether you are trying to accelerate your
computation on a multi-core laptop or on an HPC cluster. In this one-hour hands-on introduction I will go
over several of Chapel's high-level abstractions.

**Requirements**: All attendees will need to bring their laptops with a remote ssh client installed (on
Windows I recommend the free edition of https://mobaxterm.mobatek.net/download.html; on Mac and Linux no
need to install anything). No need to install Chapel -- we'll play with it on a cluster, and I will
provide guest accounts.

You can find a much longer version of these notes at:

* basic language features http://bit.ly/2CDRuxQ
* task parallelism http://bit.ly/2CDHCUS
* data parallelism http://bit.ly/2CC8MLW

## Task parallelism on a single locale

We'll use a Slurm cluster inside a VM:

~~~ {.bash}
ssh user01@206.12.100.115   # user01..user10
$ . /home/centos/startSingleLocale.sh
$ which chpl
$ mkdir tmp && cd tmp
~~~

A Chapel program always starts as a single main thread. You can then start concurrent tasks with the
`begin` statement. A task spawned by the `begin` statement will run in a different thread while the main
thread continues its normal execution. Let's start a new code `begin.chpl` with the following lines:

~~~
var x = 100;
writeln('This is the main thread starting first task');
begin {
  var count = 0;
  while count < 10 {
    count += 1;
    writeln('thread 1: ', x + count);
  }
}
writeln('This is the main thread starting second task');
begin {
  var count = 0;
  while count < 10 {
    count += 1;
    writeln('thread 2: ', x + count);
  }
}
writeln('This is the main thread, I am done ...');
~~~
~~~ {.bash}
$ chpl begin.chpl -o begin
$ ./begin
~~~

If we have many cores, this would normally run on three cores. In our case, I don't know how many cores
our code ran on -- most likely on two on our login node. Let us submit a job to the cluster asking for
two cores on a single node (one MPI task, two cores inside), by writing `job.sh`:

~~~ {.bash}
#!/bin/bash
#SBATCH --cpus-per-task=2   # number of cores per task
#SBATCH --mem-per-cpu=1000
#SBATCH --time=00:5:00   # maximum walltime
./begin
~~~
~~~ {.bash}
$ sbatch job.sh
$ squeue -u user01 
~~~

The command `coforall` will start a new task for each iteration. Each tasks will then perform all the
instructions inside the curly brackets, and then all tasks will synchronize at the end. Let's write
`coforall.chpl`:

~~~
var x = 10;
config var numtasks = 2;
writeln('This is the main task: x = ', x);
coforall taskid in 1..numtasks do {
  var c = taskid**2;
  writeln('this is task ', taskid, ': my value of c is ', c, ' and x is ', x);
}
writeln('This message will not appear until all tasks are done ...');
~~~
~~~ {.bash}
$ chpl coforall.chpl -o coforall
$ sed -i -e 's|./begin|./coforall|' job.sh
$ sbatch job.sh
$ squeue -u user01
~~~

Let's modify the number of threads:

~~~ {.bash}
$ sed -i -e 's|./coforall|./coforall --numtasks=5|' job.sh
$ sbatch job.sh
$ squeue -u user01
~~~

Messages appear in random order since they run in parallel in two threads. Let's make them appear in the
right order, by writing from each thread into a string, and then printing this string in serial:

~~~ {.bash}
2a3
> var messages: [1..numtasks] string;
6c7
<   writeln('this is task ', taskid, ': my value of c is ', c, ' and x is ', x);
---
>   messages[taskid] = 'this is task ' + taskid + ': my value of c is ' + c + ' and x is ' + x;
8a10,11
> for i in 1..numtasks do  // serial loop, will be printed in sequential order
>   writeln(messages[i]);
~~~

## Data parallelism on a single locale

As we mentioned in the previous section, **Data Parallelism** is a style of parallel programming in which
parallelism is driven by *computations over collections of data elements or their indices*. The main tool
for this in Chapel is a `forall` loop -- it'll create an *appropriate* number of tasks to execute a loop,
dividing the loop's iterations between them.

What is the *appropriate* number of tasks?
* on a single core: single task
* on multiple cores on the same nodes: all cores, up to the number of elements or iterations
* on multiple cores on multiple nodes: all cores, up to the problem size, given the data distribution

Consider a simple code `array.chpl`:

~~~
const n = 1e6: int;
var A: [1..n] real;
forall a in A do
  a += 1;
~~~

We will not run it, as it does not print anything, but let's study what it does. We update all elements
of the array `A`. The code will run on a single node, lauching as many tasks as the number of available
cores.

* if we replace `forall` with `for`, we'll get a serial loop on a sigle core
* if we replace `forall` with `coforall`, we'll create 1e6 tasks (definitely an overkill!)

Consider a simple code `forall.chpl` that we'll run inside our 2-core job. We have a range of
indices 1..1000, and they get broken into groups that are processed by different tasks:

~~~
var count = 0;
forall i in 1..1000 with (+ reduce count) {   // parallel loop
  count += i;
}
writeln('count = ', count);
~~~
~~~ {.bash}
$ chpl forall.chpl -o forall
$ sed -i -e 's|./coforall --numtasks=5|./forall|' job.sh
$ sbatch job.sh
$ squeue -u user01
~~~

We get a sum 1..1000 which is 500500.

> ## Exercise
> We computed the sum of integers from 1 to 1000 in parallel. How many cores did the code run on? Looking
> at the code or its output, **we don't know**. Most likely, on two cores available to us inside the
> job. But we can actually check that!
>
>> ## Solution
>> (1) replace `count += i;` with `count = 1;`  
>> (2) change the last line to `writeln('actual number of threads = ', count);`
>> ~~~ {.bash}
>> $ chpl forall.chpl -o forall
>> $ sbatch job.sh
>> $ squeue -u user01
>> ~~~

We should get the actual CPU count!

## Running Chapel code on multiple locales

~~~ {.bash}
$ . /home/centos/startMultiLocale.sh
$ which chpl
~~~

Let's start by writing the following code `locales.chpl`:

~~~
writeln(Locales);
~~~
~~~ {.bash}
$ chpl locales.chpl -o locales   # notice now we have two executables!
$ nano job.sh
  add line `#SBATCH --nodes=2` to
  replace `./forall` with `./locales`
$ sbatch job.sh
$ squeue -u user01
~~~

We get an error message! Need to specify the number of locales!

~~~ {.bash}
$ sed -i -e 's|./locales|./locales -nl 2|' job.sh
$ sbatch job.sh
$ squeue -u user01
~~~

Ignore the warnings and study the output:
~~~ {.bash}
LOCALE0 LOCALE1
~~~

Now let's add this to our code 'locales.chpl`:

~~~
for loc in Locales do   // this is still a serial program
  on loc do             // run the next line on locale `loc`
    writeln("this locale is named ", here.name[1..6]);   // `here` is the locale on which the node is running
~~~
~~~ {.bash}
$ chpl locales.chpl -o locales
$ sbatch job.sh
$ squeue -u user01
~~~

Now let's replace the last `for` loop with the following:

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
~~~ {.bash}
$ chpl locales.chpl -o locales
$ sbatch job.sh
$ squeue -u user01
~~~

## Data parallelism on multiple locales

In a new code `domains.chpl`, let us define an n^2 domain called `mesh`. It is defined by the single task
in our code and is therefore defined in memory on the same node (locale 0) where this task is
running. For each of n^2 mesh points, let us print out

1. m.locale.id = the ID of the locale holding that mesh point (should be 0)  
1. here.id = the ID of the locale on which the code is running (should be 0)  
1. here.maxTaskPar = the number of available cores (max parallelism with 1 task/core) (should be 2)  

~~~
config const n = 8;
const mesh: domain(2) = {1..n, 1..n};  // a 2D domain defined in shared memory on a single locale
forall m in mesh do   // go in parallel through all n^2 mesh points
  writeln(m, ' ', m.locale.id, ' ', here.id, ' ', here.maxTaskPar);
~~~  
~~~ {.bash}
$ chpl domains.chpl -o domains
$ sed -i -e 's|./locales -nl 2|./domains -nl 2|' job.sh
$ sbatch job.sh
$ squeue -u user01
~~~
~~~ {.bash}
(1, 1) 0 0 2
(5, 1) 0 0 2
(1, 2) 0 0 2
(5, 2) 0 0 2
(1, 3) 0 0 2
...
~~~

Now, let's introduce two important properties of Chapel domains:

1. Domains can be used to define arrays of variables of any type on top of them.
1. Domains can span multiple locales.

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
  a = a.locale.id + '-' + here.name[1..5] + '-' + here.maxTaskPar + '  ';
}
writeln(A);
~~~~
~~~ {.bash}
$ chpl domains.chpl -o domains
$ sbatch job.sh
$ squeue -u user01
~~~
~~~ {.bash}
0-node1-2   0-node1-2   0-node1-2   0-node1-2   0-node1-2   0-node1-2   0-node1-2   0-node1-2
0-node1-2   0-node1-2   0-node1-2   0-node1-2   0-node1-2   0-node1-2   0-node1-2   0-node1-2
0-node1-2   0-node1-2   0-node1-2   0-node1-2   0-node1-2   0-node1-2   0-node1-2   0-node1-2
0-node1-2   0-node1-2   0-node1-2   0-node1-2   0-node1-2   0-node1-2   0-node1-2   0-node1-2
1-node2-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2
1-node2-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2
1-node2-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2
1-node2-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2
~~~

Finally, let us count the number of threads by adding the following to our code:

~~~
var count = 0;
forall a in A with (+ reduce count) { // go in parallel through all n^2 elements
  count = 1;
}
writeln("actual number of threads = ", count);
~~~
~~~ {.bash}
$ chpl domains.chpl -o domains
$ sbatch job.sh
$ squeue -u user01
~~~
~~~ {.bash}
...
actual number of threads = 4
~~~
