# Process

What happens when we run a program in shell: A process is created and the executable object file runs in this context.

The process control block stores all the information the operating system needs about a particular process: where it is stored in memory, where its executable image is on disk, which user asked it to start executing, what privileges the process has, and so forth.

A typical x86-64 virtual memory layout is presented below. The `.data` segment is primarily used for storing global and static variables that are initialized. The `.bss` segment is used for uninitialized global and static variables. The `.rodata` stores constant data and string literals. The `.text` is the read-only code.

<img src="https://p.ipic.vip/q52zgk.png" alt="Screenshot 2023-08-15 at 4.20.59 PM" style="zoom: 33%;" />

> In Intel processors, the privilege level of a process is stored in a **field** within the Code Segment (CS) register. This **field** is known as the Current Privilege Level (CPL).
>
> The CPL can have four levels of privilege from 0 to 3, with level 0 being the most privileged and level 3 being the least. Typically, the kernel code runs at privilege level 0 (known as Ring 0), and user applications run at privilege level 3 (Ring 3).
>
> When a user-level process executes a system call, it triggers a transition from Ring 3 to Ring 0. The processor automatically changes the CPL to 0. When the kernel code has finished handling the system call, it executes a special return-from-system-call instruction, and the processor switches the CPL back to 3.

**Each thread of a multi-thread program have its own stack and PC.** They do **share the data (typically global variables)** and code from the process. 

## Process Control

### Obtaining Process IDs

`getpid` and `getppid`

### Creating and Terminating Processes

A process in five states:

* Creating
* Ready

* Running

* Blocked

  The execution of this process is suspended and will not be scheduled. A process stops as a result of receiving a `SIGSTOP`, `SIGTSP`, `SIGTTIN` or `SIGTTOU` signal, and remains stopped until it receives a `SIGCONT` signal, at which point it becomes running again.

  | Signal    | Number | Description                                      |
  | --------- | ------ | ------------------------------------------------ |
  | SIGINT    | 2      | Interrupt signal                                 |
  | SIGQUIT   | 3      | Quit signal                                      |
  | SIGILL    | 4      | Illegal instruction signal                       |
  | SIGTRAP   | 5      | Trap signal                                      |
  | SIGABRT   | 6      | Abort signal                                     |
  | SIGBUS    | 7      | Bus error signal                                 |
  | SIGFPE    | 8      | Floating-point exception signal                  |
  | SIGKILL   | 9      | Kill signal                                      |
  | SIGSEGV   | 11     | Segmentation fault signal                        |
  | SIGPIPE   | 13     | Broken pipe signal                               |
  | SIGALRM   | 14     | Alarm clock signal                               |
  | SIGTERM   | 15     | Termination signal                               |
  | SIGSTKFLT | 16     | Stack fault on coprocessor (unused)              |
  | SIGCHLD   | 17     | Child process status change signal               |
  | SIGCONT   | 18     | Continue executing, if stopped                   |
  | SIGSTOP   | 19     | Stop executing (cannot be caught or ignored)     |
  | SIGTSTP   | 20     | Terminal stop signal                             |
  | SIGTTIN   | 21     | Background process attempting read from terminal |
  | SIGTTOU   | 22     | Background process attempting write to terminal  |
  | SIGURG    | 23     | Urgent condition on socket                       |
  | SIGXCPU   | 24     | CPU time limit exceeded                          |
  | SIGXFSZ   | 25     | File size limit exceeded                         |
  | SIGVTALRM | 26     | Virtual timer expired                            |
  | SIGPROF   | 27     | Profiling timer expired                          |
  | SIGWINCH  | 28     | Window size change signal                        |
  | SIGIO     | 29     | I/O now possible (asynchronous I/O)              |
  | SIGPWR    | 30     | Power failure restart                            |
  | SIGSYS    | 31     | Bad system call                                  |
  
* Terminated

  A process become terminated for:

  * Receving a signal whose default action is to terminate the process
  * Returning from the main routine
  * Calling the `exit` function

The child gets an identical (but separate) copy of the parent’s user-level virtual address space. The child also gets identical copies of any of the parent’s open file descriptors.

In parent, the `fork()` function returns the pid of the child process.

In child, the `fork()` function returns 0.

### Reaping Child Processes

The child process terminated → It becomes a zombie → reaped by its parent → Cease to exist

The `init` process is the adopted parent of any orphaned children. It has a PID of 1. It is created by the kernel in the system start-up, never terminates. If a parent terminates without reaping its children, the `init` process is to reap them.

Even though zombies are not running, they still consume system memory resources.

A process waits for its children to terminate or stop by calling the `waitpid` function.

```c
#include <sys/types.h>
#include <sys/wait.h>

pid_t waitpid(pid_t pid, int *statusp, int options);

//Returns PID of child if OK, 0 if WNOHANG, -1 if error
```

By default (`options = 0`) `waitpid` temporarily suspends execution of the calling process until a child process in its wait set terminates. After the function returns, **the termianted child has been reaped** and the kernel removes all traces of it from system.

