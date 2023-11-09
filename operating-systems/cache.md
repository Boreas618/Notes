# Memory Technologies

**SRAM** is used for cache memories, both on and off the CPU chip. SRAM stores each bit in a bistable memory cell. 

**DRAM** is used for the main memory plus the frame buffer of a graphics system. DRAM stores each bit as charge on a capacitor. It's very sensitive to disturbances and loses charge in 10-100ms, too long for computers. **Refresh** must periodically occur by reading & rewriting. **Error-correcting code**s may also be used to detect & correct single bit errors.

The cells(bits) in a DRAM chip are partitioned into $d$ supercells, each consisting of $w$ DRAM cells. A $d\times w$ DRAM stores a total of $dw$ bits of information. Information flows in and out of the chip via external connectors called pins. Each pin carries **a 1-bit signal**. 

<img src="https://p.ipic.vip/3q25zr.jpg" alt="Untitled" style="zoom:50%;" />

The memory controller sends the row address to the DRAM chip via the address pin. The DRAM chip then copies the requested row to its internal row buffer. Next, the memory controller sends the **column** address to the DRAM chip to retrieve the desired supercell.

The row address i is called a RAS (row access strobe) request and the column address is called a CAS(column access strobe) request.

**Memory Modules**

DRAM chips are packaged in memory modules that plug into expansion slots in the main system board (mother board).

Core i7 systems use the 240-pin dual inline memory module(DIMM), which transfers data to and from the memory controller in 64-bit chunks. Each DRAM chip has a total of 8M supercells. 

There are a total of 8 DRAM chips. For each chip, there are 8M supercells. For each supercell, there are 8 cells. The total size of the memory module is $\frac{8\times8M\times8}{8} =64MB$.

<img src="https://p.ipic.vip/yk4m87.png" alt="Untitled" style="zoom:50%;" />

**Nonvolatile Memory**

ROMs are non-volatile memories, meaning they retain their contents even when the power is turned off. They are referred to as read-only memories for historical reasons.

Programs stored in ROM devices are often referred to as firmware. When a computer system is powered up, it runs firmware stored in ROM.

**Accessing Main Memory**

- Read transaction
- Write transaction

A bus is a collection of parallel wires that carry address, data, and control singals. More than 2 devices can share the same bus. The control wires carry signals that synchronize the transaction and identify what kind of transaction is currently being performed.

<img src="https://p.ipic.vip/qle0ya.jpg" alt="Untitled" style="zoom:50%;" />

**The I/O bridge includes the memory controller**. DRAM makes up the main memory. A system bus connects CPU to I/O bridge. The I/O bridge translates the electrical signals of the system bus into the electrical of the memory bus.

# Locality

- Temporal locality (same items)
- Spatial locality (adjacent items)

A funtion visits each element of a vector sequentially is said to have a **stride-1 reference pattern**. As the stride increases, the spatial locality decreases.

Accessing multidimensional arrays in row-major order ensures good locality. 

**Locality of Instruction Fetches**: Loops have good temporal and spatial locality with respect to instruction fetches. The smaller the loop body and the greater the number of loop iterations, the better the locality.

# The Memory Hierarchy

In general, a cache is a small, fast storage device that acts as a staging area for the data objects stored in a larger, slower device. The process of using a cache is known as caching.

Data is copied between level $k$ and $k+1$ in block-sized units. Lower-level devices have longer access times, so larger block sizes are used to amortize the time.

- Cache Hits
- Cache Misses

When a miss occurs, the cache at level $k$ fetches the block containing the data from the cache at level $k+1$. If the cache of  level $k$ is full, an existing block may be replaced (known as replacing or evicting). The decision about which block to replace is governed by the cache’s **replacement policy.**

## Different Cache Misses

**Compulsory/Cold misses**: An empty cache (cold cache).

**Conflict misses**: Caused when multiple memory addresses map to the same cache location (due to the set-associative nature of most caches), causing cache evictions and subsequent misses.

**Capacity misses**: The size of the working set exceeds the size of the cache.

**Coherence misses**: Arise in multiprocessor systems when one processor modifies a location in memory and another processor tries to access the modified location, causing a miss because the copy in its cache is now stale or outdated. Coherence misses ensure that all processors in the system observe a single coherent view of memory.

# Cache Memories

<img src="https://p.ipic.vip/18tx80.png" alt="Screenshot 2023-08-31 at 5.08.03 PM" style="zoom:50%;" />

<img src="https://p.ipic.vip/iby9wn.png" alt="Screenshot 2023-08-31 at 5.07.34 PM" style="zoom:50%;" />

# Classification Based on E

* **Direct-Mapped Caches**

  A cache with exactly one line per set (E=1) is known as a direct-mapped cache. 

  **Conflict Misses in Direct-Mapped Caches**

  Confict misses in direct-mapped caches typically occur when programs **access arrays whose sizes are a power of 2.**

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

