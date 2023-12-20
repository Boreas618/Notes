# Scheduling

**Execution model**: Programs alternate between bursts of CPU and I/O.

> In the context of computing and operating systems, a "burst" typically refers to a period of continuous execution by a process or thread without giving up the CPU

Each scheduling decision is about which job to give to the CPU for use by its (the job) next CPU burst (execution time).

With timeslicing, thread may be forced to give up CPU before finishing current CPU burst (execution time).

## Scheduling Policy

There are three types of scheduling:

- **Long-term scheduling**: Determines whether a job can enter the operating system and controls the degree of multiprogramming.
- **Medium-term scheduling**: Swaps processes between suspended and active states to regulate resource utilization.
- **Short-term scheduling**: Allocates processor resources to processes (or threads) in the ready queue.

<img src="https://p.ipic.vip/tpmfhr.png" alt="Screenshot 2023-10-16 at 2.05.30 PM" style="zoom:50%;" />

## Simple Algorithms

### FCFS

**Non-preemptive**: favor CPU-bound processes over I/O-bound processes. favor long jobs over short jobs.

**Convoy effect**: short process stuck behind long process.

### Round Robin

$n$ processes in ready queue and time quantum is $q$ : No process waits more than $(n-1)q$ time units.

$q$ must be large with respect to context switch, othewise the overhead is too high.

**If one task is done within its quantum, we immediately switch to next task.** We just don't want to waste any resource.

**Pros**: Better for short jobs and fair

**Cons**: Context-switching time adds up for long jobs. Poor performance for I/O-bound processes.

**A Refinement**: Virtual Round Robin. Processes are moved into this queue after being released from an I/O block. When a dispatching decision is to be made, processes in auxiliary queue get preference over those in the main Ready queue.

### Shortest Job First

If all jobs are of the same length, SJF is the same as FCFS.

**Pros**: **Optimal** in terms of average response time and average turnaround time.

**Cons**: Hard to predict future and unfair. Favor short jobs over long jobs. No notion of "priority": a long task may be more urgent than a short task.

Due to the unpredictable nature of SJF, it is more suitable for long-term scheduling, where the job time is more predictable.

SJF can be preemptive and non-preemptive at the same time. The preemptive version is called **S**hortest **R**emaining **J**ob **F**irst.

<img src="https://p.ipic.vip/c0p3l5.png" alt="Screenshot 2023-12-17 at 6.01.43 AM" style="zoom: 25%;" />

### High Response Ratio First

**Normalized turnaround time**: the ratio of turn around time to service time.
$$
R=\frac{\text{Waiting Time}+\text{Expected Service Time}}{\text{Expected Service Time}}
$$

**Pros**: balance short jobs and starving jobs at the same time. **Not prone to starvation.**

**Cons**: have to predict the future.

### Multi-Level Feedback Scheduling

<img src="https://p.ipic.vip/f7awx4.png" alt="Screenshot 2023-06-18 at 5.09.57 AM" style="zoom:50%;" />

Job starts in highest priority queue.

* If timeout expires, drop one level,
* If timeout doesn't expire, push up one level.

The result **approximates SRTF**. CPU bound jobs drop like a rock and short-running I/O bound job stay near top.

Scheduling must be done between the queues:

* **Fixed priority scheduling**

* **Time slice**: Each queue gets a certain amount of CPU time. It may be like 70% to highest, 20% next and 10% lowest.

## Priority-Based Approaches

* To prevent high-priority processes from running indefinitely, the scheduler may **decrease the priority** of the currently running process at each clock interrupt.

* **Highly I/O bound processes** (interactive processes) should be given higher priorities.

* A **preemptive priority scheduling** algorithm will preempt the CPU if the priority of the newly arrived process is higher than the priority of the currently running process

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

* Solution: **Dynamic priorities** We adjust base-level priority up or down based on heuristics about interactivity, locking, burst behavior, etc.

### Starvation

Starvation is not deadlock but deadlock is starvation. 

A **work-conserving** scheduler is one that does not leave the CPU idle when there is work to do. A non-work-conserving scheduler could trivially lead to starvation.

The starvation could happen when arrival rate (offered load) exceeds service rate (delivered load). Queue builds up faster than it drains. Thus, FCFS, priority scheduling, SRTF and MLFS are also prone to starvation.

<img src="https://p.ipic.vip/gr3d12.png" alt="Screenshot 2023-12-17 at 6.57.38 AM" style="zoom: 33%;" />

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

## Multi-Core Scheduling

Two categories: **single-queue scheduling** strategies and **multi-queue scheduling** strategies.

**Affinity scheduling**: Once a thread is scheduled on a CPU, OS tries to reschedule it on the same CPU. That is for cache reuse.

**Gang Scheduling**: When multiple threads work together on a multi-core system, try to schedule them together (spread the threads on different cores) .This makes the spin-waiting more efficient. Otherwise if we schedule the threads on a single core we have waste a lot of time on context switching.

**Dedicated Scheduling**: A program is made up of multiple threads. Several processors are allocated them. Each thread is dedicated to one processor thoroughout its life cycle.

## Real-Time Scheduling

The goal to the predictability of performance.

**Hard real-time**: meet all deadlines. Ideally we determine in advance if this is possible.

**Soft real-time**: for multi-media.

Tasks in real time systems can be categorized as:

* **Periodic Task**
* **Non-periodic Task**

### Earliest Deadline First

Periodic Tasks with period $P$ (arrive every $P$ frames) and computation $C$ in each period for each task $i$. We adopt a preemptive priority-based dynamic scheduling. Each task is assigned a priority based on how close the absolute deadline is (i.e. $D_i^{t+1}=D_{i}^{t}+P_{i}$ for task $i$) The scheduler always schedules the active task with the closest absolute deadline.

<img src="https://p.ipic.vip/js984z.png" alt="Screenshot 2023-06-22 at 12.05.07 PM" style="zoom:50%;" />

EDF won't work if you have too many tasks. For $n$ tasks with computation time $C$ and deadline $D$, a feasible schedule exists if:
$$
\sum_{i=1}^{n} \frac{C_i}{D_i} \leq 1
$$

### Rate-Monotonic Scheduling

The frequency of a periodic real-time task with a cycle is $p$ . Therefore, the priority of a group of periodic real-time tasks can be calculated based on the frequency of that task.

<img src="https://p.ipic.vip/d1tyjy.png" alt="Screenshot 2023-12-17 at 6.53.58 AM" style="zoom: 33%;" />

Rate monotonic scheduling optimally schedules processes based on static priority. If a group of processes can't be scheduled using this method, they won't meet hard real-time requirements with any other scheduling strategies.
