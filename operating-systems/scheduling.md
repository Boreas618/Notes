# **Execution model**

Programs alternate between bursts of CPU and I/O.

> In the context of computing and operating systems, a "burst" typically refers to a period of continuous execution by a process or thread without giving up the CPU

Each scheduling decision is about which job to give to the CPU for use by its (the job) next CPU burst (execution time).

With timeslicing, thread may be forced to give up CPU before finishing current CPU burst (execution time).

# Scheduling Policy

## Simple Algorithms

### FCFS

**Convoy effect**: short process stuck behind long process.

### Round Robin

$n$ processes in ready queue and time quantum is $q$ : No process waits more than $(n-1)q$ time units.

$q$ must be large with respect to context switch, othewise the overhead is too high.

**If one task is done within its quantum, we immediately switch to next task.** We just don't want to waste any resource.

Pros: Better for short jobs, Fair

Cons: Context-switching time adds up for long jobs

### Shortest Remaining Time First

If all jobs are of the same length, SRTF is the same as FCFS.

Suppose there are three jobs:

* A, B: both CPU bound, run for week
* C: I/O bound, loop 1ms CPU, 9ms disk I/O

<img src="https://p.ipic.vip/5d8waz.png" alt="Screenshot 2023-06-17 at 8.21.57 PM" style="zoom:50%;" />

<img src="https://p.ipic.vip/xi2ywz.png" alt="Screenshot 2023-06-17 at 8.26.00 PM" style="zoom:25%;" />

**Pros**: **Optimal** in terms of average response time

**Cons**: Hard to predict future and unfair

### Multi-Level Feedback Scheduling

<img src="https://p.ipic.vip/f7awx4.png" alt="Screenshot 2023-06-18 at 5.09.57 AM" style="zoom:50%;" />

Job starts in highest priority queue. If timeout expires, drop one level and if timeout doesn't expire, push up one level.

The result approximates SRTF. CPU bound jobs drop like a rock and short-running I/O bound job stay near top.

Scheduling must be done between the queues:

* Fixed priority scheduling

* Time slice

  Each queue gets a certain amount of CPU time. It may be like 70% to highest, 20% next and 10% lowest.

## Priority-Based Approach

### Strict Priority Scheduling

<img src="https://p.ipic.vip/pscxpe.png" alt="Screenshot 2023-06-17 at 8.02.24 PM" style="zoom:50%;" />

**Starvation**: Lower priority jobs don't get to run because higher priority jobs.

**Deadlock**: Priority Inversion

* Happens when low priority task has lock needed by high-priority task

* Usually involves third, inntermediate priority task preventing high-priority task from running. Otherwise, the dead lock can be easily resolved.

  > The whole picture is:
  >
  > The high priority job is waiting for the low priority job to run and release the lock. The medium priority job is running and will run for a long time.
  >
  > The solution is the high priority job temporarily grants the low priority job its "high priority" to run on its behalf.

* Solution: **Dynamic priorities** We adjust base-level priority up or down based on heuristics about interactivity, locking, burst behavior, etc...

### Case Study: Linux O(1) Scheduler

<img src="https://p.ipic.vip/prt58w.png" alt="Screenshot 2023-06-18 at 5.17.55 AM" style="zoom:50%;" />

**Priority Queues and Time Slices**： In the Linux O(1) scheduler, tasks are organized based on their priority, which is categorized into 140 distinct levels:

- **User Tasks**: These are tasks that are initiated by users, allocated 40 out of the 140 priorities.
- **Realtime/Kernel Tasks**: These are tasks that are either kernel-centric or require real-time processing. They're assigned the remaining 100 priorities.

The scheduler employs two distinct priority queues:

1. **Active Queue**: This is where tasks are initially placed and are allowed to use up their allocated timeslices.
2. **Expired Queue**: Once tasks exhaust their timeslices in the active queue, they are moved to the expired queue.

After all tasks in the active queue have consumed their timeslices, the roles of the active and expired queues are swapped, allowing for a continuous execution of tasks without delay.

The duration of a task's timeslice is directly proportional to its priority. That is, higher-priority tasks are allotted longer timeslices. The scheduler maps these priorities linearly onto a predefined timeslice range.

**Heuristic-based Priority Adjustments**: A unique aspect of the Linux O(1) scheduler is its ability to adapt to different types of tasks. It employs a variety of heuristics to fine-tune task priorities. The primary goal of these heuristics is to ensure I/O-bound tasks and tasks that have been starved of CPU time receive priority boosts.

