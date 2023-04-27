# Exception Control Flow

The control flow of the processor:

Each transition from one address of a certain instruciton to another address of instruction is called **flow of control**, or **control flow** of the processor.

Applications request services from the operating system by using a form of ECF known as a **trap** or **system call**.

## Exceptions

An **exception** is an abrupt change in the control flow in response to some change in the processor’s state.

Processor’s state: encoded in various bits and signals inside the processor. The change in state is known as an *event*.

Change of processor’s state: event

In the case, when the processor detects that the event has occured, it makes an indirect procedure call(the exception), through a jump table called an **exception** **table**, to an operating system subroutine(the **exception hanlder**) that is specially designed to process this particular kind of event. When the exception handler finishes processing, one of three things happens, depending on the type of event that caused the exception:

- returns to $I_{curr}$
- returns to $I_{next}$
- Aborts the interrupted program

### Exception Handling

Exceptions denoted with *exception number*s. Assigned by processor or kernel.

At system boot time, the operating system allocates and initializes a jump table called an exception table.

The exception number is an index into the exception table, whose starting address is contained in a special CPU register called the *exception table base register.*

When control is being transfered from a user program to the kernel, all of these items are pushed onto the kernel’s stack rather than onto the user’s stack.

After the handler has processed the event, it optionally returns to the interrupted program by executing a special “return from interrupt” instruction.

### Classes of Exceptions

- Interrupts
- traps
- faults
- aborts

