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

# Deadlocks

## Requirements for deadlock occurrence

Four requirements for occurrence of Deadlock:

* Mutual exclusion

  Only one thread at a time can use a resource.

* Hold and wait

  Thread holding at least one resource is waiting to acquire additional resources held by other threads

* No preemption

  Resources are released only voluntarily by the thread holding the resource, after thread is finished with it

* Circular wait

 Having a circle doesn't mean that a dead lock exists. 

<img src="https://p.ipic.vip/jalu9q.png" alt="Screenshot 2023-06-23 at 1.20.00 AM" style="zoom:50%;" />

## Detection Algorithm

```pseudocode
[Avail] = [FreeResources]
Add all nodes to UNFINISHED 	
do {
			done = true
      Foreach node in UNFINISHED {
      	if ([Requestnode] <= [Avail]) {
      		remove node from UNFINISHED
        	[Avail] = [Avail] + [Allocnode]
        	done = false
       }
      }
} until(done)
```

Nodes left in `UNFINISHED` are deadlocked.

## Dealing With Deadlock

Four different approaches:

1. Deadlock prevention: write your code in a way that it isn’t prone to deadlock

2. Deadlock recovery: let deadlock happen, and then figure out how to recover from it

3. Deadlock avoidance: dynamically delay resource requests so deadlock doesn’t happen

4. Deadlock denial: ignore the possibility of deadlock

For modern operating systems, we make sure that the system isn't involved in any deadlock. We must ignnore deadlock in applications.

### Deadlock Prevention

* Infinite resources

  Include enough resources so that no one ever runs out of resources.

  Doesn’t have to be infinite, just large

  Give illusion of infinite resources (e.g. virtual memory)

  Examples:

  * Bay bridge with 12,000 lanes. Never wait!
  * Infinite disk space (not realistic yet?)

* No Sharing of resources (totally independent threads)

  Not very realistic

* Don’t allow waiting 

  How the phone company avoids deadlock

  * Call Mom in Toledo, works way through phone network, but if blocked get busy signal. 

  Technique used in Ethernet/some multiprocessor nets

  * Everyone speaks at once. On collision, back off and retry

  Inefficient, since have to keep retrying

  * Consider: driving to San Francisco; when hit traffic jam, suddenly you’re transported back home and told to retry!

* Make all threads request everything they’ll need at the beginning.

  Problem: Predicting future is hard, tend to over-estimate resources

  Example:

  * If need 2 chopsticks, request both at same time
  * Don’t leave home until we know no one is using any intersection between here and where you want to go; only one car on the Bay Bridge at a time

* Force all threads to request resources in a particular order preventing any cyclic use of resources

  Thus, preventing deadlock

  Example `(x.Acquire(), y.Acquire(), z.Acquire(),…)`

  Make tasks request disk, then memory, then…

  Keep from deadlock on freeways around SF by requiring everyone to go clockwise

### Deadlock Recovery

Roll back the actions of deadlocked threads

### Deadlock Avoidance

**Safe state**: System can delay resource acquisition to prevent deadlock

**Unsafe state**: No deadlock yet…But threads can request resources in a pattern that **unavoidably** leads to deadlock

**Deadlocked state**: There exists a deadlock in the system. Also considered “unsafe”

The idea is that when a thread requests a resource, OS checks if it would result in an unsafe state.

## Banker's Algorithm for Avoiding Deadlock

**Basic Idea**: we state the maximum resource needs in advance. We allow particular thread to proceed if: available resources $-$ #requested $\geq$ max

**Banker's Algorithm** (less conservative than the basic idea): allocate resources dynamically. We evaluate each request and grant if some ordering of threads is still deadlock free afterward.

The implementation of the Banker's algorithm lies in pretending each request is granted then run deadlock detection algorithm above. Each thread will have a maximum resource requirement ([Max]), and a currently allocated resource amount ([Alloc]). [Avail] is the amount of resources currently available in the system. We substitute the `([Request] <= [Avail])` with `([Max]-[Alloc] <= [Avail])`

We Keep system in a “SAFE” state: there exists a sequence ${T_1, T_2, … T_n}$ with $T_1$ requesting all remaining resources, finishing, then $T_2$ requesting all remaining resources, etc..

