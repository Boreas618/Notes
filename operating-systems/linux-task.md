# Case Study: Linux Task Model

In the Linux kernel, both threads and processes are encapsulated using the `task_struct` structure, making them both "tasks." Their distinction arises from the concept of thread groups.

1. **task_struct**: The Linux kernel's primary structure for representing either a process or a thread. It encompasses all essential details about a task, such as state, priority, open files, memory mappings, and more.
2. **Thread Group**: In Linux, a process is a collection of one or more threads. Each thread is a task. The distinction between a standalone task (process) and a task within a group (thread) is made using the thread group ID (`tgid`). For a single-threaded process, its PID and TGID are identical. For a multi-threaded process, the TGID matches the main thread's PID, and all threads within this process share this TGID. However, each thread retains its unique PID.
   - `getpid()` returns the PID of the calling process.
   - `gettid()` returns the PID of the calling thread.

## Task Life Cycle

| State                  | Description                                                  |
| ---------------------- | ------------------------------------------------------------ |
| `TASK_NEW` (Linux 4.8) | Prevents the task from executing.                            |
| `TASK_RUNNING`         | Represents tasks either currently executing or waiting in the runqueue. Tasks in this state are eligible for CPU execution. |
| `TASK_INTERRUPTIBLE`   | Tasks wait for specific conditions or resources and can be awakened by signals. Typically seen when tasks await disk data. |
| `TASK_UNINTERRUPTIBLE` | Like `TASK_INTERRUPTIBLE`, but tasks don't wake up for signals. Used for operations where interruptions may cause inconsistencies. |
| `TASK_STOPPED`         | Tasks are halted by signals like `SIGSTOP` and can resume with `SIGCONT`. This state is signal-induced, not due to an awaited condition. |
| `EXIT_ZOMBIE`          | Tasks have finished but remain in the task table, awaiting their parent to read the exit status. They are removed once the status is read. |

## Thread Family

The kernel's first process, often known as the idle or swapper process, is initialized at launch with PID 0 and is responsible for managing CPU idle time. When the kernel is booted, the `start_kernel()` function is called to establish its core data structures, including initializing the `init_task` and call `cpu_idle()` to make itself be the idle process.

After completing the initial setup, the `rest_init()` function is called at the end of `start_kernel()` to continue the initialization process, which includes starting kernel threads that will eventually launch the first user-space process, `init`, with PID 1. This `init` process then invokes the `execve` system call to execute the user-space initialization program, typically `/sbin/init` or a modern equivalent like `/bin/systemd`. This process becomes the parent of all other user-space processes and is responsible for the system's further initialization, which may or may not involve reading the `/etc/inittab` file, depending on the init system in use.

## `task_struct` 

| Field Name     | Data Type       | Description                                                  |
| -------------- | --------------- | ------------------------------------------------------------ |
| `tasks`        | `list_head`     | All `task_struct` instances are connected via this doubly linked list. The head of the list is represented by `init_task`. |
| `thread_group` | `list_head`     | Represents all the threads within the same process group.    |
| `stack`        | `void*`         | Pointer to the kernel stack of the task.                     |
| `mm`           | `mm_struct *`   | Points to the memory layout information of the task. This structure can be used to locate the user-space stack. |
| `pid`          | `pid_t`         | Process ID of the task.                                      |
| `state`        | `long`          | Current state of the process. Common states include `TASK_RUNNING`, `TASK_INTERRUPTIBLE`, and `TASK_UNINTERRUPTIBLE`. |
| `priority`     | `int`           | Static priority of the task.                                 |
| `cpus_allowed` | `cpumask_t`     | CPU affinity mask.                                           |
| `flags`        | `unsigned int`  | Process flags.                                               |
| `exit_code`    | `int`           | Exit code returned when the task terminates.                 |
| `parent`       | `task_struct *` | Pointer to the parent process.                               |
| `children`     | `list_head`     | List of child processes.                                     |
| `next_task`    | `task_struct *` | Pointer to the next task in the runnable list.               |
| `prev_task`    | `task_struct *` | Pointer to the previous task in the runnable list.           |

