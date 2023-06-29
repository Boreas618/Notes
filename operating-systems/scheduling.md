# Schedulinng

**Execution model**: programs alternate between bursts of CPU and I/O.

> In the context of computing and operating systems, a "burst" typically refers to a period of continuous execution by a process or thread without giving up the CPU

Each scheduling decision is about which job to give to the CPU for use by its (the job) next CPU burst.

With timeslicing, thread may be forced to give up CPU before finishing current CPU burst.

## Scheduling Policy

### FCFS

**Convoy effect**: short process stuck behind long process.

### Round Robin

$n$ processes in ready queue and time quantum is $q$ : No process waits more than $(n-1)q$ time units.

$q$ must be large with respect to context switch, othewise the overhead is too high.

If one task is done within its quantum, we immediately switch to next task.

Pros: Better for short jobs, Fair

Cons: Context-switching time adds up for long jobs

### Strict Priority Scheduling

<img src="https://p.ipic.vip/pscxpe.png" alt="Screenshot 2023-06-17 at 8.02.24 PM" style="zoom:50%;" />

**Starvation**: Lower priority jobs don't get to run because higher priority jobs.

**Deadlock**: Priority Inversion

* Happens when low priority task has lock needed by high-priority task

* Usually involves third, inntermediate priority task preventing high-priority task from running

  > The whole picture is:
  >
  > The high priority job is waiting for the low priority job to run and release the lock. The medium priority job ios running and will run for a long time.
  >
  > The solution is the high priority job temporarily grants the low priority job its "high priority" to run on its behalf.

How to fix the problems: **Dynamic priorities** We adjust base-level priority up or down based on heuristics about interactivity, locking, burst behavior, etc...

### Shortest Remaining Time First

If all jobs are of the same length, SRTF is the same as FCFS.

Suppose there are three jobs:

* A, B: both CPU bound, run for week
* C: I/O bound, loop 1ms CPU, 9ms disk I/O

<img src="https://p.ipic.vip/5d8waz.png" alt="Screenshot 2023-06-17 at 8.21.57 PM" style="zoom:50%;" />

<img src="https://p.ipic.vip/xi2ywz.png" alt="Screenshot 2023-06-17 at 8.26.00 PM" style="zoom:25%;" />

Pros: Optimal in terms of average response time

Cons: Hard to predict future and unfair

### Lottery Scheduling

Assign tickets:

* Short running jobs get more and long running jobs get fewer.
* To avoid starvation, every job gets at least one ticket.

Advantage over strict priority scheduling: behaves gracefully as load changes

* Adding or deleting a job affects all jobs proportionally, independent of how many tickets each job possesses

**What if too many short jobs to give reasonable response time?**: Short jobs or processes typically require less CPU time to complete. However, if there are too many such jobs, they can overwhelm the system. Even though each job may take a short time, the cumulative effect may lead to longer response times. This is because every time a job starts, the operating system must perform some overhead operations, such as allocating resources, which can become significant with many short jobs.

**If load average is 100, hard to make progress**: Load average is a measure of the amount of computational work that a computer system performs. A load average of 100 is incredibly high, meaning the system is severely overloaded. In such a situation, any scheduling algorithm, including lottery scheduling, will struggle to make significant progress on any single task because it has so many tasks to handle.

### Stride Scheduling 

We can achieve proportional share scheduling without resorting to randomness and overcome the "law of small numbers" problem.

