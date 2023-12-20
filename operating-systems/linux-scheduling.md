# Case Study: Linux Scheduling

## Priority and Weight

There is a user-space variable called `nice` that indicates the priority of ordinary processes. This variable ranges from -20 to +19 and is mapped to a priority range of 100 to 139. The lower the value of `nice`, the higher the priority of the process.

There are 4 members of `task_struct` indicating the priority of the task.

* `prio` is the dynamic priority of the process which can be adjusted to tackle priority donation.
* `static_prio` is the priority set when the task starts.
* `normal_prio` is calculated based on `static_prio` and scheduling policy. For ordinary processes, it is the same as `static_prio`. For real time processes, it will be calculated according to `rt_priority`
* `rt_priority` is the priority of real time processes.

The schedulers also adopts the concept of 'weight' to denote the urgency of different processes. In the CFS, each task is assigned a weight. The default weight for a normal priority task (nice value of 0) is defined as a specific value, e.g., 1024. Tasks with higher priority (negative nice values) will have higher weights, while those with lower priority (positive nice values) will have lesser weights. The scheduler uses these weights to calculate the length of time each task should run on the CPU. A task with double the weight of another should, in theory, get double the CPU time.

## Policy

There are five scheduler classes implemented in Linux kernel right now: stop, deadline, realtime, CFS, idle. Each of these scheduler classes defines its own scheduling policy and determines how processes in that class will be scheduled on the CPU.

1. **Stop Scheduler Class**: This is the highest priority class. Tasks in the stop class are special tasks that need to run immediately. They are designed to stop other tasks and must not be preempted by any other task. For example, tasks that handle critical operations like system shutdown would fall into this category.
2. **Deadline Scheduler Class**: As the name suggests, this scheduling class is focused on tasks that have strict timing requirements. Tasks with deadline requirements are guaranteed to be executed before a specified deadline. This scheduler uses the Earliest Deadline First (EDF) algorithm. Tasks in this class specify their runtime and deadline requirements.
3. **Realtime Scheduler Class**: Realtime tasks are divided into two categories, depending on their scheduling policies: SCHED_FIFO (First In, First Out) and SCHED_RR (Round Robin). 
   - **SCHED_FIFO**: A task scheduled with SCHED_FIFO will run until it relinquishes the CPU, either because it completes or because it is forced to yield.
   - **SCHED_RR**: A task scheduled with SCHED_RR gets a fixed time slice to run. Once this time slice is used up, it is moved to the back of the queue, allowing the next task in line to execute.
4. **CFS (Completely Fair Scheduler)**: This is the default Linux process scheduler. The CFS tries to ensure a fair distribution of the CPU processing power among all tasks. It uses a time-ordered Red-Black tree to track task execution and employs the concept of 'virtual runtime' to decide which task should run next. The less a task has run, the higher its priority becomes. Policies: **SCHED_NORMAL**, **SCHED_BATCH**, **SCHED_IDLE**.
5. **Idle Scheduler Class**: This class is for tasks that run when there is absolutely no other work that can be done. It’s the lowest priority class and runs tasks when the CPU is idle.

POSIX interface:

```c
#include <sched.h>

int sched_setscheduler(pid_t pid, int policy, const struct sched_param *param);
int sched_setparam(pid_t pid, const struct sched_param *param);
```

## Multi-level Feedback Queue

Rules:

1. **Priority-based Scheduling:** If Process A has a higher priority than Process B, the scheduler will choose Process A.
2. **Round Robin (RR) within the Same Priority:** If Process A and Process B have the same priority and are in the same queue, the Round Robin (RR) scheduling algorithm is used to decide which process runs next.
3. **Initial Queue Placement:** When a process enters the scheduler for the first time, it is placed in the highest priority queue.
4. **Demotion for Full Time Slice Usage:** If a process consumes its entire time slice without yielding, it is moved to a lower priority queue.
5. **Maintain Queue Position on Early Yield:** If a process yields the CPU before its time slice is fully consumed, it remains in its current queue.

Some modifications to tackle edge cases:

1. Every time period T, the priority of all processes is raised to the highest level. (To avoid starvation)
2. Once the process uses up its time slice, it's priority is lowered even if it initiates an I/O request.

## Tuning and Configuration

Tuning the scheduler can lead to performance improvements for specific workloads. Linux provides various tools and interfaces, such as `sysctl` and `/proc` filesystem, to adjust scheduler parameters. Additionally, tools like `chrt` can be used to change real-time attributes of processes.

## Linux O(1) Scheduler

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

## Linux CFS

The **Completely Fair Scheduler (CFS)** is the default process scheduler in the Linux kernel. Its main objective is to ensure fair access to the CPU for all tasks, providing a good mix of interactive and throughput performance.

The CFS scheduler uses a red-black tree to track runnable tasks, ensuring a logarithmic time complexity for insertion and deletion operations. The scheduler's decisions are based on a task's `vruntime`, which reflects how much time a task has been running relative to others.

### `sched_entity`

Every task in the CFS scheduler is represented by a `sched_entity` structure.

