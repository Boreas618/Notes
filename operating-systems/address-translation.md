# Address Translation Concept

The translation is usually implemented in hardware, and the operating system kernel configures the hardware to accomplish its aims.

**Goals**:

* **Memory protection** We need the ability to limit the access of a process to certain regions of memory
* **Memory sharing** We want to allow multiple processes to share selected regions of memory
* **Flexible memory placement** We want to allow the operating system the flexibility to place a process (and each part of a process) anywhere in physical memory
* **Sparse address** Many programs have multiple dynamic memory regions that can change in size over the course of the execution of the program: the heap for data objects, a stack for each thread, and memory mapped files. 
* **Runtime lookup efficiency** Time overhead
* **Compact transalation tables** Space overhead
* **Portability**

Addresses can be translated to final values anywhere in the path from source program to the in-memory binary image.

<img src="https://p.ipic.vip/72ggqw.png" alt="image-20230629020250798" style="zoom: 33%;" />

## Translation Strategies

* Uniprogramming: no translation or protection

* Primitive Multiprogramming: use loader/linker to adjust addresses while program loaded into memory

  The translation is done by a linker-loader (relocation)

  No protection

* Multiprogramming with Protection

  Base and bound

  ...

# Towards Flexible Address Translation

Address translation with base and bounds registers

* Fragmentation problem over time

  Not every process is same size Þ memory becomes fragmented over time

* Missing support for sparse address space

  Would like to have multiple chunks/program (Code, Data, Stack, Heap, etc)

* Hard to do inter-process sharing

  Want to share code segments when possible

  Want to share memory between processes

  Helped by providing multiple segments per process

