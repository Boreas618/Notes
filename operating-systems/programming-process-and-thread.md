# Programming: Process and Thread

## Process Control

Unix provides a number of system calls for manipulating processes from C programs. This section describes the important functions and gives examples of how they are used.

### Obtaining Process IDs

`getpid` and `getppid`

```c
#include <sys/types.h>
#include <unistd.h>

pid_t getpid(void);
pid_t getppid(void);

// Returns: PID of either the caller or the parent
```

### Creating and Terminating Processes

From the perspective of programmers, we care the following states:

* **Running**
* **Blocked**: The execution of this process is suspended and will not be scheduled. A process stops as a result of receiving a `SIGSTOP`, `SIGTSP`, `SIGTTIN` or `SIGTTOU` signal, and remains stopped until it receives a `SIGCONT` signal, at which point it becomes running again. It can be further categorized as **uninterruptible sleep** and **interruptible sleep**.
* **Terminated**: A process become terminated for: Receving a signal whose default action is to terminate the process; Returning from the main routine; Calling the `exit` function.

```c
#include <stdlib.h>

void exit(int status);

// This function does not return
```

```c
#include <sys/types.h>
#include <unistd.h>

pid_t fork(void);

// Returns: 0 to child, PID of child to parent, −1 on error
```

### Reaping Child Processes

The child process terminated → It becomes a zombie → reaped by its parent → Cease to exist

**The `init` process is the adopted parent of any orphaned children.** It has a PID of 1. It is created by the kernel in the system start-up, never terminates. If a parent terminates without reaping its children, the `init` (``systemd` or `Upstart` in Linux)process is to reap them.

Even though zombies are not running, they still occupy system memory resources. A process waits for its children to terminate or stop by calling the `waitpid` function.

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

```c
#include <unistd.h>

int pause(void);

// Always returns −1
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

<img src="https://p.ipic.vip/sqoabq.png" alt="Screenshot 2023-12-16 at 9.57.13 PM" style="zoom: 33%;" />

```c
#include <stdlib.h>

char *getenv(const char *name);

// Returns: pointer to name if it exists, NULL if no match

int setenv(const char *name, char *newvalue, int overwrite);

// Returns: 0 on success, -1 on error

void unsetenv(const char *name);

// Returns: nothing
```

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

```c
int pthread_mutex_init(pthread_mutex_t *mutex, const pthread_mutexattr_t *attr);
```

- `mutex`: A pointer to the mutex to initialize.
- `attr`: A pointer to a `pthread_mutexattr_t` structure that specifies attributes for the mutex. If NULL, default attributes are used.

Initializes a mutex object, which may be used to control access to a resource.

```c
int pthread_mutex_lock(pthread_mutex_t *mutex);
```

- `mutex`: A pointer to the mutex to lock.

This function locks the specified mutex. If the mutex is already locked by another thread, the calling thread blocks until the mutex becomes available.

```c
int pthread_mutex_unlock(pthread_mutex_t *mutex);
```

- `mutex`: A pointer to the mutex to unlock.

Unlocks the specified mutex, allowing other threads to acquire the mutex if they are blocked waiting for it.

```c
int pthread_cond_wait(pthread_cond_t *cond, pthread_mutex_t *mutex);
```

- `cond`: A pointer to the condition variable to wait on.
- `mutex`: A pointer to the mutex that must be locked by the calling thread.

Blocks the calling thread on the condition variable specified by `cond`. The thread remains blocked until another thread signals the condition variable.

```c
int pthread_cond_signal(pthread_cond_t *cond);
```

- `cond`: A pointer to the condition variable to signal.

Wakes up at least one thread that is currently blocked on the specified condition variable.

```c
int pthread_attr_init(pthread_attr_t *attr);
```

- `attr`: A pointer to the thread attributes object.

Initializes a thread attributes object with default values.

```c
int pthread_attr_setdetachstate(pthread_attr_t *attr, int detachstate);
```

- `attr`: A pointer to the thread attributes object.
- `detachstate`: The detach state to set, which can be either `PTHREAD_CREATE_DETACHED` or `PTHREAD_CREATE_JOINABLE`.

Sets the detach state of a thread, determining whether the thread is joinable or detached upon creation.
