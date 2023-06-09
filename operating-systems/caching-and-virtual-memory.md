# The Memory Hierarchy

Program with good locality tend to access the same or nearby set of data items over and over again.

## Storage Technologies

### Random Access Memory

- Static RAM
- Dynamic RAM

**SRAM**

SRAM is used for cache memories, both on and off the CPU chip. DRAM is used for the main memory plus the frame buffer of a graphics system. 

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

Core i7 systems use the 240-pin dual inline memory module(DIMM), which transfers data to and from the memory controller in 64-bit chunks. Each DRAM chip has a total of 8M supercells. Each DRAM chip has 512MB.

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

A flash memory consists of a sequence of B blocks, where each block consists of P pages. Typically, pages are 512 bytes to 4KB in size, and s block consists of 32-128 pages, with total block sizes ranging from 16KB to 512 KB. Data are read and written in units of pages. A page can be written only after the entire block to which it belongs has been erased.(All bits in the block set to 1) It takes a long time.

### Storage Technology Trends

Latency is the time it takes for a storage device to respond to a request, while throughput is the amount of data that can be transferred within a certain amount of time. 

With the advent of multi-core processors around 2003, the performance gap between DRAM and DISK performance and CPU performance shifted from latency to throughput.

## Locality

Computer programs that are well-written tend to have good locality, meaning they reference data items that are close to one another or the same.

- Temporal locality(same items)
- Spatial locality(adjacent items)

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

Placement policy: A small subset of the blocks at level k for the data to be cached

Conflict misses: multiple blocks are attempting to occupy the same block.

Capacity misses: the size of the working set exceeds the size of the cache.

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

# Case Study: Memory-Mapped Files

Read/write system calls allow the program to work on a copy of file data. The file is copied on to the buffer.

For a **memory-mapped file**, the operating system provides the illusion that the file is a program segment; like any memory segment, the program can directly issue instructions to load and store values to the memory. Unlike file read/write, the load and store instructions do not operate on a copy; they directly access and modify the contents of the file (**a reflection**) , treating memory as a write-back cache for disk.

We saw an example of a memory-mapped file in the previous chapter: the program executable image. 

> The key difference between the two approaches is that read/write system calls involve explicit I/O operations, while memory-mapped files use the virtual memory system to handle I/O implicitly. In other words, with memory-mapped files, the process operates on the file as though it were in memory, and the operating system takes care of fetching pages from disk and writing them back as needed.

## Implementation

A system call to map the file into a portion of the virtual address space. Initialize a set of page table entries for that region of the virtual address space. Each entry is **invalid**.

When the process issues an instruction that touches an invalid mapped address, a sequence of events occurs

**TLB miss.** The hardware looks the virtual page up in the TLB, and finds that there is not a valid entry. This triggers a full page table lookup in hardware.

> TLB is a cache for page table

**Page table exception.** The hardware walks the multi-level page table and finds the page table entry is invalid. This causes a hardware *page fault* exception trap into the operating system kernel.

**Convert virtual address to file offset.** In the exception handler, the kernel looks up in its segment table to find the file corresponding to the faulting virtual address and converts the address to a file offset.

**Disk block read.** The kernel allocates an empty page frame and issues a disk operation to read the required file block into the allocated page frame. While the disk operation is in progress, the processor can be used for running other threads or processes.

**Disk interrupt.** The disk interrupts the processor when the disk read finishes, and the scheduler resumes the kernel thread handling the page fault exception.

**Page table update.** The kernel updates the page table entry to point to the page frame allocated for the block and sets the entry to valid.

**Resume process.** The operating system resumes execution of the process at the instruction that caused the exception.

**TLB miss.** The TLB still does not contain a valid entry for the page, triggering a full page table lookup.

**Page table fetch.** The hardware walks the multi-level page table, finds the page table entry valid, and returns the page frame to the processor. The processor loads the TLB with the new translation, evicting a previous TLB entry, and then uses the translation to construct a physical address for the instruction.

To make this work, we need an empty page frame to hold the incoming page from disk. To create an empty page frame, the operating system must:

**Select a page to evict.** Assuming there is not an empty page of memory already available, the operating system needs to select some page to be replaced. We discuss how to implement this selection in Section 9.6.3 below.

**Find page table entries that point to the evicted page.** The operating system then locates the set of page table entries that point to the page to be replaced. It can do this with a *core map* — an array of information about each physical page frame, including which page table entries contain pointers to that particular page frame.

**Set each page table entry to invalid.** The operating system needs to prevent anyone from using the evicted page while the new page is being brought into memory. Because the processor can continue to execute while the disk read is in progress, the page frame may temporarily contain a mixture of bytes from the old and the new page. Therefore, because the TLB may cache a copy of the old page table entry, a TLB shootdown is needed to evict the old translation from the TLB.

**Copy back any changes to the evicted page.** If the evicted page was modified, the contents of the page must be copied back to disk before the new page can be brought into memory. Likewise, the contents of modified pages must also be copied back when the application closes the memory-mapped file.

How do we know some page have been modified?

> Hardware to keep track of which pages have been modified. Most processor architectures reserve a bit in each page table entry to record whether the page has been modified. This is called a *dirty bit*. The operating system initializes the bit to zero, and the hardware sets the bit automatically when it executes a store instruction for that virtual page.
>
> Since the TLB can contain a copy of the page table entry, the TLB also needs a dirty bit per entry. The hardware can ignore the dirty bit if it is set in the TLB, but whenever it goes from zero to one, the hardware needs to copy the bit back to the corresponding page table entry.

To minimize this delay, the system can proactively clean dirty pages via a background thread. The thread identifies good eviction candidates (e.g., rarely accessed pages), then:

1. The kernel marks the page as clean (without changing its content), indicating upcoming disk write-back.
2. It performs a TLB shootdown, removing the page table entry from the TLB, erasing the old dirty bit.
3. The kernel writes the page to disk. All changes in CPU's on-chip memory cache and write buffers are ensured to be written back to main memory first.
4. After disk write, process execution resumes. If the page is modified again, the dirty bit is reset, indicating the page cannot be evicted without saving changes.