The user-task priority adjusted $\pm$ 5 based on heuristics. The sleep time is calculated based on `p->sleep_avg = sleep_time - run_time`. The higher the `sleep_avg`, the more I/O bound the task and the more reward we get.

The **interactive credit** is earned when a task sleeps for a long time and spend when a task runs for a long time. Interactive credit is used to provide hysteresis to avoid changing interactivity for temporary changes in behavior.

The "interactive tasks" get special dispensation. They are simply placed back into active queue unless some other task has been starved for too long.

### Drawback: Starvation

Starvation is not deadlock but deadlock is starvation. 

A **work-conserving** scheduler is one that does not leave the CPU idle when there is work to do. A non-work-conserving scheduler could trivially lead to starvation.

Stack (LIFO) as a scheduling data structure. It's extremely unfair for possible starvation.

The starvation could happen when arrival rate (offered load) exceeds service rate (delivered load). Queue builds up faster than it drains.

FCFS, priority scheduling, SRTF and MLFS are also prone to starvation.

## Proportional-Share Scheduling

The policies we’ve covered: **Always prefer to give the CPU to a prioritized job **and Non-prioritized jobs may never get to run. Instead, we can share the CPU propotionally.

### Lottery Scheduling

**Assign tickets**:

* Short running jobs get more and long running jobs get fewer.
* To avoid starvation, every job gets at least one ticket.

**Advantage over strict priority scheduling**: behaves gracefully as load changes without leading to problems such as starvation.

* Adding or deleting a job affects all jobs proportionally, independent of how many tickets each job possesses

### Stride Scheduling 

We can achieve proportional share scheduling without resorting to randomness and overcome the "law of small numbers" problem.

The stride of each job is $\frac{big\#W}{N_i}$. The total number of tickets across all jobs is represented by `big#W`. Each job has its own number of tickets represented by `N_i`. The larger your share of tickets, the smaller your stride. For each job, there's a "pass" counter. The scheduler picks a job with lowest pass and runs it, adds its stride to its pass.

The difference between lottery scheduling and stride scheduling is that the latter ensures predictability.

> Assumptions encoded into many schedulers:
>
> Apps that compute a lot should get lower priority, since they won't notice intermittent bursts from interactive apps.
>
> Apps that sleep a lot and have short bursts must be interactive apps - they should get high priority.

### Case Study: Linux Completely Fair Scheduler

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

# Special Scheduling Scenarios

## Multi-Core Scheduling

**Affinity scheduling**: once a thread is scheduled on a CPU, OS tries to reschedule it on the same CPU. That is for cache reuse.

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

**Challenges with too many short jobs**: Short jobs or processes typically require less CPU time to complete. However, if there are too many such jobs, they can overwhelm the system. Even though each job may take a short time, the cumulative effect may lead to longer response times. This is because every time a job starts, the operating system must perform some overhead operations, such as allocating resources, which can become significant with many short jobs.

**Challenges with high load average**: Load average is a measure of the amount of computational work that a computer system performs. A load average of 100 is incredibly high, meaning the system is severely overloaded. In such a situation, any scheduling algorithm, including lottery scheduling, will struggle to make significant progress on any single task because it has so many tasks to handle.

## Real-Time Scheduling

The goal to the predictability of performance.

**Hard real-time**: meet all deadlines. Ideally we determine in advance if this is possible.

**Soft real-time**: for multi-media.

### Earliest Deadline First

Tasks periodic with period P (arrive every P frames) and computation C in each period for each task i. We adopt a preemptive priority-based dynamic scheduling. Each task is assigned a priority based on how close the absolute deadline is (i.e. $D_i^{t+1}=D_{i}^{t}+P_{i}$ for each task) The scheduler always schedules the active task with the closest absolute deadline.

![Screenshot 2023-06-22 at 12.05.07 PM](https://p.ipic.vip/js984z.png)

EDF won't work if you have too many tasks. For $n$ tasks with computation time $C$ and deadline $D$, a feasible schedule exists if:
$$
\sum_{i=1}^{n} \frac{C_i}{D_i} \leq 1
$$

### Last-come, First-Served

Use stack (LIFO) as a scheduling data structure. Late arrivals get fast service and early ones just wait – extremely unfair.

When would starvation occur ? When arrival rate (offered load) exceeds service rate (delivered load) and the queue builds up faster than it drains.

Queue can build in FIFO too, but “serviced in the order received”…

