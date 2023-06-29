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

When `pthread_join()` is called, **the calling thread blocks until the specified thread terminates.** Once the thread has terminated, `pthread_join()` returns and the exit status of the thread is stored in the location pointed to by `retval`.

```c
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

*   **Kernel threads and single-threaded processes.** An operating system with kernel threads might also run some single-threaded user processes.

    <img src="https://p.ipic.vip/3i4w5q.png" alt="" width="375">

    Every thread corresponds to a user stack and kernel stack. 
    
*   **Multi-threaded processes using kernel threads**

    <img src="https://p.ipic.vip/xdh0ef.png" style="zoom:50%;" />

    Each thread needs a kernel stack in the kernel. Here, "in the kernel" means that the data structure is managed and controlled by the kernel of the operating system.
    
*   **User-level threads**

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

In Linux, the registers are stored in the `thread_struct` which is stored in the `task_struct`. The `task_struct` is actually the TCB implementation in Linux.

```c
struct task_struct {
  volatile long state;    /* -1 unrunnable, 0 runnable, >0 stopped */
  void *stack;            /* Points to the start of the stack */
  atomic_t usage;         /* Usage count, see below. */
  unsigned int flags;     /* Per process flags */
  int prio, static_prio, normal_prio;  /* Process priority */
  struct list_head tasks; /* List of tasks with the same real parent. */
  struct mm_struct *mm, *active_mm;    /* Memory management information */
  
  // ...
  struct thread_struct thread;
  // ...
}
```

```c
struct thread_struct {
    struct cpu_context cpu_context; /* CPU context. */
    /* ... other fields ... */
};

struct cpu_context {
    long sp; /* Stack pointer */
    long pc; /* Program counter */
    /* ... other registers ... */
};
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

# Mutual Exclusion

**Atomic Operations**: Either complete or not.

Many instructions are not atomic: double-precision floating point store often not atomic. 

**Synchronization**: using atomic operations to ensure cooperation between threads.

**Mutual Exclusion**: ensuring that only one thread does a particular thing at a time.

**Race condition**: Two threads attempting to **access same data** simultaneously with one of them performing a write.

## Semaphore

Semaphores are a kind of generalized lock. They are the main synchronization primitive used in original UNIX.

**Definition**: a Semaphore has a non-negative integer value and supports the following two operations:

* `Down()` or `P()`: an atomic operation that waits for semaphore to become positive, then decrements it by 1
* `Up()` or `V()`: an atomic operation that increments the semaphore by 1, waking up a waiting P, if any.

### Two Uses of Semaphores

* **Mutual Exclusion** : also called "Binary Semaphore" or "mutex". Can be used for mutual exclusion, just like a lock.

* **Scheduling Constraints** (initial value = 0)

  Allow thread 1 to wait for a signal from thread 2

  Thread 2 schedules thread 1 when a given event occurs

  Example: suppose you had to implement ThreadJoin which must wait for thread to terminate:

  ```pseudocode
  Initial value of semaphore = 0
  
  ThreadJoin {
  	semaP(&mysem);
  }
  
  ThreadFinish {
  	semaV(&mysem);
  }
  ```

Use semaphores for bounded buffer:

* Consumer must wait for producer to fill buffers, if none full (scheduling constraint)

* Producer must wait for consumer to empty buffers, if all full (scheduling constraint)

* Only one thread can manipulate buffer queue at a time (mutual exclusion)

```c
Semaphore fullSlots = 0; 	// Initially, no coke
Semaphore emptySlots = bufSize;				// Initially, num empty slots
Semaphore mutex = 1;	// No one using machine

Producer(item) {
  semaP(&emptySlots);	// Wait until space
  semaP(&mutex);	// Wait until machine free
  Enqueue(item);	
  semaV(&mutex);	
  semaV(&fullSlots);	// Tell consumers there is more coke
}

Consumer() {	
  semaP(&fullSlots);	// Check if there’s a coke	
  semaP(&mutex);	// Wait until machine free	
  item = Dequeue();	
  semaV(&mutex);	
  semaV(&emptySlots);	// tell producer need more
  return item;
}
```

## Locks

Lock prevents someone from doing something. Threads should wait if a region is locked and should sleep if waiting for a long time.

**Difference between locks and semaphores:**

