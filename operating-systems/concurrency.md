# Concurrency

Any thread running in a process can make system calls into the kernel, blocking that thread until the call returns but allowing other threads to continue to run.

When the processor gets an I/O interrupt, it preempts one of the running threads so the kernel can run the interrupt handler; when the handler finishes, the kernel resumes that thread.

The operating system kernel itself implements the thread abstraction for its own use.

A thread is a single sequence that represents a separately schedulable task.

Threads provide an execution model in which each thread runs on a dedicated virtual processor with unpredictable and variable speed.

Execution speeds for the different threads of a program are hard to predict, can vary on different hardware, and can even vary from run to run on the same hardware. As a result, we must coordinate thread actions through explicit synchronization rather than by trying to reason about their relative speed.

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

When `pthread_join()` is called, the calling thread blocks until the specified thread terminates. Once the thread has terminated, `pthread_join()` returns and the exit status of the thread is stored in the location pointed to by `retval`.

```
void pthread_exit(void *retval);
```

When `pthread_exit()` is called, the calling thread is terminated and its resources are freed. The exit status of the thread is returned in the `retval` parameter.

**An application of thread: Parallel block zero**

In practice, the operating system will often create a thread to run block zero in the background. The memory of an exiting process does not need to be cleared until the memory is needed — that is, when the next process is created.

## Thread Data Structures and Life Cycle

### Per-Thread State and Thread Control Block (TCB)

Thread Control Block (TCB)

The thread control block holds two types of per-thread information:

1. The state of the computation being performed by the thread.
2. Metadata about the thread that is used to manage the thread.

**Per-thread Computation State**

A pointer to the thread’s stack and a copy of its processor registers.

In some systems, the general-purpose registers for a stopped thread are stored on the top of the stack, and the TCB contains only a pointer to the stack. In other systems, the TCB contains space for a copy of all processor registers.

<figure><img src="https://p.ipic.vip/at02p3.png" alt="" width="375"><figcaption></figcaption></figure>

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

## Implementing Kernel Threads

* **Kernel threads** The simplest case is implementing threads inside the operating system kernel, sharing one or more physical processors. A _kernel thread_ executes kernel code and modifies kernel data structures. Almost all commercial operating systems today support kernel threads. (e.g., process scheduling, memory management)

*   **Kernel threads and single-threaded processes.** An operating system with kernel threads might also run some single-threaded user processes.

    <figure><img src="https://p.ipic.vip/3i4w5q.png" alt="" width="375"><figcaption></figcaption></figure>

    Every thread corresponds to a user stack and kernel stack. 
    