**`pid` Value**:

| `pid` Value | Description                              |
| ----------- | ---------------------------------------- |
| `pid` > 0   | …                                        |
| `pid` = -1  | Wait set includes all parent's children. |

**Options**:

| Option               | Description                                                  |
| -------------------- | ------------------------------------------------------------ |
| `WNOHANG`            | Return immediately if no child in wait set terminated.       |
| `WUNTRACED`          | Wait until child is terminated or stopped. Return the PID of that child. |
| `WCONTINUED`         | Wait until a running child is terminated or a stopped child resumes (via `SIGCONT`). |
| `WNOHANG\|WUNTRACED` | Return immediately unless a child has stopped or terminated. |

**Exit Status (`*statusp`)**:

| Function               | Description                                                 |
| ---------------------- | ----------------------------------------------------------- |
| `WIFEXITED(status)`    | True if child terminated normally.                          |
| `WEXITSTATUS(status)`  | Exit status if `WIFEXITED` is true.                         |
| `WIFSIGNALED(status)`  | True if child terminated from an uncaught signal.           |
| `WTERMSIG(status)`     | Signal number causing termination if `WIFSIGNALED` is true. |
| `WIFSTOPPED(status)`   | True if the returning child is currently stopped.           |
| `WSTOPSIG(status)`     | Signal number causing stop if `WIFSTOPPED` is true.         |
| `WIFCONTINUED(status)` | True if child was restarted by `SIGCONT`.                   |

A simpler version of `waitpid`

```c
#include <sys/types.h>
#include <sys/wait.h>

pid_t wait(int *statusp);
```

The order to reap the child processes is nondeterministic behavior that can make reasoning about concurrency so diffcult.

### Putting Processes to Sleep

The `sleep` function suspends a process for a specified period of time.

```c
#include <unistd.h>

unsigned int sleep(unsigned int secs);

// Returns: seconds left to sleep
```

**Returns after the process has been woken up.** Returns zero if the requested amount of time has elapsed, and the number of seconds still left to sleep otherwise. The function returns the number of seconds left unslept if the sleep is interrupted by a signal handler.

```c
#include <unistd.h>
#include <stdio.h>
#include <signal.h>

void signalHandler(int sig) {
    printf("Signal caught, interrupting sleep\n");
}

int main() {
    // set signal handler for SIGALRM
    signal(SIGALRM, signalHandler);

    printf("Sleeping for 10 seconds...\n");
    // set an alarm for 3 seconds
    alarm(3);
    unsigned int unslept = sleep(10);
    printf("Woke up!\n");
    if (unslept > 0)
        printf("Sleep was interrupted with %u seconds left\n", unslept);

    return 0;
}
```

### Loading and Running Programs

The `execve` function loads and runs the executable object file `filename` with the argument list `argv` and the environment variable `envp`. `execve` returns to the calling program only there is an error, such as not being able to find `filename`. It is called once and never returns.

```c
#include <unistd.h>

int execve(const char *filename, const char *argv[], const char *envp[]);
```

<img src="https://p.ipic.vip/w9j5nw.png" alt="Untitled" style="zoom:50%;" />

```c
#include <stdlib.h>

char *getenv(const char *name);

// Returns: pointer to name if it exists, NULL if no match

int setenv(const char *name, char *newvalue, int overwrite);

// Returns: 0 on success, -1 on error

void unsetenv(const char *name);

// Returns: nothing
```

# Singals

A signal is a small message that notifies a process that an event of some type has occured in the system.

Singals provide an mechanism for exposing the occurence of some hardware exceptions to user processes.

## Singal Terminology

The kernel sends a signal to a destination process by updating some state in the context of the destination process.

The signal is delivered for 2 reasons:

* The kernel has detected a system event such as divide-by-zero or the termination of a child process
* A process has invoked the `kill` function to explicitly **reuquest the kernel to send a signal** to the signal to the destination process. A process can send a signal to itself.

A destination process receives a signal when it is forced by the kernel to react in some way to the delivery of the signal. The process can either ignore the signal, terminate or catch the signal by executing a user-level function called a signal handler.

A signal that has been sent but not yet received is called a pending signal. At any point in time, there can be **at most one** pending signal of a particular type. If a process has a pending signal of type _k_, then any subsequent signals of type _k_ sent to that process are not required; they are simply discarded.

A pending signal is received at most once. **For each process**, th kernel maintains the set of pending signals in the `pending` bit vector( but the `pending` bit vector is not stored in PCB! ), and the set of blocked signals in the `blocked` bit vector. The kernel sets bit _k_ in `pending` whenever a signal of type _k_ is delivered and clears it whenever it is received.