The stride of each job is $\frac{big\#W}{N_i}$. The larger your share of tickets, the smaller your stride. For each job, there's a "pass" counter. The scheduler picks a job with lowest pass and runs it, adds its stride to its pass.

The difference between lottery scheduling and stride scheduling is that the latter ensures predictability.

----

Assumptions encoded into many schedulers:

Apps that sleep a lot and have short bursts must be interactive apps - they should get high priority.

Apps that compute a lot should get lower priority, since they won't notice intermittent bursts from interactive apps.

### Multi-Level Feedback Scheduling

<img src="https://p.ipic.vip/f7awx4.png" alt="Screenshot 2023-06-18 at 5.09.57 AM" style="zoom:50%;" />

Job starts in highest priority queue. If timeout expires, drop one level and if timeout doesn't expire, push up one level.

The result approximates SRTF. CPU bound jobs drop like a rock and short-running I/O bound job stay near top.

Scheduling must be done between the queues:

* Fixed priority scheduling

* Time slice

  Each queue gets a certain amount of CPU time. It may be like 70% to highest, 20% next and 10% lowest.

## Linux O(1) Scheduler

<img src="https://p.ipic.vip/prt58w.png" alt="Screenshot 2023-06-18 at 5.17.55 AM" style="zoom:50%;" />

There are 140 priorities. 40 for "user tasks" and 100 for "realtime/kernel"

All algorithms O(1). Timeslices/priorities/interactivity credits all computed when job finishes time slice.

Two separate priority queues: "active" and "expire"

All tasks in the active queue use up their timeslices and get placed on the expired queue, after which queues swapped.

Timeslice depends on priority - linearly mapped onto timeslice range

There are lots of ad-hoc heuristics. We try to boost priority of I/O-bound tasks and starved tasks.

### Heuristics

The user-task priority adjusted $\pm$ 5 based on heuristics. The sleep time is calculated based on `p->sleep_avg = sleep_time - run_time`. The higher the `sleep_avg`, the more I/O bound the task and the more reward we get.

The **interactive credit** is earned when a task sleeps for a long time and spend when a task runs for a long time. Interactive credit is used to provide hysteresis to avoid changing interactivity for temporary changes in behavior.

The "interactive tasks" get special dispensation. They are simply placed back into active queue unless some other task has been starved for too long.

## Multi-Core Scheduling

**Affinity scheduling**: once a thread is scheduled on a CPU, OS tries to reschedule it on the same CPU. That is cache reuse.

> Spinlocks for multiprocessing
>
> Spinlocks doesn't put the calling thread to sleep, it just busy waits.
>
> This might be preferable when we are waiting for a limited amount of threads at a barrier in a multiprocessing (multicore) program. It guarantees that we don't need to bother waking all the threads up when all the threads end their jobs.
>
> ```python
> import threading
> import time
> 
> # Number of threads
> N = 4
> 
> # Initialize the barrier object with the number of threads
> barrier = threading.Barrier(N)
> 
> def worker():
>     # Simulate some work
>     time.sleep(0.1)
> 
>     # Wait at the barrier
>     # When we enter the loop, we are acutally waiting at the barrier
>     while barrier.n_waiting < N:
>         pass
> 
>     print(f'Thread {threading.current_thread().name} passed the barrier')
> 
> # Create and start N worker threads
> for i in range(N):
>     threading.Thread(target=worker).start()
> ```
>
> Every test&set is a write, which will value ping-pong around between core-local caches.

When multiple threads work together on a multi-core system, try to schedule them together (spread the threads on different cores) .This makes the spin-waiting more efficient. Otherwise if we schedule the threads on a single core we have waste a lot of time on context switching.

OS can inform a parallel program how many processors its threads are scheduled on and applications can adapt to number of cores that it has scheduled.

## Real-Time Scheduling

The goal to the predictability of performance.

**Hard real-time**: meet all deadlines. Ideally we determine in advance if this is possible.

**Soft real-time**: for multi-media.

### Earliest Deadline First

Tasks periodic with period P (arrive every P frames) and computation C in each period for each task i. We adopt a preemptive priority-based dynamic scheduling. Each task is assigned a priority based on how close the absolute deadline is (i.e. $D_i^{t+1}=D_{i}^{t}+P_{i}$ for each task) The scheduler always schedules the active task with the closest absolute deadline.

![Screenshot 2023-06-22 at 12.05.07 PM](https://p.ipic.vip/js984z.png)

EDF won't work if you have too many tasks. For $n$ tasks with computation time $C$ and deadline $D$, a feasible schedule exists if:
$$
\sum_{i=1}^{n} (\frac{C_i}{D_i}) \leq 1
$$

### Strawman: Last-come, First-Served

Starvation is not deadlock but deadlock is starvation. 

A **work-conserving** scheduler is one that does not leave the CPU idle when there is work to do. A non-work-conserving scheduler could trivially lead to starvation.

Stack (LIFO) as a scheduling data structure. It's extremely unfair for possible starvation.

The starvation could happen when arrival rate (offered load) exceeds service rate (delivered load). Queue builds up faster than it drains.

FCFS, priority scheduling, SRTF and MLFS are also prone to starvation.

## Linux Complexity Fair Scheduler

**Goal**: Each process gets a equal share of CPU

In general, can’t do this with real hardware. OS needs to give out full CPU in time slices. Thus, we must use something to keep the threads roughly in sync with one another.

**Basic idea**: is that we track CPU per thread and schedule threads to match up average rate of execution.

**Scheduling Decision**: repair illusion of complete fairness and choose thread with minimum CPU time. It is related to Fair Queueing.

We use a heap-like scheduling queue for this. $O(\log N)$ to add/remove threads where $N$ is the number of threads.

Since short jobs tend to be chosen to run, we automatically get the interactivity.

**Constraint 1**: Target Latency. It is the period of time over whcih every process gets service. The quanta is $\frac{Target Latecncy}{n}$. This ensures low resposnse time ad starvation freedom.

**Constraint 2**: Throughput. It is the minimum length of any time slice.

> Priority in Unix - Being Nice
>
> `nice` values range from -20 to 19
>
> * Negative values are “not nice”
>
> * If you wanted to let your friends get more time, you would nice up your job
>
> Scheduler puts higher nice-value tasks (lower priority) to sleep more …
>
> * In $O(1)$ scheduler, this translated fairly directly to priority (and time slice)

Introduce priority: we assign a weight $w_i$ to each process and compute the switching quanta $Q_i$.

We reuse `nice` value to reflect share. CFS uses nice values to scale weights exponentially: $W=\frac{1024}{{1.25}^{nice}}$. The denominator can be an arbitary number. For example, for two CPU tasks separated by nice value of 5, task with lower nice value has 3 times the weight, since $1.25^{3}\approx3$.

![Screenshot 2023-06-23 at 12.46.26 AM](https://p.ipic.vip/iavdkg.png)

We track a thread's virtual runtime rather than its true physical runtime.

* Higher weight: virtual runtime increases more slowly, so we can get more physical runtime.
* Lower weight: virtual runtime increases more quickly, so we can get less physical runtime.

So the virtual time of the two threads grows at the same weight.

We use red-black tree to hold all runable processes as sorted on `vruntime` variable.

* $O(1)$ time to find next thread to run.
* $O(\log N)$ time to perform insertions/deletions
* When we ready to reschedule, we grab the version with the samllest `vruntime` (which will be the item at the far left)

# Deadlocks

Four requirements for occurrence of Deadlock:

* Mutual exclusion

  Only one thread at a time can use a resource.

* Hold and wait

  Thread holding at least one resource is waiting to acquire additional resources held by other threads

* No preemption

  Resources are released only voluntarily by the thread holding the resource, after thread is finished with it

* Circular wait

 Having a circle doesn't mean that a dead lock exists. 

<img src="https://p.ipic.vip/jalu9q.png" alt="Screenshot 2023-06-23 at 1.20.00 AM" style="zoom:50%;" />

## Detection Algorithm

```pseudocode
[Avail] = [FreeResources]
Add all nodes to UNFINISHED 	
do {
			done = true
      Foreach node in UNFINISHED {
      	if ([Requestnode] <= [Avail]) {
      		remove node from UNFINISHED
        	[Avail] = [Avail] + [Allocnode]
        	done = false
       }
      }
} until(done)
```

Nodes left in `UNFINISHED` are deadlocked.

## Dealing With Deadlock

Four different approaches:

1. Deadlock prevention: write your code in a way that it isn’t prone to deadlock

2. Deadlock recovery: let deadlock happen, and then figure out how to recover from it

3. Deadlock avoidance: dynamically delay resource requests so deadlock doesn’t happen

4. Deadlock denial: ignore the possibility of deadlock

For modern operating systems, we make sure that the system isn't involved in any deadlock. We must ignnore deadlock in applications.

### Preventing Deadlock

* Infinite resources

  Include enough resources so that no one ever runs out of resources.

  Doesn’t have to be infinite, just large

  Give illusion of infinite resources (e.g. virtual memory)

  Examples:

  * Bay bridge with 12,000 lanes. Never wait!
  * Infinite disk space (not realistic yet?)

* No Sharing of resources (totally independent threads)

  Not very realistic

* Don’t allow waiting 

  How the phone company avoids deadlock

  * Call Mom in Toledo, works way through phone network, but if blocked get busy signal. 

  Technique used in Ethernet/some multiprocessor nets

  * Everyone speaks at once. On collision, back off and retry

  Inefficient, since have to keep retrying

  * Consider: driving to San Francisco; when hit traffic jam, suddenly you’re transported back home and told to retry!

* Make all threads request everything they’ll need at the beginning.

  Problem: Predicting future is hard, tend to over-estimate resources

  Example:

  * If need 2 chopsticks, request both at same time
  * Don’t leave home until we know no one is using any intersection between here and where you want to go; only one car on the Bay Bridge at a time

* Force all threads to request resources in a particular order preventing any cyclic use of resources

  Thus, preventing deadlock

  Example `(x.Acquire(), y.Acquire(), z.Acquire(),…)`

  Make tasks request disk, then memory, then…

  Keep from deadlock on freeways around SF by requiring everyone to go clockwise

### Recovery

Roll back the actions of deadlocked threads

### Avoidance

**Safe state**: System can delay resource acquisition to prevent deadlock

**Unsafe state**: No deadlock yet…But threads can request resources in a pattern that **unavoidably** leads to deadlock

**Deadlocked state**: There exists a deadlock in the system. Also considered “unsafe”

The idea is that when a thread requests a resource, OS checks if it would result in an unsafe state.

**Banker's Algorithm for Avoiding Deadlock**

**Basic Idea**: we state the maximum resource needs in advance. We allow particular thread to proceed if: available resources $-$ #requested $\geq$ max

**Banker's Algorithm** (less conservative than the basic idea): allocate resources dynamically. We evaluate each request and grant if some ordering of threads is still deadlock free afterward.

The implementation of the Banker's algorithm lies in pretending each request is granted then run deadlock detection algorithm above. Each thread will have a maximum resource requirement ([Max]), and a currently allocated resource amount ([Alloc]). [Avail] is the amount of resources currently available in the system. We substitute the `([Request] <= [Avail])` with `([Max]-[Alloc] <= [Avail])`

We Keep system in a “SAFE” state: there exists a sequence ${T_1, T_2, … T_n}$ with $T_1$ requesting all remaining resources, finishing, then $T_2$ requesting all remaining resources, etc..