| Member                  | Type                    | Description                                                  |
| ----------------------- | ----------------------- | ------------------------------------------------------------ |
| `on_rq`                 | `int`                   | On-runqueue flag: indicates if this entity is currently on a runqueue. |
| `exec_start`            | `u64`                   | Start time (in nanoseconds) of the current execution period. |
| `sum_exec_runtime`      | `u64`                   | Accumulated runtime for this entity.                         |
| `vruntime`              | `u64`                   | Virtual runtime for this entity.                             |
| `prev_sum_exec_runtime` | `u64`                   | Previous entity runtime sum; used for throttling.            |
| `load`                  | `struct load_weight`    | Load tracking.                                               |
| `run_node`              | `struct rb_node`        | Red-black tree node used for task time ordering.             |
| `group_node`            | `struct list_head`      | Queue node for task groups.                                  |
| `cfs_rq`                | `struct cfs_rq *`       | Pointer to the `cfs_rq` this entity runs on.                 |
| `parent`                | `struct sched_entity *` | Pointer to the parent entity.                                |
| `cfs_bands`             | `int`                   | Control group's task band.                                   |
| ...                     | ...                     | ...                                                          |

### `load_weight`

```c
struct load_weight {
  unsigned long weight;
  u32 inv_weight;
};
```

The concept of weight is central to CFS, allowing it to ensure that tasks get CPU time proportional to their weight. A higher weight means a task will get more CPU time. The `load_weight` is an attribute of the task that reflects its weight.

The weight is determined from the task's priority (nice value). The default weight for a normal-priority (nice 0) task is `NICE_0_LOAD` (which is defined as 1024). We often use `p->se.load` to get the weight of a process `p`.

### `nice` to priority

In Linux, tasks have a "nice" value, ranging from -20 (highest priority) to 19 (lowest priority). The default is 0.

The formula to convert from nice to priority (`static_prio`) is:

```
static_prio = MAX_RT_PRIO + NICE_TO_PRIO(nice)
```

Where `MAX_RT_PRIO` is 100 and `NICE_TO_PRIO(nice)` translates the nice value into a priority offset.

### priority to weight

The priority of a task is then translated into a weight, which helps the scheduler determine the share of CPU time it should get. There is a predefined array `prio_to_weight` that provides a weight for each priority value.

For instance, a task with nice value 0 (which translates to a static priority of 120) has a default weight of 1024 (`NICE_0_LOAD`).

### priority to wmult

`wmult` is a multiplier used in calculations to speed up division operations (by replacing them with shift operations). Like `prio_to_weight`, there's a predefined array `prio_to_wmult` that provides this multiplier for each priority.

`wmult` is calculated as $inv\_weight = \frac{2^{32}}{weight}$.

When CFS computes the amount of time a task should run, it uses the task's weight, its delta execution time, and `wmult` in its calculations. The array `prio_to_wmult` provides optimized multipliers for each priority to make these computations efficient.

The kernel provides a function to query `prio_to_weight` and `prio_to_wmult`  and store the values in `p->se.load` (which is of type `load_weight`)

### vruntime

$$
vruntime = \frac{delta\_exec \times nice\_0\_weight}{weight}
$$

Where `vruntime` is the virtual runtime, `delta_exec` is the real runtime, `nice_0_weight` is the weight corresponding to the nice value of 0 and `weight` is the real weight of the process.

When perform calculation, we can make use of `inv_weight`:
$$
vruntime = (\frac{delta\_exec \times  nice\_0\_weight \times \bold{2^{32}}}{\bold{weight}}) >> 32
$$

### `rq`

| Member Name    | Type                    | Description                                                |
| -------------- | ----------------------- | ---------------------------------------------------------- |
| curr           | `struct task_struct*`   | Points to the currently executing task.                    |
| idle           | `struct task_struct*`   | Points to the idle task for this CPU.                      |
| nr_running     | `int`                   | Number of runnable tasks.                                  |
| cpu_load       | `int[NR_CPUS]`          | Array of recent load averages for this CPU.                |
| rq_lock        | `spinlock_t`            | Lock to protect this runqueue's data.                      |
| cfs            | `struct cfs_rq`         | Completely Fair Scheduler runqueue details.                |
| rt             | `struct rt_rq`          | Real-Time tasks runqueue details.                          |
| dl             | `struct dl_rq`          | Deadline tasks runqueue details.                           |
| tasks_timeline | `struct rb_root_cached` | Red-black tree to organize tasks based on their deadlines. |
| clock          | `u64`                   | Current clock value.                                       |
| clock_task     | `u64`                   | Per-task clock value.                                      |
| ...            | ...                     | ...                                                        |

We can get the run queue of the current cpu through `this_rq(cpu)`.

### `cfs_rq`

| Member          | Type                    | Description                               |
| --------------- | ----------------------- | ----------------------------------------- |
| load            | `struct load_weight`    | Aggregate load of tasks in the queue      |
| runnable_weight | `struct load_weight`    | Sum of weights of runnable tasks          |
| nr_running      | `unsigned int`          | Number of runnable tasks                  |
| h_nr_running    | `unsigned int`          | Number of tasks in the hierarchy          |
| exec_clock      | `u64`                   | Accumulated runtime of tasks in the queue |
| min_runtime     | `u64`                   | Minimum guaranteed runtime for tasks      |
| tasks_timeline  | `struct rb_root_cached` | RB-tree root of tasks based on vruntime   |
| curr            | `struct task_struct *`  | Currently running task                    |
| next            | `struct task_struct *`  | Next task to run (if preemption)          |

We can get the run queue that corresponds to the current process through `task_cfs_rq()`.