* **Set Associative Caches**: A cache with $1<E<\frac{C}{B}$ is often called an $E$-way set associative cache. $E$ is the number of lines per set.

* **Fully Associative Caches**: A cache with $S=1$ is often called a fully associative cache.

# Decisions About Writes

* **Write Updates**

  After the cache updates its copy of $w$, what does it do about updating the copy of $w$ in the next lower level of the hierarchy.

  * **Write-through**: immediately write the next low level
  * **Write-back**: defer the update until it is evicted from the cache

  Both of them assume that the data is in the cache.

* **Write Misses**
  * **Write-allocate**: loads the corresponding block from the next lower level into the cache and updates the cache block
  * **No-write-allocate**: bypasses the cache and writes the word directly to the next lower level.

**Write-through caches are typically no-write-allocate.** Cause the update immediately propagate to the lower level, we don't need bother updating the cache and lower level both.

**Write-back caches are typically write-allocate.** Caise we need a copy tempoprarily.

> Caches can hold instructions as well as data. A cache that holds instructions only is called an i-cache. A cache that holds program data only is called a d-cache. A cache that holds both instructions and data is known as a unified cache.

# Performance Impact of Cache Parameters

**Impact of Cache Size:** A larger cache ensures higher hit rate, but increase the hit time.

**Impact of Block Size:** Larger blocks increase the hit rate, but larger block sizes imply a smaller number of cache lines, hurting the programs with more temporal locality than spatial locality. Larger transfer time: miss penalty escalates

> A cache with 4-byte blocks offers 1000 lines, while an 8-byte block cache offers 500 lines. Sequential access benefits from 8-byte blocks, but frequent jumps favor more cache lines. Larger blocks also mean longer transfer times on misses.

**Impact of Associativity:** Higher associativity decreases the vulunerability of the cache to thrashing due to confict misses. Larger assocaitivity is hard to implement and hard to make fast(higher hit time). A trade-off between the hit time and miss penalty.

# Replacement Policies

## Random

Particularly for a first-level hardware cache, the system may not have the time to make a more complex decision, and the cost of making the wrong choice can be small if the item is in the next level cache. The bookkeeping cost for more complex policies can be non-trivial: keeping more information about each block requires space that may be better spent on increasing the cache size.

## FIFO

Worst scenario when iterating through an array.

<img src="https://p.ipic.vip/zq18pw.png" alt="Screenshot 2023-05-30 at 10.09.05 PM" style="zoom:50%;" />

## Optimal Cache Replacement (MIN)

Need to have the ability to predict the future.

## Least Recently Used (LRU)

If programs exhibit temporal locality, the locations they reference in the future are likely to be the same as the ones they have referenced in the recent past.

Using a linked list to record. On every cache hit, you move the block to the front of the list, and on a cache miss, you evict the block at the end of the list.

In hardware, keeping a linked list of cached blocks is too complex to implement at high speed; instead, we need to approximate LRU, and we will discuss exactly how in a bit.

<img src="https://p.ipic.vip/zz3xmr.png" alt="Screenshot 2023-05-30 at 10.14.16 PM" style="zoom:50%;" />

Can be the optimal while also the worst solution.

This method is not feasible in terms of hardware or cost. We can use **ageing**  algorithm to achieve this. 

<img src="https://p.ipic.vip/zqkg4t.png" alt="image-20230607101017289" style="zoom:50%;" />

There's **a R bit and a counter** associated with each page. At each clock tick, the algorithm left shifts the counter of each page and  fill the R bit in the leftmost slot. In other words, the counter will record the recent 8 clock ticks' reference behavior. In the figure above, we can choose between the page 3 and 5 to replace at the clock tick 4. We break the tie by the fact that age 5 was referenced twice while page 3 was referenced only once. It is possible that the reference behaviors of two pages more than 8 ticks ago are quite difference, but we don't care.

## Least Frequently Used (LFU)

**Belady's Anomaly**: Sometimes add the space of the cache may cause a lower hit rate.

<img src="https://p.ipic.vip/xy4nma.png" alt="Screenshot 2023-05-30 at 10.22.16 PM" style="zoom:50%;" />

# Writing Cache-Friendly Code

**Working Set Model**: Most programs will have an inflection point, or knee of the curve, where a critical mass of program data can just barely fit in the cache. This critical mass is called the program’s *working set*. As long as the working set can fit in the cache, most references will be a cache hit, and application performance will be good.

**Zipf Model**: For web proxy page, frequency of visits to the $k$th most popular page $\propto \frac{1}{k^{\alpha}}$

A characteristic of a Zipf curve is a **heavy-tailed distribution**. Although a significant number of references will be to the most popular items, a substantial portion of references will be to less popular ones. 

**Rules**:

- Repeated references to local variables are good because the compiler can cache them in the register file(temporal locality)
- Stride-1 reference patterns are good because caches at all levels of the memory hierarchy store data as contiguous blocks.