> In Linux
>
> 1. **Thread-level (Task-level) signals**: These are signals directed to a specific thread within a process. Each `task_struct` (which represents a thread/task in the kernel) has a `pending` field that holds the signals pending for that specific thread.
> 2. **Process-level (Thread-group-level) signals**: These are signals directed to the process as a whole. The kernel will select one of the process's threads to handle the signal. For this, there's a `signal_struct` associated with each thread group, and this struct contains a `shared_pending` field that holds the signals pending for the entire process (or thread group).

## Sending Signals

### **Process Groups**

Every process belongs to exactly one process group, which is identified by a positive integer **process group ID.** The `getpgrp` function returns the process group ID of the current process.

```java
#include <unistd.h>

pid_t getpgrp(void);
```

By default, a child process belongs to the same process group as its parent. A process can change the process group of itself or another process by using the `setpgid` function.

```java
#inlcude <unistd.h>

int setpgid(pid_t pid, pid_t pgid);

//Returns 0 on success, -1 on error
```

If pid is `0`, the PID of the current process is used.

If pgid is `0`, the PID of the process specified by pid is used for the process group ID.

### **Sending Signals with the `/bin/kill` Program**

```java
/bin/kill -9 15213
```

Send signal 9 to the process `15213`. If the process ID is negative, the signal is sent to be sent to every process in process group PID.

### **Sending Signals from the Keyboards**

Unix shells use the abstraction of a **job** to represent the processes that are created as a result of evaluating a single command line. At any point in time, there is at most one foreground job and zero or more background jobs.

**The shell creates a separate group for each job.** Typically, the process group ID is taken from one of the parent processes in the job.

CMD+C: send a `SIGINT` signal to every process in the foreground process group.

CMD+Z: send a `SIGTSTP` signal to every process in the foreground process group.

### **Sending Signals with the `kill` Function**

Processes send signals to other processes (including themselves) by calling the `kill` function.

```java
#include <sys/types.h>
#include <signal.h>

int kill(pid_t pid, int sig);
```

If the pid is equal to 0, then `kill` sends signal to every process in the process group of the calling process, including the calling prcess itself. If pid is negative, then `kill` sends signal to every process in process group `|pid|`

<img src="https://p.ipic.vip/3gvr0s.jpg" alt="Untitled" style="zoom:50%;" />

### **Sending Signals with the `alarm` Function**

A process can send SIGALRM signals to itself by calling the `alarm` function.

```java
#include <unistd.h>

unsigned int alarm(unsigend int secs);

//Returns: remaining seconds of previous alarm, or 0 if no previous alarm
```

In any event, the call to `alarm` cancels any pending alarms and returns the number of seconds remaining until any pending alarm was due to be delivered(had not this call to `alarm` canceled it), or 0 if there were no pending alarms.

## Receiving Signals

When the kernel switches a process p from kernel mode to user mode, it checks the set of unblocked pending signals (`pending&~blocked`) for p.

If the set is empty, then the kernel passes control to the next instruction in the logical control flow of p.

If the set is nonempty, then the kernel chooses some signal k in the set (typically the smallest k) and forces p to receive signal k.

The receipt of the signal triggers some action by the process. Once the process completes the action, then control passes back to the next instruction in the logical control flow of p. Each signal type has a predefined default action, which is one of the following:

* The process terminates
* The process terminates and dumps core
* The process stops until restarted by a `SIGCONT` signal
* The process ignores the signal

A process can modify the default action associated with a signal by using the `signal` function. The only exceptions are `SIGSTOP` and `SIGKILL`, whose default actions cannot be changed.

```java
#include <signal.h>

typedef void (*sighandler)(int);

sighandler_t signal(int signum, sighandler_t handler);

//Returns: pointer to previous handler if OK, SIG_ERR on error(does not set errno)
```

The `signal` functioncan change the action associated with a signal `signum` in one of three ways:

* If `handler` is `SIG_IGN`, then signals of type `signum` are ignored
* If `handler` is `SIG_DFL`, then the action of signals of type `signum` reverts to the default action
* Otherwise, `handler` is the address of a user-defined function, called a signal handler, that will be called whenever the process receives a signal of type `signum`. Changing the default action by passing the address a handler to the `signal` function is known as **installing the handler.** The invocation of the handler is called **catching the signal.** The execution of the handler is referred to as **handling the signal.**

When a process catches a signal of type _k_, the handler installed for signal _k_ is invoked with a single integer argument set to _k_. **This argument allows the same handler function to catch different types of signals.**

<img src="https://p.ipic.vip/1a4uxg.jpg" alt="Untitled" style="zoom:50%;" />

## Blocking and Unblocking Signals

**Implicit blocking mechanism**: By default, the kernel blocks any pending signals of the type currently being processed by a handler.

**Explicit blocking mechanism:** Appications can explicitly block and unblock selected signals using the `sigprocamsk` function and its helpers.

```java
#include <signal.h>

int sigprocmask(int how, const sigset_t *set, sigset_t *oldset);
int sigemptyset(sigset_t *set);
int sigfillset(sigset_t *set);
int sigaddset(sigset_t *set, int signum);
int sigdelset(sigset_t *set, int signum);

//Returns: 0 if OK, -1 on error

int sigismember(const sigset_t *set, int signum);

//Returns: 1 if remember, 0 if not, -1 on error
```

