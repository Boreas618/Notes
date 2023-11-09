# Deadlocks

<img src="https://p.ipic.vip/mhh6l6.png" alt="Screenshot 2023-10-29 at 10.43.39 PM" style="zoom: 33%;" />

# Requirements for Deadlock

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

# Dealing With Deadlock

* **The Ostrich algorithm**: Just ignore the problem (Adopted by most operating systems)
* **Detection and recovery**: Let dead lock occur,detect them, and take action
* **Dynamic avoidance**: by carefully resource allocation
* **Prevention**: by structurally negating one of the four required conditions

For modern operating systems, we make sure that the system isn't involved in any deadlock. We must ignnore deadlock in applications.

## Deadlock Prevention

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

## Deadlock Detection

Given Matrices:

- **Request Matrix Q**
- **Allocation Matrix A**
- **Resource Vector**
- **Available Vector**

1. **Mark each process** that has a row in the Allocation Matrix of all zeros.
2. **Initialize a temporary vector W** to equal the Available vector.
3. **Find an index i** such that process i is currently unmarked and the ith row of Q is less than or equal to W. If no such row is found, terminate the algorithm.
4. If such a row is found, **mark process i** and add the corresponding row of the allocation matrix to W. Return to Step 3.

The core of this algorithm is to determine whether, given the current situation of requests, we can find a sequence of allocations to make.

**Mark**s mean granting the requests. We perform marks on the request matrix.

There are several candidate strategies for determining when to detect deadlocks.：

* To check every time a resource request is made (certain to detect deadlocks as early as possible, but potentially expensive in terms of CPU time)
* To check every $k$ minutes
* To check only when the CPU utilization has below some threshold (if enough processes are deadlocked, there will be few runnable processes)

## Deadlock Recovery

Roll back the actions of deadlocked threads

## Deadlock Avoidance

Two approaches to avoid deadlock:

* Process Initailization Denial
* Resource Allocation Denial

### Process Initialization Denial

A process $\text{P}_\text{{n+1}}$ can start only if
$$
R_i \geq C_{n+1\space i} + \sum_{k=1}^{n} C_{ki}
$$

### Resource Alloaction Denial

**Safe state** is where there is **at least one** sequence that does not result in deadlock.

**Unsafe state** is a state that is not safe.

We can make sure that the system is always in a safe state to avoid deadlock. For a resource allocation request, we assume that the resource is granted and examine the state of the system after allocation. If the new state is unsafe, then we deny the request.

### Banker's Algorithm for Avoiding Deadlock

**Basic Idea**: we state the maximum resource needs in advance. We allow particular thread to proceed if: available resources $-$ #requested $\geq$ max

**Banker's Algorithm** (less conservative than the basic idea): allocate resources dynamically. We evaluate each request and grant if some ordering of threads is still deadlock free afterward.

The implementation of the Banker's algorithm lies in pretending each request is granted then run deadlock detection algorithm above. Each thread will have a maximum resource requirement ([Max]), and a currently allocated resource amount ([Alloc]). [Avail] is the amount of resources currently available in the system. We substitute the `([Request] <= [Avail])` with `([Max]-[Alloc] <= [Avail])`

We Keep system in a “SAFE” state: there exists a sequence ${T_1, T_2, … T_n}$ with $T_1$ requesting all remaining resources, finishing, then $T_2$ requesting all remaining resources, etc..

