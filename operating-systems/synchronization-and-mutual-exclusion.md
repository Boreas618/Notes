# Synchronization and Mutual Exclusion

**Atomic Operations**: Either complete or not.

> Many instructions are not atomic: double-precision floating point store often not atomic. 

**Synchronization**: using atomic operations to ensure cooperation between threads.

**Mutual Exclusion**: ensuring that only one thread does a particular thing at a time.

**Race Condition**: Two threads attempting to **access same data** simultaneously with one of them performing a write.

## Process-level Approaches

The two algorithms that will be introduced in this sections do not rely on programming language and operating system supports. 

### Dekker's Algorithm

1. Process 0 checks the intention of the other process 1.

2. If 1 also wants to enter the critical section:

   0 checks the turn variable. If it's 1's turn, then 0 recalls its desiration and waits (spins) until the turn changes. If it's not 1's turn, then 1 may decide to withdraw its request to enter the critical section, allowing 0 to proceed.

3. Once inside the critical section, the process will eventually exit, and the turn is given to the other process, ensuring fairness.

```c
bool flag[2] = {false, false};
int turn = 1;

void *p0(void *arg) {
  while(ture) {
    flag[0] = true;
    while(flag[1]) {
      if (turn == 1) {
        flag[0] = false;
        while (turn == 1)
          // just spin for a while
        flag[0] = true;
      }
    }
    // critical section
    turn = 1;
    flag[0] = false;
  }
}
```

### Peterson's Algorithm

```c
boolean flag[2] = {false, false};
int turn;

void *p0(void *arg) {
  while(true) {
    flag[0] = true;
    turn = 1;
    while(flag[1] && turn == 1) {
      // just spin for a while
    }
    // critical section
    flag[0] = false;
  }
}
```

## Hardware-level Approaches

Two commly implemented instructions:

* **Test and Set** Instruction
* **Exchange** Instruction

```c
#include <stdio.h>
#include <pthread.h>
#include <stdatomic.h>

atomic_flag bolt = ATOMIC_FLAG_INIT;

void *p(void *arg) {
    while(1) {
      while(atomic_flag_test_and_set(&bolt)); 
    	// Critical section
    	atomic_flag_clear(&bolt);
    }
}
```

We use the `atomic_flag_test_and_set` instruction here to implement a spin lock. The test&set instruction reads the value from a memory location (`bolt` in this case) and sets it to 1 in an atomic manner.

In x86, the test&set operation can be implemented using **XCHG**, which is an atomic instruction that performs an exchange.

```assembly
; EAX contains the value we want to set the lock to (typically 1 to indicate locked)
; [lock_variable] is the memory address of the lock variable

spin_lock:
    XCHG EAX, [lock_variable]  ; Atomically swap the values of EAX and the lock variable
    TEST EAX, EAX              ; Test if the original value of the lock variable (now in EAX) was 0
    JNZ spin_lock              ; If it was not 0, the lock was held, so we loop and try again
```

## Semaphore

Semaphore is a variable that has an integer value, with three operations defined upon it:

* `Init`

* `Down` or `P` or `Wait`: an atomic operation that waits for semaphore to become positive, then decrements it by 1
* `Up` or `V` or `Signal` or `Post`: an atomic operation that increments the semaphore by 1, waking up a waiting P, if any.

> **Binary semaphores** are considered as a special type of semaphore in COMP130110.03. When we refer to semaphores, we actually mean counting semaphores instead of binary semaphores.

**Strong semaphore**: the process that has been blocked the longest is released from the queue first.

**Weak semaphore**: doesn't specify the order in which processes are removed from the queue.

**The encapsulated variable `val` of semaphores**:

* `val >= 0` number of processes that can execute waits without blocking
* `val < 0` the magnitude of `val` is the number of processes blocked in the queue

### Producer/Consumer Problem

Producer/consumer problem with finite buffer can be solved by two semaphores.

```c
sem_t empty; // the "empty" slots
sem_t mutex;
sem_t full; // the "full" slots

void *producer(void *arg) {
    item_type item;
    while (TRUE) {
        item = produce();
        sem_wait(&empty);
        sem_wait(&mutex);
        enqueue(&buffer, item);
        sem_post(&mutex);
        sem_post(&full);
    }
    return NULL;
}

void *consumer(void *arg) {
    item_type item;
    while (TRUE) {
        sem_wait(&full);
        sem_wait(&mutex);
        dequeue(&buffer, &item);
        sem_post(&mutex);
        sem_post(&empty);
        consume(item);
    }
    return NULL;
}
```

