# Exceptions, Kernels and Processes

A central role of operating system: protection

The control flow of the processor: Each transition from one address of a certain instruciton to another address of instruction is called **flow of control**, or **control flow** of the processor.

Applications request services from the operating system by using a form of ECF known as a **trap** or **system call**.

## Exceptions

An **exception** is an abrupt change in the control flow in response to some change in the processor’s state.

Processor’s state: encoded in various bits and signals inside the processor. The change in state is known as an **event**.

Change of processor’s state is triggered by events.

In the case, when the processor detects that the event has occured, it makes an indirect procedure call(the exception), through a jump table called an **exception** **table**, to an operating system subroutine(the **exception hanlder**) that is specially designed to process this particular kind of event. When the exception handler finishes processing, one of three things happens, depending on the type of event that caused the exception:

* returns to $I\_{curr}$
* returns to $I\_{next}$
* Aborts the interrupted program

### Exception Handling

Exceptions denoted with **exception number**s. Assigned by processor or kernel.

At system boot time, the operating system allocates and initializes a jump table called an exception table.

The exception number is an index into the exception table, whose starting address is contained in a special CPU register called the **exception table base register**.

When control is being transfered from a user program to the kernel, all of these items are pushed onto the kernel stack rather than onto the user’s stack(The difference between kernel stack adn user stack is explained later on).

After the handler has processed the event, it optionally returns to the interrupted program by executing a special “return from interrupt” instruction.

### Classes of Exceptions

| Class     | Cause                           | Async/sync | Return behavior           |
| --------- | ------------------------------- | ---------- | ------------------------- |
| Interrupt | Signal from I/O device / Signal | Async      | Next instruction          |
| Trap      | Intentional exception           | Sync       | Next instruction          |
| Fault     | Potentially recoverable error   | Sync       | Might current instruction |
| Abort     | Nonrecoverable error            | Sync       | Never returns             |

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

| Exception number | Description              | Exception class   |
| ---------------- | ------------------------ | ----------------- |
| 0                | Divide error             | Fault             |
| 13               | General protection fault | Fault             |
| 14               | Page fault               | Fault             |
| 18               | Machine check            | Abort             |
| 32-255           | OS-defined exceptions    | Interrupt or trap |

Numbers in the range from 0 to 31 is defined by Intel architecture and thus are identical to any x86-64 system.