The specific behavior depends on the value of `how`:

* `SIG_BLOCK`. Add the signals in `set` to blocked (`blocked = blocked | set`).
* `SIG_UNBLOCK`. `blocked = blocked & ~set`.
* `SIG_SETMASK`. `blocked = set`.

If `oldset`is non-NULL, the previous value of the `blocked` bit vector is stored in `oldset`.

## Writing Signal Handlers

### **Safe Signal Handling**

The signal handlers run concurrently with the main program. If they try to access the same global data structure concurrently,the results can be unpredictable.

Guidelines:

* Keep handlers as simple as possible

* Call only **async-signal-safe (or simple “safe”)** functions in your handlers i.e. can be safely called from a signal handler.

  Either it is _reentrant_ (e.g. accesses only local variables) or because it cannot be interrupted by a signal handler.

  **Example 1: Reentrant**

  Here's the signal handler:

  ```c
  volatile sig_atomic_t count = 0;
  void handler(int signum) {
    ++count;
    printf("Signal caught %d time(s)\n", count);
  }
  ```
  
  If 2 signals arrive at nearly the same time, two handlers may run concurrently. We increment the count to 1. Before we can printf the count, the count is incremented by another handler to 2. Here is the race condition. To solve this:
  
  ```c
  volatile sig_atomic_t count = 0;
  pthread_mutex_t count_mutex = PTHREAD_MUTEX_INITIALIZER;
  
  void handler(int signum) {
  	pthread_mutex_lock(&count_mutex);
  	++count;
  	printf("Signal caught %d time(s)\n", count);
  	pthread_mutex_unlock(&count_mutex);
  }
  ```
  
  **Example 2: Interrupted**
  
  ```c
  void allocate_memory() {
  	char* buffer = malloc(1024);
  	// do something with buffer
  	free(buffer);
  }
  ```
  
  The ONLY safe way to generate output from a signal handler is to use the `write` function. Calling `printf` or `sprintf` is unsafe.
  
  <img src="https://p.ipic.vip/th5z4d.png" alt="Untitled" style="zoom:50%;" />

* **Save and restore `errno`.**

  Many of the Linux async-signal-safe functions set `errno` when they return with an error. Calling such functions inside a handler might interfere with other parts of the program that rely on `errno`.

  Workaround: save `errno` to a local variable on entry to the handler can restore it before the handler returns. It’s not necessary if the handler terminates the process by calling `_exit`.

* **Protecting accesses to shared global data structures by blocking all signals.**

  If sharing a global data structure, then the handlers and main program should temporarily block all signals when accessing(reading or writing) that data structure.

* **Declare global variables with `volatile`**

  To an optimizing compiler, the compiler will cache a global variable in register. Using `volatile` will force the compiler to read the value from memory each time it is referenced in the code. **The source of truth is on memory.**

  Since threads run asynchronously, any update of global variables due to one thread should be fetched freshly by the other consumer thread.

* **Declare flags with `sig_atomic_t`**

  In one common handler design, the handler records the receipt of the signal by writing to a global **flag**. The main program periodically reads the **flag**, responding to the signal, and clears the flag.

  For flags shared in this way, we declare it in the way which reads and writes are guaranteed to be atomic (uninterruptible) because they can be implemented with a single instruction:

  ```java
  volatile sig_atomic_t flag;
  ```

  ```c
  #include <signal.h>
  #include <stdio.h>
  #include <stdlib.h>
  
  sig_atomic_t sig_received = 0;
  
  void sig_handler(int sig)
  {
      sig_received = 1;
  }
  
  int main()
  {
      signal(SIGINT, sig_handler);
  
      while (1) {
          if (sig_received) {
              printf("Signal received!\n");
              sig_received = 0;
          }
      }
  
      return 0;
  }
  ```

### **Correct Signal Handling**

The key ides is that the existence of a pending signal merely indicates that _**at least**_ one signal has arrived.

The parent installs a `SIGCHLD` handler and then creates **three** children. In the meantime, the parent waits for a line of input from the terminal and then process it.

```c
void handler1(int sig){
	int olderrno = errno;
	if((waitpid(-1, NULL, O)) < 0)
		sio_error("waitpid error");
	Sio_puts("Handler reaped child\n")
	Sleep(1);
	errno = olderrno;
}
```

```c
void handler2(int sig){
	int olderrno = errno;

	//waitpid can block the loop
	while(waitpid(-1, NULL, 0) > 0){
		Sio_puts("Handler reaped child\n")
	}

	if(errno != ECHILD)
		Sio_error("waitpid error");
	Sleep(1);
	errno = olderrno;
}
```

### **Portable Signal Handling**

Different systems have different signal-handling semantics. For example:

