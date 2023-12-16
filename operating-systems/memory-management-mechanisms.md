# Memory Management (Mechanisms)

Background concepts:

* Address Binding
* Logical Address vs. Physical Address
* Object Modules &  Load Modules
* Linkage Editors
* Static Linker vs. Dynamic Linker
* Absolute Loading vs. Relocatable Loading vs. Dynamic Loading

## Memory Partitioning

Fixed partitioning is trivial. We focus on dynamic partitioning and buddy systems.

### Dynamic Partitioning

When a process arrives and needs memory, the system searches the set for a hole that is large enough for this process. 

<img src="https://p.ipic.vip/31bd1r.png" alt="Screenshot 2023-12-01 at 3.46.07 AM" style="zoom:50%;" />

* **Best-fit**: Choose the block that is closest in size to the request.
* **First-fit**: Scan the memory from the beginning and choose the first available block that is large enough.
* **Next-fit**: Scan the memory from the location of the last placement and choose the next available block that is large enough.

**Discussion**:

* The first-fit algorithm is not only the simplest but usually the best and fastest as well.
* The next-fit tends to be slightly worse than first-fit.
* The best-fit algorithm is usually the worst performer.

### Buddy System

<img src="https://p.ipic.vip/qywc83.png" alt="Screenshot 2023-12-01 at 4.00.23 AM" style="zoom: 33%;" />



## Segmented Memory

The hardware can support an array of pairs of base and bounds registers, for each process. This is called *segmentation*.

The high order bits of the virtual address are used to index into the array; the rest of the address is then treated as above — added to the base and checked against the bound stored at that index.

In addition, the operating system can assign different segments different permissions, e.g., to allow execute-only access to code and read-write access to data.

<img src="https://p.ipic.vip/ocs8d1.png" alt="Screenshot 2023-05-25 at 11.20.11 PM" style="zoom:50%;" />

x86 Example: `mov [es:bx],ax`.