## Macros

### **`next_task(p)`**

This macro fetches the next task in the task list, given the current task `p`.

```c
#define next_task(p) \
	list_entry_rcu((p)->tasks.next, struct task_struct, tasks)
```

The `list_entry_rcu` function is a mechanism to fetch the entry from a given linked list pointer, in this case, the `tasks` list of task structures. It ensures the safe traversal of the list in a RCU (Read-Copy-Update) protected context.

### **`next_thread(p)`**

This macro is used to fetch the next thread of a given task `p`. In Linux, threads are essentially tasks that share certain resources with other tasks. Threads are connected in a list, similar to tasks, which this macro helps traverse.

### **`for_each_process(p)`**

This macro provides a way to iterate over all tasks in the system, starting from the `init_task`, which is the initial task started by the kernel.

```c
#define for_each_process(p) \
	for (p = &init_task ; (p = next_task(p)) != &init_task ; )
```

Here, the loop will continue fetching the next task until it wraps around and reaches the `init_task` again.

### **`current()`**

In Linux kernel 4.0 and earlier versions, the `thread_info` structure, which contains information about a thread, is stored at the bottom of the kernel stack. To fetch the current process (or thread) on architectures like Arm32, we can locate the `thread_info` by first accessing the kernel stack using the `SP` (Stack Pointer) register.

However, with the introduction of Linux kernel 5.0, there was a significant change. A new configuration option called `CONFIG_THREAD_INFO_IN_TASK` was added. When this option is enabled, the `thread_info` is situated directly within the `task_struct`, which is the main data structure representing a task or thread in the kernel. By placing the `thread_info` inside the `task_struct`, it becomes less prone to corruption due to certain stack overflow scenarios, thereby enhancing the kernel's stability and security.

## Primitives

Linux augments the POSIX `fork()` primitive with the additions of `vfork()` and `clone()`. All three methods are underpinned by the `_do_fork()` function.

```c
#include <unistd.h>
#include <sys/types.h>

pid_t fork(void);
```

The `fork()` syscall is implemented as:

```c
SYSCALL_DEFINE0(fork)
{
  return _do_fork(SIGCHLD, 0, 0, NULL, NULL, 0);
}
```

However, `fork()` can be less efficient as it necessitates the copying of the parent process's page table, even with copy-on-write optimization.

`vfork()` addresses some of the drawbacks of `fork()`:

```c
SYSCALL_DEFINE0(vfork)
{
  return _do_fork(CLONE_VFORK | CLONE_VM | SIGCHLD, 0, 0, NULL, NULL, 0);
}
```

### Key Differences between `fork()` and `vfork()`

| Criteria                       | `fork()`                                                     | `vfork()`                                                    |
| ------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Memory Semantics**           | Duplicates the address space of the parent (uses "copy-on-write"). | Shares the address space with the parent until a call to `execve()` or `_exit()`. |
| **Blocking of Parent Process** | Both processes run concurrently.                             | The parent process is halted until the child process exits or executes another program. |
| **Overhead**                   | Can be higher, especially with large address spaces.         | Typically lower since it doesn't duplicate the address space. |
| **Usage**                      | For duplicating processes.                                   | Primarily for the child to execute another program.          |
| **Safety**                     | Safer due to memory isolation.                               | Riskier due to shared memory.                                |

`clone()` is a more versatile syscall, typically employed for creating user-level threads. It provides granular control over the inheritance of the parent's resources:

```c
#include <sched.h>

int clone(int (*fn)(void *), void *child_stack, int flags, void *arg, ...);
```

Its syscall definition is:

