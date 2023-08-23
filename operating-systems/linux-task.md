# Linux Task Model

In the Linux kernel, both threads and processes are represented using the `task_struct` structure. In this sense, they're both considered "tasks." However, how the kernel differentiates between threads and processes is through the concept of thread groups.

1. **task_struct**: This is the primary data structure used by the Linux kernel to represent a process or thread. Each instance of this structure contains all the necessary information about a task, such as its state, priority, open files, memory mappings, etc.

2. **Thread Group**: Linux treats processes as a group of one or more threads, with each thread being a task. The thread group ID (`tgid`) is introduced to differentiate between a standalone task and a task that belongs to a group. For a single-threaded process, the thread ID (PID) and thread group ID (TGID) are the same. For a multi-threaded process, the TGID is the same as the PID of the main thread, and all threads (including the main thread) in this process share this TGID. The individual threads, meanwhile, will each have their own unique PID.

   `getpid()` returns the current `tgid` and `getttid()` returns the current `pid`.

## Task Life Cycle

* `TASK_NEW` (Linux 4.8) 

  Ensure that the task won't run.

* `TASK_RUNNING`

  This is the state of a task that is either currently running or on the runqueue waiting to run. A task in this state can be selected by the scheduler to run on a CPU.

* `TASK_INTERRUPTIBLE`

  A task in this state is waiting for a certain condition to come true or for a resource to become available. It remains in the sleep queue and can be woken up by signals. If the required condition is satisfied or if the task receives a signal, it transitions back to `TASK_RUNNING`. A common scenario for this state is when a task waits for data from disk. If the data isn't readily available, the task doesn't waste CPU cycles and instead sleeps until the data is fetched.

* `TASK_UNINTERRUPTIBLE`

  Similar to the `TASK_INTERRUPTIBLE` state, a task in this state is waiting for a condition to become true or for a resource. However, unlike `TASK_INTERRUPTIBLE`, tasks in this state do not wake up for signals. They only transition back to `TASK_RUNNING` once the condition they are waiting for becomes true. This state is used in situations where an interruption can cause inconsistency or incoherence, like certain I/O operations.

* `TASK_STOPPED`

  A task enters this state when it receives a signal like `SIGSTOP`, which instructs it to stop executing. While in the `_TASK_STOPPED` state, the task does not run, but it can be resumed if it receives a `SIGCONT` signal. This state is also different from `TASK_INTERRUPTIBLE` and `TASK_UNINTERRUPTIBLE` in that a task in those states waits for an event or condition, whereas a `_TASK_STOPPED` task is explicitly paused by a signal.

* `EXIT_ZOMBIE`

  When a task has completed its execution but still has an entry in the task table, it's in the `EXIT_ZOMBIE` state. In this state, the task is not running, but it hasn't been completely removed from the process table because its parent hasn't read its exit status yet. Once the parent process reads the exit status using system calls like `wait()` or `waitpid()`, the zombie task is removed from the process table and it's said to be reaped.

## Thread Family

`init_task` is created when the kernel is launching. It is also called process 0 or idle process or swapper process. It is the first-ever kernel process.

`init` is created when the initialization of the system is about to reach an end. It is also called process 1.

After `start_kernel()` has initialized all the data structures, it will create `init` process which shares all data structures with `init_task`.

After process 0 has created process 1, it will run `cpu_idle()`. When there are no tunable processes on the ready task of CPU, the scheduler will run process 0 and push CPU into idle state.

Process 1 will call `kerel_init()` function which will call `execve` to load executable inits(/sbin/init, /bin/init). Finally, process 1 becomes a normal process. After it has become a normal process, it will do some work as instructed in /etc/inittab.

## `task_struct`

**`tasks`: `list_head` **All `task_structrues` are connected by a doubly linked list. The head of the list is `init_task`.

**`thread_group`: `list_head`**: All the threads within the same process group.

**`stack`: `void*`** Points to the kernel stack.

**`mm`: `mm_struct *`** Points to the memory layout information. We can locate the user-space stack with this struct.

## Macros

**`next_task()`**

```c
#define next_task(p) \
	list_entry_rcu((p)->tasks.next, struct task_struct, tasks)
```

**`next_thread()`**

**`for_each_process()`**

```c
#define for_each_process(p) \
	for (p = &init_task ; (p = next_task(p)) != &init_task ; )
```

**`current()`**

In Linux kernel 4.0, a `thread_info` is stored on the bottom of the kernel stack. For example, in Arm32, to obtain the current process, we can locate the `thread_info` by first finding the kernel stack through the `SP` register.

The release of Linux kernel 5.0 includes a new configuration option called `CONFIG_THREAD_INFO_IN_TASK`. This option specifies that the `thread_info is situated` in the`task_struct`. This helps to reduce the likelihood of `thread_info` corruption in certain stack overflow scenarios.

## Primitives

Linux extends `fork()` primitive of POSIX to `vfork()` and `clone()`. The underlying implementation of `fork()`,  `vfork()` and `clone()` is realized by `_do_fork()`.

```c
#include <unistd.h>
#include <sys/types.h>

pid_t fork(void);
```

```c
SYSCALL_DEFINE0(fork)
{
  return _do_fork(SIGCHLD, 0, 0, NULL, NULL, 0);
}
```

There are drawbacks with `fork()`. Though it adopts copy-on-write, it has to copy the page table of the parent process which may result it in a lower performance.

```c
SYSCALL_DEFINE0(vfork)
{
  return _do_fork(CLONE_VFORK | CLONE_VM | SIGCHLD, 0, 0, NULL, NULL, 0);
}
```