* Some older Unix systems restore the action for signed k to its default after signal k has been caught by a handler. The handler should be reinstalled.
* On some older versions of Unix, slow system calls that are interrupted when a handler catches a signal do not resume when the signal handler returns but instead return immediately to the user with an error condition and `errno` set to $\tiny{EINTR}$. On these systems, programmers must include code that manually restarts interrupted sysetm calls.

The POSIX standard defines the `sigaction` function, which allows users to clearly speicfy the signal-handling semantics they want when they install a handler.

```c
#include <signal.h>

int sigaction(int signum, struct sigaction *act, struct sigaction *oldact);
```

The function is unwieldy. A wrapper function `Signal` is introduced:

```c
handler_t *Signal(int signum, handler_t *handler){
	struct sigaction action, old_action;

	action.sa_handler = handler;
	// Block sigs of type being handled
	sigemptyset(&action.sa_mask); 
	// Restart syscalls if possible
	action.sa_flags = SA_RESTART;
	
	if(sigaction(signum, &action, &old_action) < 0)
		unix_error("Signal error");
	return(old_action.sa_handler);
}
```

Once the signal handler is installed, it remains installed until Signal is called with a handler argument of either `SIG_IGN` or `SIG_DFL`.

## Explicitly Waiting for Signals

Sometimes we need to exlicitly wait for a certain signal handler to run. Like the Linux shell wait for the foreground job to terminate and be reaped by the SIGCHLD handler before accepting the next user command.

**Solution 1:**

After creating the child, it resets pid to zero, unblocks SIGCHLD, and then waits in a spin loop for pid to become nonzero. After the child terminates, the handler reaps it and assigns its nonzero PID to the global pid variable. This terminates the spin loop, and the parent continues with additional work before starting the next iteration.

Cost: the spin loop is wasteful of processor resources

**Solution 2:**

```c
while(!pid)
  pause();
```

A loop is still needed though because the `pause` may be interrupted by `SIGINT` signals. 

Use `pause` to wait for the `SIGCHLD` signal. If `SIGCHLD` is caught, then the main routine will resume. 

Cost: a race condition. If the `SIGCHLD` is received between the condition test and `pause`. Then the main routine wil pause forever.

**Solution 3**:

```c
while(!pid)
  Sleep(1);
```

It won't pause forvever. But it is costly to sleep for 1 second. Also, it's not likely that you find a fesible length of session of sleep. 

**Solution 4:**

The `sigsuspend` function will replace the current blocked set with mask temporarily and then suspend the process. It will wait until a signal is received that either runs a handler or terminates the process.

 If the action is to terminate, then the process terminates without returning from `sigsuspend`. 

If the action is to run a handler, then sigsuspend returns after the handler returns, restoring the blocked set to its state when `sigsuspend` was called.

## Nonlocal Jumps

```c
#include <setjump.h>

int setjmp(jmp_buf env);
int sigsetjmp(sigjmp_buf env, int savesigs);

// returns o from setjump, nonzero from longjmps


void longjmp(jmp_buf env, int retval);
void siglongjmp(sigjmp_buf env, int retval);

//never returns
```

The `setjmp` function saves the current _calling environment_ in the env buffer, for later use by longjmp, and returns 0. The calling environment includes the program counter, stack pointer, and general-purpose registers.

The `longjmp` function restores the calling environment from the env buffer and then triggers a return from the most recent `setjmp` call that initialized `env`. The `setjmp` then returns with the nonzero return value retval.

An important application of nonlocal jumps is to permit an immediate return from a deeply nested function call, usually as a result of detecting some error condition.

The `longjump` call to jump from the nested function calls can skip some deallocation of dynamcially allocated memory, thus causing memory leak.

The `sigsetjmp` and `siglongjmp` functions are versions of setjmp and longjmp that can be used by signal handlers.

Another important application of nonlocal jumps is to branch out of a signal handler to a specific code location, rather than returning to the instruction that was interrupted by the arrival of the signal.

**Example: A soft restart**

```c
#include "csapp.h"

sigjmp_buf buf;

void handler(int sig) {
  siglongjmp(buf, 1);
}

int main() {
  if(!sigsetjmp(buf, 1)) {
    Signal(SIGINT, handler);
    Sio_puts("starting\n");
  } else
    Sio_puts("restarting\n");
  
  while(1) {
    Sleep(1);
    Sio_puts("processing...\n");
  }
  exit(0);
}
```

To avoid a race, we must install the handler _after_ we call `sigsetjmp`. If not, we would run the risk of the handler running before the initial call to `sigsetjmp` sets up the calling environment for `siglongjmp`.

The `sigsetjmp` and `siglongjmp` functions are not on the list of async-signal-safe functions. The reason is that in general `siglongjmp` can jump into arbitrary code, so we must be careful to call only safe functions in any code reachable from a `siglongjmp`. In our example, we call the safe `sio_puts` and `sleep` functions. The unsafe `exit` function is unreachable.

# Threads