```c
SYSCALL_DEFINE5(clone, unsigned long, clone_flags, unsigned long, newsp,
		 int __user *, parent_tidptr, int __user *, child_tidptr, unsigned long, tls)
{
	return _do_fork(clone_flags, newsp, 0, parent_tidptr, child_tidptr, tls);
}
```

### Thread Termination Scenarios

Threads can terminate due to the following:
- Voluntary return from `main`
- Voluntary call to `exit()`
- Unhandled signals
- Kernel mode exceptions
- Reception of `SIGKILL`

Post-termination, if the child concludes before the parent, the latter must reap the former's "zombie" state using `wait()`. If the child outlives the parent, the `init` process adopts it.

```c
SYSCALL_DEFINE1(exit, int, error_code)
{
  do_exit((error_code&0xff)<<8);
}
```

Zombies exist to let the system retrieve termination reasons. After using the `wait()` syscall to glean this information, the kernel discards the child's `task_struct`.

Related system calls to `wait()` include:

```c
asmlinkage long sys_wait4(pid_t pid, int __user *stat_addr, int options, struct rusage __user *ru);
asmlinkage long sys_waitpid(pid_t pid, int __user *stat_addr, int options);
asmlinkage long sys_waitid(int which, pid_t pid, struct siginfo __user *infop, int options, struct rusage __user *ru);
```

## Kernel Thread

Kernel threads do not have a standalone address space and can only run in the memory space of the kernel. A typical kernel thread includes the page recycle thread kswapd.

To create a Linux kernel thread, you can use the following functions:

```
kthread_create(threadfn, data, namefmt, arg, ...)
kthread_run(threadfn, data, namefmt, ...)
```

The thread created by `kthread_create` is stopped, and you need to call `wake_up_process` to wake it up and add it to the ready queue. To create a thread that is ready to run, simply use `kthread_run()`.

The underlying implementation of `kernel_thread` is done through `_do_fork()`.

## Creation of Tasks

Typical call stack:

```c
fork() -> _do_fork() -> copy_process()
```

Functions called in `copy_process()`:

1. **dup_task_struct()**: This creates a duplicate of the current task's task structure. 
2. **init_task()**: It initializes various task parameters for the newly created process.
3. **copy_flags()**: Adjusts flags based on the original process and the nature of the forking/cloning.
4. **copy_files()**, **copy_fs()**, **copy_sighand()**, **copy_signal()**, **copy_mm()**, **copy_namespaces()**, etc.: These functions copy or share different aspects of the process context depending on the flags passed to the clone system call. For example, if `CLONE_FILES `is set, `copy_files()` would share file descriptors between the parent and child processes, otherwise, it creates a copy.
5. **copy_thread()**: Responsible for copying architecture-specific thread-related data. In the x86 architecture, it sets up the new process's kernel stack and process state.
6. **pid_alloc()**: Allocates a PID (Process ID) for the new process.
7. **sched_fork()**: Prepares the new process for scheduling.
8. **wake_up_new_task()**: Once the new process has been set up, this function is called to put the new task on the run queue, ready to be scheduled by the CPU.
9. **audit_fork()**: If auditing is enabled, this function records the fork event.
10. **ptrace_fork()**: If the parent process is being traced (e.g., by a debugger), this function ensures that the child is also traced.
11. **perf_event_fork()**: If performance events are being tracked, they are updated for the new process here.

### `_do_fork()`

```c
long _do_fork(unsigned long clone_flags,
	      unsigned long stack_start,
	      unsigned long stack_size,
	      int __user *parent_tidptr,
	      int __user *child_tidptr,
	      unsigned long tls)
```

