# Concurrency and Threads

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

- `thread`: A pointer to a `pthread_t` variable that will be filled in with a unique thread ID for the new thread.
- `attr`: A pointer to a `pthread_attr_t` structure that specifies attributes for the new thread.
- `start_routine`: A pointer to the function that the new thread will execute.
- `arg`: An argument that will be passed to the `start_routine` function when it is called by the new thread.

```c
int pthread_yield(void);
```

Give up the processor to let other threads run.

```c
int pthread_join(pthread_t thread, void **retval);
```

- `thread`: The thread ID of the thread to join.
- `retval`: A pointer to a variable that will be filled in with the exit status of the joined thread.

When `pthread_join()` is called, the calling thread blocks until the specified thread terminates. Once the thread has terminated, `pthread_join()` returns and the exit status of the thread is stored in the location pointed to by `retval`.

```
void pthread_exit(void *retval);
```

When `pthread_exit()` is called, the calling thread is terminated and its resources are freed. The exit status of the thread is returned in the `retval` parameter.

**An application of thread: Parallel block zero**

In practice, the operating system will often create a thread to run blockzero in the background. The memory of an exiting process does not need to be cleared until the memory is needed — that is, when the next process is created.

## Thread Data Structures and Life Cycle

### Per-Thread State and Thread Control Block (TCB)

Thread Control Block (TCB)

The thread control block holds two types of per-thread information: 

1. The state of the computation being performed by the thread. 
2. Metadata about the thread that is used to manage the thread.

**Per-thread Computation State**

A pointer to the thread’s stack and a copy of its processor registers.

In some systems, the general-purpose registers for a stopped thread are stored on the top of the stack, and the TCB contains only a pointer to the stack. In other systems, the TCB contains space for a copy of all processor registers.

<img src="https://p.ipic.vip/at02p3.png" alt="Screenshot 2023-05-11 at 2.57.37 AM" style="zoom:50%;" />

**Per-thread Metadata**

Thread ID, scheduling priority and status

### Shared State

Code, global variables and heap

## Thread Life Cycle

![Screenshot 2023-05-11 at 3.19.51 AM](https://p.ipic.vip/ut6ns7.png)

**INT** Thread creation puts a thread into its INIT state and allocates and initializes per-thread data structures. Once that is done, thread creation code puts the thread into the READY state by adding the thread to the *ready list*.

**READY** A thread in the READY state can be run but is not running. Its TCB is on the ready list, and its register values are stored in the TCB. The scheduler can make a thread go from READY to RUNNING at any time by copying its register values from its TCB to a processor's registers.

**RUNNING**  A RUNNING thread can transition to the READY state in two ways:

* The scheduler can preempt a running thread and move it to the READY state by: (1) saving the thread’s registers to its TCB and (2) switching the processor to run the next thread on the ready list.
* A running thread can voluntarily relinquish the processor and go from RUNNING to READY by calling yield (e.g., thread_yield in the thread library).

Linux keep a running thread in the ready list too.

A thread in the **WAITING** state is waiting for some event. Whereas the scheduler can move a thread in the READY state to the RUNNING state, a thread in the WAITING state cannot run until some action by another thread moves it from WAITING to READY.

When a thread is in the **WAITING** state, Rather than continuing to run the thread or storing the TCB on the scheduler’s ready list, the TCB is stored on the *waiting list* of some *synchronization variable* associated with the event. When the required event occurs, the operating system moves the TCB from the synchronization variable’s waiting list to the scheduler’s ready list, transitioning the thread from WAITING to READY.

## Implementing Kernel Threads

* **Kernel threads** The simplest case is implementing threads inside the operating system kernel, sharing one or more physical processors. A *kernel thread* executes kernel code and modifies kernel data structures. Almost all commercial operating systems today support kernel threads.

* **Kernel threads and single-threaded processes.** An operating system with kernel threads might also run some single-threaded user processes.

  <img src="https://p.ipic.vip/3i4w5q.png" alt="Screenshot 2023-05-11 at 11.34.02 AM" style="zoom:50%;" />

  In this figure, the stack in the kernel is the **user interrupt stack**, while the stack in the process is the **user-level stack**.

* **Multi-threaded processes using kernel threads**

  ![Screenshot 2023-05-11 at 11.57.29 AM](https://p.ipic.vip/xdh0ef.png)

  Each thread needs a kernel interrupt stack in the kernel. Here, "in the kernel" means that the data structure is managed and controlled by the kernel of the operating system.

* **User-level threads**

  The thread operations — create, yield, join, exit, and the synchronization routines is completely controlled by the user.

### Creating a Thread

```pseudocode
void
thread_create(thread_t *thread, void (*func)(int), int arg) {
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
 
 void
 stub(void (*func)(int), int arg) {
     (*func)(arg);           // Execute the function func()
     thread_exit(0);         // If func() does not call exit,  call it here.
 }
```

When we create a stack, we get the address of the stack. The stack is a continuous memory region from the starting address. We need to get to the top of the stack and the stack grows from higher address to lower address.

Noting the step of calling stub. We need this extra step in case the func procedure returns instead of calling `thread_exit`. Without the stub, func would return to whatever random location is stored at the top of the stack! Instead, func returns to stub and stub calls thread_exit to finish the thread.