![Untitled](https://p.ipic.vip/dythmj.jpg)

All arguments to Linux system calls are passed through general purpose registers rather than the stack. By convention, `%rax`contains the syscall number, with up to six arguments in `%rdi`,`%rsi`,`%rdx`, `%r10`,`%r8` and `%r9`. On return from the system, registers `%r11`and `%rcx`are destroyed, and `%rax` contains the return value. A negative return value between -4095 and -1 indicates an error corresponding to negative `errno`.

## System Call Error Handling

When Unix-like systems' system-level functions encounter an error, they typically return $-1$ and set the global integer variable `errno` to indicate what went wrong.

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

## Process

A process is an instance of program in execution.

What happens when we run a program in shell: A process is created and the executable object file runs in this context.

The operating system keeps track of the various processes on the computer using a data structure called the **process control block(PCB).** The process control block stores all the information the operating system needs about a particular process: where it is stored in memory, where its executable image is on disk, which user asked it to start executing, what privileges the process has, and so forth.

The private address space for a process is like:
![Untitled](https://p.ipic.vip/hxsui1.jpg)

> In Intel processors, the privilege level of a process is stored in a field within the Code Segment (CS) register. This field is known as the Current Privilege Level (CPL).
>
> The CPL can have four levels of privilege from 0 to 3, with level 0 being the most privileged and level 3 being the least. Typically, the kernel code runs at privilege level 0 (known as Ring 0), and user applications run at privilege level 3 (Ring 3).
>
> When a user-level process executes a system call, it triggers a transition from Ring 3 to Ring 0. The processor automatically changes the CPL to 0. When the kernel code has finished handling the system call, it executes a special return-from-system-call instruction, and the processor switches the CPL back to 3.

**Each thread of a multi-thread program have its own stack and PC.** They do **share the data(typically global variables)** and code from the process. 

> Because they share the data, race conditions may happen from time to time. Mutex is used for thread synchronization. It ensures that only one thread has access to a critical section or data by using operations like a lock and unlock. A thread having the lock of mutex can use the critical section while other threads must wait till the lock is released. (This will be covered later) 

## Process Control

### Obtaining Process IDs

`getpid` and `getppid`

### Creating and Terminating Processes

A process in three states:

* Running

* Stopped

  The execution of this process is suspended and will not be scheduled. A process stops as a result of receiving a `SIGSTOP`, `SIGTSP`, `SIGTTIN` or `SIGTTOU` signal, and remains stopped until it receives a `SIGCONT` signal, at which point it becomes running again.

* Terminated

  A process become terminated for:

  * Receving a signal whose default action is to terminate the process
  * Returning from the main routine
  * Calling the `exit` function

The child gets an identical (but separate) copy of the parent’s user-level virtual address space. The child also gets identical copies of any of the parent’s open file descriptors.

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

By default(`options = 0`) `waitpid` temporarily suspends execution of the calling process until a child process in its wait set terminates. After the function returns, the termianted child has been reaped and the kernel removes all traces of it from system.

The value of `pid` determines the members of the wait set:

* If `pid` > 0, …
* If `pid` = -1, the wait set contains all of the parent’s child processes.

The default behavior can be modified by setting the options:

* `WNOHANG` Return immediately (with a return value of 0) if none of the child processes in the wait set has terminated yet
* `WUNTRACED` Suspend execution of the calling process until a process in the wait set becomes **either terminated or stopped**. Return the PID of the terminated or stopped child that caused the return.
* `WCONTINUED` Suspend execution of the calling process until a running process in the wait set is terminated or until a stopped process in the wait set has been resumed by the receipt of a `SIGCONT` signal.
* `WNOHANG|WUNTRACED` Return immediately, with a return value of 0, if none of the children in the wait set has stopped or terminated, or with a return value equal to the PID of one of the stopped or terminated children.

The exit status of a reaped child in `*statusp`:

* `WIFEXITED(status)` Returns true if the child terminated normally, via a call to exit or a return. 
* `WEXITSTATUS(status)` Returns the exit status of a normally terminated child. This status is only defined if `WIFEXITED()`returned true. 
* `WIFSIGNALED(status)` Returns true if the child process terminated because of a signal that was not caught. 
* `WTERMSIG(status)` Returns the number of the signal that caused the child process to terminate. This status is only defined if `WIFSIGNALED()` returned true. 
* `WIFSTOPPED(status)` Returns true if the child that caused the return is currently stopped. 
* `WSTOPSIG(status)` Returns the number of the signal that caused the child to stop. This status is only defined if `WIFSTOPPED()` returned true. 
* `WIFCONTINUED(status)` Returns true if the child process was restartedby receipt of a SIGCONT signal.

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

Returns zero if the requested amount of time has elapsed, and the number of seconds still left to sleep otherwise. The function returns the number of seconds left unslept if the sleep is interrupted by a signal handler.

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

![Untitled](https://p.ipic.vip/w9j5nw.png)

```c
#include <stdlib.h>

char *getenv(const char *name);

// Returns: pointer to name if it exists, NULL if no match

int setenv(const char *name, char *newvalue, int overwrite);

// Returns: 0 on success, -1 on error

void unsetenv(const char *name);

// Returns: nothing
```

## Dual-mode operation

In user-mode, the processor checks each instruction before executing it to verify that the instruction is permitted to be performed by that process.

In kernel-mode, there’s no verification.

Hardware needed to do the protection:

* Privileged instructions
* Memory Protection
* Timer interrupts

### Privileged instructions

Process isolation is only possible if there is a way to **limit programs running in user-mode from directly changing their privilege level.**

Other than trapping into the system kernel at the system call locations, an application process **cannot be allowed to change its privilege level**.

**Cannot be allowed to change the set of memory locations it can access**

**Cannot disable processor interrupts**

**Privileged instructions:** avaliable only in kernel-mode

When a process attempts to access the memory it is not allowed to access, exception happens. Transfer control to exception handler.

Usually, the operating system kernel simply halts the process on privilege violations, as it often means that the application’s code has encountered a bug.

### Memory protection

Early computers: base and bounds

Can only be changed by privileged instructions

Cost: two extra comparisons for each instruction

Gain: memory protection

Only needed to check against base and bounds in user-mode

Unable to provide:

* Expandable heap and stack
* Memory sharing
* Non-relative memory address
* Memory fragmentation

Modern solution: virtual address

### Timer interrupts

A hardware timer: interrupt the processor after a certain delay.

After resetting the timer, the operating system will resume execution of the process.

## Safe control transfer

Three reasons for transferring from user-mode to kernel-mode:

* **Exceptions catched** 

  Can be used to set breakpoints

*   **Interrupts arrived**

    **Hardware Interrupt**: timer events/ the completion of I/O requests

    An alternative to interrupts: polling. The kernel loop and check if an event has occured and is needed to be handled

    **Interprocessor interrupts**: coorindate actions across the multiprocessor

*   System calls

    Most processors implement system calls using a special trap instruction.

    Applications trap only to a pre-defined address

### Safe mode switch

A common sequence for entering the kernel and returning from the kernel

*   Limited entry points

    The user program cannot jump arbitrarily.
*   Automic changes to processor state

    mode, program counter, stack and memory protection all changed at the same time.
* Transparent, restartable execution

**Interrupt vector**

A **special register** pointing to **an area of kernel memory** called the interrupt vector. The interrupt vector is an array of pointers, with each pointer pointing to the first instruction of a handler procedure.

x86: 0-31 hardware exceptions    32-255 interrupts     entry 64 points to the system call trap handler

**Interrupt handler stack**

A **privileged hardware register** pointing to a region of **kernel memory** called the interrupt handler stack.

Procedure:

* Save some of the interrupted process’s registers onto the interrupt stack (done by hardware)
* call the kernel handler
* Save the remaining registers(done by the handler)
* do the handler work

Procedure of returning from the interrupt, exception or trap:

* pop the registers stored by the handler
* hardware restore the registers it saved into the interrupt stack

> When a system call/software interrupt/exception occurs, the user's register information is saved on the **kernel stack**.
>
> When a thread is rescheduled, the user's register information is saved on the **user stack**.
>
> When a ***hardware*** interrupt is triggered, the user's register information is saved on the **interrupt handler stack**.
>
> When an interrupt or exception occurs, the user's register information is saved **on the interrupt handler stack**.
>
> Each thread corresponds to one user stack and one kernel stack, and each processor corresponds to one interrupt handling stack.
>
> This applies to Linux x86 and x86_64

> Linux has a per-process kernel stack that is used for system calls, exceptions, and other kernel-level operations. However, when it comes to handling hardware interrupts, Linux uses a single, per-processor interrupt stack.

**Interrupt masking**

Interrupts arrive asynchronously may cause confusion when one interrupt handler is executing and another interrupt comes.

The hardware provides a privileged instruction to temporarily defer delivery of interrupt until it is safe to do so. On the x86 and several other processors, this instruction is called **disable interrupts.** The interrupt is only deferred(masked) and not ignored. The instruction are previleged.

**Hardware support for saving and restoring registers**

Once the handler starts running, it can use the `pushad` instruction to save the remaining registers onto the stack.

`pushad` saves the x86 integer registers; because the kernel does not typically do ﬂoating point operations, those do not need to be saved unless the kernel switches to a new process.

`popad ` pop an array of integer register values off the stack into the registers

`iret` instruction that loads a stack pointer, instruction pointer and processor status word off the stack into the appropriate processor registers.

**Putting it all together: Mode switch on the x86**

The x86 is segmented. Pointers come in 2 parts: a segment, such as code, data or stack, and an offset within that segment.

The current user-level instruction is based on a combination of the code segment(`cs` register plus the instruction pointer `eip`)

> 1. **Code Segment (CS)**: This segment contains the actual executable code of the program.
> 2. **Data Segment (DS)**: This segment contains static data such as global variables.
> 3. **Stack Segment (SS)**: This segment contains the program's execution stack, which includes local variables and function call information.
> 4. **Extra Segment (ES)**: This segment is generally used for extra data and is sometimes used by certain instructions that need to access data in a different segment.
> 5. **FS, GS**: Additional segments that can be used for various purposes depending on the specific needs of a program.

The current stack position is based on the stack segment `ss` and the stack pointer within the stack segment `esp`.

1. Save three key values. The hardware **internally** saves the value of the stack pointer (the x86 `esp` and `ss` registers), the execution ﬂags (the x86 `eflags` register), and the instruction pointer (the x86 `eip` and `cs` registers).
2. Switch onto the interrupt handler stack. The hardware then switches the stack pointer to the base of the kernel handler stack. The hardware switches to a new stack if the Interrupt Stack Table (IST) feature is used. The new stack's address is found in the IST, which is a part of the Task State Segment (TSS).
3. Push the three key values onto the new stack. The hardware then stores the internally saved values onto the stack.
4. Optionally save error code. Certain types of exceptions such as page faults **generate an error code** to provide more information about the event; for these exceptions, the hardware pushes this code as the last item on the stack. For other types of events, the software interrupt handler typically pushes a dummy value onto the stack so that the stack format is identical in both cases.
5. Invoke the interrupt handler. Finally, the hardware changes the program counter to the address of the interrupt handler procedure, speciﬁed via a special register in the processor that is accessible only to the kernel. This register contains a pointer to an array of exception handler addresses in memory. The type of interrupt is mapped to an index in this array, and the program counter is set to the value at this index.

In the interrupt handler process, `pushad` pushes the rest of the registers, **including the current stack pointer**, onto the stack. x86 `pushad` pushes the contents of all general purpose registers onto the stack.

At this point the kernel’s interrupt handler stack holds

1. the stack pointer, execution ﬂags, and program counter saved by the hardware
2. an error code or dummy value
3. a copy of all of the general registers (including the stack pointer but not the instruction pointer or eﬂags register)

To prevent an inﬁnite loop, the handler modiﬁes the program counter stored at the base on the stack to point to the instruction immediately after the one causing the mode switch.

### System calls

Trap instructions are instructions used in computer processors to cause the operating system to **switch from user mode to kernel mode.**

Issue a system call by **executing the trap instruction** to transfer control to the operating system kernel

To issue a system call:

| Architecture | Method  |
| ------------ | ------- |
| x86          | int     |
| x86-64       | syscall |

The system call handler will implement each system call. It runs in kernel mode. When a system call is made, the arguments of it should be carefully validated by the system call handler.

A pair of stubs are two short procedures that mediate between two environments, in this case between the user program and the kernel.

![Screenshot 2023-05-24 at 7.22.07 PM](https://p.ipic.vip/kfuqa8.png)

The syscall function takes care of marshalling the arguments passed by the user program into a format that can be understood by the system call handler, and handles any necessary validation of the arguments.

**x86** The system call calling convention is arbitrary, so here we pass arguments on the user stack, with a code indicating the type of system call in the register `%eax`. The return value comes back in `%eax` so there is no work to do on the return.

The kernel stub has four tasks:

* **Locate system call arguments:** the arguments are stored on the process’s user stack. We should convert the virtual addresses of the arguments to physical addresses.
* **Validate parameters**: you cannot trust the processes.
* **Copy before check:** the kernel copies system call parameters into kernel memory before performing the necessary checks.
* **Copy back any results**

In turn, the system call handler pops any saved registers (except `%eax`) and uses the `iret` instruction to return back to the user stub immediately after the trap, allowing the user stub to return to the user program.

### Staring a new process

The kernel allocates and initializes the process control block, allocates memory for the process, copies the program from disk into the newly allocated memory, and allocates **both** a **user-level** stack for normal execution and a **kernel-level** stack for handling system calls, interrupts and exceptions.

Arguments of a **program** are stored in the higher address.

When we create the new process, we allocate it a kernel stack, and we reserve room at the bottom of the kernel stack for the initial values of its registers, program counter, stack pointer, and processor status word. To start the new program, we can then switch to the new stack and **jump to the end of the interrupt handler**. When the handler executes `popad` and `iret`, the processor “returns” to the start of the user program.

`Exit` is a system call that terminates the process.

### Upcalls

We call virtualized interrupts and exceptions **upcalls.** In UNIX, they are called **signals**, and in Windows they are called **asynchronous events**.

Each process deﬁnes its own handlers for each signal type, much as the kernel deﬁnes its own interrupt vector table. If a process does not deﬁne a handler for a speciﬁc signal, then the kernel calls a default handler instead.

Applications have the option to run UNIX signal handlers either on the process’s normal execution stack or on a special signal stack allocated by the user process in user memory. Running signal handlers on the normal stack can reduce the ﬂexibility of the signal handler in manipulating the stack, e.g., to cause a language-level exception to be raised.

A UNIX signal handler automatically masks further delivery of that type of signal until the handler returns. The program can mask other signals, either all together or individually, as needed.

**Handling signals in user-level program**

When the timer interrupt occurs, the hardware generates an interrupt request(IRQ) signal to the processor, which causes the processor to switch from user mode to kernel mode.

The kernel interrupt handler saves the current state of the user-level computation onto the kernel interrupt handler stack.

The kernel then copies the saved state from the kernel stack to a **user-level buffer**, which is a special area of memory reserved for handling signals and interrupts. This buffer contains the saved state of the user-level program that was interrupted by the timer interrupt.

The kernel then sets the program counter register to the singal handler and the stack pointer register to signal stack. The signal stack is a special area of memory reserved ofr handling signals.

The kernel then returns from the interrupt handler, and the `reti` instruction resumes execution at the signal handler, rather than the original program counter. The signal handler is a user-level function that is responsible for handling the timer interrupt.

When the signal handler has finished executing, it returns control to the kernel.

The kernel copies the processor state from the signal handler back into kernel memory.

The kernel then returns to the interrupted user-level program, using the saved state from the user-level buffer to restore the program's original state.

## Singals

![Untitled](https://p.ipic.vip/ycf4ng.jpg)

A signal is a small message that notifies a process that an event of some type has occured in the system.

Singals provide an mechanism for exposing the occurence of some hardware exceptions to user processes.

### Singal Terminology

**Sending a signal**

The kernel sends a signal to a destination process by updating some state in the context of the destination process.

The signal is delivered for 2 reasons:

* The kernel has detected a system event such as divide-by-zero or the termination of a child process
* A process has invoked the kill function to explicitly **reuquest the kernel to send a signal** to the signal to the destination process. A process can send a signal to itself.

**Receiving the signal**

A destination process receives a signal when it is forced by the kernel to react in some way to the delivery of the signal. The process can either ignore the signal, terminate or catch the signal by executing a user-level function called a signal handler.

A signal that has been sent but not yet received is called a pending signal. At any point in time, there can be **at most one** pending signal of a particular type for a. If a process has a pending signal of type _k_, then any subsequent signals of type _k_ sent to that process are not required; they are simply discarded.

A pending signal is received at most once. **For each process**, th kernel maintains the set of pending signals in the `pending` bit vector( but the `pending` bit vector is not stored in PCB! ), and the set of blocked signals in the `blocked` bit vector. The kernel sets bit _k_ in `pending` whenever a signal of type _k_ is delivered and clears it whenever it is received.

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

When the kernel switches a process p from kernel mode to user mode, it checks the set of unblocked pending signals (`pending&~blocked`) for p.

If the set is empty, then the kernel passes control to the next instruction in the logical control flow of p.

If the set is nonempty, then the kernel chooses some signal k in the set(typically the smallest k) and forces p to receive signal k.

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

* SIG\_BLOCK. Add the signals in `set` to blocked (`blocked = blocked | set`).
* SIG\_UNBLOCK. `blocked = blocked & ~set`.
* SIG\_SETMASK. `blocked = set`.

If `oldset`is non-NULL, the previous value of the `blocked` bit vector is stored in `oldset`.

### Writing Signal Handlers

**Safe Signal Handling**

The signal handlers run concurrently with the main program. If they try to access teh same global data structure concurrently,the results can be unpredictable.

Guidelines:

* Keep handlers as simple as possible

* Call only **async-signal-safe (or simple “safe”)** functions in your handlers

  i.e. can be safely called from a signal handler.

  Either it is _reentrant_ (e.g. accesses only local variables)

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

* Save and restore `errno`.

  Many of the Linux async-signal-safe functions ser `errno` when they return with an error. Calling such functions inside a handler might interfere with other parts of the program that rely on `errno`.

  Workaround: save `errno` to a local variable on entry to the handler can restore it before the handler returns. It’s not necessary if the handler terminates the process by calling `_exit`.

* Protecting accesses to shared global data structures by blocking all signals.

  If sharing a global data structure, then the handlers and main program should temporarily block all signals when accessing(reading or writing) that data structure.

* Declare global variables with `volatile`

  To an optimizing compiler, the compiler will cache a global variable in register. Using `volatile` will force the compiler to read the value from memory each time it is referenced in the code.

  Since threads run asynchronously, any update of global variables due to one thread should be fetched freshly by the other consumer thread.

* Declare flags with `sig_atomic_t`

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

The key ides is that the existence of a pending signal merely indicates that _**at least**_ one signal has arrived.

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

* Some older Unix systems restore the action for signed k to its default after signal k has been caught by a handler. The handler should be reinstalled.
* On some older versions of Unix, slow system calls that are interrupted when a handler catches a signal do not resume when the signal handler returns but instead return immediately to the user with an error condition and `errno` set to $\tiny{EINTR}$. On these systems, programmers must include code that manually restarts interrupted sysetm calls.

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

By blocking `SIGCHLD` signals before the call to fork and then unblocking them only after we have called `addjob`, we guarantee that the child will be reaped _after_ it is added to the job list. Notice that children inherit the blocked set of their parents, so we must be careful to unblock the `SIGCHLD` signal in the child before calling `execve`.

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

## Case Study: Booting an operating system kernel

The BIOS reads bootloader from flash RAM or disk.

## Case Study: Virtual machines

**host operating system**

**guest operating system**

# The Programming Interface

A **micro kernel** isolates privileged but less critical parts of operaing systems to such as the file system and window system, from the rest of the kernel.

The functions are ecapsulated into user-level processes or servers and access from user programs via interprocess communication.

However, transferring control to a user-level file system server via the kernel is even more costly than transferring control into the kernel. 

## Process Management

A shell is a job control system.

### Windows Process Management

Add a system call to create a process. In Windows, there is a routine called `CreateProcess`. The steps that are needed to take are:

* Create and initialize the process control block (PCB) in the kernel
* Create and initialize a new address space
* Load the program `prog` into the address space
* Copy arguments `args` into memory in the address space
* Initialize the hardware context to start execution at “start”
* Inform the scheduler that the new process is ready to run

### UNIX Process Management

UNIX splits `CreateProcess` in two steps, called `fork` and `exec`.

`fork()` gets a copy of the program. The copy can then do whatever is necessary to set up the context of the child. Once the context is set, the copy then calls UNIX `exec` to copy in the new program and start running it.

```c
pid = fork();
if(pid == 0)
	exec(...);
else
	wait(pid);
```

The steps for implementing UNIX `fork` are:

* Create and initialize the PCB in the kernel
* Create a new address space
* Initialize the address space with a copy of the entire contents of the address space of the parent
* Inherit the execution context of the parent
* Inform the scheduler that the new process is ready to run

The steps for implementing UNIX `exec` are:

* Load the program `prog` into the current address space
* Copy arguments `args` into memory in the address space
* Initialize the hardware context to start execution at "start"

UNIX has a system call `wait(pid)`. The parent process waits the child process to finish, crash or terminate.

> Waiting for a thread to complete is called "thread join"
>
> In Windows, however, there is a single function called "WaitForSingleObject" that wait for process completion, thread completion, or on a condition variable.

UNIX also provides a system call to send another process an instant notification, or upcall. It is sent by calling `signal`.

## Input/output

All device I/O, file operations and interprocess communication use the same set of system calls: open, close, read and write.

`open` wil return a handle to be used in the later calls to read, write and close to identify the file, device or channel. This is called "file descriptor".

Stream data such as from the network or keyboard is stored in a kernel buffer and returned to the application on request. If no data is available to be returned immediately, the `read` call blocks until it arrives.

The outgoing data is stored in a kernel buffer for transmission when the device becomes available. If the application generates data faster than the device can receive it (as is common when spooling data to a printer), the write system call blocks in the kernel until there is enough room to store the new data in the buﬀer.

Explicitly call the close will decrement the reference-count on the device, and garbage collect any unused kernel data structures.

For interprocess communication:

* Pipes

  A UNIX pipe is a kernel buﬀer with two ﬁle descriptors, one for writing and one for reading.

  Data is read exactly the same order sequence it is written. The pipe terminates when either endpoint closes the pipe or exits.

  While pipe connects two processes on the same machine, TCP provides a bi-directional pipe between two processes running on different machines.

* Replace file descriptor

  `dup(from, to)` replaces the `to` file descriptor with a copy of the `from` file descriptor.

* Wait for multiple reads

  The UNIX system call `select(fd[], number)` addresses this. Select allows the server to wait for input from any of a set of ﬁle descriptors; it returns the ﬁle descriptor that has data, but it does not read the data.

  Windows has an equivalent function, called `WaitForMultipleObjects`.

## Case Study: Implementing a Shell

```c
int main() {
  char *prog = NULL;
  char **args = NULL;
  
  while(readAndParseCmdLine(&prog, &args)) {
    int child_pid = fork();
    if (child_pid == 0) {
      exec(prog, args);
    } else {
      wait(child_pid);
      return 0;
    }
  } 
}
```

My question: why do we mention pipe here?

A **producer-consumer** relationship: the output of the preprocessor is sent to to the parser.

## Case Study: Interprocess Communication

Three widely used forms of interprocess communication:

* Producer-consumer
* Client server: two way communication between processes, as in client-server computing.
* File system: can be separated in time

### Producer-consumer communication

![image-20230426160834164](https://p.ipic.vip/lyaunf.png)

As one process computes and produces a stream of output data, it issues a sequence of **write** system calls **on the pipe into the kernel**. Each write can be of variable size. Assuming there is room in the **kernel buﬀer**, the kernel copies the data into the buﬀer, and returns immediately back to the producer.

The consumer issues a sequence of **read** calls. The consumer can read the data out in any convenient chunking.

If the kernel buffer is full, the kernel stalls the producer process until there is room to store the data.

If the kernel buffer is empty, the kernel stalls the consumer process until there the producer produces more data.

Eventually, the consumer reads the last of the data, and the read system call will return an “end of ﬁle” marker. Thus, to the consumer, there is no diﬀerence between reading from a pipe and reading from a ﬁle.

Decoupling the execution of the producer and consumer through the use of kernel buﬀers reduces the number and cost of context switches. This means that the producer and consumer runs concurrently. If not, the producer needs to be defered and picked up frequently . A lot of context switches is costly.

Modern computers make extensive use of hardware caches to improve performance, but caches are ineﬀective if a program only runs for a short period of time before it must yield the processor to another task. Hence, it is not feasible to do a lot of context swithes. The kernel buﬀer allows the operating system to run each process long enough to beneﬁt from reuse, rather than alternating between the producer and consumer on each system call.

### Client-server Communication

There are two pipes, one for each direction.

A server can be connected to a couple of clients. For this, the server uses the select system call, to identify the pipe containing the request to be read.

```c
char request[RequestSize];
char replt[ReplySize];
FileDescriptor clientInput[NumClients];
FileDescriptor clientOutput[NumClients];

while(fd = select(clientInput, NumClients)) {
  read(clientInput[fd], request, RequestSize);
  write(clientOutput[fd], reply, ReplySize);
}
```

## Operating System Structure

Almost all modern operating systems have both a hardware abstraction layer and dynamically loaded device drivers.

The hardware abstraction layer (HAL) is a portable interface to machine-speciﬁc operations within the kernel.

A dynamically loadable device driver is software to manage a speciﬁc loadable device driver device or interface or chipset, that is added to the operating system kernel after the kernel starts running, to handle the devices that are present on a particular machine.

Some device drivers may crash or even corrupt the kernel. Therefore, we can run them inside driver sandbox ot a guest operating system running on a virtual machine.

In practice, most systems adopt a hybrid model where some operating system services are run at user-level and some are in the kernel, depending on the speciﬁc tradeoﬀ between code complexity and performance.