The parent process of `vfork()` will block until the child process calls `exit()` or `execve()`. The implementation of `v_fork()` has two more signal flag than that of `fork()`. 

**Difference between `fork()` and `vfork()`**

1. **Memory Semantics**:
   - **fork()**: When a new process is created using `fork()`, it creates a complete copy of the address space of the parent process. This means both the parent and the child processes have separate memory areas. This is known as a "copy-on-write" mechanism, where the actual memory copying is delayed until one of the processes (either parent or child) modifies the memory.
   - **vfork()**: With `vfork()`, the child process shares the address space of the parent process until it either calls `execve()` to replace its memory space with a new program or calls `_exit()` to terminate. Due to this shared address space, the child process can modify the parent's memory and variables.
2. **Blocking of Parent Process**:
   - **fork()**: After calling `fork()`, both the parent and child processes can run concurrently.
   - **vfork()**: After calling `vfork()`, the parent process is suspended and doesn't run until the child process either calls `execve()` or `_exit()`.
3. **Overhead**:
   - **fork()**: Due to the copy-on-write mechanism, `fork()` can have more overhead, especially if the parent process has a large address space. However, in modern systems with copy-on-write optimizations, the overhead can often be minimal until the memory is actually written to.
   - **vfork()**: `vfork()` has less overhead compared to `fork()` since it doesn't copy the address space.
4. **Usage**:
   - **fork()**: `fork()` is widely used when a process wants to duplicate itself to run similar tasks or processes concurrently.
   - **vfork()**: `vfork()` is generally used when the child process is created solely to execute a new program using `execve()`. Given its specific use-case and potential risks due to shared memory, `vfork()` is used less frequently.
5. **Safety**:
   - **fork()**: `fork()` is safer in a sense that the child doesn't modify the parent's memory.
   - **vfork()**: `vfork()` can lead to issues if not used carefully, due to shared memory. If the child process modifies any data before calling `execve()` or `_exit()`, it can alter the parent's execution.

`clone()` is usually used to create user-level thread. It can selectively inherit the parent process's resource.

The encapsulation of glibc library:

```c
#include <sched.h>

// 'fn' is a function pointer to the routine that will be executed by the child process
// 'child_stack' is the stack for the child process
// 'flag' sets the signal flag that denotes the resource that needs to be inherited
// 'arg' denotes the parameters for the child process
int clone(int (*fn)(void *), void *child_stack, int flags, void *arg, ...);

long clone(unsigned long flags, void *child_stack, void *ptid, void *ctid, struct pt_regs *regs);
```

```c
SYSCALL_DEFINE5(clone, unsigned long, clone_flags, unsigned long, newsp,
		 int __user *, parent_tidptr,
		 int __user *, child_tidptr,
		 unsigned long, tls)
{
	return _do_fork(clone_flags, newsp, 0, parent_tidptr, child_tidptr, tls);
}
```

Scenarios when a thread is terminated:

* (Volun) Return from `main`
* (Volun) Call `exit()`
* The thread receives a signal that cannot be handled
* The thread excounters a exception in kernel mode
* The thread receives `SIGKILL`

If the child process terminates earlier than parent process, then the parent process has to reap the zombie through `wait()`

If the child process terminates later than parent process, then `init` becomes its' new parent.

```c
SYSCALL_DEFINE1(exit, int, error_code)
{
  do_exit((error_code&0xff)<<8);
}
```

Linux designed the zombie mechanism to allow the system to obtain the reasons for termination of child processes. After the parent process calls the `wait()` system call and obtains the reasons for the termination of child processes, the kernel releases the `task_struct` of the child process.

Some system calls related to `wait()`:

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
fork() -> _do_fork() -> copy_process() -> dup_task_structre()
```

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

### `sched_fork()`

* Initialize data structures with regard to scheduling.
* Denote running state with `state` in `task_struct`.
* Inherit the priority from parent.
* Set up priority class of child task.
* Call `__set_task_cpu()` function to set CPU for child process.
* Call `task_fork()` of the scheduling class to finish some initialization of scheduler.
* Call `init_task_preempt_count()` to initialize `preempt_count` inside `thread_info`.

### Return from Creating Task Routines

After calling `_do_fork()`, the child process is added to the scheduler and will be scheduled sooner or later. Therefore, `fork()` will return twice: once from the parent process and once from the child process after being scheduled.

`copy_thread_tls()` in `copy_process()`is a wrapper around `copy_thread()` in architectures where `tls` is not defined. It mainly set up the kernel context for the new child process. In ARM64, the `copy_thread()` function copies the parent's stack frame to the child process and sets the X0 register in the stack frame to 0. This indicates that `_do_fork` will return 0 when returning to user space.

# Linux Scheduling

## Priority and Weight

There is a user-space variable called `nice` that indicates the priority of ordinary processes. This variable ranges from -20 to +19 and is mapped to a priority range of 100 to 139. The lower the value of `nice`, the higher the priority of the process.

There are 4 members of `task_struct` indicating the priority of the task.

* `prio` is the dynamic priority of the process which can be adjusted to tackle priority donation.
* `static_prio` is the priority set when the task starts.
* `normal_prio` is calculated based on `static_prio` and scheduling policy. For ordinary processes, it is the same as `static_prio`. For real time processes, it will be calculated according to `rt_priority`
* `rt_priority` is the priority of real time processes.

The schedulers also adopts the concept of 'weight' to denote the urgency of different processes.

## Policy

There are five scheduler classes implemented in Linux kernel right now: stop, deadline, realtime, CFS, idle