Kernel interrupt handler is not a thread. A interrupt handler is not independently schedulable. It is triggered by a hardware I/O event, rather than a decision by the thread scheduler in the kernel. Once started, the interrupt handler runs to completion, unless preempted by another (higher priority) interrupt.

## Simple Thread API

```c
int pthread_create(pthread_t *thread, const pthread_attr_t *attr,
                   void *(*start_routine) (void *), void *arg);
```

* `thread`: A pointer to a `pthread_t` variable that will be filled in with a unique thread ID for the new thread.
* `attr`: A pointer to a `pthread_attr_t` structure that specifies attributes for the new thread.
* `start_routine`: A pointer to the function that the new thread will execute.
* `arg`: An argument that will be passed to the `start_routine` function when it is called by the new thread.

```c
int pthread_yield(void);
```

Give up the processor to let other threads run.

```c
int pthread_join(pthread_t thread, void **retval);
```

* `thread`: The thread ID of the thread to join.
* `retval`: A pointer to a variable that will be filled in with the exit status of the joined thread.

When `pthread_join()` is called, **the calling thread blocks until the specified thread terminates.** Once the thread has terminated, `pthread_join()` returns and the exit status of the thread is stored in the location pointed to by `retval`.

```c
void pthread_exit(void *retval);
```

When `pthread_exit()` is called, the calling thread is terminated and its resources are freed. The exit status of the thread is returned in the `retval` parameter.

> **An application of thread: Parallel block zero**
>
> In practice, the operating system will often create a thread to run block zero in the background. The memory of an exiting process does not need to be cleared until the memory is needed — that is, when the next process is created.

## Thread Data Structures and Life Cycle

### Per-Thread State and Thread Control Block (TCB)

Thread Control Block (TCB)

The thread control block holds two types of per-thread information:

1. The state of the computation being performed by the thread.
2. Metadata about the thread that is used to manage the thread.

**Per-thread Computation State**

A pointer to the thread’s stack and a copy of its processor registers.

In some systems, the general-purpose registers for a stopped thread are stored on the top of the stack, and the TCB contains only a pointer to the stack. In other systems, the TCB contains space for a copy of all processor registers.

<img src="https://p.ipic.vip/at02p3.png" alt="" width="375">

**Per-thread Metadata**

Thread ID, scheduling priority and status

### Shared State

Code, global variables and heap

## Thread Life Cycle

