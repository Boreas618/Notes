# Singals

A signal is a small message that notifies a process that an event of some type has occured in the system.

Singals provide an mechanism for exposing the occurence of some hardware exceptions to user processes.

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
> 1. **Thread-level (Task-level) Signals**: These are signals directed to a specific thread within a process. Each `task_struct` (which represents a thread/task in the kernel) has a `pending` field that holds the signals pending for that specific thread.
> 2. **Process-level (Thread-group-level) Signals**: These are signals directed to the process as a whole. The kernel will select one of the process's threads to handle the signal. For this, there's a `signal_struct` associated with each thread group, and this struct contains a `shared_pending` field that holds the signals pending for the entire process (or thread group).

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