![Screenshot 2023-06-29 at 2.15.19 AM](https://p.ipic.vip/emei15.png)

There are 4 segments for a single process. We define the format the virtual address:

* The top 2 bits are segment ID
* The lower bits are offset

<img src="https://p.ipic.vip/9bf5pp.png" alt="image-20230629133908094" style="zoom:50%;" />

In this process:

* `0x240` is seg 0 + offset 0x240
* `0x4050` is seg 1 + offset 0x50

With this formulation, there are definitely holes in the **virtual memory space**. The holes are referred to as **external fragmentation**s. If a program branches into or tries to load data from one of these gaps, the hardware will generate an exception, trapping into the operating system kernel. On UNIX systems, this is called a **segmentation fault**.

**External fragmentation**: the operating system may reach a point where there is enough free space for a new segment, but the free space is not contiguous.

If there is not enough space in memory, we need to perform an extreme form of context switch: swapping. In order to make room for next process, some or all of the previous process is moved to disk. This greatly increases the cost of context-switching.

> For dynamically allocated memory like heap, we don't know how much memory space will be consumed, therefore we don't know how much memory we need to zero.
>
> We address this issue by **zero-on-reference**. With zero-on-reference, the operating system allocates a memory region for the heap, but only zeroes the first few kilobytes. Instead, it sets the bound register in the segment table to limit the program to just the zeroed part of memory. If the program expands its heap, it will take an **exception**, and the operating system kernel can zero out additional memory before resuming execution.

> UNIX `fork` and **copy-on-write**
>
> When creating a new child, the segment table of the parent is copied and all of the segments shared between the parent and child process are marked “read-only”. If either side modifies data in a segment, an exception is raised and a full memory copy of that segment is made at that time.

## Paged Memory

With paging, physical memory is allocated in fixed-sized chunks called **page frames**. Address translation is similar to how it works with segmentation.

The page is fix-sized so the page table only need to provide the upper bits of the page frame address. 

<img src="https://p.ipic.vip/ocyhdh.png" alt="Screenshot 2023-05-26 at 2.02.31 AM" style="zoom:50%;" />

The range of the offset should be the size the frame. The bigger the frame size, the smaller the size of the page table.

Paging addresses the issue of free-space allocation. The operating system represents the memory in a bit map. Finding a free frame is just a matter of finding an empty bit.

**Support Inter Process Communication**: set the page table entry for each process sharing a page to point to the same physical page frame.

When performing context switch, the page table pointer and limit need to be changed.

**Features**:

* Copy-on-write

* Zero-on-reference

* Load the pages and execute the program at the same time

  Once the first few pages are in memory, the operating system can start execution of the program in user-mode, while the kernel continues to transfer the rest of the program’s code in the background. As the program starts up, if it happens to jump to a location that has not been loaded yet, the hardware will cause an exception, and the kernel can stall the program until that page is available.

* Data breakpoints

**Downside**:

The tradeoff between the size of the page and the size of the page table.

The layout of the stacks and heaps of the multi-thread programs.

**NOTICE**: All page frames should be recorded in the page table. Even if we only use two pages, such as the lowest virtual address and the highest virtual address, all virtual addresses and their corresponding physical addresses should be recorded. This is because we use the page number to index into the page table. If we need to add a new mapping in the case where only the required pages are recorded, we will not be able to index into the correct location. Later, we will see that this requirement explains why a multi-level page table is better than a single-level page table.

## Multi-Level Transalation

### Paged Segmentation

<img src="https://p.ipic.vip/pbm6oj.png" alt="Screenshot 2023-05-26 at 2.33.02 AM" style="zoom:50%;" />

Segment tables are sometimes stored in special hardware registers, the page tables for each segment are quite a bit larger in aggregate, so they are normally stored in physical memory.

To keep the memory allocator simple (Because the page table is stored in the memory, we expect the page table to be the same size as the size of one page), the maximum segment size is usually chosen to allow the page table for each segment to be a small multiple of the page size.

For example, with 32-bit virtual addresses and 4 KB pages, we might set aside the upper ten bits for the segment number, the next ten bits for the page number, and twelve bits for the page offset. In this case, if each page table entry is four bytes, **the page table for each segment would exactly fit into one physical page frame.**

### Multi-Level Paging

<img src="https://p.ipic.vip/3iq3ep.png" alt="Screenshot 2023-05-26 at 2.42.31 AM" style="zoom: 33%;" />

Each level of page table is designed to fit in a physical page frame. **Only the top-level page table must be filled in.**

The lower levels of the tree are allocated only if those portions of the virtual address are in use by a particular process.

> **Vrtual memory model adopted by AArch64**
>
> * `Bits[63:48]` are all zeros or ones.
> * `Bits[47:39]` can be used to index into level 0 of the page table.
> * `Bits[38:30]` can be used to index into level 1 of the page table.
> * `Bits[29:21]` can be used to index into level 2 of the page table.
> * `Bits[20:12]` can be used to index into level 3 of the page table.
> * `Bits[11:0]` represent the offset within the page.

### Muti-Level Paged Segmentation

We can combine these two approaches by using a segmented memory where each segment is managed by a multi-level page table. In the protected mode of the 80386, memory addressing involves two distinct elements: processor registers dedicated to holding segment numbers, and the virtual addresses used within these segments. 

> However, you must notice that the segment numbers are used for backward compatibility with 8086 and 80286. A flat model is more mainstream in the 80386.

The  protected mode 80386 has a per-process **Local Descriptor Table** (LDT), equivalent to a segment table. Each entry (descriptor) points to the (multi-level) page table for that segment along with the segment length and segment access permissions. To start a process, the operating system sets up the LDT and initializes a register, the **Local Descriptor Table Register** (LDTR), that contains the address and length of the LDT.

There's also a **Global Descriptor Table** (GDT). The GDT describes system segments, including the operating system itself.

There are 6 segment registers: **SS, CS, DS, ES, FS, GS**. Inside a segment register: 

<img src="https://p.ipic.vip/xh6gcz.png" alt="Screenshot 2023-06-29 at 4.41.46 PM" style="zoom:50%;" />

* G/L: selects between GDT and LDT tables.
* RPL: **R**equestor’s **P**rivilege **L**evel

> **About Segment**
>
> 1. **CS (Code Segment)**: Points to the segment containing the current program code.
> 2. **DS (Data Segment)**: Generally points to the segment where variables are stored.
> 3. **SS (Stack Segment)**: Points to the segment where the stack is located.
> 4. **ES (Extra Segment)**: A general-purpose segment register that can be used for data storage and to hold segment addresses for some instructions.
> 5. **FS**: An additional segment register introduced in the 386 processor, typically used for specific operating system tasks.
> 6. **GS**: Another segment register introduced in the 386 processor, typically used alongside FS for operating system tasks.
>
> This is different from sections in ELF. A **section** is a subdivision of a file that contains information of a similar type for linking and relocation, while a **segment** is a larger unit that groups various sections together based on their memory access properties for loading and executing the program in memory.

This is the format of a 64-bit descriptor:

<img src="https://p.ipic.vip/w14xox.png" alt="image-20230629164336699" style="zoom:50%;" />

| Portion | Meaning                                                      |
| ------- | ------------------------------------------------------------ |
| G       | Granularity of segment [ Limit Size ] (0: 16bit, 1: 4KiB unit) |
| DB      | Default operand size (0: 16bit, 1: 32bit)                    |
| A       | Freely available for use by software                         |
| P       | Segment present                                              |
| DPL     | Descriptor Privilege Level: Access requires Max(CPL,RPL)$\leq$DPL |
| S       | System Segment (0: System, 1: Code or data)                  |
| Type    | Code, Data, Segment                                          |

**16-bit Mode Legacy Applications**: In the older 16-bit mode of x86 processors, segmentation was used to provide protection and separation between different components of a program. There could be separate segments for code, data, and stack. The Requestor Privilege Level (RPL) of the code segment would determine the Current Privilege Level (CPL), which is a mechanism used for protection. In this mode, the system could support up to 64K segments.

**Modern x86-64**: In the 32-bit mode of x86 processors, although the full functionality of segments is still available, it is typically not used in the same way as in 16-bit mode. Instead, segments are set up to cover the entire address space - they are "flattened". 

However, one exception to this is the use of the GS (or FS) segment as a pointer to "Thread Local Storage" (TLS). TLS is a mechanism that allows each thread to have its own copy of data. A thread can access its TLS by using an instruction like `mov eax, gs(0x0)`, which moves the value at the start of the GS segment into the `%eax` register.

## PTE

A page table entry (PTE) is a pointer to next-level page table or to actual page. 

* **Present/absent bit**
* **Protection bits**: tell what kinds of access are permitted
* **Modified/dirty bit**
* **Referenced bit**: is set whenever a page is referenced, either for reading or for writing.
* **Caching Disabled bit**: If the kernel is waiting for some I/O device to respond to a command, it is essential that the hardware keep fetching the word from the device instead of using the old copy.

The Intel x86 architecture PTE for the last level is like:

<img src="https://p.ipic.vip/dtk9y1.png" alt="image-20230629155143574" style="zoom:50%;" />

| Portion | Meaning                                                      | Type           |
| ------- | ------------------------------------------------------------ | -------------- |
| P       | Present (same as “valid” bit in other architectures)         | Present/absent |
| W       | Writeable                                                    | Protection     |
| U       | User accessible                                              | Protection     |
| PWT     | Page write transparent: external cache write-through         | Configuration  |
| PCD     | Page cache disabled (page cannot be cached)                  | Configuration  |
| A       | Accessed: page has been accessed recently                    | Referenced     |
| D       | Dirty (PTE only): page has been modified recently            | Modified/dirty |
| PS      | Page Size: PS=1indicates a 4MB page (directory only). Bottom 22 bits of virtual address serve as offset | Configuration  |

For the 32-bit x86, the virtual address space within a segment has a two-level page table. 

* **The first 10 bits of the virtual address index the top level page table, called the *page directory*.**

* **The next 10 bits index the second level page table.**
* **The final 12 bits are the offset within a page.** 

The number of second-level page tables needed depends on the length of the segment; **they are not needed to map empty regions of virtual address space** (The minimum memory space needed to store page tables is 4MB thus. This feature ensures that a privilege over single level paging in terms of memory requirement).

Both the top-level and second-level page table entries have permissions, so fine-grained protection and sharing is possible within a segment.

For x86-64:

<img src="https://p.ipic.vip/cjfly8.png" alt="Screenshot 2023-06-29 at 5.11.06 PM" style="zoom:50%;" />

As an optimization, x86-64 has the option to **eliminate one or two levels of the page table**. Each physical page frame on the x86 is 4 KB. Each page of fourth level page table maps 2 MB of data, and each page of the third level page table maps 1 GB of data. If the operating system places data such that the entire 2 MB covered by the fourth level page table is allocated contiguously in physical memory, then the page table entry one layer up can be marked to point directly to this region instead of to a page table. Likewise, a page of third level page table can be omitted if the operating system allocates the process a 1 GB chunk of physical memory.

## Managing Virtual Memory

OS will modify page table in the following scenarios:

- When creating a new process, the operating system (OS) needs to map the virtual addresses of instructions and data to the physical addresses.
- When executing the processes, the processes may increase or shrink stack and heap space; the processes may dynamically load libraries; the processes may add and delete mappings through `mmap` and `munmap`.
- When exiting a process, the virtual memory space should be deallocated.

When it comes to adding mappings to the page table, there are two different timing strategies.

* **Instant Mapping**: When the process is created, the operating system (OS) allocates the physical pages and adds the mappings to the page table for the code and data sections. Similarly, when the user calls `mmap`, the physical pages and mappings are instantly allocated and added.

  This method can contribute to the overhead of creating a new process. Additionally, the user can claim more memory than actually needed, resulting in significant memory waste as the physical page is allocated without hesitation.

* **Lazy Mapping**: The virtual memory space is documented in PCB

## Translation Lookaside Buffer

A **translation lookaside buffer (TLB**) is a small hardware table containing the results of recent address translations. Each entry in the TLB maps a virtual page to a physical page:

<img src="https://p.ipic.vip/8h4boh.png" alt="Screenshot 2023-12-01 at 8.58.29 PM" style="zoom:50%;" />

Instead of finding the relevant entry by a multi-level lookup or by hashing, the TLB hardware (typically) checks all of the entries simultaneously against the virtual page.

When an entry is purged from the TLB, the modified bit is copied back into the page table entry in memory. The other values are already there, except the reference bit.

<img src="https://p.ipic.vip/nbbpvl.png" alt="Screenshot 2023-05-29 at 10.19.51 AM" style="zoom:50%;" />

Multiple levels of TLB are used to keep lookups rapid, with smaller first level TLBs close to the processor and larger second level TLBs consulted if necessary.

Some TLBs are **set associative**. In a set associative TLB, each virtual address can be mapped to any one of N entries in a given set of the TLB, where N is the set's associativity. Compared to fully associative TLBs, set associative ones need fewer comparators, but they may have a higher miss rate.

* **Hardware-managed TLB**: onolder computer with complex-instruction set (CISC). Hardware has to know where the page tables are located in memory (via a page-table-base-register, PTBR), as well as their exact format. Example: Intel x86 architecture
* **Software-managed TLB**: on more modern architectures. On TLB miss, the hardware raises an exception (an internal interrupt). OS raises to kernel model, and jumps to a trap handler. The trap handler lookups the page table, uses specialized privileged instructions to update TLB, and returns from the trap.

### Superpages

Superpage can drastically reduce the number of TLB entries needed to map large, contiguous regions of memory. Each entry in the TLB has a flag, signifying whether the entry is a page or a superpage.

<img src="https://p.ipic.vip/0p6hnk.png" alt="Screenshot 2023-05-29 at 10.35.59 AM" style="zoom: 33%;" />

When looking for a match against a superpage, the TLB only considers the most significant bits of the address, ignoring the offset within the superpage. For a 2 MB superpage, the offset is the lowest 21 bits of the virtual address. For a 1 GB superpage it is the lowest 30 bits.

Use case: map the frame buffer for computer display

### Tagged TLB

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

### Permission Reduction

When the operating system modifies the page table, it must ensure the TLB (Translation Lookaside Buffer) does not hold incorrect mappings. 

**Adding permissions**: like extending the heap or stack or changing a page from read-only to read-write, doesn't require TLB updates; hardware will handle new permissions automatically. 

**Reducing permissions**: changing a page to read-only, necessitates removing old entries from the TLB, especially for shared pages, to maintain correct access controls.

Historically, TLB was entirely reset after page table changes, but modern systems like x86 and ARM can selectively remove individual TLB entries.

### TLB Shootdown

In multiprocessor systems, each processor's TLB must be updated for page table changes. Since only the current processor can invalidate its own TLB, the OS must interrupt all processors to ensure the old entry is removed from each TLB. The process concludes once all processors confirm the removal.