![](https://p.ipic.vip/ut6ns7.png)

**INIT** Thread creation puts a thread into its INIT state and allocates and initializes per-thread data structures. Once that is done, thread creation code puts the thread into the READY state by adding the thread to the _ready list_.

**READY** A thread in the READY state can be run but is not running. Its TCB is on the ready list, and its register values are stored in the TCB. The scheduler can make a thread go from READY to RUNNING at any time by copying its register values from its TCB to a processor's registers.

**RUNNING** A RUNNING thread can transition to the READY state in two ways:

* The scheduler can preempt a running thread and move it to the READY state by: (1) saving the thread’s registers to its TCB and (2) switching the processor to run the next thread on the ready list.
* A running thread can voluntarily relinquish the processor and go from RUNNING to READY by calling yield (e.g., `thread_yield` in the thread library).

Linux keeps a running thread in the ready list.

A thread in the **WAITING** state is waiting for some event. Whereas the scheduler can move a thread in the READY state to the RUNNING state, a thread in the WAITING state cannot run until some action by another thread moves it from WAITING to READY.

When a thread is in the **WAITING** state, rather than continuing to run the thread or storing the TCB on the scheduler’s ready list, the TCB is stored on the _waiting list_ of some _synchronization variable_ associated with the event. When the required event occurs, the operating system moves the TCB from the synchronization variable’s waiting list to the scheduler’s ready list, transitioning the thread from WAITING to READY.

## Ready Queue And Various I/O Device Queues

Porocess not running: PCB or TCB is in some scheduler queue

* There are separate queues for each device/signal/condition
* Each queue can have a differernt scheduler policy

![Screenshot 2023-05-27 at 8.15.47 AM](https://p.ipic.vip/5ep9k1.png)

## Implementing Kernel Threads

* **Kernel threads** The simplest case is implementing threads inside the operating system kernel, sharing one or more physical processors. A _kernel thread_ executes kernel code and modifies kernel data structures. Almost all commercial operating systems today support kernel threads. (e.g., process scheduling, memory management)

* **Kernel threads and single-threaded processes.** An operating system with kernel threads might also run some single-threaded user processes.

  <img src="https://p.ipic.vip/3i4w5q.png" alt="" width="375">

  Every thread corresponds to a user stack and kernel stack. 

* **Multi-threaded processes using kernel threads**

  <img src="https://p.ipic.vip/xdh0ef.png" style="zoom:50%;" />

  Each thread needs a kernel stack in the kernel. Here, "in the kernel" means that the data structure is managed and controlled by the kernel of the operating system.

* **User-level threads**

  The thread operations — `create`, `yield`, `join`, `exit`, and the synchronization routines is completely controlled by the user.

### Creating a Thread


```c
void thread_create(thread_t *thread, void (*func)(int), int arg) {
    // Allocate TCB and stack
    TCB *tcb = new TCB();
    thread->tcb = tcb;
    tcb->stack_size = INITIAL_STACK_SIZE;
    tcb->stack = new Stack(INITIAL_STACK_SIZE);
    
    // Initialize registers so that when thread is resumed, it will start running at
    // stub.  The stack starts at the top of the allocated region and grows down.
    tcb->sp = tcb->stack + INITIAL_STACK_SIZE;
    tcb->pc = stub;
    
    // Create a stack frame by pushing stub’s arguments and start address
    // onto the stack: func, arg
  	// This whole process is necessary because the thread isn't being started by a normal function call. Instead, it's 		
  	// being started by the system's thread scheduler, which doesn't know anything about the stub function's arguments.
    *(tcb->sp) = arg;
    tcb->sp--;
    *(tcb->sp) = func;
    tcb->sp--;
    
    // Create another stack frame so that thread_switch works correctly.
    // This routine is explained later in the chapter.
    thread_dummySwitchFrame(tcb);
		tcb->state = READY;
    readyList.add(tcb);    // Put tcb on ready list
 }
 
void stub(void (*func)(int), int arg) {
     (*func)(arg);           // Execute the function func()
     thread_exit(0);         // If func() does not call exit,  call it here.
 }
```

When we create a stack, we get the address of the stack. The stack is a continuous memory region from the starting address. We need to get to the top of the stack and the stack grows from higher address to lower address.

Noting the step of calling `stub`. We need this extra step in case the `func` procedure returns instead of calling `thread_exit`. Without the stub, func would return to whatever random location is stored at the top of the stack! Instead, func returns to stub and stub calls thread\_exit to finish the thread.

### Deleting a Thread

When a thread calls `thread_exit`,  there are two steps to deleting the thread: 

* Remove the thread from the ready list so that it will never run again. 
* Free the per-thread state allocated for the thread.

An important subtlety: After the thread remove itself from the ready list, an interrupt occurs before the thread finishes deallocating its memory. **Memory leak**.

A thread never deletes its own state. Some other thread must do it. On exit, the thread transitions to the **FINISHED** state, moves its TCB from the ready list to a list of finished threads the scheduler should never run. Once the finished thread is no longer running, it is safe for some *other* thread to free the state of the thread.

### Thread Context Switch

The switch saves the currently running thread’s registers to the thread’s TCB and stack, and then it restores the new thread’s registers from that thread’s TCB and stack into the processor.

A thread context switch can be triggered by either a voluntary call into the thread library, or an involuntary interrupt or processor exception.

**Voluntary** **(Internal Events)**

`thread_yield` call lets the currently running thread voluntarily give up processor to the next thread on the ready list.

`thread_join` and `thread_exit`.

* Blocking on I/O
* Waiting for a signal

**Involuntary**

An interrupt or processor exception could invoke a interrupt handler. The interrupt hardware saves the state of the running thread and executes the handler’s code. The handler can decide that some other thread should run, and then switch to it. Alternatively, if the current thread should continue running, the handler restores the state of the interrupted thread and resumes execution.

> **What happens when an interrupt or exception happens during the execution of a thread**
>
> * The hardware saves some registers information of the thread to the interrupt stack
>
> * The interrupt handler takes control
> * The handler saves the registers **it needs to use** to the kernel stack
> * The handler does its work
> * The handler pop some registers back
> * Hardware restore the registers from the interrupt stack
>
> At a minimum, the hardware typically saves:
>
> 1. **Program Counter (PC)**: This register contains the memory address of the next instruction to be executed. By saving this, the system knows where to resume execution once the interrupt has been handled.
> 2. **Processor Status Register (PSR)**: This register contains flags that indicate the status of the processor. For example, it can contain flags to indicate whether the last arithmetic operation resulted in a zero value or caused an overflow. It may also contain flags to indicate the current privilege level of the CPU (user mode or kernel mode), whether interrupts are enabled, etc.

For example, **timer interrupt** handler saves the state of the running thread, chooses another thread to run, and runs that thread by restoring its state to the processor. **IO hardware events** also invoke interrupt handlers.

<img src="https://p.ipic.vip/bylv3f.png" alt="Screenshot 2023-06-12 at 6.46.09 PM" style="zoom:50%;" />

We do not want to do an involuntary context switch while we are in the middle of a voluntary one. When switching between two threads, we need to temporarily defer interrupts until the switch is complete, to avoid confusion. Processors contain privileged instructions to defer and re-enable interrupts; we make use of these in our implementation below.

> **Why it is necessary to turn off interrupts during thread switch**
>
> A scenario:
>
> * A low priority thread is about to voluntarily switch to a high priority thread. It has pulled the high priority thread off the ready list
> * An interrupt occurs. A medium priority thread is moved from WAITING to READY
> * Since it appears that the processor is still running the low priority thread, the interrupt handler immediately switches to the medium thread.
> * It has to wait the low thread to reschedule to switch from the low priority to high priority

Prototype:

```pseudocode
while(1){
	RunThread();
	ChooseNextThread();
	SaveStateOfCPU(curTCB);
	LoadStateOfCPU(newTCB);
}
```

First, to run a thread:

* Load its state into CPU
* Load environment (virtual memory space, etc)
* Jump to the PC

To return the control back to the dispatcher:

* Internal events: thread returns control voluntarily
  * Requesting I/O
  * Thread asks to `wait`
  * call `yield()`
* External events: thread gets preempted

```c
// We enter as oldThread, but we return as newThread.
// Returns with newThread’s registers and stack.
void thread_switch(oldThreadTCB, newThreadTCB) {
    oldThreadTCB.regs.r7 = CPU.r7;
  	// ...
  	oldThreadTCB.regs.r0 = CPU.r0;
  	oldThreadTCB.regs.retpc = CPU.retpc;
  
  	CPU.r7 = newThreadTCB.regs.r7;
  	// ...
  	CPU.r0 = newThreadTCB.regs.r0;
  	CPU.retpc = newThreadTCB.regs.retpc;
		return; 
}

void thread_yield() {
    TCB *chosenTCB, *finishedTCB;
    // Prevent an interrupt from stopping us in the middle of a switch.
    disableInterrupts();
  	// Choose another TCB from the ready list.
    chosenTCB = readyList.getNextThread();
    if (chosenTCB == NULL) {
      // Nothing else to run, so go back to running the original thread.
    } else {
    	// Move running thread onto the ready list.
    	runningThread->state = ready;
      readyList.add(runningThread);
      thread_switch(runningThread, chosenTCB); // Switch to the new thread.
      // restore virtual memory space
      runningThread->state = running;
		}
     // Delete any threads on the finished list.
     while ((finishedTCB = finishedList->getNextThread()) != NULL) {
         delete finishedTCB->stack;
         delete finishedTCB;
     }
     enableInterrupts();
 }

 // thread_create must put a dummy frame at the top of its stack:
 // the return PC and space for pushad to have stored a copy of the registers.
 // This way, when someone switches to a newly created thread,
 // the last two lines of thread_switch work correctly.
 void thread_dummySwitchFrame(newThread) {
     *(tcb->sp) = stub;      // Return to the beginning of stub.
     tcb->sp--;
     tcb->sp -= SizeOfPopad;
}
```

To facilitate the seamless switching between threads, we set up the newly created thread as if it had already been executing and then voluntarily suspended or yielded its execution.

This means **arranging the data on the stack** to look as if the thread had been running and then voluntarily yielded control. This setup usually includes the return address where execution should resume (which, in this case, would be the start of the thread's function), and possibly also dummy values for saved register contents.

<img src="https://p.ipic.vip/axelj0.png" alt="Screenshot 2023-06-12 at 6.56.46 PM" style="zoom:50%;" />

**Involuntary Kernel Thread Context Switch**

Comparing a switch between kernel threads to what happens on a user-mode transfer:

* There is no need to switch modes (and therefore no need to switch stacks) and
* The handler can resume any thread on the ready list rather than always resuming the thread or process that was just suspended.

On most processor architectures, a simple (but inefficient) way to swap to the next thread from within an interrupt handler is to call `thread_switch` just before the handler returns. As we have already seen, thread_switch saves the state of the current thread (that is, the state of the interrupt handler) and switches to the new kernel thread. When the original thread resumes, it will return from `thread_switch`, and immediately pop the interrupt context off the stack, resuming execution at the point where it was interrupted. That is, the sequence of execution is $A\rightarrow Handler\rightarrow B \rightarrow A$.

To optimize this, some OSs like Linux imitate the interrupt hardware's fashion of saving context. That's to say, saving return instruction pointer and the `eflags` register first and then calls the `pushad` instruction to save the state of the general-purpose registers. This way, whether a thread was suspended by a hardware interrupt or a software call (like `thread_switch()`), resuming the thread is the same operation.

Switching Thread is much cheaper than switching processes. There is no need to change address space. Switching threads in user-space is even cheaper.

In Linux and pintos, we adopt the simple one-to-one threading model:

<img src="https://p.ipic.vip/f4kkqc.png" alt="image-20230611012613977" style="zoom:50%;" />

Every user thread has its kernel thread. In kernel, we use the kernel stack and in user mode we use the user stack. 

However, we can try out this model:

<img src="https://p.ipic.vip/nduys5.png" alt="image-20230611012833249" style="zoom:50%;" />

We can perform user-level yield without borthering switching to the kernel mode. However, this model has some downsides. If one user thread initiates an I/O job, all the other user threads will also go to sleep. The reason is that they share a common kernel thread.

To address this issue, we propose Many-toMany model:

<img src="https://p.ipic.vip/pdn8cv.png" alt="image-20230611013251886" style="zoom:50%;" />