*   **Multi-threaded processes using kernel threads**

    ![](https://p.ipic.vip/xdh0ef.png)

    Each thread needs a kernel interrupt stack in the kernel. Here, "in the kernel" means that the data structure is managed and controlled by the kernel of the operating system.
    
*   **User-level threads**

    The thread operations — create, yield, join, exit, and the synchronization routines is completely controlled by the user.

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

Noting the step of calling stub. We need this extra step in case the func procedure returns instead of calling `thread_exit`. Without the stub, func would return to whatever random location is stored at the top of the stack! Instead, func returns to stub and stub calls thread\_exit to finish the thread.

### Deleting a Thread

When a thread calls `thread_exit`,  there are two steps to deleting the thread: 

* Remove the thread from the ready list so that it will never run again. 
* Free the per-thread state allocated for the thread.

An important subtlety: After the thread remove itself from the ready list, an interrupt occurs before the thread finishes deallocating its memory. **Memory leak**.

A thread never deletes its own state. Some other thread must do it. On exit, the thread transitions to the FINISHED state, moves its TCB from the ready list to a list of finished threads the scheduler should never run. Once the finished thread is no longer running, it is safe for some *other* thread to free the state of the thread.

### Thread Context Switch

The switch saves the currently running thread’s registers to the thread’s TCB and stack, and then it restores the new thread’s registers from that thread’s TCB and stack into the processor.

A thread context switch can be triggered by either a voluntary call into the thread library, or an involuntary interrupt or processor exception.

**Voluntary**

`thread_yield` call lets the currently running thread voluntarily give up processor to the next thread on the ready list.

`thread_join` and `thread_exit`.

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

We do not want to do an involuntary context switch while we are in the middle of a voluntary one. When switching between two threads, we need to temporarily defer interrupts until the switch is complete, to avoid confusion. Processors contain privileged instructions to defer and re-enable interrupts; we make use of these in our implementation below.

> **Why it is necessary to turn off interrupts during thread switch**
>
> A scenario:
>
> * A low priority thread is about to voluntarily switch to a high priority thread. It has pulled the high priority thread off the ready list
> * An interrupt occurs. A medium priority thread is moved from WAITING to READY
> * Since it appears that the processor is still running the low priority thread, the interrupt handler immediately switches to the medium thread.
> * It has to wait the low thread to reschedule to switch from the low priority to high priority

```c
// We enter as oldThread, but we return as newThread.
// Returns with newThread’s registers and stack.
void thread_switch(oldThreadTCB, newThreadTCB) {
    pushad;                  // Push general register values onto the old stack.
    oldThreadTCB->sp = %esp; // Save the old thread’s stack pointer.
    %esp = newThreadTCB->sp; // Switch to the new stack.
    popad;            // Pop register values from the new stack.
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

This means arranging the data on the stack to look as if the thread had been running and then voluntarily yielded control. This setup usually includes the return address where execution should resume (which, in this case, would be the start of the thread's function), and possibly also dummy values for saved register contents.

**Involuntary Kernel Thread Context Switch**

Comparing a switch between kernel threads to what happens on a user-mode transfer:

* There is no need to switch modes (and therefore no need to switch stacks) and
* The handler can resume any thread on the ready list rather than always resuming the thread or process that was just suspended.

On most processor architectures, a simple (but inefficient) way to swap to the next thread from within an interrupt handler is to call `thread_switch` just before the handler returns. As we have already seen, thread_switch saves the state of the current thread (that is, the state of the interrupt handler) and switches to the new kernel thread. When the original thread resumes, it will return from `thread_switch`, and immediately pop the interrupt context off the stack, resuming execution at the point where it was interrupted. That is, the sequence of execution is $A\rightarrow Handler\rightarrow B \rightarrow A$.

To optimize this, some OSs like Linux imitate the interrupt hardware's fashion of saving context. That's to say, saving return instruction pointer and the `eflags` register first and then calls the `pushad` instruction to save the state of the general-purpose registers. This way, whether a thread was suspended by a hardware interrupt or a software call (like `thread_switch()`), resuming the thread is the same operation.

## Combing Kernel Threads and Single Threaded User Processes

Like a TCB, a PCB for a single-threaded process must store the processor registers when the process’s thread is not running. In addition, the PCB has information about the process’s address space.

An important detail is that many processor architectures have extra co-processor state, e.g., floating registers, for user-level code. Typically, the operating system kernel does not make use of floating point operations. Therefore, the kernel does not need to save those registers when switching between kernel threads, but it does save and restore them when switching between processes.

When switching from user thread to kernel handler (triggered by software interrupt), the IP, eflags and stack pointer are pushed onto the kernel stack.

However, switching between a kernel thread and a kernel handler is slightly different:

**Entering the handler.** When an interrupt or exception occurs, if the processor detects that it is already in kernel mode (by inspecting the eflags register), it just pushes the instruction pointer and eflags registers (but not the stack pointer) onto the existing stack. On the other hand, if the hardware detects that it is switching from user-mode to kernel-mode, then the processor also changes the stack pointer to the base of the interrupt stack and pushes the original stack pointer along with the instruction pointer and eflags registers onto the new stack.

**Returning from the handler.** When the iret instruction is called, it inspects both the current eflags register and the value on the stack that it will use to restore the earlier eflags register. If the mode bit is identical, then iret just pops the instruction pointer and eflags register and continues to use the current stack. On the other hand, if the mode bit differs, then the iret instruction pops not only the instruction pointer and eflags register, but also the saved stack pointer, thus switching the processor’s stack pointer to the saved one.

## Implementing Multi-Threaded Processes

### Implementing Multi-Threaded Processes Using Kernel Threads

User-level threads are abstractions provided by a library in user space (like POSIX threads, or "pthreads") and their scheduling can be managed within the user space. Multiple user-level threads can be multiplexed on a single kernel thread, or they can be implemented with a 1:1 mapping to kernel threads, depending on the threading library and the specific operating system's implementation.

User-level threads still rely on kernel services, such as for creating or destroying threads, or for handling synchronization primitives like mutexes or semaphores. These services are typically accessed by making system calls into the kernel.

To create a thread, the user library allocates a user-level stack for the thread and then does a system call into the kernel. The kernel allocates a TCB and interrupt stack, and arranges the state of the thread to start execution on the user-level stack at the beginning of the requested procedure. The kernel needs to store a pointer to the TCB in the process control block; if the process exits, the kernel must terminate any other threads running in the process.

### Implementing User-Level Threads Without Kernel Support

The basic idea is simple. The thread library instantiates all of its data structures within the process: TCBs, the ready list, the finished list, and the waiting lists all are just data structures in the process’s address space. Then, calls to the thread library are just procedure calls, akin to how the same functions are implemented within a multi-threaded kernel.

A limitation of green threads is that the operating system kernel is unaware of the state of the user-level ready list. If the application performs a system call that blocks waiting for I/O, the kernel is unable to run a different user-level thread. Likewise, on a multiprocessor, the kernel is unable to run the different threads running within a single process on different processors.

You can achieve preemptive user-level threads by means of signals

1. The user-level thread library makes a system call to register a timer signal handler and signal stack with the kernel.
2. When a hardware timer interrupt occurs, the hardware saves P ’s register state and runs the kernel’s handler.
3. Instead of restoring P ’s register state and resuming P where it was interrupted, the kernel’s handler copies P ’s saved registers onto P ’s signal stack.
4. The kernel resumes execution in P at the registered signal handler on the signal stack.
5. The signal handler copies the processor state of the preempted user-level thread from the signal stack to that thread’s TCB.
6. The signal handler chooses the next thread to run, re-enables the signal handler (the equivalent of re-enabling interrupts), and restores the new thread’s state from its TCB into the processor. execution with the state (newly) stored on the signal stack.

### Implementing User-Level Threads With Kernel Support

**Hybrid Thread Join** A special case: the objective thread has finished before the current thread calling `thread_join()`. In this case, we can store the exit status of the objective thread in the process's address space.

**Per-Processor Kernel Threads** When the application starts up, the user-level thread library creates one kernel thread for each processor on the host machine. As long as there is no other activity on the system, the kernel will assign each of these threads a processor. Each kernel thread executes the **user-level** scheduler in parallel: pull the next thread off the user-level ready list, and run it. Because thread scheduling decisions occur at user level, they can be flexible and application-specific; for example, in a parallel graph algorithm, the programmer might adjust the priority of various threads based on the results of the computation on other parts of the graph.

**Scheduler Activations**

Based on the above approach, the user-level thread scheduler is notified (or activated) for every kernel event that might affect the user-level thread system.
