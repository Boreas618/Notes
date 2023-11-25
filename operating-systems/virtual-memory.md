# Preventing Page Fault

**Compulsory Misses**: Pages that have never been paged into memory before.

Prefetching: loading them into memory before needed. Need to predict future somehow!

**Capacity Misses**: Not enough memory. Must somehow increase available memory size.

* **Option 1**: Increase amount of DRAM (not quick fix)
* **Option 2**: If multiple processes in memory: adjust percentage of memory allocated to each one!

**Conflict Misses**: Technically, conflict misses don’t exist in virtual memory, since it is a "fully-associative" cache

**Policy Misses**: Caused when pages were in memory, but kicked out prematurely because of the replacement policy

# Replacement Policies

We introduce some classic replacement policies here.

## FIFO (First In, First Out)
- Replace the oldest page.
- Aims for fairness by allowing each page to reside in memory for an equivalent duration.
- **Disadvantage:** Can replace heavily accessed pages over seldom-used ones.

## RANDOM
- Randomly select a page for replacement.
- Commonly used for Translation Lookaside Buffers (TLB) due to its hardware simplicity.
- **Disadvantage:** Unpredictability can challenge real-time performance guarantees.

## MIN (Minimum)
- Replace the page that will remain unused for the longest upcoming duration.
- Theoretically optimal, but forecasting future access is infeasible.
- **Note:** We can use past access patterns for future predictors. This is basically what LRU done.

## LRU (Least Recently Used)
- Replace the page that has been unused for the longest duration.
- Based on the principle of locality: infrequently accessed items will likely remain so.
- LRU is considered a good practical approximation to MIN.

> **For MIN and LRU**, adding memory generally decreases the miss rate. But for FIFO, this reduction isn't guaranteed, leading to Bélády’s anomaly：
>
> <img src="https://p.ipic.vip/sjopmr.png" alt="Screenshot 2023-06-30 at 3.15.53 AM" style="zoom:50%;" />

## Approximating LRU: Clock Algorithm

**Clock Algorithm**: 
- Envision physical pages in a circular arrangement with a clock hand.
- Approximates LRU.
- Replaces an old page but not necessarily the oldest.

There is a hardware-supported "use" bit for each physical page (termed “accessed” in some architectures). The hardware updates this "use" bit upon each reference. A cleared (not set) "use" bit indicates the page hasn't been referenced for a while.

The clock hand progresses (on a page fault) and checks the use bit:

- 1: Recently used; reset it and proceed.
- 0: Designate as a replacement candidate.

The algorithm's design guarantees eventual page replacement, unless all pages are frequently accessed.

**Partitioning of Pages: Young and Old** 
Pages are informally classified based on their "use" bit status:
- Set (1): "Young" or recently accessed.
- Cleared (0): "Old" or infrequently accessed.

## N-th Chance Version of Clock Algorithm

This version enhances the standard Clock Algorithm by tracking the number of "sweeps" (clock hand rotations) a page remains unused.
- The counter is reset when a page's "use" bit is 1.
- If the "use" bit is 0, the counter increments, and replacement occurs when it hits N.
- A higher N brings the algorithm closer to LRU. However, finding a replaceable page might necessitate numerous clock cycles.
- The "dirty" status of a page (indicating modifications) is also considered. This ensures data consistency by writing back changes before potential replacements.

## Second-Chance List Algorithm

This blends the simplicity of FIFO with the efficacy of LRU.
- Memory is bifurcated into the **Active List** (containing pages accessible at full speed) and the **Second Chance (SC) List** (with pages marked as invalid).
- On accessing a page, the algorithm:
  - Checks the Active List. If absent, moves the oldest Active List page to the SC List's start, marking it invalid.
  - If found in the SC List, reinstates it to the Active List's start, marking it valid.
  - If absent in both, brings the new page into the Active List, displacing the LRU page from the SC List.

Depending on the SC List's allocation, the algorithm can behave like FIFO, LRU, or a hybrid.

# Allocation of Page Frames

Possible Replacement Scopes:

* Global replacement – process selects replacement frame from set of all frames; one process can take a frame from another
* Local replacement – each process selects from only its own set of allocated frames

## Equal allocation (Fixed Scheme):

Every process gets same amount of memory

Example: 100 frames, 5 processes : process gets 20 frames

## Proportional allocation (Fixed Scheme)

Allocate according to the size of process

Computation proceeds as follows:

$s_i$ = size of process $p_i$ and $S=\sum s_i$ 

$m$ = total number of physical frames in the system

$a_i$ = (allocation for $p_i$) =$\frac{s_i}{S} \times m$

## Priority Allocation

Proportional scheme using priorities rather than size

* Same type of computation as previous scheme

Possible behavior: If process $p_i$ generates a page fault, select for replacement a frame from a process with lower priority number.

We want find a balance between page-fault rate and number of frames. We dynamically adjust the number of frames a process is allocated.

<img src="https://p.ipic.vip/sxoaml.png" alt="image-20230630214738969" style="zoom:50%;" />

**Thrashing**: if a process does not have "enough" pages, the page-fault rate is very high. This leads to low CPU utilization and the operating system spends most of its time swapping to disk. If there are more threads, more memory is needed and thus we need to spend a lot time on paging and swapping.

![Screenshot 2023-06-30 at 9.52.41 PM](https://p.ipic.vip/6dabts.png)

**Working-Set Model**: $\Delta$is the working-set window º= fixed number of page references (Example: 10,000 instructions)

$WS_i$ (working set of Process $P_i$) = total set of pages referenced in the most recent $D$ (varies in time)

* If $D$ is too small, we will not encompass entire locality
* If $D$ is too large, we will encompass several localities
* If $D$ = $\infin$, we will encompass entire program

$D$ = $\sum|WS_i|$ total demand frames 

If D > m $\rightarrow$Thrashing:

* Policy: if D > m, then suspend/swap out processes
* This can improve overall system behavior by a lot!

For compulsory misses:

**Clustering:** This is a strategy to optimize memory management and reduce the impact of page faults. The idea behind clustering is that when a page fault occurs, instead of just bringing the faulting page into memory, the operating system also brings in a set of pages around the faulting page. This is based on the principle of locality, which states that if a process accesses a particular page, it's likely to access nearby pages in the near future. Clustering takes advantage of the fact that disk reads are more efficient when reading sequential pages, as it reduces the need to move the disk read/write head.

**Working Set Tracking:** This is a strategy used to manage memory more efficiently. The working set of a process is the set of pages that the process is currently using or is likely to use in the near future. By tracking the working set of a process, the operating system can make better decisions about which pages to keep in memory and which pages to swap out. When a process is swapped out and then later swapped back in, if the operating system has been tracking the working set of the process, it can bring the entire working set back into memory. This can significantly reduce the number of page faults after the process is swapped back in, as the pages the process is likely to access are already in memory.

# Linux Memory

## Memory Zones

The Linux kernel divides physical memory into zones, each of which represents a class of memory pages that can be used for different purposes:

- **ZONE_DMA**: This is for memory that is accessible by direct memory access (DMA). DMA is a feature of computer systems that allows certain hardware subsystems to access main system memory independently of the central processing unit. ZONE_DMA is typically for memory under 16MB, which is DMAable on the ISA bus.
- **ZONE_NORMAL**: This zone typically includes memory from 16MB to 896MB. It is called "normal" because it's the zone where the kernel's code and data structures live, and most of the system's operations happen.
- **ZONE_HIGHMEM**: This zone includes all physical memory above approximately 896MB. The memory in this zone is not permanently mapped into the kernel's address space. Instead, it is temporarily mapped when needed.

Each of these zones has one freelist and two least recently used (LRU) lists (Active and Inactive). The freelist is used to track free memory pages, while the LRU lists are used to manage page caching

**Memory Allocation Types**

Linux supports many different types of memory allocation, including SLAB allocators, per-page allocators, and mapped/unmapped memory.

- **SLAB allocators**: SLAB allocation is a memory management mechanism within the Linux kernel which helps to efficiently manage the memory allocation of kernel objects. The concept behind SLAB allocation is to create caches for commonly used objects to minimize the overhead of object creation and destruction. For example, whenever a network packet arrives, the kernel needs to create a new object to handle this packet, and when it's done processing, it needs to destroy this object. By using SLAB allocation, these objects can be cached for later use, which can significantly speed up these operations.
- **Per-page allocators**: This is a type of memory allocation where memory is allocated one page at a time. This is often used for larger allocations, as it can be more efficient than allocating memory in smaller chunks.
- **Mapped/Unmapped memory**: Mapped memory is memory that has been mapped into the address space of a process. Unmapped memory, on the other hand, has not been mapped into the address space of any process. Mapped memory can be backed by a file, while unmapped memory cannot.

**Types of Allocated Memory**

The allocated memory can be of different types such as:

- **Anonymous memory**: This is memory that is not backed by a file. It is typically used for a process's heap and stack.
- **Mapped memory**: This is memory that is backed by a file. It is used for things like memory-mapped files and shared memory.

**Allocation Priorities and Blocking**

The Linux kernel also has a notion of allocation priorities. When a process requests memory, it can specify a priority for the allocation. The kernel will then try to satisfy the allocation request based on the priority, the amount of available memory, and other factors.

In addition to allocation priorities, the kernel also has mechanisms to determine whether or not blocking is allowed during a memory allocation. If blocking is allowed, then the process requesting memory can be put to sleep (i.e., blocked) until enough memory is available to satisfy the request. If blocking is not allowed, then the kernel must either immediately satisfy the request or immediately fail it.

![Screenshot 2023-06-30 at 11.53.58 PM](https://p.ipic.vip/klj3d1.png)

One exception to this user/kernel separation is the special Virtual Dynamically linked Shared Objects (VDSO) facility that Linux offers. The purpose of VDSO is to map certain parts of kernel code into user space, providing quicker access to some system call mechanisms. VDSO is a shared library that the kernel automatically maps into the address space of all user-space applications. It contains code that runs in user space but can accomplish the same work as some system calls. A common example is the `gettimeofday()` system call which provides the current system time. By offering a user-space VDSO version of `gettimeofday()`, the system can avoid a context switch to kernel mode, improving performance.

In the Linux kernel, every physical page of memory is described by a "page" structure. This provides metadata about the physical memory page, including status information like whether the page is currently being used, and if so, by which process. These "page" structures are collected together in lower physical memory and can be accessed in the kernel's virtual space.

The Linux kernel also employs the LRU (Least Recently Used) algorithm for managing memory, particularly for its page replacement strategy. It organizes physical pages into several "LRU" lists, which help the system decide which pages should be evicted when the memory is full.

In a 32-bit virtual memory architecture, the way the kernel maps physical memory depends on the amount of physical memory available. When physical memory is less than 896MB, all physical memory is mapped at the memory location 0xC0000000. However, if the physical memory is 896MB or more, the kernel does not map all physical memory all the time. Portions of the memory can be temporarily mapped with addresses greater than 0xCC000000.

In a 64-bit virtual memory architecture, things are different due to the larger address space available. All physical memory can be mapped above the memory address 0xFFFF800000000000. This makes memory management more straightforward and efficient, as there's a direct correlation between virtual and physical memory.

The memory management strategy adopted by an operating system like Linux has profound implications for the system's performance and efficiency, and is a critical part of the kernel's functionality. Understanding this can provide useful insights into the inner workings of operating systems.

## Post Meltdown Memory Map

**Meltdown flaw**: 

```c++
// Set up side channel (array flushed from cache)
uchar array[256 * 4096];
flush(array);	// Make sure array out of cache

try { 	 // … catch and ignore SIGSEGV (illegal access)
  uchar result = *(uchar *)kernel_address;	// Try access
  uchar dummy = array[result * 4096];	// leak info
} catch(){;} // Could use signal() and setjmp/longjmp

// scan through 256 array slots to determine which is fast
```

Patch: different page tables for user and kernel. But if without PCID tag in TLB, we need to flush TLB *twice* on each syscall (800% overhead!)

Fix: better hardware without timing side-channels