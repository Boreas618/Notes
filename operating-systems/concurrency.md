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

- `thread`: A pointer to a `pthread_t` variable that will be filled in by `pthread_create()` with a unique thread ID for the new thread. This ID can be used later to refer to the thread.
- `attr`: A pointer to a `pthread_attr_t` structure that specifies attributes for the new thread, such as its stack size or scheduling policy. If NULL is passed, default attributes will be used.
- `start_routine`: A pointer to the function that the new thread will execute. This function should take a single argument of type `void*` and return a `void*`.
- `arg`: An argument that will be passed to the `start_routine` function when it is called by the new thread.

```c
int pthread_yield(void);
```

Give up the processor to let some other threads run.

```c
int pthread_join(pthread_t thread, void **retval);
```

- `thread`: The thread ID of the thread to join. This should be a thread ID previously returned by `pthread_create()`.
- `retval`: A pointer to a variable that will be filled in with the exit status of the joined thread. If the exit status is not needed, NULL can be passed instead.

When `pthread_join()` is called with a thread ID, the calling thread blocks until the specified thread terminates. Once the thread has terminated, `pthread_join()` returns and the exit status of the thread is stored in the location pointed to by `retval`. The exit status is a `void*` pointer that was returned by the thread function passed to `pthread_create()`, or NULL if the thread did not explicitly return a value.

```c
void pthread_exit(void *retval);
```

When `pthread_exit()` is called, the calling thread is terminated and its resources are freed. The exit status of the thread is returned in the `retval` parameter, which is a `void*` pointer. If no exit status is needed, `pthread_exit(NULL)` can be called instead.