![Screenshot 2023-05-25 at 11.13.01 PM](https://p.ipic.vip/08j9wf.png)

We can assign segments for code, stack, data, ...

We mantain base and bounds for different regions of a signle program.

x86 Example: `mov [es:bx],ax`.

![Screenshot 2023-06-29 at 2.15.19 AM](https://p.ipic.vip/emei15.png)

## Sgemented Memory

The hardware can support an array of pairs of base and bounds registers, for each process. This is called *segmentation*.

The high order bits of the virtual address are used to index into the array; the rest of the address is then treated as above — added to the base and checked against the bound stored at that index.

In addition, the operating system can assign different segments different permissions, e.g., to allow execute-only access to code and read-write access to data.

![Screenshot 2023-05-25 at 11.20.11 PM](https://p.ipic.vip/ocs8d1.png)

If a program branches into or tries to load data from one of these gaps, the hardware will generate an exception, trapping into the operating system kernel. On UNIX systems, this is called a *segmentation fault*.

- [x] Share regions

- [x] Facilitate interprocess communication

- [x] Efficient management of dynamically allocated memory

> For dynamically allocated memory like heap, we don't know how much memory space will be consumed, therefore we don't know how much memory we need to zero.
>
> We address this issue by zero-on-reference. With zero-on-reference, the operating system allocates a memory region for the heap, but only zeroes the first few kilobytes. Instead, it sets the bound register in the segment table to limit the program to just the zeroed part of memory. If the program expands its heap, it will take an exception, and the operating system kernel can zero out additional memory before resuming execution.

External fragmentation: the operating system may reach a point where there is enough free space for a new segment, but the free space is not contiguous.

> UNIX fork and copy-on-write
>
> When creating a new child, the segment table of the parent is copied and all of the segments shared between the parent and child process are marked “read-only”. If either side modifies data in a segment, an exception is raised and a full memory copy of that segment is made at that time.

## Paged Memory

With paging, memory is allocated in fixed-sized chunks called *page frames*. Address translation is similar to how it works with segmentation.

The page is fix-sized so the page table only need to provide the upper bits of the page frame address. 

![Screenshot 2023-05-26 at 2.02.31 AM](https://p.ipic.vip/ocyhdh.png)

The range of the offset should be the size the frame. The bigger the frame size, the smaller the size of the page table.

Paging addresses the issue of free-space allocation. The operating system represents the memory in a bit map. Finding a free frame is just a matter of finding an empty bit.

Sharing memory between processes is also convenient: we need to set the page table entry for each process sharing a page to point to the same physical page frame.

Features:

* Copy-on-write

* Zero-on-reference

* Load the pages and execute the program at the same time

  Once the first few pages are in memory, the operating system can start execution of the program in user-mode, while the kernel continues to transfer the rest of the program’s code in the background. As the program starts up, if it happens to jump to a location that has not been loaded yet, the hardware will cause an exception, and the kernel can stall the program until that page is available.

* Data breakpoints

Downside:

The tradeoff between the size of the page and the size of the page table.

The layout of the stacks and heaps of the multi-thread programs.

## Multi-Level Transalation

**Paged Segmentation**

![Screenshot 2023-05-26 at 2.33.02 AM](https://p.ipic.vip/pbm6oj.png)

Segment tables are sometimes stored in special hardware registers, the page tables for each segment are quite a bit larger in aggregate, so they are normally stored in physical memory.

To keep the memory allocator simple(Cause the page table is stored in the memory, we expect the page table to be the same size as the size of one page), the maximum segment size is usually chosen to allow the page table for each segment to be a small multiple of the page size.

For example, with 32-bit virtual addresses and 4 KB pages, we might set aside the upper ten bits for the segment number, the next ten bits for the page number, and twelve bits for the page offset. In this case, if each page table entry is four bytes, **the page table for each segment would exactly fit into one physical page frame.**

**Multi-Level Paging**

![Screenshot 2023-05-26 at 2.42.31 AM](https://p.ipic.vip/3iq3ep.png)

Each level of page table is designed to fit in a physical page frame. Only the top-level page table must be filled in; the lower levels of the tree are allocated only if those portions of the virtual address are in use by a particular process.

**Muti-Level Paged Segmentation**

We can combine these two approaches by using a segmented memory where each segment is managed by a multi-level page table. This is the approach taken by the x86, for both its 32-bit and 64-bit addressing modes.

The x86 has a per-process *Global Descriptor Table* (GDT), equivalent to a segment table. The GDT is stored in memory; each entry (descriptor) points to the (multi- level) page table for that segment along with the segment length and segment access permissions. To start a process, the operating system sets up the GDT and initializes a register, the *Global Descriptor Table Register (GDTR)*, that contains the address and length of the GDT.

The x86 uses separate processor registers to specify the segment number (that is, the index into the GDT) and the virtual address for use by each instruction. For example, on the “32-bit” x86, there is both a segment number and 32 bits of virtual address within each segment. On the 64-bit x86, the virtual address within each segment is extended to 64 bits. 

The segment register is often implicit as part of the instruction. For example, the x86 stack instructions such as push and pop assume the stack segment (the index stored in the stack segment register), branch instructions assume the code segment (the index stored in the code segment register), and so forth.

For the 32-bit x86, the virtual address space within a segment has a two-level page table. **The first 10 bits of the virtual address index the top level page table, called the *page directory*, the next 10 bits index the second level page table, and the final 12 bits are the offset within a page.** Each page table entry takes four bytes and the page size is 4 KB, so the top-level page table and each second-level page table fits in a single physical page. The number of second-level page tables needed depends on the length of the segment; they are not needed to map empty regions of virtual address space. Both the top-level and second-level page table entries have permissions, so fine-grained protection and sharing is possible within a segment.

>  **About Segment**
>
> 1. **Text Segment (Code Segment)**: This contains the executable code of the program.
> 2. **Data Segment**: This segment usually contains the global and static variables that are initialized by the programmer.
> 3. **BSS Segment (Block Started by Symbol)**: This segment contains the global and static variables that are not initialized by the programmer.
> 4. **Stack Segment**: This contains the program's runtime stack. Local variables and function call information typically reside here.
> 5. **Heap Segment**: This is used for dynamic memory allocation during the program execution.

As an optimization, the 64-bit x86 has the option to eliminate one or two levels of the page table. Each physical page frame on the x86 is 4 KB. Each page of fourth level page table maps 2 MB of data, and each page of the third level page table maps 1 GB of data. If the operating system places data such that the entire 2 MB covered by the fourth level page table is allocated contiguously in physical memory, then the page table entry one layer up can be marked to point directly to this region instead of to a page table. Likewise, a page of third level page table can be omitted if the operating system allocates the process a 1 GB chunk of physical memory.

## Towards Efficiency Address Translation

A ***translation lookaside buffer (TLB**)* is a small hardware table containing the results of recent address translations. Each entry in the TLB maps a virtual page to a physical page:

```c
TLB entry = {
  virtual page number,
  physical page frame number,
  access permissions
}
```

Instead of finding the relevant entry by a multi-level lookup or by hashing, the TLB hardware (typically) checks all of the entries simultaneously against the virtual page.

An new translation will replace hte least used entry.

<img src="https://p.ipic.vip/nbbpvl.png" alt="Screenshot 2023-05-29 at 10.19.51 AM" style="zoom:50%;" />

TLB table entries are implemented in fast on-chip static memory near the processor. Multiple levels of TLB are used to keep lookups rapid, with smaller first level TLBs close to the processor and larger second level TLBs consulted if necessary.

A TLB also requires an address comparator for each entry to check in parallel if there is a match. To reduce this cost, some TLBs are *set associative*. Compared to fully associative TLBs, set associative ones need fewer comparators, but they may have a higher miss rate.
$$
Cost(address\space translation)= Cost(TLB\space lookup) + Cost (full\space  translation) × (1 - P(hit))
$$

### Superpages

Superpage can drastically reduce the number of TLB entries needed to map large, contiguous regions of memory. Each entry in the TLB has a flag, signifying whether the entry is a page or a superpage.

![Screenshot 2023-05-29 at 10.35.59 AM](https://p.ipic.vip/0p6hnk.png)

When looking for a match against a superpage, the TLB only considers the most significant bits of the address, ignoring the offset within the superpage. For a 2 MB superpage, the offset is the lowest 21 bits of the virtual address. For a 1 GB superpage it is the lowest 30 bits.

Use case: map the frame buffer for computer display

### TLB Consistency

* Process context switch

  When a process context switch happens, the TLB of old process should be not accessed by the new process. We introduce **tagged TLB**.

  ```pseudocode
  tagged TLB entry = {
  	process ID,
  	virtual page number,
  	physical page frame number,
  	access permissions
  }
  ```

  When performing a lookup, the hardware ignores TLB entries from other processes, but it can reuse any TLB entries that remain from the last time the current process executed.

* Permission reduction

  Whenever the operating system changes the page table, it ensures that the TLB does not contain an incorrect mapping.

  Nothing needs to be done when the operating system adds permissions to a portion of the virtual address space. 

  > The operating system might dynamically extend the heap or the stack by allocating physical memory and changing invalid page table entries to point to the new memory, or the operating system might change a page from read-only to read-write. In these cases, the TLB can be left alone because any references that require the new permissions will either cause the hardware load the new entries or cause an exception, allowing the operating system to load the new entries.

  However, if the operating system needs to *reduce permissions* to a page, then the kernel needs to ensure the TLB does not have a copy of the old translation before resuming the process. If the page was shared, the kernel needs to ensure that the TLB does not have the copy for any of the process ID’s that might have referenced the page. For example, to mark a region of memory as copy-on-write, the operating system must reduce permissions to the region to read-only, and it must remove any entries for that region from the TLB, since the old TLB entries would still be read- write.

  Early computers discarded the entire contents of the TLB whenever there was a change to a page table, but more modern architectures, including the x86 and the ARM, support the removal of individual TLB entries.

* TLB shootdown

  For multiprocessor, each processor's TLB need to be discarded when the page table is changed. Only the current processor can invalidate its own TLB, so removing the entry from all processors on the system requires that the operating system interrupt each processor and send a TLB shutdown to all of them. The original processor can continue only when *all* of the processors have acknowledged removing the old entry from their TLB.

### Virtually Addressed Caches

Another step to improving the performance of address translation is to include a virtually addressed cache *before* the TLB is consulted. A virtually addressed cache stores a copy of the contents of physical memory, indexed by the virtual address.

Often, like the TLB, the virtually addressed cache will be split in half, one for instruction lookups and one for data.

![Screenshot 2023-05-29 at 11.06.33 AM](https://p.ipic.vip/ee64ou.png)

Most systems with virtually addressed caches use them in tandem with the TLB. Each virtual address is looked up in both the cache and the TLB at the same time; the TLB specifies the permissions to use, while the cache provides the data if the access is permitted. This way, only the TLB’s permissions need to be kept up to date. The TLB and virtual cache are co-designed to take the same amount of time to perform a lookup, so the processor does not stall waiting for the TLB.

**Memory aliasing**: different process, different virtual memory, same physical memory.

Store the physical address along with the virtual address in the virtual cache. In parallel with the virtual cache lookup, the TLB is consulted to generate the physical address and page permissions. On a store instruction modifying data in the virtual cache, the system can do a reverse lookup to find all the entries that match the same physical address, to allow it to update those entries.

### Physically Addressed Caches

![Screenshot 2023-05-29 at 11.14.26 AM](https://p.ipic.vip/zhbc78.png)

Typically, the second-level cache is per-core and is optimized for latency; a typical size is 256 KB. The third-level cache is shared among all of the cores on the same chip and will be optimized for size; it can be as large as 2 MB on a modern chip.
