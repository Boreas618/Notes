# Case Study: Linux Futex (Fast Userspace Mutex)

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
> * `FUTEX_WAIT` – if `val == *uaddr` then sleep till FUTEX_WAKE
>   **Atomic** check that condition still holds after we disable interrupts (in kernel!)
> * `FUTEX_WAKE` – wake up at most `val` waiting threads
> * `FUTEX_FD`, `FUTEX_WAKE_OP`, `FUTEX_CMP_REQUEUE`: More interesting operations! 
> * `timeout` - `ptr` to a *timespec* structure that specifies a timeout for the op

Interface to the kernel `sleep()` functionality. Let the thread put themselves to sleep conditionally.

## T&S and Futex

```c
acquire(int *thelock) {
  while(__atomic_test_and_set(thelock, __ATOMIC_SEQ_CST)) {
    futex(thelock, FUTEX_WAIT, 1);
  }
}

release(int *thelock) {
  thelock = 0;
  futex(thelock, FUTEX_WAKE, 1);
}
```

**`acquire(int *thelock)`** This function is used to acquire the lock. The argument is a pointer to the lock variable.

- `futex(thelock, FUTEX_WAIT, 1)` is called when the lock is not available. This puts the calling thread to sleep until the lock becomes available. The `FUTEX_WAIT` operation suspends the thread if the current value of the futex word (i.e., the lock variable) is `1` (the third argument to `futex`).

**`release(int *thelock)`** This function is used to release the lock. The argument is a pointer to the lock variable.

- `futex(&thelock, FUTEX_WAKE, 1);` wakes up one of the threads waiting on the lock, if any. The `FUTEX_WAKE` operation wakes up a number of threads waiting on the futex word (i.e., the lock variable). The third argument to `futex` specifies the maximum number of threads to wake up, which in this case is `1`.

There is no busy waiting here. The acquire procedure simply sleeps until being waken up by the release. But we still have to tap into the kernel to **release** the lock even if there is no one who is acquring the lock.

So we provide a second implementation to be syscall-free in the uncontended case:

```c
void acquire(int *thelock, bool *maybe) {
    while (__atomic_test_and_set(thelock, __ATOMIC_SEQ_CST)) {
        // Sleep, since lock busy!
        *maybe = true;
        futex(thelock, FUTEX_WAIT, 1);
        // Make sure other sleepers are not stuck
        *maybe = true;
    }
}

void release(int *thelock, bool *maybe) {
    __atomic_clear(thelock, __ATOMIC_SEQ_CST);

    if (*maybe) {
        *maybe = false;
        // Try to wake up someone
        futex(thelock, FUTEX_WAKE, 1);
    }
}
```

A more elegant implementation:

```c
#include <stdint.h>
#include <linux/futex.h>
#include <unistd.h>

void acquire(Lock *thelock) {
    Lock expected = UNLOCKED;
    
    // If unlocked, grab lock!
    if (__atomic_compare_exchange_n(thelock, &expected, LOCKED, 0, __ATOMIC_SEQ_CST, __ATOMIC_SEQ_CST))
        return;

    // Keep trying to grab lock, sleep in futex
    while (__atomic_exchange_n(thelock, CONTESTED, __ATOMIC_SEQ_CST) != UNLOCKED)
        // Sleep unless someone releases here!
        futex((int *)thelock, FUTEX_WAIT, CONTESTED, NULL, NULL, 0);
}

void release(Lock *thelock) {
    // If someone is sleeping, 
    if (__atomic_exchange_n(thelock, UNLOCKED, __ATOMIC_SEQ_CST) == CONTESTED)
        futex((int *)thelock, FUTEX_WAKE, 1, NULL, NULL, 0);
}
```