1. `clone_flags`: the flags for creating a process

   | Flag                   | Description                                                  |
   | ---------------------- | ------------------------------------------------------------ |
   | **`CLONE_VM`**         | **Child shares the same memory space as the parent (typically for threads).** |
   | **`CLONE_FS`**         | **Child shares file system information (root, current directory, umask) with the parent.** |
   | `CLONE_FILES`          | Child shares the file descriptor table with the parent.      |
   | **`CLONE_SIGHAND`**    | **Child shares signal handlers with the parent.**            |
   | `CLONE_PTRACE`         | Retain any `ptrace` relationship with the parent.            |
   | `CLONE_VFORK`          | Parent is suspended until child calls `execve()` or `_exit()`. |
   | **`CLONE_PARENT`**     | **Child's parent is the same as the caller's parent. (Create sibling thread)** |
   | **`CLONE_THREAD`**     | **Child is placed in the same thread group as the caller. Ideal to be used with `CLONE_SIGHAND` and `CLONE_VM`.** |
   | **`CLONE_NEWNS`**      | **Create the child in a new mount namespace.** Cannot be used with `CLONE_FS`. |
   | `CLONE_SYSVSEM`        | Child shares System V semaphore undo values with the parent. |
   | `CLONE_SETTLS`         | Use the `tls` argument.                                      |
   | `CLONE_PARENT_SETTID`  | Parent's TID is set to the value pointed by `parent_tidptr`. |
   | `CLONE_CHILD_CLEARTID` | Child's kernel thread ID is cleared when the child terminates. |
   | `CLONE_CHILD_SETTID`   | Child's TID is set to the value pointed by `child_tidptr`.   |
   | `CLONE_DETACHED`       | Historically implied that the child would be detached (now unused). |
   | `CLONE_UNTRACED`       | Tracing process cannot force `CLONE_PTRACE` on this child process. |
   | `CLONE_NEWCGROUP`      | Child is created in a new cgroup namespace.                  |
   | `CLONE_NEWUTS`         | Child is created in a new UTS namespace.                     |
   | `CLONE_NEWIPC`         | Child is created in a new IPC namespace.                     |
   | **`CLONE_NEWUSER`**    | **Child is created in a new user namespace.** Cannot be used with `CLONE_FS`. |
   | **`CLONE_NEWPID`**     | **Child is created in a new PID namespace.**                 |
   | `CLONE_NEWNET`         | Child is created in a new network namespace.                 |
   | `CLONE_IO`             | Child shares an I/O context with the parent.                 |

2. `stack_start`: the starting address for user mode stack
3. `stack_size`: the size of user stack, set to 0 usually
4. `parent_tidptr` and `child_tidptr`: points to id of parent and child process
5. `tls`: thread local storage

```c
long _do_fork(unsigned long clone_flags,
	      unsigned long stack_start,
	      unsigned long stack_size,
	      int __user *parent_tidptr,
	      int __user *child_tidptr,
	      unsigned long tls)
{
	struct completion vfork;
	struct pid *pid;
	struct task_struct *p;
	int trace = 0;
	long nr;

	/*
	 * Determine whether and which event to report to ptracer.  When
	 * called from kernel_thread or CLONE_UNTRACED is explicitly
	 * requested, no event is reported; otherwise, report if the event
	 * for the type of forking is enabled.
	 */
	if (!(clone_flags & CLONE_UNTRACED)) {
		if (clone_flags & CLONE_VFORK)
			trace = PTRACE_EVENT_VFORK;
		else if ((clone_flags & CSIGNAL) != SIGCHLD)
			trace = PTRACE_EVENT_CLONE;
		else
			trace = PTRACE_EVENT_FORK;

		if (likely(!ptrace_event_enabled(current, trace)))
			trace = 0;
	}

  // create a child process and return the task_struct of child process
	p = copy_process(clone_flags, stack_start, stack_size,
			 child_tidptr, NULL, trace, tls, NUMA_NO_NODE);
	add_latent_entropy();

	if (IS_ERR(p))
		return PTR_ERR(p);

	/*
	 * Do this prior waking up the new thread - the thread pointer
	 * might get invalid after that point, if the thread exits quickly.
	 */
	trace_sched_process_fork(current, p);

	pid = get_task_pid(p, PIDTYPE_PID);
	nr = pid_vnr(pid);

	if (clone_flags & CLONE_PARENT_SETTID)
		put_user(nr, parent_tidptr);

	if (clone_flags & CLONE_VFORK) {
		p->vfork_done = &vfork;
		init_completion(&vfork);
		get_task_struct(p);
	}

	wake_up_new_task(p);

	/* forking complete and child started to run, tell ptracer */
	if (unlikely(trace))
		ptrace_event_pid(trace, pid);

	if (clone_flags & CLONE_VFORK) {
		if (!wait_for_vfork_done(p, &vfork))
			ptrace_event_pid(PTRACE_EVENT_VFORK_DONE, pid);
	}

	put_pid(pid);
	return nr;
}
```