> **Solving by [Condition Variables](https://boreas618.github.io/posts/monitor.html)**
>
> ```c
> pthread_cond_t empty = PTHREAD_COND_INITIALIZER;
> pthread_cond_t fill = PTHREAD_COND_INITIALIZER;
> pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
> 
> void *producer(void *arg) {
>   	while (TRUE) {
>   		pthread_mutex_lock(&mutex);
>     		while (count == MAX)
>       		pthread_cond_wait(&empty, &mutex);
>     		put(i);
>     		pthread_cond_signal(&fill);
>     		pthread_mutex_unblock(&mutex);
>   	}
> }
> 
> void *consumer(void *arg) {
>   	while (TRUE) {
>        pthread_mutex_lock(&mutex);
>        while (count == 0)
>          pthread_cond_wait(&fill, &mutex);
>        int tmp = get(i);
>        pthread_cond_signal(&empty);
>        pthread_mutex_unblock(&mutex);
>        print("%d\n", tmp);
>     }
> }
> ```
>
> **Discussions**: 
>
> > 1. **Why we need two condition variables?**
> >
> >    In cases with multiple producers and consumers, using a single condition variable can result in a consumer waking up another consumer, which is not desired.
>
> > 2. **Why we use `while` loops instead of `if`s?**
> >    * The `signal` function only wakes up a thread, putting it in the ready queue. However, it is highly possible that the world has changed between the thread waking up and being executed. Using `while` allows for the ability to double-check the condition.
> >    * Spurious wakeups.

### Barber Shop Problem

- A customer will not enter the shop if it is filled to the capacity with other customers.
- Once inside, the customer takes a seat on the sofa or stands if the sofa is filled.
- When a barber is free, the customer that has been on the sofa the longest is served.
- When a customer's haircut is finished, payment is accepted for one customer at a time.

<img src="https://p.ipic.vip/91ugbs.png" alt="Screenshot 2023-10-28 at 5.07.15 PM" style="zoom:50%;" />

### Readers/Writers Problem

```c
sem_t x；
sem_t wsem;

void *reader(void *arg) {
    while (TRUE) {
        sem_wait(&x);
        readcount++;
        if (readcount == 1)
            sem_wait(&wsem);
        sem_post(&x);
        // Read
        sem_wait(&x);
        readcount--;
        if (readcount == 0)
            sem_post(&wsem);
        sem_post(&x);
    }
    return NULL;
}

void *writer(void *arg) {
    while (TRUE) {
        sem_wait(&wsem); 
        // Write     
        sem_post(&wsem);
    }
    return NULL;
}
```

### Implementation of Semaphores

1. Disabling Interrupts

   Feasible for uniprocessor system. For multiprocessor system, it's not a fancy idea to disable interrupts for all cores.

2. Semaphore Implementation by test&set instruction

   The semaphore now includes a new integer component `s.flag` which controls access to the semaphore.

## Locks

Lock prevents someone from doing something. Threads should wait if a region is locked and should sleep if waiting for a long time.

**Difference between locks and semaphores:**

1. **Count:** A lock (mutex) is binary; it is either locked or unlocked. On the other hand, a semaphore maintains a count and allows more than one thread to access a resource, up to a limit specified by the count.
2. **Ownership:** A lock must be released by the thread that acquired it, while a semaphore can be posted by any thread.
3. **Use cases:** Locks are typically used when a piece of code or resource should only be accessed by one thread at a time (critical section). Semaphores are used when a number of identical resources are available, or to control access to a pool of resources.

### Implementation of locks

**Hardware** is responsible for implementing this correctly. It is effective on both uniprocessors and multiprocessors.

**Implementing Locks with test&set** We can implement a lock without needing to enter kernel mode (disabling interrupts) using test&set.

```c
#include <stdatomic.h>

acquire(int *thelock) {
	while(atomic_flag_test_and_set(thelock)); 
}

release(int *thelock) {
  atomic_flag_clear(thelock);
}
```

Even though this is busy waiting, we don't need to enter kernel mode and it works well with multiprocessor.

**Negatives**: 

* This is very inefficient as thread will consume cycles waiting. 
* Waiting thread may take cycles away from thread holding lock (no one wins). 
* **Priority Inversion**: If busy-waiting thread has higher priority than thread holding lock. It simply waits instead of preempting.
* Waiting thread may wait for an arbitrary long time.

* **Cache coherency overhead**: In a multiprocessor system, the flag variable is likely to be stored in the cache of each processor. If one processor changes the flag, all other caches must invalidate their copies of the flag (Even if the origin value is 1 and we write 1 to the variable). This constant invalidation and refreshing of cache entries leads to a significant overhead, further hampering the system's performance.

**An Improvement for the Ping-pong Issue**: test&test&set

```c
acquire(int *thelock) {
  do {
    while(*thelock);
  } while(atomic_flag_test_and_set(thelock));
}

release(int *thelock) {
  atomic_flag_clear(thelock);
}
```

* We wait until the lock might be free (only reading (compared with the above implementation that everytime a test&set is called, the cache has to be refreshed), the lock variable stays in cache)
* We try to grab the lock with test&set
* We repeat if we fail to actually get the lock