1. **Count:** A lock (mutex) is binary; it is either locked or unlocked. On the other hand, a semaphore maintains a count and allows more than one thread to access a resource, up to a limit specified by the count.
2. **Ownership:** A lock must be released by the thread that acquired it, while a semaphore can be released by any thread.
3. **Use cases:** Locks are typically used when a piece of code or resource should only be accessed by one thread at a time (critical section). Semaphores are used when a number of identical resources are available, or to control access to a pool of resources.

### Implementation of locks

The key idea is to maintiain a lock variable and impose mutual exclusion only during operations on that variable.

```pseudocode
Acquire() {
	disable interrupts;
	if (value == BUSY) {
		wait.enqueue(thread);
		sleep();
	} else {
		value = BUSY;
	}
	enable interrupts;
}
```

**Note**: we need to disable interrupts in the implementation of acquire. Disabling interrupts here won't cause serious problems because the critical region is rather short.

Acquire a lock needs to be in the kernel mode.

### Atomic Instructions

Problems with lock implementation:

* Can't give lock implementation to users
* Doesn't work well on multiprocessor

Alternative: atomic instruction sequences

These instructions read a value and write a new value atomically

**Hardware** is responsible for implementing this correctly. It is effective on both uniprocessors and multiprocessors.

![Screenshot 2023-06-16 at 8.01.11 PM](https://p.ipic.vip/0arlai.png)

 An atomic add to linked-list function:

```pseudocode
addToQueue(&object) {
  do {
    ld r1, M[root]
    st r1, M[object]
  } until (compare&swap(&root, r1, object));
}
```

<img src="https://p.ipic.vip/kyctue.png" alt="Screenshot 2023-06-13 at 2.48.10 AM" style="zoom:50%;" />

`M[root]` stores the address of the first node. `M[object]` stores the address of its successor.

**Implementing Locks with test&set** We can implement a lock without needing to enter kernel mode.

```c
int mylock = 0;

acquire(int *thelock) {
  while(test&set(thelock));
}

release(int *thelock) {
  *thelock = 0;
}
```

Even though this is busy waiting, we don't need to enter kernel mode and it works well with multiprocessor.

**Negatives**: 

* This is very inefficient as thread will consume cycles waiting. 
* Waiting thread may take cycles away from thread holding lock (no one wins). 
* **Priority Inversion**: If busy-waiting thread has higher priority than thread holding lock.
* For semaphores and monitors, waiting thread may wait for an arbitrary long time.

* **Cache coherency overhead**: In a multiprocessor system, the flag variable is likely to be stored in the cache of each processor. If one processor changes the flag, all other caches must invalidate their copies of the flag (Even if the origin value is 1 and we write 1 to the variable). This constant invalidation and refreshing of cache entries leads to a significant overhead, further hampering the system's performance.

**A improvement for the ping-pong issue**: test&test&set

```c
acquire(int *thelock) {
  do {
    while(*thelock);
  } while(test&set(thelock));
}

release(int *thelock) {
  *thelock = 0;
}
```

* We wait until the lock might be free (only reading (compared with the above implementation that everytime a test&set is called, the cache has to be refreshed), the lock variable stays in cache)
* We try to grab the lock with test&set
* We repeat if we fail to actually get the lock

We can **nearly** build test&set locks without busy-waiting

```c
int guard = 0;
int mylock = FREE;

acquire(int *thelock) {
  while(test&set(guard));
  if (*thelock == BUSY) {
    put thread on wait queue;
    go to sleep and set guard = 0
    // guard == 0 on wakeup!
  } else {
    *thelock = BUSY;
    guard = 0;
  }
}

release(int *thelock) {
  while(test&set(guard));
  if anyone on wait queue {
    take thread off wait queue
    place on ready queue;
  } else {
    *thelock = FREE;
  }
  guard = 0;
}
```

* `while(testandset(guard))` is much shorter than `while(test&set(thelock))` cause the guard only indicates the critical area of the acquire and release.
* The use of `guard` implies that we don't need to enter the kernel mode and disable all interrupts.

> Linux futex: Fast Userspace Mutex
>
> ```c
> #include <linux/futex.h>
> #include <sys/time.h>
> 
> int futex(int *uaddr, int futex_op, int val, const struct timespec *timeout);
> ```
>
> `uaddr` points to a 32-bit value in user space 
>
> `futex_op`
>
> * `FUTEX_WAIT` – if val == *uaddr then sleep till FUTEX_WAKE
>    **Atomic** check that condition still holds after we disable interrupts (in kernel!)
> * `FUTEX_WAKE` – wake up at most `val` waiting threads
> * `FUTEX_FD`, `FUTEX_WAKE_OP`, `FUTEX_CMP_REQUEUE`: More interesting operations! 
> * `timeout` - `ptr` to a *timespec* structure that specifies a timeout for the op
>
> Interface to the kernel sleep() functionality. Let the thread put themselves to sleep conditionally.

**T&S and futex**

```c
int mylock = 0;

acquire(int *thelock) {
  while(test&set(thelock)) {
    futex(thelock, FUTEX_WAIT, 1);
  }
}

release(int *thelock) {
  thelock = 0;
  futex(&thelock, FUTEX_WAKE, 1);
}
```

**`acquire(int *thelock)`** This function is used to acquire the lock. The argument is a pointer to the lock variable.

- `while(test&set(thelock))` is a loop that continuously attempts to acquire the lock. The `test&set` function is an atomic operation that tests the lock and sets it in a single, uninterruptible step. If the lock is available (i.e., its value is `0`), `test&set` sets it to a non-zero value and returns `0`, causing the while loop to exit and the calling thread to acquire the lock. If the lock is already acquired, `test&set` returns a non-zero value and the while loop continues.
- `futex(thelock, FUTEX_WAIT, 1)` is called when the lock is not available. This puts the calling thread to sleep until the lock becomes available. The `FUTEX_WAIT` operation suspends the thread if the current value of the futex word (i.e., the lock variable) is `1` (the third argument to `futex`).

**`release(int *thelock)`** This function is used to release the lock. The argument is a pointer to the lock variable.

- `thelock = 0;` releases the lock by setting its value to `0`.
- `futex(&thelock, FUTEX_WAKE, 1);` wakes up one of the threads waiting on the lock, if any. The `FUTEX_WAKE` operation wakes up a number of threads waiting on the futex word (i.e., the lock variable). The third argument to `futex` specifies the maximum number of threads to wake up, which in this case is `1`.

There is no busy waiting here. The acquire procedure simply sleeps until being waken up by the release. But we still have to tap into the kernel to release the lock even if there is no one who is acquring the lock.

So we provide a second implementation to be syscall-free in the uncontended case:

```c
acquire(int *thelock, bool *maybe) {
	while (test&set(thelock)) {
		// Sleep, since lock busy!
		*maybe = true;
		futex(thelock, FUTEX_WAIT, 1);

		// Make sure other sleepers not stuck
		*maybe = true;
	}
}

release(int*thelock, bool *maybe) {
  value = 0;
	if (*maybe) {
		*maybe = false;
		// Try to wake up someone
		futex(&value, FUTEX_WAKE, 1);
	}
}
```

A more elegant implementation:

```c
typedef enum { UNLOCKED,LOCKED,CONTESTED } Lock;
Lock mylock = UNLOCKED; // Interface: acquire(&mylock);
                        //            release(&mylock);

acquire(Lock *thelock) {
	// If unlocked, grab lock!
	if (compare&swap(thelock,UNLOCKED,LOCKED))
		return;

	// Keep trying to grab lock, sleep in futex
	while (swap(mylock,CONTESTED) != UNLOCKED))
		// Sleep unless someone releases hear!
		futex(thelock, FUTEX_WAIT, CONTESTED);
}

release(Lock *thelock) {
	// If someone sleeping, 
  if (swap(thelock,UNLOCKED) == CONTESTED)
		futex(thelock,FUTEX_WAKE,1);
}
```

## Monitors

**Monitor**: a lock and zero or more condition variables for managing concurrent access to shared data.

**Condition Variable**: a queue of variables waiting for something inside a critical section. The key idea of condition variable is that we allow sleeping inside critical section 

**Operations:** 

* `Wait(&lock)`: Atomically release lock and go to sleep
* `Signal()`: Wake up one waiter, if any
* `Broadcast()`: Wake up all waiters

```c
lock buf_lock;	// Initially unlocked
condition buf_CV;	// Initially empty	queue queue;

Producer(item) {		
  acquire(&buf_lock);	// Get Lock
  enqueue(&queue,item);	// Add item
  cond_signal(&buf_CV);	// Signal any waiters
  release(&buf_lock);	// Release Lock
}

Consumer() {
	acquire(&buf_lock);	// Get Lock
	while (isEmpty(&queue)) {
    cond_wait(&buf_CV, &buf_lock); // If empty, sleep
	}
	item = dequeue(&queue);	// Get next item
	release(&buf_lock);	// Release Lock
	return(item);
}
```

### Mesa vs. Hoare Monitors

The above example is the so-called Mesa monitors. We need to check condition again after wait. By the time the waiter gets scheduled, condition may be false again. So, we just check again with the "while" loop.

Apart from this, in the mesa monitor implementation, the waiting thread is put on the ready queue when the producer is in its critical region.

In Hoare monitors, however, we don't check the condition once again and the CPU is immediately handed over to consumer on emitting the signal.

### Readers/Writers

```c
Reader() {
  // First check self into system
  acquire(&lock);
	while ((AW + WW) > 0) {	// Is it safe to read?		
  	WR++;	// No. Writers exist
  	cond_wait(&okToRead,&lock);// Sleep on cond var
    WR--;	// No longer waiting	
  }
  AR++;		// Now we are active!
  release(&lock); 
  // the lock is used to check the conditions
  // we release the lock because another reader may come along
	
  // Perform actual read-only access
  AccessDatabase(ReadOnly);
	
  // Now, check out of system
  acquire(&lock);
  AR--;		// No longer active
  if (AR == 0 && WW > 0) // No other active readers		
    cond_signal(&okToWrite); // Wake up one writer
  release(&lock);
}


Writer() {
  // First check self into system
  acquire(&lock);
  while ((AW + AR) > 0) {	// Is it safe to write?
    WW++;	// No. Active users exist
    cond_wait(&okToWrite,&lock);	// Sleep on cond var
    WW--;	// No longer waiting
  }
  AW++;		// Now we are active!
  release(&lock);
  
  // Perform actual read/write access
  AccessDatabase(ReadWrite);
  
  // Now, check out of system
  acquire(&lock);
  AW--;		// No longer active
  if (WW > 0){	// Give priority to writers
    cond_signal(&okToWrite);// Wake up one writer
  } else if (WR > 0) {	// Otherwise, wake reader
    cond_broadcast(&okToRead); // Wake all readers
  }
  release(&lock);
}
```

**State variables** (Protected by a lock called “lock”):

* int AR: Number of active readers; initially = 0

* int WR: Number of waiting readers; initially = 0

* int AW: Number of active writers; initially = 0

* int WW: Number of waiting writers; initially = 0

* Condition okToRead = NIL

* Condition okToWrite = NIL

**C-Language Support for Synchronization**

We should make sure we know all code paths out of a critical section.

```c
int Rtn() {
  acquire(&lock);
  …
  if (exception) {
    release(&lock);
    return errReturnCode;
  }
  …
  release(&lock);
  return OK;
}
```

We should watch out for `setjmp`/`longjmp`. In the example below, if E calls a longjmp to jump to B and in the meantime C holds a lock, then problems may occur.

<img src="https://p.ipic.vip/aekeih.png" alt="Screenshot 2023-06-17 at 6.35.50 PM" style="zoom:50%;" />

Languages that support exceptions are problematic because it's easy to make a non-local exit without releasing lock. In C++, for example, we should catch all exceptions in the critical region and drop the lock.

```c++
void Rtn() {
  lock.acquire();
  try {
    …
    DoFoo();
    …
  } catch (…) {	
    // catch exception
    lock.release();	
    // release lock
    throw; 	
    // re-throw the exception
  }
  lock.release();
}

void DoFoo() {
  …
  if (exception) throw errException;
  …
}
```

Lock guard is a good solution:

```c++
std::mutex mtx;

void print_block(int n, char c) {
    std::lock_guard<std::mutex> guard(mtx);
    for(int i=0; i<n; ++i) { std::cout << c; }
    std::cout << '\n';
}
```

In Java, each object has ann associated lock:

* Lock is acquired on entry and released on exit from a synchronized method

* Lock is properly released if exception occurs inside a synchronized method

```Java
class Account {
  private int balance;
  
  // object constructor
  public Account (int initialBalance) {
    balance = initialBalance;
  }
  
  public synchronized int getBalance() {
    return balance;
  }
  
  public synchronized void deposit(int amount) {
    balance += amount;
  }
}
```

Along with a lock, every object has a single condition variable associated with it:

* To wait inside a synchronized method:

  `void wait();`

  `void wait(long timeout);`

* To signal while in a synchronized method:
* `void notify();`
* `void notifyAll();`