### `copy_process()`

* Verify the flags as specified in the table above. For example, `CLONE_NEWNS` cannot be used with `CLONE_FS`.
* Call `dup_task_struct()` to create a new `tak_struct` for the new task
* Call `copy_creds()` to copy the certificate of the parent process.
* Call `sched_fork()` to initialize some data structures with regrad to task scheduling.
* Call `copy_files()`, `copy_fs()`, `copy_signal()`, `copy_mm()`, `copy_namespaces()`, `copy_io()`, `copy_thread_tls()` to copy necessaty information.
* Call `alloc_pid()` to allocate `pid` structure and PID to the new task
* Call `pid_vnr()` to allocate a global PID
* Set `group_leader` and TGID

### `dup_task_struct()`

* Call `alloc_task_struct_node()` to allocate a new process descriptor for the new process.
* Call `alloc_thread_stack_node()` to allocate kernel stack for the new process.
* Call `arch_dup_task_struct()` to copy the parents' process descriptor to the new process's.
* Make the `stack` in the new process's process descriptor point to the new kernel stack.
* Call `set_task_stack_end_magic()` to set a magic number at the top of the kernel stack to detect stack overflow.

### `copy_thread()`

<img src="http://lastweek.io/notes/linux/stack_layout_fork.png" alt="stack_layout_fork" style="zoom: 25%;" />

This is the stack layout after `copy_thread()`. Also the rough layout when the newly created thread is enqueued into runqueue.

The top of the stack can be calculated with `task_stack_page(p)+THREAD_SIZE`. The stack grows from top to bottom.

To the top of the kernel stack is the `struct pt_regs`. Here, `copy_thread()` used a structure called `struct fork_frame`, which contains a `struct inactive_task_frame` and a `struct pt_regs`. The bottom of the `struct fork_frame` is a field called `ret_addr`. This is essentially the first function (`ret_from_fork()`) gets run when this newly created thread gets running (scheduled by runqueue). 

When the scheduler decides to run a thread, it will call `context_switch()`, which internally calls `switch_to()`, which is just a macro around `__switch_to_asm`.

```c
#define switch_to(prev, next, last)                                     \    
do {                                                                    \    
        ((last) = __switch_to_asm((prev), (next)));                     \    
} while (0) 
```

`__switch_to_asm` is simply playing around the `struct fork_frame` we discussed above. It:

- Saves the callee-saved registers.
- Switches the stack pointer (`%rsp`) from Thread A's stack to Thread B's stack using the `TASK_threadsp` macro. This ensures that when Thread B is executed, it has its own separate stack.
- Restores callee-saved registers for Thread B from its stack.

Eventually, only the `ret_addr` field remains in the stack.

**This is very important**: we **`jump`** to the `__switch_to()` (Note that it is different from `switch_to()`) function. Hence no return address will be pushed into the stack. Later on, when `__switch_to()` finishes and returns, the hardware will use the last field in the stack, which is the `ret_addr` field we placed there during `copy_thread()`.

### `sched_fork()`

* Initialize data structures with regard to scheduling.

* Set the `state` of the `task_struct` to `TASK_NEW` to indicate that the task has just beenc created and not yet able to be added to the scheduler.

