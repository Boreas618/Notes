# Process

> What happens when we run a program in shell: A process is created and the executable object file runs in this context.

Process is the basic unit the oprating system allocating resources for.

A process is made up of three parts:

* PCB
* Program segment
* Data segment

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

# Threads

> Kernel interrupt handler is not a thread. A interrupt handler is not independently schedulable. It is triggered by a hardware I/O event, rather than a decision by the thread scheduler in the kernel. Once started, the interrupt handler runs to completion, unless preempted by another (higher priority) interrupt.

> **An application of thread: Parallel block zero**
>
> In practice, the operating system will often create a thread to run block zero in the background. The memory of an exiting process does not need to be cleared until the memory is needed — that is, when the next process is created.

## Thread Data Structures and Life Cycle

### Per-Thread State

The thread control block holds two types of per-thread information:

1. The state of the computation being performed by the thread.
2. Metadata about the thread that is used to manage the thread.

**Per-thread Computation State**

A pointer to the thread’s stack and a copy of its processor registers.

In some systems, the general-purpose registers for a stopped thread are stored on the top of the stack, and the TCB contains only a pointer to the stack (Many early and simplistic computer architectures and operating systems employed this method). In other systems, the TCB contains space for a copy of all processor registers (*nix).

<img src="https://p.ipic.vip/at02p3.png" alt="" width="375">

**Per-thread Metadata**

Thread ID, scheduling priority and status

### Shared State

Code, global variables and heap

## Thread Life Cycle

<img src="https://p.ipic.vip/ut6ns7.png" style="zoom:50%;" />

**INIT** Thread creation puts a thread into its INIT state and allocates and initializes per-thread data structures. Once that is done, thread creation code puts the thread into the READY state by adding the thread to the _ready list_.

**READY** A thread in the READY state can be run but is not running. Its TCB is on the ready list, and its register values are stored in the TCB. The scheduler can make a thread go from READY to RUNNING at any time by copying its register values from its TCB to a processor's registers.

**RUNNING** A RUNNING thread can transition to the READY state in two ways:

* The scheduler can preempt a running thread and move it to the READY state by: (1) saving the thread’s registers to its TCB and (2) switching the processor to run the next thread on the ready list.
* A running thread can voluntarily relinquish the processor and go from RUNNING to READY by calling yield (e.g., `thread_yield` in the thread library).

A thread in the **WAITING** state is waiting for some event. Whereas the scheduler can move a thread in the READY state to the RUNNING state, a thread in the WAITING state cannot run until some action by another thread moves it from WAITING to READY.

When a thread is in the **WAITING** state, rather than continuing to run the thread or storing the TCB on the scheduler’s ready list, the TCB is stored on the _waiting list_ of some _synchronization variable_ associated with the event. When the required event occurs, the operating system moves the TCB from the synchronization variable’s waiting list to the scheduler’s ready list, transitioning the thread from WAITING to READY.

## Ready Queue And Various I/O Device Queues

Porocess not running: PCB or TCB is in some scheduler queue

* There are separate queues for each device/signal/condition
* Each queue can have a differernt scheduler policy

![Screenshot 2023-05-27 at 8.15.47 AM](https://p.ipic.vip/5ep9k1.png)

## Implementing Threads

**User Level Threads**

User threads function fully within user space, bypassing kernel intervention for operations. They're managed by user-level thread libraries like POSIX Pthreads or Java Thread API. They switch contexts faster than kernel threads but can block other user threads if one performs a blocking operation.

**Kernel Level Threads**

Kernel threads can execute kernel code and alter kernel data structures. Managed by the OS, they're used for tasks like process scheduling. They offer more concurrency than user threads but have higher context-switching overhead.

**Hybrid Thread Approach**

This method merges benefits of user and kernel threads. The OS recognizes both thread types and can map multiple user threads to one kernel thread or keep a one-to-one ratio. It balances fast context switching of user threads with kernel thread integration, but managing this system can be complex.

### Creating a Thread

The creation of a thread involves the setup of its own stack, where local variables, function arguments, and return addresses are stored. Let's break down the following code that demonstrates this:

```c
void thread_create(thread_t *thread, void (*func)(int), int arg) {
    // Allocate a Thread Control Block (TCB) and a dedicated stack
    TCB *tcb = new TCB();
    thread->tcb = tcb;
    tcb->stack_size = INITIAL_STACK_SIZE;
    tcb->stack = new Stack(INITIAL_STACK_SIZE);
    
    // Initialize the stack pointer (sp) to point to the top of the stack. Recall in Linux, the stack grows downwards
    // (from higher memory address to lower). Also, set the program counter (pc) to point to our stub function.
    tcb->sp = tcb->stack + INITIAL_STACK_SIZE;
    tcb->pc = stub;
    
    // When the system's thread scheduler starts this thread, it's unaware of the specific arguments required by our stub function.
    // Hence, we manually set up the stack to contain these arguments: the function `func` and its integer argument `arg`.
    *(tcb->sp) = arg;        // Push the argument onto the stack
    tcb->sp--;
    *(tcb->sp) = func;       // Push the function pointer onto the stack
    tcb->sp--;
    
    // Set up another stack frame to ensure proper context switching. This aspect will be elaborated in later sections.
    thread_dummySwitchFrame(tcb);
    tcb->state = READY;
    readyList.add(tcb);      // Enqueue the TCB to the list of ready-to-run threads
}

void stub(void (*func)(int), int arg) {
    (*func)(arg);            // Invoke the actual thread function
    thread_exit(0);          // Ensure the thread is properly terminated if `func` doesn't call `thread_exit` on its own
}
```

The essence of the `stub` function is to act as a safety net. When our thread function `func` completes its execution, it needs to know where to return. If we didn't have the `stub` function, and if `func` didn't call `thread_exit`, it would try to return to whatever memory address was initially at the top of the stack – which could be any arbitrary value, leading to unpredictable behavior. Instead, by using the `stub` function, we ensure that once `func` completes, it returns to a controlled environment (`stub`), which then ensures the thread is correctly terminated by invoking `thread_exit`.

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

## POSIX Thread APIs

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