![Untitled](https://p.ipic.vip/8bkqob.jpg)

**Interrupts**

Occur asynchronously as a result of signals from I/O devices that are external to the processor.

I/O devices such as network adapters, disk controllers, and timer chips trigger interrupts by signaling a pin on the processor chip and placing onto the system bus the exception number that identifies the device that caused the interrupt.

**After the current instruction finishes executing**, the processor notices that the interrupt pin has gone high, reads the exception number from the system bus, and then calls the appropriate interrupt handler.

Apart from interrupts, all the other interrupts occur synchronously as result of executing the current instruction, like `syscall`.

**Traps and System calls**

Traps provide a procedure-like interface between user program and the kernel, known as system call.

`syscall n` Executing `syscall` instruction causes a trap to an exception handler that decodes the argument and calls the appropriate kernel routine.

**Faults**

If the handler is able to correct the error condition, it returns control to the faulting instruction, thereby re-executing it. Otherwise, the handler returns to an `abort`routine in the kernel that terminates the application program that caused the fault.

**Aborts**

Aborts result from unrecoverable fatal errors.

### Exceptions in Linux/x86-64 Systems

![Untitled](https://p.ipic.vip/iy3me8.jpg)

Numbers in the range from 0 to 31 is defined by intel architecture and thus are identical to any x86-64 system.

![Untitled](https://p.ipic.vip/dythmj.jpg)

All arguments to Linux system calls are passed through general purpose registers rather than the stack. By convention, `%rax`contains the syscall number, with up to six arguments in `%rdi`,`%rsi`,`%rdx`, `%r10`,`%r8` and `%r9`. On return from the system, registers `%r11`and `%rcx`are destroyed, and `%rax` contains the return value. A negative return value between -4095 and -1 indicates an error corresponding to negative `errno`.

## Processes

An instance of a program in execution.

Run program in shell: create a process and run the executable object file in this context.

Key abstractions of concept **process:**

- An independent control flow that provides the illusion that our program has exclusive use of the processor.
- A private address space that provides the illusion that our program has exclusive use of the memory system.

Serveral processors: execute → preempted→another execute

### Concurrent Flows

A logical flow whose execution overlaps in time with another flow is called a concurrent flow, and the two flows are said to run concurrently.

The general phenomenon of mulitple flows executing concurrently is known as concurrency. The notion of a process taking turns with other processes is also known as multitasking.

If two flows are running concurrently on different processor cores or computers, then we say that they are **parallel flows.** The idea is a subset of concurrent flows.

### Private Address Space

![Untitled](https://p.ipic.vip/hxsui1.jpg)

A mode bit in some control register characterize the privilleges that the process currently enjoys. **kernel mode**

`/proc` file system: allow user mode processes access the concepts of kernel data structures. 

`/proc/cpuinfo`

### Context Switches

Preempt the current process and restart a previously preempted process. 

Scheduling handled by scheduler.

A context switch can occur while the kernel is executing a system call on behalf of the user. If the system call blocks because it is waiting for some event to occur, then the kernel can put the current process to sleep and switch to another process.

## System Call Error Handling

Unix system-level functions encounter an error, they typically return $-1$ and set the global integer variable `errno` to indicate what went wrong. 

Example for error checking:

```c
if((pid = fork()) < 0) {
	fprintf(stderr, "fork error: %s\n", sterror(errno));
	exit(0);
}
```

The `strerror` function returns a text string that describes the error associated with a particular value of `errno`.

error-handling wrapper:

```c
pid_t Fork(void){
	pid_t pid;

	if((pid = fork()) < 0)
		unix_error("Fork error");
	return pid;
}
```

## Process Control

### Obtaining Process IDs

`getpid` and `getppid`

### Creating and Terminating Processes

A process in three states:

- Running

- Stopped

  The execution of this process is suspended and will not be scheduled. A process stops as a result of receiving a SIGSTOP, SIGTSP, SIGTTIN or SIGTTOU signal, and remains stopped until it receives a SIGCONT signal, at which point it becomes running again.

- Terminated

  A process become terminated for:

  - Receving a signal whose default action is to terminate the process
  - Returning from the main routine
  - Calling the `exit` function

---

The child gets an identical(but separate) copy of the parent’s user-level virtual address space. The child also gets identical copies of any of the parent’s open file descriptors.

In parent, the `fork()` function returns the pid of the child process.

In child, the `fork()` function returns 0.

The instructions in their logical control flows can be interleaved by the kernel in an arbitrary way.

When the parent calls `fork`, the `stdout` file is open and directed to the screen.

For a program running on a single processor, any **topological sort** of the vertices in the corresponding process graph represents a feasible total ordering of the statements in the program.

### Reaping Child Processes

The child process terminated → It becomes a zombie → reaped by its parent → Cease to exist

The `init` process is the adopted parent of any orphaned children. It has a PID of 1. It is created by the kernel in the system start-up, never terminates.

If a parent terminates without reaping its children, the `init` process is to reap them.

Even though zombies are not running, they still consume system memory resources.

A process waits for its children to terminate or stop by calling the `waitpid` function.

```c
#include <sys/types.h>
#include <sys/wait.h>

pid_t waitpid(pid_t pid, int *statusp, int options);

//Returns PID of child if OK, 0 if WNOHANG, -1 if error
```

By default(`options = 0`) `waitpid` suspends execution of the calling process until a child process in its wait set terminates. After the function returns, the termianted child has been reaped and the kernel removes all traces fo it from system.

The value of `pid` determines the members of the wait set:

- If `pid` > 0, …
- If `pid` = -1, the wait set contains all of the parent’s child processes.

The default behavior can be modified by setting the options:

- `WNOHANG` Return immediately (with a return value of 0) if none of the child processes in the wait set has terminated yet
- `WUNTRACED` Suspend execution of the calling process until a process in the wait set becomes either terminated or stopped. Return the PID of the terminated or stopped child that caused the return.
- `WCONTINUED` Suspend execution of the calling process until a running process in the wait set is terminated or until a stopped process in the wait set has been resumed by the receipt of a `SIGCONT` signal.
- `WNOHANG|WUNTRACED` Return immediately, with a return value of 0, if none of the children in the wait set has stopped or terminated, or wuth a return value equal to the PID of one of the stopped or terminated children.

The exit status of a reaped child in `*statusp`:

![Untitled](https://p.ipic.vip/1sqkzz.jpg)

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

//Returns: seconds left to sleep
```

Returns zero if the requested amount of time has elapsed, and the number of seconds still left to sleep otherwise. The latter case is possible if the `sleep` function returns prematurely because it was interrupted by a signal.

### Loading and Running Programs

The `execve` function loads and runs the executable object file `filename` with the argument list `argv` and the environment variable `envp`. `execve` returns to the calling program only there is an error, such as not being able to find `filename`. It is called once and never returns.

```c
#include <unistd.h>

int execve(const char *filename, const char *argv[], const char *envp[]);
```

![Untitled](https://p.ipic.vip/w9j5nw.png)

```c
#include <stdlib.h>

char *getenv(const char *name);

//Returns: pointer to name if it exists, NULL if no match

int setenv(const char *name, char *newvalue, int overwrite);

//Returns: 0 on success, -1 on error

void unsetenv(const char *name);

//Returns: nothing
```

## Singals

![Untitled](https://p.ipic.vip/ycf4ng.jpg)

A signal is a small message that notifies a process that an event of some type has occured in the system.

Singals provide an mechanism for exposing the occurence of some hardware exceptions to user processes.

### Singal Terminology

**Sending a signal**

The kernel sends a signal to a destination process by updating some state in the context of the destination process.

The signal is delivered for 2 reasons:

- The kernel has detected a system event such as divide-by-zero or the termination of a child process
- A process has invoked the kill function to explicitly **reuquest the kernel to send a signal** to the signal to the destination process. A process can send a signal to itself.

**Receiving the signal**

A destination process receives a signal when it is forced by the kernel to react in some way to the delivery of the signal. The process can either ignore the signal, terminate ot catch the signal by executing a user-level function called a signal handler.

A signal that has been sent but not yet received is called a pending signal. At any point in time, there can be **at most one** pending signal of a particular type for a. If a process has a pending signal of type *k*, then any subsequent signals of type *k* sent to that process are not required; they are simply discarded.

A pending signal is received at most once. **For each process**, th kernel maintains the set of pending signals in the `pending` bit vector( but the `pending` bit vector is not stored in PCB!  ), and the set of blocked signals in the `blocked` bit vector. The kernel sets bit *k* in `pending` whenever a signal of type *k* is delivered adn clears it whenever it is received.

### Sending Signals

**Process Groups**

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

**Sending Signals with the `/bin/kill` Program**

```java
/bin/kill -9 15213
```

Send signal 9 to the process `15213`. If the process ID is negative, the signal is sent to be sent to every process in process group PID.

**Sending Signals from the Keyboards**

Unix shells use the abstraction of a **job** to represent the processes that are created as a result of evaluating a single command line. At any point in time, there is at most one foreground job and zero or more background jobs.

**The shell creates a separate group for each job.** Typically, the process group ID is taken from one of the parent processes in the job.

CMD+C: send a `SIGINT` signal to every process in the foreground process group.

CMD+Z: send a `SIGTSTP` signal to every process in the foreground process group.

**Sending Signals with the `kill` Function**

Processes send signals to other processes (including themselves) by calling the `kill` function.

```java
#include <sys/types.h>
#include <signal.h>

int kill(pid_t pid, int sig);
```

If the pid is equal to 0, then `kill` sends signal to every process in the process group of the calling process, including the calling prcess itself. If pid is negative, then `kill` sends signal to every process in process group `|pid|`

![Untitled](https://p.ipic.vip/3gvr0s.jpg)

**Sending Signals with the `alarm` Function**

A process can send SIGALRM signals to itself by calling the `alarm` function.

```java
#include <unistd.h>

unsigned int alarm(unsigend int secs);

//Returns: remaining seconds of previous alarm, or 0 if no previous alarm
```

In any event, the call to `alarm` cancels any pending alarms and returns the number of seconds remaining until any pending alarm was due to be delivered(had not this call to `alarm` canceled it), or 0 if there were no pending alarms.

### Receiving Signals

When the kernel switches a process p from kernel mode to user mode, it checks the set of unblocked pending signals(`pending&~blocked`) for p.

 If the set is empty, then the kernel passes control to the next instruction in the logical control flow of p. 

If the set is nonempty, then the kernel chooses some signal k in the set(typically the smallest k) and forces p to receive signal k. 

The receipt of the signal triggers some action by the process. Once the process completes the action, then control passes back to the next instruction in the logical control flow of p. Each signal type has a predefined default action, which is one of the following:

- The process terminates
- The process terminates and dumps core
- The process stops until restarted by a `SIGCONT` signal
- The process ignores the signal

A process can modify the default action associated with a signal by using the `signal` function. The only exceptions are `SIGSTOP` and `SIGKILL`, whose default actions cannot be changed.

```java
#include <signal.h>

typedef void (*sighandler)(int);

sighandler_t signal(int signum, sighandler_t handler);

//Returns: pointer to previous handler if OK, SIG_ERR on error(does not set errno)
```

The `signal` functioncan change the action associated with a signal `signum` in one of three ways:

- If `handler` is `SIG_IGN`, then signals of type `signum` are ignored
- If `handler` is `SIG_DFL`, then the action of signals of type `signum` reverts to the default action
- Otherwise, `handler` is the address of a user-defined function, called a signal handler, that will be called whenever the process receives a signal of type `signum`. Changing the default action by passing the address a handler to the `signal` function is known as **installing the handler.** The invocation of the handler is called **catching the signal.** The execution of the handler is referred to as **handling the signal.**

When a process catches a signal of type *k*, the handler installed for signal *k* is invoked with a single integer argument set to *k*. **This argument allows the same handler function to catch different types of signals.**

![Untitled](https://p.ipic.vip/1a4uxg.jpg)

### Blocking and Unblocking Signals

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

- SIG_BLOCK. Add the signals in `set` to blocked (`blocked = blocked | set`).
- SIG_UNBLOCK. `blocked = blocked & ~set`.
- SIG_SETMASK. `blocked = set`.

If `oldset`is non-NULL, the previous value of the `blocked` bit vector is stored in `oldset`.

### Writing Signal Handlers

**Safe Signal Handling**

The signal handlers run concurrently with the main program. If they try to access teh same global data structure concurrently,the results can be unpredictable.

Guidelines:

- Keep handlers as simple as possible

- Call only **async-signal-safe (or simple “safe”)** functions in your handlers

  i.e. can be safely called from a signal handler.

  Either it is *reentrant* (e.g. accesses only local variables)

  Or because it cannot be interrupted by a signal handler.

  **Example 1: Not reentrant**

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

  **Example 2: Can be interrupted**

  ```c
  void allocate_memory() {
      char* buffer = malloc(1024);
      // do something with buffer
      free(buffer);
  }
  ```

  The ONLY safe way to generate output from a signal handler is to use the `write` function. Calling `printf` or `sprintf` is unsafe.

  ![Untitled](https://p.ipic.vip/th5z4d.png)

- Save and restore `errno`.

  Many of the Linux async-signal-safe functions ser `errno` when they return with an error. Calling such functions inside a handler might interfere with other parts of the program that rely on `errno`.

  Workaround: save `errno` to a local variable on entry to the handler can restore it before the handler returns. It’s not necessary if the handler terminates the process by calling `_exit`.

- Protecting accesses to shared global data structures by blocking all signals.

  If sharing a global data structure, then the handlers and main program should temporarily block all signals when accessing(reading or writing) that data structure.

- Declare global variables with `volatile`

  To an optimizing compiler, the compiler will cache a global variable in register. Using `volatile` will force the compiler to read the value from memory each time it is referenced in the code.

  Since threads run asynchronously, any update of global variables due to one thread should be fetched freshly by the other consumer thread.

- Declare flags with `sig_atomic_t`

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

**Correct Signal Handling**

The key ides is that the existence of a pending signal merely indicates that ***at least*** one signal has arrived.

The parent installs a SIGCHLD handler and then creates **three** children. In the meantime, the parent waits for a line of input from the terminal and then process it. 

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

**Portable Signal Handling**

Different systems have different signal-handling semantics. For example:

- Some older Unix systems restore the action for signed k to its default after signal k has been caught by a handler. The handler should be reinstalled.
- On some older versions of Unix, slow system calls that are interrupted when a handler catches a signal do not resume when the signal handler returns but instead return immediately to the user with an error condition and `errno` set to $\tiny{EINTR}$. On these systems, programmers must include code that manually restarts interrupted sysetm calls.

The Posix standard defines the `sigaction` function, which allows users to clearly speicfy the signal-handling semantics they want when they install a handler.

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

### Synchronizing Flows to Avoid Nasty Concurrency Bugs

```c
void handler(int sig) {
  int olderrno = errno;
  sigset_t mask_all, prev_all;
  pid_t pid;
  
  Sigfillset(&mask_all);
  while((pid = waitpid(-1, NULL, 0)) > 0) {
    Sigprocmask(SIG_BLOCK, &mask_all, &prev_all);
    deletejob(pid);
    Sigprocmask(SIG_SETMASK, &prev_all, NULL);
  }
  if(errno != ECHILD)
    Sio_error("waitpid error");
  errno = olderrno;
}
```

```C
int main(int argc, char **argv) {
  int pid;
  sigset_t mask_all, prev_all;
  
  Sigfillset(&mask_all);
  Signal(SIGCHLD, handler);
  initjobs();
  
  while(1) {
    if((pid = Fork()) == 0) {
      Execve("/bin/date", argv, NULL);
    }
    Sigprocmask(SIG_BLOCK, &mask_all, &prev_all);
    addjob(pid);
    Sigprocmask(SIG_SETMASK, &prev_all, NULL);
  }
  exit(0);
}
```

Chances are that a race condition may happen:

After the `fork` function the newly created child is instantly scheduled. The child terminates and call `deletejob`. But the job haven't been added!

By blocking `SIGCHLD` signals before the call to fork and then unblocking them only after we have called `addjob`, we guarantee that the child will be reaped *after* it is added to the job list. Notice that children inherit the blocked set of their parents, so we must be careful to unblock the `SIGCHLD` signal in the child before calling `execve`.

### Explicitly Waiting for Signals

Sometimes we need to exlicitly wait for a certain signal handler to run. Like the Linux shell wait for the foreground job to terminate and be reaped by the SIGCHLD handler before accepting the next user command.

**Solution 1:**

After creating the child, it resets pid to zero, unblocks SIGCHLD, and then waits in a spin loop for pid to become nonzero. After the child terminates, the handler reaps it and assigns its nonzero PID to the global pid variable. This terminates the spin loop, and the parent continues with additional work before starting the next iteration.

Cost: the spin loop is wasteful of processor resources

**Solution 2:**

```c
while(!pid)
  pause();
```

This aims to solve the problem of spin loop by simply pausing the main routine. A loop is still needed though because the `pause` may be interrupted by `SIGINT` signals. We use `pause` to wait for the `SIGCHLD` signal. If `SIGCHLD` is caught, then the main routine will resume. There's a race condition. If the `SIGCHLD` is received between the condition test and `pause`. Then the main routine wil pause forever.

**Solution 3**:

```c
while(!pid)
  Sleep(1);
```

It won't pause forvever. But it is costly to sleep for 1 second. Also, it's not likely that you find a fesible length of session of sleep.

The proper solution is to use `sigsuspend`.

The `sigsuspend` function temporarily replaces the current blocked set with mask and then suspends the process until the receipt of a signal whose action is either to run a handler or to terminate the process. If the action is to terminate, then the process terminates without returning from sigsuspend. If the action is to run a handler, then sigsuspend returns after the handler returns, restoring the blocked set to its state when `sigsuspend` was called.

Here a example implementation of shell using `sigsuspend`:

```c
#include "csapp.h"

volatile sig_atomic_t pid;

void sigchld_handler(int s) {
  int olderrno = errno;
  pid = Waitpid(-1, NULL, 0);
  errno = olderrno;
}

void sigint_handler(int s) {
  
}

int main(int argc, char **argv) {
  sigset_t mask, prev;
  
  Signal(SIGCHLD, sigchld_handler);
  Signal(SIGINT, sigint_handler);
  Sigemptyset(&mask);
  Sigaddset(&mask, SIGCHLD);
  
  while(1) {
    Sigprocmask(SIG_BLOCK, &mask, &prev);
    if(Fork() == 0)
      exit(0);
    
    pid = 0;
    while(!pid)
      sigsuspend(&prev);
    
    Sigprocmask(SIG_SETMASK, &prev, NULL);
    
    printf(".");
  }
  
}
```

If `SIGCHLD` is handled, then the loop will break.

If `SIGINT` is handled, then the loop will resume.

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

The `setjmp` function saves the current *calling environment* in the env buffer, for later use by longjmp, and returns 0. The calling environment includes the program counter, stack pointer, and general-purpose registers.

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

To avoid a race, we must install the handler *after* we call `sigsetjmp`. If not, we would run the risk of the handler running before the initial call to `sigsetjmp` sets up the calling environment for `siglongjmp`. 

The `sigsetjmp` and `siglongjmp` functions are not on the list of async-signal-safe functions. The reason is that in general `siglongjmp` can jump into arbitrary code, so we must be careful to call only safe functions in any code reachable from a `siglongjmp`. In our example, we call the safe `sio_puts` and `sleep` functions. The unsafe `exit` function is unreachable.
