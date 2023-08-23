# The Memory Hierarchy

Program with good locality tend to access the same or nearby set of data items over and over again.

## Storage Technologies

### Random Access Memory

- Static RAM
- Dynamic RAM

SRAM is used for cache memories, both on and off the CPU chip. DRAM is used for the main memory plus the frame buffer of a graphics system.

**SRAM**

SRAM stores each bit in a bistable memory cell. Each cell is implemented with a six-transistor circuit.

**DRAM**

DRAM stores each bit as charge on a capacitor. It's very sensitive to disturbances and loses charge in 10-100ms, too long for computers. Refresh must periodically occur by reading & rewriting. Error-correcting codes may also be used to detect & correct single bit errors.

**Conventional DRAMS**

The cells(bits) in a DRAM chip are partitioned into $d$ supercells, each consisting of $w$ DRAM cells. A $d\times w$ DRAM stores a total of $dw$ bits of information. 

Information flows in and out of the chip via external connectors called pins. Each pin carries **a 1-bit signal**. 

![Untitled](https://p.ipic.vip/3q25zr.jpg)

The memory controller sends the row address to the DRAM chip via the address pin. The DRAM chip then copies the requested row to its internal row buffer. Next, the memory controller sends the **column** address to the DRAM chip to retrieve the desired supercell.

The row address i is called a RAS (row access strobe) request and the column address is called a CAS(column access strobe) request.

**Memory Modules**

DRAM chips are packaged in memory modules that plug into expansion slots in the main system board (mother board).

Core i7 systems use the 240-pin dual inline memory module(DIMM), which transfers data to and from the memory controller in 64-bit chunks. Each DRAM chip has a total of 8M supercells. 

There are a total of 8 DRAM chips. For each chip, there are 8M supercells. For each supercell, there are 8 cells. The total size of the memory module is $\frac{8\times8M\times8}{8} =64MB$.

![Untitled](https://p.ipic.vip/yk4m87.png)

**Nonvolatile Memory**

ROMs are non-volatile memories, meaning they retain their contents even when the power is turned off. They are referred to as read-only memories for historical reasons.

Programs stored in ROM devices are often referred to as firmware. When a computer system is powered up, it runs firmware stored in ROM.

**Accessing Main Memory**

- Read transaction
- Write transaction

A bus is a collection of parallel wires that carry address, data, and control singals. More than 2 devices can share the same bus. The control wires carry signals that synchronize the transaction and identify what kind of transaction is currently being performed.

![Untitled](https://p.ipic.vip/qle0ya.jpg)

**The I/O bridge includes the memory controller**. DRAM makes up the main memory. A system bus connects CPU to I/O bridge. The I/O bridge translates the electrical signals of the system bus into the electrical of the memory bus.

### Disk Storage

Disks are composed of platters, each of which has two sides coated with magnetic recording material. These surfaces are divided into concentric rings called tracks, and each track is further divided into sectors. Each sector holds an equal number of data bits (typically 512 bytes) encoded in the magnetic material. Sectors are separated by gaps, which store formatting bits that identify them.

A disk consists of one or more platters stacked on top of each other and encased in a sealed package.

A cylinder is the collection of tracks on all surfaces that are equi-distant from the center of the spindle.

$bytes \rightarrow sectors\rightarrow tracks\rightarrow surfaces\rightarrow platters\rightarrow disks$

![Untitled](https://p.ipic.vip/1kuvm1.jpg)

**Disk with multiple platters have a separate read/write head for each surface.** The heads are lined up vertically and move At any point in time, all heads are positioned on the same cylinder.

**Disk Operation**

Disk read and write data in sector-size blocks. The access time for a secotr has three main components: seek time, rotational latency, and transfer time.

- Seek time: move the arm to the specified track
- Rotational latency: find the first bit of the target sector
- Transfer time: read or write the contents of the sector

---

The time to access the 512 bytes in a disk sector is dominated by the seek time and rotational latency.

**Logic Disk Blocks**

To hide the comlexity of the disk from the operating system, modern disks present a simpler view of their geometry as a sequence of B sector-size logical blocks numbered 0, 1, … , B - 1. A small hardware/firmware device in the disk package, called disk controller, maintains the mapping between logical block numbers and actual (physical) disk sectors.

**Connecting I/O Devices**

I/O devices such as graphics cards, monitors, mice, keyboards and disks are connected to the CPU and main memory using an I/O bus. I/O buses are designed to be independent of the underlying CPU. 

![Untitled](https://p.ipic.vip/9t7zs2.jpg)

A host bus adapter connects one or more disks to the I/O bus using a communication protocol defined by a particular host bus interface, such as SCSI or SATA. A SCSI host bus adapter can support multiple disk drives as opposed to SATA adpaters, which can only support one drive.

**Accessing Disks**

The CPU issues commands to I/O devices using a technique called **memory-mapped I/O**. A block of addresses in the address in the address space is known as an I/O port. Each device is assoicated with(or mapped to) one or more ports when it is attached to the bus.

The disk reads the dameta and transfers it directly to the memory without going through the CPU. The process is called direct memory access (DMA)

After the DMA transfer is complete and the contents of the disk sector are safely stored in main memory, the disk controller notifies the CPU by sending an interrupt signal to the CPU. An interrupt signals an external pin on the CPU chip. This causes the CPU to stop what it is currently working on and jump to an operating system routine. The routine records the fact that the I/O has finished and then returns control to the point where the CPU was interrupted

### Solid State Disks

An SSD package consists of one or more flash memory chips and **a flash translation layer**, which is a hardware/firmware device that plays the same role as a disk controller.

$page\rightarrow block \rightarrow falsh\space memory$

A flash memory consists of a sequence of B blocks, where each block consists of P pages. Typically, pages are 512 bytes to 4KB in size, and a block consists of 32-128 pages, with total block sizes ranging from 16KB to 512 KB. Data are read and written in units of pages. A page can be written only after the entire block to which it belongs has been erased.(All bits in the block set to 1) It takes a long time.

### Storage Technology Trends

Latency is the time it takes for a storage device to respond to a request, while throughput is the amount of data that can be transferred within a certain amount of time. 

With the advent of multi-core processors around 2003, the performance gap between DRAM and DISK performance and CPU performance shifted from latency to throughput.

## Locality

Computer programs that are well-written tend to have good locality, meaning they reference data items that are close to one another or the same.

- Temporal locality (same items)
- Spatial locality (adjacent items)

A funtion visits each element of a vector sequentially is said to have a **stride-1 reference pattern**. As the stride increases, the spatial locality decreases.

Accessing multidimensional arrays in row-major order ensures good locality. 

### Locality of Instruction Fetches

Loops have good temporal and spatial locality with respect to instruction fetches. The smaller the loop body and the greater the number of loop iterations, the better the locality.

## The Memory Hierarchy

In general, a cache is a small, fast storage device that acts as a staging area for the data objects stored in a larger, slower device. The process of using a cache is known as caching.

Data is copied between level k and k+1 in block-sized units. Lower-level devices have longer access times, so larger block sizes are used to amortize the time.

- Cache Hits
- Cache Misses

When a miss occurs, the cache at level `k` fetches the block containing `d` from the cache at level `k+1`. If the level `k` cache is full, an existing block may be replaced (known as replacing or evicting). The decision about which block to replace is governed by the cache’s **replacement policy.**

**Kinds of Cache Misses**

An empty cache: cold cache; Compulsory/Cold misses

Conflict misses:  occur in a set-associative or direct-mapped cache when multiple blocks of memory map to the same set or slot in the cache and they continually evict each other.

Capacity misses: the size of the working set exceeds the size of the cache.

Coherence

**Cache Management**

![Untitled](https://p.ipic.vip/o5vxhz.jpg)

## Cache Memories

<img src="https://p.ipic.vip/vownm4.jpg" alt="Untitled" style="zoom:50%;" />

### Direct-Mapped Caches

A cache with exactly one line per set (E = 1) is known as a direct-mapped cache. 

**Conflict Misses in Direct-Mapped Caches**

Confict misses in direct-mapped caches typically occur when programs access arrays whose sizes are a power of 2.

```c
float dotprod(float x[8], float y[8]) {
  float sum = 0.0
  int i;
  for(i = 0; i < 8; i++)
    sum += x[i] * y[i];
  return sum;
}
```

The first iteration of the loop references `x[0]`, a miss that causes the block containing `x[0]`-`x[3]` to be loaded into set 0. However, the cache line will soon be altered by  `y[0]`-`y[3]` . The term thrashing describes any situation where a cache is repeatedly loading and evicting the same sets of cache blocks.

`x[0]-x[3]` and `y[0]-y[3]` are both blocks of memory of the same size (4 floats * 4 bytes/float = 16 bytes). If the starting address of `y` is a multiple of 16 bytes away from the starting address of `x`, then the blocks `x[0]-x[3]` and `y[0]-y[3]` will map to the same cache line, leading to cache thrashing as described earlier.

To fix this, we can add padding to the trail of `x`. 

### Set Associative Caches

A cache with $1<E<\frac{C}{B}$ is often called an $E$-way set associative cache. $E$ is the number of lines per set.

**Fully Associative Caches**

### Issues with Writes

After the cache updates its copy of $w$, what does it do about updating the copy of $w$ in the next lower level of the hierarchy.

- Write-through: immediately write the next low level
- Write-back: defer the update until it is evicted from the cache

Both of them assume that the data is in the cache.

Another issue: write misses

- Write-allocate: loads the corresponding block from the next lower level into the cache and updates the cache block

- No-write-allocate: bypasses the cache and writes the word directly to the next lower level.

  Write-through caches are typically no-write-allocate. 

  Cause the update immediately propagate to the lower level, we don't need bother updating the cache and lower level both.

  Write-back caches are typically write-allocate.

  We need a copy tempoprarily.

### Anatomy of a Real Cache Hierarchy

Caches can hold instructions as well as data. A cache that holds instructions only is called an i-cache. A cache that holds program data only is called a d-cache. A cache that holds both instructions and data is known as a unified cache.

### Performance Impact of Cache Parameters

**Impact of Cache Size:** A larger cache: higher hit rate, but increase the hit time.

**Impact of Block Size:** Larger blocks: increase the hit rate, but larger block sizes imply a smaller number of cache lines, hurting the programs with more temporal locality than spatial locality. Larger transfer time: miss penalty escalates

**Impact of Associativity:** Higher associativity Decreases the vulunerability of the cache to thrashing due to confict misses. Larger assocaitivity is hard to implement and hard to make fast(higher hit time). A trade-off between the hit time and miss penalty.

> **Associativity** in a cache refers to the number of places (cache lines) a block of data can be placed when it is brought into the cache. In a direct-mapped cache (1-way associative), each block of data has exactly one place it can go. In a 2-way set associative cache, each block of data has two places it could go, and so on. In a fully associative cache, a block of data can go anywhere in the cache.

## Replacement Policies

### Random

Particularly for a first-level hardware cache, the system may not have the time to make a more complex decision, and the cost of making the wrong choice can be small if the item is in the next level cache. The bookkeeping cost for more complex policies can be non-trivial: keeping more information about each block requires space that may be better spent on increasing the cache size.

### FIFO

Worst scenario when iterating through an array.

<img src="https://p.ipic.vip/zq18pw.png" alt="Screenshot 2023-05-30 at 10.09.05 PM" style="zoom:50%;" />

### Optimal Cache Replacement (MIN)

Need to have the ability to predict the future.

### Least Recently Used (LRU)

If programs exhibit temporal locality, the locations they reference in the future are likely to be the same as the ones they have referenced in the recent past.

Using a linked list to record. On every cache hit, you move the block to the front of the list, and on a cache miss, you evict the block at the end of the list.

In hardware, keeping a linked list of cached blocks is too complex to implement at high speed; instead, we need to approximate LRU, and we will discuss exactly how in a bit.

<img src="https://p.ipic.vip/zz3xmr.png" alt="Screenshot 2023-05-30 at 10.14.16 PM" style="zoom:50%;" />

Can be the optimal while also the worst solution.

This method is not feasible in terms of hardware or cost. We can use **ageing**  algorithm to achieve this. 

![image-20230607101017289](https://p.ipic.vip/zqkg4t.png)

There's **a R bit and a counter** associated with each page. At each clock tick, the algorithm left shifts the counter of each page and  fill the R bit in the leftmost slot. In other words, the counter will record the recent 8 clock ticks' reference behavior. In the figure above, we can choose between the page 3 and 5 to replace at the clock tick 4. We break the tie by the fact that age 5 was referenced twice while page 3 was referenced only once. It is possible that the reference behaviors of two pages more than 8 ticks ago are quite difference, but we don't care.

### Least Frequently Used (LFU)

### Belady's Anomaly

Sometimes add the space of the cache may cause a lower hit rate.

<img src="https://p.ipic.vip/xy4nma.png" alt="Screenshot 2023-05-30 at 10.22.16 PM" style="zoom:50%;" />

## Writing Cache-Friendly Code

**Working Set Model**: Most programs will have an inflection point, or knee of the curve, where a critical mass of program data can just barely fit in the cache. This critical mass is called the program’s *working set*. As long as the working set can fit in the cache, most references will be a cache hit, and application performance will be good.

![Screenshot 2023-06-30 at 2.32.28 AM](https://p.ipic.vip/68e2oo.png)



**Zipf Model**: For web proxy page, frequency of visits to the $k$th most popular page $\propto \frac{1}{k^{\alpha}}$

A characteristic of a Zipf curve is a **heavy-tailed distribution**. Although a significant number of references will be to the most popular items, a substantial portion of references will be to less popular ones. 

- Repeated references to local variables are good because the compiler can cache them in the register file(temporal locality)
- Stride-1 reference patterns are good because caches at all levels of the memory hierarchy store data as contiguous blocks.

## Putting it Together: The Impact of Caches on Program Performance

### The Memory Mountain

The rate that a program reads data from the memory system is called the read throughput, or sometimes the read bandwidth. Typically expressed in units of megabytes per second.

Task: read elements of an array from the memory

Smaller values of size result in a smaller working set size, and thus better temporal locality. Smaller values of stride reslut in better spatial locality.

![Untitled](https://p.ipic.vip/a0s3qx.jpg)

Notice that even when the working set os too large to fit in any of the caches, the highest point on the main memory ridge is a factor of 8 higher than its lowest point. So even when a program has poor temporal locality, spatial locality can still come to the rescue and make a significant difference.

For Stride-1, the flat line implies a prefetching mechanism.

![Untitled](https://p.ipic.vip/mldv8n.jpg)

Here, the word size is 8 bytes and the block size is 64 bytes. The read throughput of s8-s11 is mainly dependent on the rate that cache blocks can be transferred from L3 into L2. 

### Rearranging Loops to Increase Spatial Locality

![Untitled](https://p.ipic.vip/agkess.jpg)

Miss rate in this case is a better predictor of performance than the total number of memory accesses.

# Demanding Paging

What does OS do on a Page Fault?

* Choose an old page to replace 
* If old page modified (“D=1”), write contents back to disk
* Change its PTE and any cached TLB to be invalid
* Load new page into memory from disk
* Update page table entry, invalidate TLB for new entry
* Continue thread from original faulting location

While pulling pages off disk for one process, OS runs another process from ready queue. The suspended process sits on wait queue.

What data structure maps non-resident pages to disk?

```c
FindBlock(PID, page#) → disk_block
```

Some OSs utilize spare space in PTE for paged blocks. It is like the PT, but is purely software. We can also store it in memory or even hash table. The backing store is also desired which saves a copy of code to swap file.

<img src="https://p.ipic.vip/lzq572.png" alt="image-20230630021120680" style="zoom:50%;" />

**Finding a Free Frame during a Page Fault**:

- During a page fault, when the OS needs to allocate a new frame for a page that is not currently in physical memory, it searches for a free frame.
- The OS typically maintains a data structure called a "free list" that keeps track of available free frames in the system.
- The free list can be implemented as a linked list, bitmap, or any other suitable data structure that allows efficient management of available frames.
- When a page fault occurs, the OS selects a free frame from the free list and assigns it to the requested page.

**Memory Management Strategies to Handle Memory Pressure**:

- Unix-like operating systems often employ a "reaper" mechanism when the available memory becomes too full.
- The reaper is responsible for identifying and freeing memory that is no longer in active use. It can reclaim memory from terminated or idle processes.
- To optimize memory utilization, the OS schedules dirty pages (pages modified but not yet written back to disk) to be written back to the disk. This ensures data integrity and frees up memory for other processes.
- Pages that have not been accessed for a while (considered "clean" or unchanged) can be zeroed out and reclaimed. This technique is known as "page zeroing" or "page aging."
- If there are no clean pages available or sufficient memory pressure exists, the OS may choose to evict a dirty page (one that has been modified) as a last resort. This involves writing the page back to disk and marking it as available for reuse.

## Preventing Page Fault

### Compulsory Misses

Pages that have never been paged into memory before

Prefetching: loading them into memory before needed

Need to predict future somehow!

### Capacity Misses

Not enough memory. Must somehow increase available memory size.

Can we do this?

* One option: Increase amount of DRAM (not quick fix!)
* Another option: If multiple processes in memory: adjust percentage of memory allocated to each one!

### Conflict Misses

Technically, conflict misses don’t exist in virtual memory, since it is a "fully-associative" cache

### Policy Misses

Caused when pages were in memory, but kicked out prematurely because of the replacement policy

How to fix? Better replacement policy

## Replacement Policies

### FIFO (First In, First Out)

Throw out oldest page. Be fair – let every page live in memory for same amount of time.

Bad – throws out heavily used pages instead of infrequently used

### RANDOM

Pick random page for every replacement

Typical solution for TLB’s. Simple hardware

Pretty unpredictable – makes it hard to make real-time guarantees

### MIN (Minimum)

Replace page that won’t be used for the longest time 

Great (provably optimal), but can’t really know future…

But past is a good predictor of the future …

### LRU (Least Recently Used)

Replace page that hasn’t been used for the longest time

Programs have locality, so if something not used for a while, unlikely to be used in the near future.

Seems like LRU should be a good approximation to MIN.

For MIN and LRU, when we add memory the miss rate drops. This is called rate drops. However, for FIFO, this is not necessarily true. This is called Bélády’s anomaly:

<img src="https://p.ipic.vip/sjopmr.png" alt="Screenshot 2023-06-30 at 3.15.53 AM" style="zoom:50%;" />

### Approximating LRU: Clock Algorithm

<img src="https://p.ipic.vip/4w115m.png" alt="Screenshot 2023-06-30 at 1.27.20 PM" style="zoom:50%;" />

**Clock ALgorithm**: Arrange physical pages in circle with single clock hand

* Approximate LRU
* Replace an old page, not the oldest page

Details:

* Hardware “use” bit per physical page (called “accessed” in Intel architecture):

* Hardware sets use bit on each reference

* If use bit isn’t set, means not referenced in a long time

* Some hardware sets use bit in the TLB; must be copied back to page TLB entry gets replaced

On page fault:

* Advance clock hand (not real time)

* Check use bit:

  1: used recently; **clear and move forward**

  The purpose of this action is to give the page a "second chance". Clearing the bit essentially resets the page's status, giving it an opportunity to be accessed again before it becomes a candidate for replacement. If the page is accessed again before the clock hand comes back to it, the use bit will be set back to 1.

  0: selected candidate for replacement

The Clock Page Replacement Algorithm is designed in such a way that it will eventually find a page to replace, or it could potentially loop indefinitely if all use bits are set (i.e., all pages are being frequently accessed). However, in practice, it's unlikely that all pages are being frequently accessed, so the algorithm will eventually find a page to replace.

**What happens when all use bits are set?**

If all use bits are set, the Clock Algorithm behaves like a First-In, First-Out (FIFO) algorithm. It will keep advancing the clock hand, clearing use bits along the way, until it makes a full loop and comes back to where it started. The first page it encounters with the use bit cleared (because it cleared it in the last loop) becomes the candidate for replacement. This FIFO behavior ensures that a page will eventually be found for replacement.

**What if the clock hand is moving slowly?**

If the clock hand is moving slowly, it might be a good sign. This could mean there are not many page faults occurring, which is usually a sign of efficient memory management. Alternatively, it could mean that the algorithm is finding replacement pages quickly because many pages are not being accessed frequently (their use bits are not set).

**What if the clock hand is moving quickly?**

If the clock hand is moving quickly, it could indicate a problem. This might mean there are a lot of page faults occurring, which can decrease the overall performance of the system. It could also mean that many pages are being accessed frequently (many use bits are set), causing the algorithm to search longer to find a suitable candidate for replacement.

**Partitioning of Pages: Young and Old**

One way to view the clock algorithm is as a crude partitioning of pages into two groups: "young" and "old". Pages with their use bit set (1) can be seen as "young", or recently used, while pages with their use bit cleared (0) can be seen as "old", or not recently used.

### Nth Chance Version of Clock Algorithm

1. For each page, the operating system maintains a counter that tracks the number of "sweeps" or cycles the clock hand has made without the page being used.
2. When a page fault occurs (i.e., a requested page is not found in memory), the OS checks the use bit of the current page pointed at by the clock hand.
3. If the use bit is 1 (indicating the page has been used in the last sweep), the OS clears the use bit and resets the counter for that page.
4. If the use bit is 0 (the page hasn't been used in the last sweep), the OS increments the counter for that page. If the count reaches N, the page is considered for replacement.

A large N will make the algorithm a good approximation to LRU while the OS might have to look a long way (i.e., many clock cycles) to find a page that can be replaced.

The approach also considers whether a page is "dirty" or modified. A page is marked as dirty if it has been modified since it was brought into memory. Replacing a dirty page incurs additional overhead because the changes need to be written back to disk to ensure data integrity. Therefore, the algorithm might give dirty pages an extra chance before replacing them. A common approach is to use N=1 for clean pages and N=2 for dirty pages. The dirty pages are written back to the disk when the counter reaches 1. The reason for this is to ensure that, even if the page gets replaced in the future, the most recent version of the data it contains is preserved on disk.

### Clock-Emulated-M

We don't need hardware-supported "modified" bit. We can emulate it using read-only bit. The algorithm is called **Clock-Emulated-M**.

* Initially, mark all pages as read-only (W to 0), even writable data pages. Further, clear all software versions of the “modified” bit to 0 (page not dirty)
* Writes will cause a page fault. Assuming write is allowed, OS sets software “modified” bit to 1, and marks page as writable (W to 1).
* Whenever page written back to disk, clear “modified” bit to 0, mark read-only

We can further emulate the **use** bit. That is called **Clock-Emulated-Use-and-M**.

### Second-Chance List Algorithm

Designed to balance the efficiency of the Least Recently Used (LRU) algorithm with the simplicity of the First-In, First-Out (FIFO) algorithm.

In this algorithm, memory is divided into two parts: the **Active List** and the **Second Chance** (SC) List. The Active List contains pages that can be accessed at full speed and are marked Read/Write (RW), while the SC List contains pages marked as invalid.

**When a page is accessed**:

* First checks if it is in the Active List. If the page is not found there, a Page Fault occurs.

  If a new page needs to be loaded and the Active List is full, the page at the end of the Active List is moved to the front of the SC List and **marked as invalid**. This process is also known as "paging out".

* If the desired page is on the SC List, it is moved to the front of the Active List and **marked as RW**. 
* If the desired page is not on the SC List, the new page is loaded into the front of the Active List and marked RW. In this case, the LRU (Least Recently Used) page at the end of the SC List is paged out. This is an important distinction from the FIFO policy, which simply removes the oldest page without considering its recent usage (because there's a second chance).

The number of pages assigned to the SC List can be varied. If no pages are assigned to the SC List, the algorithm becomes a FIFO policy. If all pages are assigned to the SC List, the algorithm behaves like LRU but incurs a page fault on every page reference (because it was marked as invalid). An intermediate value is usually chosen to balance the trade-off between disk accesses and overhead from trapping to the operating system.

The advantage of this algorithm is that it reduces disk accesses because a page only goes to disk if it is unused for a long time. The drawback is the increased overhead of trapping to the operating system, representing a trade-off between software efficiency and hardware resources.

### Freelist

When a page is removed from memory by the Clock algorithm, it isn't immediately discarded. Instead, it is added to a freelist, which is essentially a pool of pages that are not currently in use but are kept ready in case they are needed again.

This process of filling the freelist is managed by the "Pageout demon" or "Pageout daemon", which is a background process that runs the Clock algorithm (or other page replacement algorithms) to manage memory allocation and deallocation.

Dirty pages are pages that have been modified since they were loaded into memory. In the context of the Clock algorithm and the freelist, when a dirty page is replaced and moved to the freelist, it starts copying back to disk. This is because, being modified, it contains information that is not yet saved to disk, and losing it could lead to data loss.

## Allocation of Page Frames

Possible Replacement Scopes:

* Global replacement – process selects replacement frame from set of all frames; one process can take a frame from another
* Local replacement – each process selects from only its own set of allocated frames

### Equal allocation (Fixed Scheme):

Every process gets same amount of memory

Example: 100 frames, 5 processes : process gets 20 frames

### Proportional allocation (Fixed Scheme)

Allocate according to the size of process

Computation proceeds as follows:

$s_i$ = size of process $p_i$ and $S=\sum s_i$ 

$m$ = total number of physical frames in the system

$a_i$ = (allocation for $p_i$) =$\frac{s_i}{S} \times$

### Priority Allocation

Proportional scheme using priorities rather than size

* Same type of computation as previous scheme

Possible behavior: If process $p_i$ generates a page fault, select for replacement a frame from a process with lower priority number.

We want find a balance between page-fault rate and number of frames. We dynamically adjust the number of frames a process is allocated.

<img src="https://p.ipic.vip/sxoaml.png" alt="image-20230630214738969" style="zoom:50%;" />

**Thrashing**: if a process does not have "enough" pages, the page-fault rate is very high. This leads to low CPU utilization and the operating system spends most of its time swapping to disk. If there are more threads, more memory is needed and thus we need to spend a lot time on paging and swapping.

![Screenshot 2023-06-30 at 9.52.41 PM](https://p.ipic.vip/6dabts.png)

**Working-Set Model**: $\Delta$is the working-set window º= fixed number of page references (Example: 10,000 instructions)

WSi (working set of Process Pi) = total set of pages referenced in the most recent D (varies in time)

* if D too small will not encompass entire locality
* if D too large will encompass several localities
* if D = $\infin$ will encompass entire program

$D$ = $\sum|WSi|$ total demand frames 

If D > m $\rightarrow$Thrashing:

* Policy: if D > m, then suspend/swap out processes
* This can improve overall system behavior by a lot!

For compulsory misses:

**Clustering:** This is a strategy to optimize memory management and reduce the impact of page faults. The idea behind clustering is that when a page fault occurs, instead of just bringing the faulting page into memory, the operating system also brings in a set of pages around the faulting page. This is based on the principle of locality, which states that if a process accesses a particular page, it's likely to access nearby pages in the near future. Clustering takes advantage of the fact that disk reads are more efficient when reading sequential pages, as it reduces the need to move the disk read/write head.

**Working Set Tracking:** This is a strategy used to manage memory more efficiently. The working set of a process is the set of pages that the process is currently using or is likely to use in the near future. By tracking the working set of a process, the operating system can make better decisions about which pages to keep in memory and which pages to swap out. When a process is swapped out and then later swapped back in, if the operating system has been tracking the working set of the process, it can bring the entire working set back into memory. This can significantly reduce the number of page faults after the process is swapped back in, as the pages the process is likely to access are already in memory.

## Linux Memory

### Memory Zones

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

### Post Meltdown Memory Map

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
