<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Task Parallelism with Chapel](#task-parallelism-with-chapel)
  - [Parallel programming in Chapel](#parallel-programming-in-chapel)
  - [Fire-and-forget tasks](#fire-and-forget-tasks)
  - [Synchronization of tasks](#synchronization-of-tasks)
  - [Parallelizing the heat transfer equation](#parallelizing-the-heat-transfer-equation)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Task Parallelism with Chapel

The basic concept of parallel computing is simple to understand: we **divide our job in tasks that can be
executed at the same time**, so that we finish the job in a fraction of the time that it would have taken
if the tasks are executed one by one.  Implementing parallel computations, however, is not always easy.

<!-- A number of misconceptions arises when implementing parallel computations, and we need to address them -->
<!-- before looking into how task parallelism works in Chapel. To this effect, let's consider the following -->
<!-- analogy: -->

<!-- Suppose that we want to paint the four walls in a room. This is our problem. We can divide our problem in -->
<!-- 4 different tasks: paint each of the walls. In principle, our 4 tasks are independent from each other in -->
<!-- the sense that we don't need to finish one to start one another. We say that we have 4 **_concurrent -->
<!-- tasks_**; the tasks can be executed within the same time frame. However, this does not mean that the -->
<!-- tasks can be executed simultaneously or in parallel. It all depends on the amount of resources that we -->
<!-- have for the tasks. If there is only one painter, this guy could work for a while in one wall, then start -->
<!-- painting another one, then work for a little bit in the third one, and so on and so for. **_The tasks are -->
<!-- being executed concurrently but not in parallel_**. If we have 2 painters for the job, then more -->
<!-- parallelism can be introduced. 4 painters could executed the tasks **_truly in parallel_**. -->

<!-- Think of the CPU cores as the painters or workers that will execute your concurrent tasks -->

<!-- Now imagine that all workers have to obtain their paint form a central dispenser located at the middle of -->
<!-- the room. If each worker is using a different colour, then they can work **_asynchronously_**, however, -->
<!-- if they use the same colour, and two of them run out of paint at the same time, then they have to -->
<!-- **_synchronize_** to use the dispenser, one should wait while the other is being serviced. -->

<!-- Think of the shared memory in your computer as the central dispenser for all your workers -->

<!-- Finally, imagine that we have 4 paint dispensers, one for each worker. In this scenario, each worker can -->
<!-- complete its task totally on its own. They don't even have to be in the same room, they could be painting -->
<!-- walls of different rooms in the house, on different houses in the city, and different cities in the -->
<!-- country. We need, however, a communication system in place. Suppose that worker A, for some reason, needs -->
<!-- a colour that is only available in the dispenser of worker B, they should then synchronize: worker A -->
<!-- should request the paint to worker B and this last one should response by sending the required colour. -->

<!-- Think of the memory distributed on each node of a cluster as the different dispensers for your workers -->

- a **_fine-grained_** parallel code needs lots of communication/synchronization between tasks
- a **_coarse-grained_** code requires little communication between tasks
- in this sense **_grain size_** refers to the amount of independent computing in between communication
- an **_embarrassing parallel_** problem is one where all tasks can be executed completely independent
  from each other (no communications required)

## Parallel programming in Chapel

Chapel provides high-level abstractions for parallel programming no matter the grain size of your tasks,
whether they run in a shared memory or a distributed memory environment, or whether they are executed
"concurrently" (frequently switching between tasks) or truly in parallel. As a programmer you can focus
on the algorithm: how to divide the problem into tasks that make sense in the context of the problem, and
be sure that the high-level implementation will run on any hardware configuration. Then you could
consider the specificities of the particular system you are going to use (whether is shared or
distributed, the number of cores, etc.) and tune your code/algorithm to obtain a better performance.

To this effect, **_concurrency_** (the creation and execution of multiple tasks), and **_locality_** (on
which set of resources these tasks are executed) are orthogonal concepts in Chapel.

In summary, we can have a set of several tasks; these tasks could be running, e.g.,
```
a. concurrently by the same processor in a single compute node,
b. in parallel by several processors in a single compute node,
c. in parallel by several processors distributed in different compute nodes, or
d. serially (one by one) by several processors distributed in different compute nodes. 
```
Similarly, each of these tasks could be using variables located in: 
```
a. the local memory on the compute node where it is running, or 
b. on distributed memory located in other compute nodes. 
```

And again, Chapel could take care of all the stuff required to run our algorithm in most of the
scenarios, but we can always add more specific detail to gain performance when targeting a particular
scenario.
 
## Fire-and-forget tasks

A Chapel program always start as a single main thread. You can then start concurrent tasks with the
`begin` statement. A task spawned by the `begin` statement will run in a different thread while the main
thread continues its normal execution. Let's start a new code `begin.chpl` with the following code:

~~~
var x = 0;
writeln("This is the main thread starting first task");
begin {
  var c = 0;
  while c < 10 {
    c += 1;
    writeln('thread 1: ', x + c);
  }
}
writeln("This is the main thread starting second task");
begin {
  var c = 0;
  while c < 10 {
    c += 1;
    writeln('thread 2: ', x + c);
  }
}
writeln('This is main thread, I am done ...');
~~~
~~~
$ chpl begin.chpl -o begin
$ ./begin
~~~
~~~
This is the main thread starting first task
This is the main thread starting second task
This is main thread, I am done ...
thread 1: 1
thread 2: 1
thread 1: 2
thread 2: 2
thread 1: 3
thread 2: 3
...
thread 1: 9
thread 2: 9
thread 1: 10
thread 2: 10
~~~

As you can see the order of the output is not what we would expected, and actually it is completely
unpredictable. This is a well known effect of concurrent tasks accessing the same shared resource at the
same time (in this case the screen); the system decides in which order the tasks could write to the
screen.

> ## Discussion
> What would happen if in the last code we declare `c` in the main thread?
>
>> _Answer_: we'll create another instance of `c` with its own value.
>
> What would happen if we try to modify the value of `x` inside a begin statement?
>
>> _Answer_: can't modify it (error), but we can create another instance of `x` with its own value.

>> ## Key idea
>> All variables have a **_scope_** in which they can be used. The variables declared inside a concurrent
>> task are accessible only by the task. The variables declared in the main task can be read everywhere,
>> but Chapel won't allow two concurrent tasks to try to modify them.

> ## Try this...
> Are the concurrent tasks, spawned by the last code, running truly in parallel?
>
> _Answer_: it depends on the number of cores available to your job. If you have a single core, they'll
> run concurrently, with the CPU switching between the tasks. If you have two cores, thread1 and thread2
> will likely run in parallel using the two cores.

>> ## Key idea
>> To maximize performance, start as many tasks as cores are avaialble

A slightly more structured way to start concurrent tasks in Chapel is by using the `cobegin`
statement. Here you can start a block of concurrent tasks, **one for each statement** inside the curly
brackets. The main difference between the `begin`and `cobegin` statements is that with the `cobegin`, all
the spawned tasks are synchronized at the end of the statement, i.e. the main thread won't continue its
execution until all tasks are done. Let's start `cobegin.chpl`:

~~~
var x = 0;
writeln("This is the main thread, my value of x is ",x);
cobegin {
  {
    var x = 5;
    writeln("This is task 1, my value of x is ", x);
  }
  writeln("This is task 2, my value of x is ", x);
}
writeln("This message won't appear until all tasks are done ...");
~~~
~~~
$ chpl cobegin.chpl -o cobegin
$ ./cobegin
~~~ 
~~~
This is the main thread, my value of x is 0
This is task 2, my value of x is 0
This is task 1, my value of x is 5
This message won't appear until all tasks are done...
~~~

As you may have conclude from the Discussion exercise above, the variables declared inside a task are
accessible only by the task, while those variables declared in the main task are accessible to all tasks.

Another, and one of the most useful ways to start concurrent/parallel tasks in Chapel, is the `coforall`
loop. This is a combination of the for-loop and the `cobegin`statements. The general syntax is:

```
coforall index in iterand 
{instructions}
```

This will start **a new task for each iteration**. Each tasks will then perform all the instructions
inside the curly brackets. Each task will have a copy of the loop variable **_index_** with the
corresponding value yielded by the iterand. This index allows us to _customize_ the set of instructions
for each particular task. Let's write `coforall.chpl`:

~~~
var x = 1;
config var numoftasks = 2;
writeln("This is the main task: x = ", x);
coforall taskid in 1..numoftasks do {
  var c = taskid + 1;
  writeln("this is task ", taskid, ": my value of c is ", c, ' and x is ', x);
}
writeln("This message won't appear until all tasks are done ...");
~~~
~~~
$ chpl coforall.chpl -o coforall
$ ./coforall --numoftasks=5
~~~ 
~~~
This is the main task: x = 1
this is task 2: my value of c is 3 and x is 1
this is task 1: my value of c is 2 and x is 1
this is task 3: my value of c is 4 and x is 1
this is task 5: my value of c is 6 and x is 1
this is task 4: my value of c is 5 and x is 1
This message won't appear until all tasks are done ...
~~~

Notice the random order of the print statements. And notice how, once again, the variables declared
outside the coforall can be read by all tasks, while the variables declared inside, are available only to
the particular task.

> ## Exercise 1
> Would it be possible to print all the messages in the right order? Modify the code in the last example
> as required and save it as `exercise1.chpl`.
>
> Hint: you can use an array of strings declared in the main task, into which all the concurrent tasks
> could write their messages in the right order. Then, at the end, have the main task print all elements
> of the array.
>> ## Solution
>> The following code is a possible solution:
>> ~~~
>> var x = 1;
>> config var numoftasks = 2;
>> var messages: [1..numoftasks] string;
>> writeln("This is the main task: x = ", x);
>> coforall taskid in 1..numoftasks do {
>>   var c = taskid + 1;
>>   var s = "this is task " + taskid + ": my value of c is " + c + ' and x is ' + x;  // add to a string
>>   messages[taskid] = s;
>> }
>> for i in 1..numoftasks do
>>   writeln(messages[i]);
>> writeln("This message won't appear until all tasks are done ...");
>> ~~~
>> ~~~
>> chpl exercise1.chpl -o exercise1
>> ./exercise1 --numoftasks=5
>> ~~~
>> ~~~
>> This is the main task: x = 1
>> this is task 1: my value of c is 2 and x is 1
>> this is task 2: my value of c is 3 and x is 1
>> this is task 3: my value of c is 4 and x is 1
>> this is task 4: my value of c is 5 and x is 1
>> this is task 5: my value of c is 6 and x is 1
>> This message won't appear until all tasks are done ...
>> ~~~

> ## Exercise 2
> Consider the following code `exercise2.chpl`:
> ~~~
> use Random;
> config const nelem = 1e9:int;    // converting real to integer
> var x: [1..nelem] real;
> fillRandom(x);	// fill array with random numbers
> var gmax = 0.0;
>
> // here put your code to find gmax
>
> writeln("the maximum value in x is: ", gmax);
> ~~~
> Write a parallel code to find the maximum value in the array x. Be careful: the number of threads
> should not be excessive. Best to use `numoftasks` to organize parallel loops.
>
>> ## Solution
>> ~~~
>> config const numoftasks = 12;     // let's pretend we have 12 cores
>> const n = nelem / numoftasks;     // number of elements per task
>> const r = nelem - n*numoftasks;   // these did not fit into the last task
>> var lmax: [1..numoftasks] real;
>>
>> coforall taskid in 1..numoftasks do {
>>   var start, finish: int;
>>   start  = (taskid-1)*n + 1;
>>   finish = (taskid-1)*n + n;
>>   if taskid == numoftasks then finish += r;    // add r elements to the last task
>>   for i in start..finish do
>>     if x[i] > lmax[taskid] then lmax[taskid] = x[i];
>>  }
>>
>> for taskid in 1..numoftasks do     // no need for a parallel loop here
>>   if lmax[taskid] > gmax then gmax = lmax[taskid];
>>
>> ~~~
>> ~~~
>> $ chpl --fast exercise2.chpl -o exercise2
>> $ ./exercise2 
>> ~~~
>> ~~~
>> the maximum value in x is: 1.0
>> ~~~
>>
>> We use `coforall` to spawn tasks that work concurrently in a fraction of the array. The trick here is
>> to determine, based on the _taskid_, the initial and final indices that the task will use. Each task
>> obtains the maximum in its fraction of the array, and finally, after the coforall is done, the main
>> task obtains the maximum of the array from the maximums of all tasks.

> ## Discussion
> Run the code of last Exercise using different number of tasks, and different sizes of the array _x_ to
> see how the execution time changes. For example:
> ~~~
> $ time ./exercise2 --nelem=3000 --numoftasks=4
> ~~~
>
> Discuss your observations. Is there a limit on how fast the code could run?
>
>> Answer: (1) consider a small problem, increasing the number of tasks => no speedup
>>         (2) consider a large problem, increasing the number of tasks => speedup up to the physical
>>             number of cores

> ## Try this...
> Substitute the code to find _gmax_ in the last exercise with:
> ~~~
> gmax = max reduce x;
> ~~~
> {:.source}
> Time the execution of the original code and this new one. How do they compare?
>
>> Answer: the built-in runs in parallel utilizing all cores.
>
>> ## Key idea
>> It is always a good idea to check whether there is _built-in_ functions or methods in the used
>> language, that can do what we want as efficiently (or better) than our house-made code. In this case,
>> the _reduce_ statement reduces the given array to a single number using the operation `max`, and it is
>> parallelized (but not really optimized ...).

Finally, we will mention the `forall` loop which is similar to `coforall`, except that it is a _for_ loop
executed by **all cores** in parallel. The number of tasks created is simply equal to the number of cores
available. Consider a simple (and somewhat silly!) code `forall.chpl`:

~~~
var counter = 0;
forall i in 1..1000 with (+ reduce counter) { // go in parallel through all 1000 numbers
  counter = 1;    // set this thread's value to 1 (same for all loop iterations in this task)
}
writeln("actual number of threads = ", counter);
~~~
~~~ {.bash}
$ chpl forall.chpl -o forall
$ ./forall
~~~
~~~
actual number of threads = 2
~~~

Let's modify the code slightly to perform actual computation:

- change `counter = 1;` to `counter += i;`
- change last line to `writeln("counter = ", counter);`

Now it computes the sum of all positive integers up to 1000, using two cores!

`forall` loop will be very useful in Part 3 (Domain parallelism) where we'll use it to create tasks on
all cores on all nodes available to us.

## Synchronization of tasks

The keyword `sync` provides all sorts of mechanisms to synchronize tasks in Chapel. We can simply use
`sync` to force the _parent task_ to stop and wait until its _spawned child-task_ ends. Consider this
`sync1.chpl`:

~~~
var x = 0;
writeln("This is the main thread starting a synchronous task");
sync {
  begin {
    var c = 0;
    while c < 10 {
      c += 1;
      writeln('thread 1: ', x + c);
    }
  }
}
writeln("The first task is done ...");
writeln("This is the main thread starting an asynchronous task");
begin {
  var c = 0;
  while c < 10 {
    c += 1;
    writeln('thread 2: ', x + c);
  }
}
writeln('This is main thread, I am done ...');
~~~
~~~
$ chpl sync1.chpl -o sync1
$ ./sync1 
~~~
~~~
This is the main thread starting a synchronous task
thread 1: 1
thread 1: 2
thread 1: 3
thread 1: 4
thread 1: 5
thread 1: 6
thread 1: 7
thread 1: 8
thread 1: 9
thread 1: 10
The first task is done ...
This is the main thread starting an asynchronous task
This is main thread, I am done ...
thread 2: 1
thread 2: 2
thread 2: 3
thread 2: 4
thread 2: 5
thread 2: 6
thread 2: 7
thread 2: 8
thread 2: 9
thread 2: 10
~~~

> ## Discussion
> What would happen if we swap `sync` and `begin` in the first task:
> ~~~
begin {
  sync {
    var c = 0;
    while c < 10 {
      c += 1;
      writeln('thread 1: ', x + c);
    }
  }
}
writeln("The first task is done ...");
> ~~~
> Discuss your observations. (Answer: `sync` wouldn't do anything.)

> ## Exercise 3
> Use `begin` and `sync` statements to reproduce the functionality of `cobegin` in cobegin.chpl.
>> ## Solution
>> ~~~
>> var x = 0;
>> writeln("This is the main thread, my value of x is ",x);
>>
>> sync {
>>   begin {
>>      var x = 5;
>>      writeln("this is task 1, my value of x is ", x);
>>   }
>>   begin writeln("this is task 2, my value of x is ", x);
>> }
>>
>> writeln("this message won't appear until all tasks are done...");
>> ~~~

A more elaborated and powerful use of `sync` is as a type qualifier for variables. When a variable is
declared as _sync_, a state that can be **_full_** or **_empty_** is associated to it.

To assign a new value to a _sync_ variable,  its state must be _empty_ (after the assignment operation is
completed, the state will be set as _full_). On the contrary, to read a value from a _sync_ variable, its
state must be _full_ (after the read operation is completed, the state will be set as _empty_ again).

~~~
var x: sync int;
writeln("this is main task launching a new task");
begin {
  for i in 1..10 do
    writeln("this is new task working: ", i);
  x = 2;
  writeln("New task finished");
}
writeln("this is main task after launching new task ... I will wait until  it is done");
x;   // not doing anything with a variable, not printing, just calling it
writeln("and now it is done");
~~~
~~~
$ chpl sync2.chpl -o sync2
$ ./sync2
~~~
~~~
this is main task launching a new task
this is main task after launching new task ... I will wait until  it is done
this is new task working: 1
this is new task working: 2
this is new task working: 3
this is new task working: 4
this is new task working: 5
this is new task working: 6
this is new task working: 7
this is new task working: 8
this is new task working: 9
this is new task working: 10
New task finished
and now it is done
~~~

* Let's replace `x;` with `var a = x; writeln(a);` -- now it prints the value!
* Let's add another line `a = x; writeln(a);` -- now it is stuck since we cannot read 'x' while it's empty!

There are a number of methods defined for _sync_ variables. Suppose _x_ is a sync variable of a given type:

~~~
// general methods
x.reset()   //will set the state as empty and the value as the default of x's type
x.isfull()  //will return true is the state of x is full, false if it is empty

// blocking read and write methods
x.writeEF(value)   // will block until the state of x is empty, 
                   // then will assign the value and set the state to full 
x.writeFF(value)       // will block until the state of x is full, 
                       // then will assign the value and leave the state as full
x.readFE()      // will block until the state of x is full, 
                // then will return x's value and set the state to empty
x.readFF()           // will block until the state of x is full, 
                     // then will return x's value and leave the state as full

// non-blocking read and write methods
x.writeXF(value)   // will assign the value no matter the state of x, and then set the state as full
x.readXX()         // will return the value of x regardless its state; the state will remain unchanged
~~~

Chapel also implements **_atomic_** operations with variables declared as `atomic`, and this provides
another option to synchronize tasks. Atomic operations run completely independently of any other thread
or process. This means that when several tasks try to write an atomic variable, only one will succeed at
a given moment, providing implicit synchronization between them. There is a number of methods defined for
atomic variables, among them `sub()`, `add()`, `write()`, `read()`, and `waitfor()` are very useful to
establish explicit synchronization between tasks, as shown in the next code `atomic.chpl`:

~~~
var lock: atomic int;
const numtasks = 5;

lock.write(0);   // the main task set lock to zero

coforall id in 1..numtasks {
  writeln("greetings form task ", id, "... I am waiting for all tasks to say hello");
  lock.add(1);              // task id says hello and atomically adds 1 to lock
  lock.waitFor(numtasks);   // then it waits for lock to be equal numtasks (which will happen when all tasks say hello)
  writeln("task ", id, " is done ...");
}
~~~
~~~
$ chpl atomic.chpl -o atomic
$ ./atomic
~~~
~~~
greetings form task 4... I am waiting for all tasks to say hello
greetings form task 5... I am waiting for all tasks to say hello
greetings form task 2... I am waiting for all tasks to say hello
greetings form task 3... I am waiting for all tasks to say hello
greetings form task 1... I am waiting for all tasks to say hello
task 1 is done...
task 5 is done...
task 2 is done...
task 3 is done...
task 4 is done...
~~~

> ## Try this...
> Comment out the line `lock.waitfor(numtasks)` in the code above to clearly observe the effect of the task synchronization.

Finally, with all the material studied so far, we should be ready to parallelize our code for the
simulation of the heat transfer equation.

## Parallelizing the heat transfer equation

1. divide the entire grid of points into blocks and assign blocks to individual tasks
1. each tasks should compute the new temperature of its assigned points
1. then we must perform a **_reduction_** over the whole grid, to update the greatest difference in temperature

<!-- For the reduction of the grid we can simply use the `max reduce` statement, which is already -->
<!-- parallelized. Now, let's divide the grid into `rowtasks` x `coltasks` subgrids, and assign each subgrid -->
<!-- to a task using the `coforall` loop (we will have `rowtasks * coltasks` tasks in total). -->

Let's start with a simple 1D loop in `parallel1.chpl`:

~~~
const rows = 100;
coforall i in 1..rows do
  writeln(i);
~~~

This will create 100 tasks -- one for each loop iteration -- and will print out the numbers 1..100 in
somewhat random order. Now let's go through 1..100 with a step of 20:

~~~
const rows = 100;
const rowStride = 20;
coforall i in 1..rows by rowStride do
  writeln(i);
~~~
~~~
$ chpl parallel1.chpl -o parallel1
$ ./parallel1
~~~
~~~
1
21
61
41
81
~~~

This created five tasks. Let's make it 2D:

~~~
const rows = 100, cols = 100;
const rowStride = 20, colStride = 20;
coforall (i,j) in {1..rows,1..cols} by (rowStride,colStride) do
  writeln(i, ' ', j);
~~~
~~~
$ chpl parallel1.chpl -o parallel1
$ ./parallel1
~~~
~~~
1 1
1 21
1 61
21 1
21 41
21 81
41 21
...
41 81
61 21
61 61
81 1
81 41
81 81
~~~

This created 25 tasks. Inside each task, we have 20x20 subgrids:

~~~
const rows = 100, cols = 100;
const rowStride = 20, colStride = 20;
coforall (i,j) in {1..rows,1..cols} by (rowStride,colStride) do {
  writeln('on this subgrid, k will run ', i, '..', i+rowStride-1, ' and l will run ', j, '..', j+colStride-1);
 }
~~~
~~~
$ chpl parallel1.chpl -o parallel1
$ ./parallel1
~~~
~~~
on this subgrid, k will run 1..20 and l will run 1..20
on this subgrid, k will run 1..20 and l will run 21..40
on this subgrid, k will run 1..20 and l will run 61..80
on this subgrid, k will run 21..40 and l will run 1..20
on this subgrid, k will run 21..40 and l will run 41..60
on this subgrid, k will run 1..20 and l will run 81..100
on this subgrid, k will run 21..40 and l will run 81..100
on this subgrid, k will run 41..60 and l will run 21..40
on this subgrid, k will run 41..60 and l will run 61..80
...
on this subgrid, k will run 41..60 and l will run 41..60
on this subgrid, k will run 41..60 and l will run 81..100
on this subgrid, k will run 61..80 and l will run 21..40
on this subgrid, k will run 61..80 and l will run 61..80
on this subgrid, k will run 81..100 and l will run 1..20
on this subgrid, k will run 81..100 and l will run 41..60
on this subgrid, k will run 81..100 and l will run 81..100
~~~

Instead of `coforall` in the diffusion solver we'll use `forall`:

~~~
...
delta = 0;   // maximum temperature change from one timestep to next
forall (i,j) in {1..rows,1..cols} by (rowStride,colStride) with (max reduce delta) {
  for k in i..i+rowStride-1 {         // serial loop inside each subgrid over its local k-range
    for l in j..j+colStride-1 do {    // serial loop inside each subgrid over its local l-range
      Tnew[k,l] = (T[k-1,l] + T[k+1,l] + T[k,l-1] + T[k,l+1]) / 4;
      var tmp = abs(Tnew[k,l] - T[k,l]);
      if tmp > delta then delta = tmp;
    }
  }
}
...
~~~

It is still a parallel loop, but instead of creating 25 tasks, it'll create max(numberOfCores,25) tasks,
i.e., 3 tasks on a 3-core system, running multiple loops inside each task. The main advantage of `forall`
-- unlike `coforall` -- here is that it allows the use of `with (max reduce delta)` construct, allowing
us to compute local _delta_ on each task and then pick the largest among them on the main thread. The
calculation will still be synchronized as the reduction operation will force synchronization.

Here is what we do with `baseSolver.chpl` and save it as `parallel1.chpl`:

~~~ {.bash}
$ diff baseSolver.chpl parallel1.chpl
~~~
~~~
  config const rows = 100, cols = 100;
+ const rowStride = 20, colStride = 20;

- var tmp: real;

-   for i in 1..rows do {
-     for j in 1..cols do {
-       Tnew[i,j] = (T[i-1,j] + T[i+1,j] + T[i,j-1] + T[i,j+1])/4;
-     }
-   }

  delta = 0;
- for i in 1..rows do {
-   for j in 1..cols do {
-     tmp = abs(Tnew[i,j] - T[i,j]);
-     if tmp > delta then delta = tmp;
-   }
- }
+ forall (i,j) in {1..rows,1..cols} by (rowStride,colStride) with (max reduce delta) { // 5x5 decomposition into 20x20 blocks => up to 25 tasks
+   for k in i..i+rowStride-1 {         // serial loop inside each subgrid over its local k-range
+     for l in j..j+colStride-1 do {    // serial loop inside each subgrid over its local l-range
+       Tnew[k,l] = (T[k-1,l] + T[k+1,l] + T[k,l-1] + T[k,l+1]) / 4;
+       var tmp = abs(Tnew[k,l] - T[k,l]);
+     }
+   }
+ }

~~~

> ## Exercise 4
> Estimate parallel speedup for a 'largish' problem.
>
>> ## Solution
>> ~~~ {.bash}
>> chpl --fast baseSolver.chpl -o baseSolver
>> ./baseSolver --rows=1000 --cols=1000        # serial run
>> chpl --fast parallel1.chpl -o parallel1
>> ./parallel1 --rows=1000 --cols=1000         # parallel run on all available cores
>> ~~~
>> We get a spedup of ~2X on two cores, which is about expected.

Further possible improvements:
* parallel tasks are launched/terminated at each iteration; in principle, could do that once and iterate
inside












<!-- Note that now the nested for loops run from `row1` to `row2` and from `col1` to `col2` which are, -->
<!-- respectively, the initial and final row and column of the sub-grid associated to the task `taskid`. To -->
<!-- compute these limits, based on `taskid`, we can again follow the same ideas as in Exercise 2. -->

<!-- ~~~ -->

<!-- { -->

<!--     if taskr<rr then -->
<!--     { -->
<!--       row1=(taskr*nr)+1+taskr; -->
<!--       row2=(taskr*nr)+nr+taskr+1; -->
<!--     } -->
<!--     else -->
<!--     { -->
<!--       row1=(taskr*nr)+1+rr; -->
<!--       row2=(taskr*nr)+nr+rr; -->
<!--     } -->

<!--     if taskc<rc then -->
<!--     { -->
<!--       col1=(taskc*nc)+1+taskc; -->
<!--       col2=(taskc*nc)+nc+taskc+1; -->
<!--     } -->
<!--     else -->
<!--     { -->
<!--       col1=(taskc*nc)+1+rc; -->
<!--       col2=(taskc*nc)+nc+rc; -->
<!--     } -->

<!--     for i in row1..row2 do -->
<!--     { -->
<!--       for j in col1..col2 do -->
<!--       { -->
<!-- 	  ... -->
	
<!-- } -->
<!-- ~~~ -->
<!-- {:.source} -->

<!-- As you can see, to divide a data set (the array `temp` in this case) between concurrent tasks, could be cumbersome. Chapel provides high-level abstractions for data parallelism that take care of all the data distribution for us. We will study data parallelism in the following lessons, but for now, let's compare the benchmark solution with our `coforall` parallelization to see how the performance improved.  -->

<!-- ~~~ -->
<!-- $ chpl --fast parallel1.chpl -o parallel1 -->
<!-- $ ./parallel1 --rows=650 --cols=650 --x=200 --y=300 --niter=10000 --tolerance=0.002 --n=1000 -->
<!-- ~~~ -->
<!-- {:.input} -->

<!-- ~~~ -->
<!-- The simulation will consider a matrix of 650 by 650 elements, -->
<!-- it will run up to 10000 iterations, or until the largest difference -->
<!-- in temperature between iterations is less than 0.002. -->
<!-- You are interested in the evolution of the temperature at the position (200,300) of the matrix... -->

<!-- and here we go... -->
<!-- Temperature at iteration 0: 25.0 -->
<!-- Temperature at iteration 1000: 25.0 -->
<!-- Temperature at iteration 2000: 25.0 -->
<!-- Temperature at iteration 3000: 25.0 -->
<!-- Temperature at iteration 4000: 24.9998 -->
<!-- Temperature at iteration 5000: 24.9984 -->
<!-- Temperature at iteration 6000: 24.9935 -->
<!-- Temperature at iteration 7000: 24.9819 -->

<!-- The simulation took 17.0193 seconds -->
<!-- Final temperature at the desired position after 7750 iterations is: 24.9671 -->
<!-- The greatest difference in temperatures between the last two iterations was: 0.00199985 -->
<!-- ~~~ -->
<!-- {:.output} -->

<!-- This parallel solution, using 4 parallel tasks, took around 17 seconds to finish. Compared with the ~20 seconds needed by the benchmark solution, seems not very impressive. To understand the reason, let's analyze the code's flow. When the program starts, the main thread does all the declarations and initializations, and then, it enters the main loop of the simulation (the **_while loop_**). Inside this loop, the parallel tasks are launched for the first time. When these tasks finish their computations, the main task resumes its execution, it updates `delta`, and everything is repeated again. So, in essence, parallel tasks are launched and resumed 7750 times, which introduces a significant amount of overhead (the time the system needs to effectively start and destroy threads in the specific hardware, at each iteration of the while loop).  -->

<!-- Clearly, a better approach would be to launch the parallel tasks just once, and have them executing all the simulations, before resuming the main task to print the final results.  -->

<!-- ~~~ -->
<!-- config const rowtasks=2; -->
<!-- config const coltasks=2; -->

<!-- const nr=rows/rowtasks; -->
<!-- const rr=rows-nr*rowtasks; -->
<!-- const nc=cols/coltasks; -->
<!-- const rc=cols-nc*coltasks; -->

<!-- //this is the main loop of the simulation -->
<!-- delta=tolerance; -->
<!-- coforall taskid in 0..coltasks*rowtasks-1 do -->
<!-- { -->
<!--   var row1, col1, row2, col2: int; -->
<!--   var taskr, taskc: int; -->
<!--   var c=0; -->

<!--   taskr=taskid/coltasks; -->
<!--   taskc=taskid%coltasks; -->

<!--   if taskr<rr then -->
<!--   { -->
<!--     row1=(taskr*nr)+1+taskr; -->
<!--     row2=(taskr*nr)+nr+taskr+1; -->
<!--   } -->
<!--   else -->
<!--   { -->
<!--     row1=(taskr*nr)+1+rr; -->
<!--     row2=(taskr*nr)+nr+rr; -->
<!--   } -->

<!--   if taskc<rc then -->
<!--   { -->
<!--     col1=(taskc*nc)+1+taskc; -->
<!--     col2=(taskc*nc)+nc+taskc+1; -->
<!--   } -->
<!--   else -->
<!--   { -->
<!--     col1=(taskc*nc)+1+rc; -->
<!--     col2=(taskc*nc)+nc+rc; -->
<!--   } -->

<!--   while (c<niter && delta>=tolerance) do    -->
<!--   { -->
<!--     c=c+1; -->
    
<!--     for i in row1..row2 do -->
<!--     { -->
<!--       for j in col1..col2 do -->
<!--       { -->
<!--         temp[i,j]=(past_temp[i-1,j]+past_temp[i+1,j]+past_temp[i,j-1]+past_temp[i,j+1])/4; -->
<!--       } -->
<!--     } -->
        
<!--     //update delta -->
<!--     //update past_temp -->
<!--     //print temperature in desired position -->
<!--   } -->
<!-- } -->
<!-- ~~~ -->
<!-- {:.source} -->

<!-- The problem with this approach is that now we have to explicitly synchronize the tasks. Before, `delta` and `past_temp` were updated only by the main task at each iteration; similarly, only the main task was printing results. Now, all these operations must be carried inside the coforall loop, which imposes the need of synchronization between tasks.  -->

<!-- The synchronization must happen at two points:  -->
<!-- 1. We need to be sure that all tasks have finished with the computations of their part of the grid `temp`, before updating `delta` and `past_temp` safely. -->
<!-- 2. We need to be sure that all tasks use the updated value of `delta` to evaluate the condition of the while loop for the next iteration. -->

<!-- To update `delta` we could have each task computing the greatest difference in temperature in its associated sub-grid, and then, after the synchronization, have only one task reducing all the sub-grids' maximums. -->

<!-- ~~~ -->
<!-- var delta: atomic real; -->
<!-- var myd: [0..coltasks*rowtasks-1] real; -->
<!-- ... -->
<!-- //this is the main loop of the simulation -->
<!-- delta.write(tolerance); -->
<!-- coforall taskid in 0..coltasks*rowtasks-1 do -->
<!-- { -->
<!--   var myd2: real; -->
<!--   ... -->
  
<!--   while (c<niter && delta>=tolerance) do    -->
<!--   { -->
<!--     c=c+1; -->
<!--     ... -->
  
<!--     for i in row1..row2 do -->
<!--     { -->
<!--       for j in col1..col2 do -->
<!--       { -->
<!--         temp[i,j]=(past_temp[i-1,j]+past_temp[i+1,j]+past_temp[i,j-1]+past_temp[i,j+1])/4; -->
<!--         myd2=max(abs(temp[i,j]-past_temp[i,j]),myd2); -->
<!--       } -->
<!--     } -->
<!--     myd[taskid]=myd2     -->
    
<!--     //here comes the synchronization of tasks -->
    
<!--     past_temp[row1..row2,col1..col2]=temp[row1..row2,col1..col2]; -->
<!--     if taskid==0 then -->
<!--     { -->
<!--       delta.write(max reduce myd); -->
<!--       if c%n==0 then writeln('Temperature at iteration ',c,': ',temp[x,y]); -->
<!--     } -->
    
<!--     //here comes the synchronization of tasks again -->
<!--   } -->
<!-- }      -->
<!-- ~~~ -->
<!-- {:.source} -->

<!-- > ## Exercise 4 -->
<!-- > Use `sync` or `atomic` variables to implement the synchronization required in the code above. -->
<!-- >> ## Solution -->
<!-- >> One possible solution is to use an atomic variable as a _lock_ that opens (using the `waitFor` method) when all the tasks complete the required instructions -->
<!-- >> ~~~ -->
<!-- >> var lock: atomic int; -->
<!-- >> lock.write(0); -->
<!-- >> ... -->
<!-- >> //this is the main loop of the simulation -->
<!-- >> delta.write(tolerance); -->
<!-- >> coforall taskid in 0..coltasks*rowtasks-1 do -->
<!-- >> { -->
<!-- >>    ... -->
<!-- >>    while (c<niter && delta>=tolerance) do    -->
<!-- >>    { -->
<!-- >>       ... -->
<!-- >>       myd[taskid]=myd2     -->
<!-- >> -->
<!-- >>       //here comes the synchronization of tasks -->
<!-- >>       lock.add(1); -->
<!-- >>       lock.waitFor(coltasks*rowtasks); -->
<!-- >>        -->
<!-- >>       past_temp[row1..row2,col1..col2]=temp[row1..row2,col1..col2]; -->
<!-- >>       ... -->
<!-- >> -->
<!-- >>       //here comes the synchronization of tasks again -->
<!-- >>       lock.sub(1); -->
<!-- >>       lock.waitFor(0); -->
<!-- >>    } -->
<!-- >> } -->
<!-- >> ~~~ -->
<!-- >> {:.source}  -->
<!-- > {:.solution} -->
<!-- {:.challenge}  -->

<!-- Using the solution in the Exercise 4, we can now compare the performance with the benchmark solution -->

<!-- ~~~ -->
<!-- $ chpl --fast parallel2.chpl -o parallel2 -->
<!-- $ ./parallel2 --rows=650 --cols=650 --x=200 --y=300 --niter=10000 --tolerance=0.002 --n=1000 -->
<!-- ~~~ -->
<!-- {:.input} -->

<!-- ~~~ -->
<!-- The simulation will consider a matrix of 650 by 650 elements, -->
<!-- it will run up to 10000 iterations, or until the largest difference -->
<!-- in temperature between iterations is less than 0.002. -->
<!-- You are interested in the evolution of the temperature at the position (200,300) of the matrix... -->

<!-- and here we go... -->
<!-- Temperature at iteration 0: 25.0 -->
<!-- Temperature at iteration 1000: 25.0 -->
<!-- Temperature at iteration 2000: 25.0 -->
<!-- Temperature at iteration 3000: 25.0 -->
<!-- Temperature at iteration 4000: 24.9998 -->
<!-- Temperature at iteration 5000: 24.9984 -->
<!-- Temperature at iteration 6000: 24.9935 -->
<!-- Temperature at iteration 7000: 24.9819 -->

<!-- The simulation took 4.2733 seconds -->
<!-- Final temperature at the desired position after 7750 iterations is: 24.9671 -->
<!-- The greatest difference in temperatures between the last two iterations was: 0.00199985 -->
<!-- ~~~ -->
<!-- {:.output} -->

<!-- to see that we now have a code that performs 5x faster.  -->

<!-- We finish this section by providing another, elegant version of the task-parallel diffusion solver -->
<!-- (without time stepping) on a single locale: -->

<!-- ~~~ -->
<!-- ~~~ -->
<!-- {:.source} -->