* Inherit the priority `normal_prio` from parent.

* Set up priority class of child task.

* Call `init_entity_runable_average()` to initialize the members related to the scheduling entity of the child process.

* Call `__set_task_cpu()` function to set CPU for child process.

* Call `task_fork()` of the scheduling class to finish some initialization of scheduler. The function will execute the scheduler-specific set of methods.

  > ```c
  > static void task_fork_fair(struct task_struct *p)
  > {
  > 	struct cfs_rq *cfs_rq;
  > 	struct sched_entity *se = &p->se, *curr;
  > 	struct rq *rq = this_rq();
  > 	struct rq_flags rf;
  >  
  > 	rq_lock(rq, &rf);
  > 	update_rq_clock(rq);
  >  
  > 	cfs_rq = task_cfs_rq(current);
  > 	curr = cfs_rq->curr;   
  > 	if (curr) {
  > 		update_curr(cfs_rq);       
  > 		se->vruntime = curr->vruntime;
  > 	}
  > 	place_entity(cfs_rq, se, 1);
  >  
  > 	se->vruntime -= cfs_rq->min_vruntime;
  > 	rq_unlock(rq, &rf);
  > }
  > ```
  >
  > The `task_fork_fair` function is part of the Completely Fair Scheduler (CFS) in the Linux kernel, which is designed to ensure a fair distribution of CPU time among tasks.
  >
  > When called, the function is setting up the scheduling attributes for a newly forked task. It begins by declaring local variables for the run queue (`rq`), the scheduling entities (`se` and `curr`), and flags for the run queue (`rf`).
  >
  > To prevent concurrent modifications, the function locks the current run queue using `rq_lock(rq, &rf)`. It then updates the run queue's time reference with `update_rq_clock(rq)`.
  >
  > The function fetches the `cfs_rq`, the run queue for the Completely Fair Scheduler, associated with the currently executing task. If there is a currently executing scheduling entity on the run queue, it updates its statistics and sets the `vruntime` (virtual runtime) of the new task's scheduling entity (`se`) to match that of the current task's `vruntime`. **This ensures fairness in the task's starting execution time compared to others.**
  >
  > Next, it places the scheduling entity of the newly forked task into the run queue with the `place_entity` function.
  >
  > To make sure the newly forked task does not start at a disadvantage, it adjusts the `vruntime` of the scheduling entity by subtracting the `min_vruntime` of the run queue (the new process inherits the `vruntime` from `curr` and the `vruntime` of `curr` can be quite high. Therefore, the new process may be starved).
  >
  > Finally, the function releases the lock on the run queue using `rq_unlock(rq, &rf)`, allowing other tasks or operations to interact with the run queue.

* Call `init_task_preempt_count()` to initialize `preempt_count` inside `thread_info`.

### Return from Creating Task Routines

After calling `_do_fork()`, the child process is added to the scheduler and will be scheduled sooner or later. The new process will start from `ret_from_fork`.

> **Arm64**
>
> If the new process is a kernel thread, the address of the thread function is stored in register x19, and the parameter of the thread function is stored in register x20. If the new process is a user process, the value of register x19 is 0.
>
> The execution process of function `ret_from_fork` is as follows:
>
> * The function `schedule_tail ` called to perform cleanup operations for the previous process.
>
> * If the value of register x19 is 0, it means that the current process is a user process. In this case, register x28 stores the address of current process's `thread_info` structure, and then jumps to label `ret_to_user` to return to user mode.
>
> * If the value of register x19 is not 0, it means that the current process is a kernel thread. In this case, it calls the thread function.

`copy_thread_tls()` in `copy_process()`is a wrapper around `copy_thread()` in architectures where `tls` is not defined. It mainly set up the kernel context for the new child process. In ARM64, the `copy_thread()` function copies the parent's stack frame to the child process and sets the X0 register in the stack frame to 0. This indicates that `_do_fork` will return 0 when returning to user space.

