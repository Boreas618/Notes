# Storage Technologies

## Disk Storage

Disks are composed of platters, each of which has two sides coated with magnetic recording material. These surfaces are divided into concentric rings called tracks, and each track is further divided into sectors. Each sector holds an equal number of data bits (typically 512 bytes) encoded in the magnetic material. Sectors are separated by gaps, which store formatting bits that identify them.

A disk consists of one or more platters stacked on top of each other and encased in a sealed package.

A cylinder is the collection of tracks on all surfaces that are equi-distant from the center of the spindle.

$$
bytes \rightarrow sectors\rightarrow tracks\rightarrow surfaces\rightarrow platters\rightarrow disks
$$
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

Peripheral devices such as graphics cards, mice, keyboards, and disks communicate with the CPU and main memory using various bus systems, often facilitated by distinct bridging components.

1. **Host Bridge**: Typically, the main connection between the CPU and the primary memory (RAM) is managed by the host bridge. In some architectures, it might also handle communications with high-speed devices like graphics cards. The host bridge ensures rapid data exchanges between these components, especially where latency and bandwidth are crucial.
2. **I/O Bridge**: This bridge manages the communication between the CPU/main memory and many I/O devices. Devices like keyboards, mice, disks, and even some communication ports fall under its purview. The I/O bridge ensures data is effectively and accurately transferred between the main system and these peripherals, even if these data transfers are not as high-speed as those managed by the host bridge.

To note, I/O buses, often managed by the I/O bridge, are designed to be compatible across a variety of CPU architectures, allowing for a wide range of devices to be connected without significant compatibility issues.

> I/O bridge can be part of the southbridge in older PC architectures, which is responsible for connecting lower-speed peripheral devices to the system.
>
> Host bridge is also known as the Northbridge in traditional PC architectures.

A host bus adapter connects one or more disks to the I/O bus using a communication protocol defined by a particular host bus interface, such as SCSI or SATA. A SCSI host bus adapter can support multiple disk drives as opposed to SATA adpaters, which can only support one drive.

**Accessing Disks**

The CPU issues commands to I/O devices using a technique called **memory-mapped I/O**. A block of addresses in the address in the address space is known as an I/O port. Each device is assoicated with(or mapped to) one or more ports when it is attached to the bus.

The disk reads the data and transfers it directly to the memory without going through the CPU. The process is called direct memory access (DMA).

After the DMA transfer is complete and the contents of the disk sector are safely stored in main memory, the disk controller notifies the CPU by sending an interrupt signal to the CPU. An interrupt signals an external pin on the CPU chip. This causes the CPU to stop what it is currently working on and jump to an operating system routine. The routine records the fact that the I/O has finished and then returns control to the point where the CPU was interrupted.

## Solid State Disks

An SSD package consists of one or more flash memory chips and **a flash translation layer**, which is a hardware/firmware device that plays the same role as a disk controller.

$$
page\rightarrow block \rightarrow flash\space memory
$$
A flash memory consists of a sequence of $B$ blocks, where each block consists of $P$ pages. Typically, pages are 512 bytes to 4KB in size, and a block consists of 32-128 pages, with total block sizes ranging from 16KB to 512 KB. **Data are read and written in units of pages.** A page can be written only after the entire block to which it belongs has been erased (All bits in the block set to 1). It takes a long time.

# File System Design

**File systems**: Layer of OS that transforms block interface of disks (or other block devices) into Files, Directories, etc.

## Disk Management

Basic entities on a disk:

* **File**: user-visible group of blocks arranged sequentially in logical space

* **Directory**: user-visible index mapping names to files

The disk is accessed as linear array of sectors

How to identify a sector?

* **Physical position**：A Sector is identified by a vector `[cylinder, surface, sector]`. It is not used anymore & OS/BIOS must deal with bad sectors.

* **Logical Block Addressing**: Every sector has integer address. Controller translates from address $\Rightarrow$ physical position.

## Components of a File System

<img src="https://p.ipic.vip/ow5aqg.png" style="zoom:50%;" />

Open file description is better described as remembering the inumber of the file, not its name.

![Screenshot 2023-07-05 at 11.31.11 PM](https://p.ipic.vip/6qejby.png)

Open performs ***Name Resolution***: translates path name into a “file number”

Read and Write operate on the file number: use file number as an “index” to locate the blocks

### File Number

How to get the file number? Look up in **directory structure**

A directory is a file containing `<file_name : file_number>` mappings

* File number could be a file or another directory
* Operating system stores the mapping in the directory in a format it interprets
* Each `<file_name : file_number>` mapping is called a directory entry

Process isn’t allowed to read the raw bytes of a directory

* The read function doesn’t work on a directory

* Instead, see `readdir`, which iterates over the map without revealing the raw bytes

### Directory Structure

How many disk accesses to resolve “/my/book/count”?

* Read in file header for root (fixed spot on disk)

* Read in first data block for root

  Table of file name/index pairs. 

  Search linearly – ok since directories typically very small

* Read in file header for “my”

* Read in first data block for “my”; search for “book”

* Read in file header for “book”

* Read in first data block for “book”; search for “count”

* Read in file header for “count”

### In-Memory File System Structures

![image-20230706000111552](https://p.ipic.vip/8hvl4i.png)

Open syscall: find inode on disk from pathname (traversing directories)

* Create “in-memory inode” in system-wide open file table

* One entry in this table no matter how many instances of the file are open

Read/write syscalls look up in-memory inode using the file handle

## FAT (File Allocation Table)

File is a collection of disk blocks.

<img src="https://p.ipic.vip/iuhoj0.png" alt="Screenshot 2023-07-06 at 12.08.07 AM" style="zoom:50%;" />

Assume we have a way to translate a path to a "file number".

Disk Storage is a collection of Blocks. It just hold file data (offset o = < B, x >)

Example: `file_read 31, <2,x>`

* Index into FAT with **file number** (31)

* Follow **linked list** to block
* Read the block from disk into memory

The FAT matches the layout of disk blocks one by one.

The FAT is stored on disk.

In order to format a disk, we zero the blocks and mark FAT entries "free".

In order to quick format a disk, we mark FAT entries "free".

### Directories

![image-20230706003323265](https://p.ipic.vip/cdba92.png)

A directory is a file containing `<file_name: file_number>` mappings

In FAT: file attributes are kept in directory (!!!)

* Not directly associated with the file itself

Each directory a linked list of entries

* Requires linear search of directory to find particular entry

Where do you find root directory (“/”)?

* At well-defined place on disk

* For FAT, this is at block 2 (there are no blocks 0 or 1)

## Unix File System

File Number is index into set of inode arrays

Index structure is an array of *inodes*

* File Number (inumber) is an index into the array of inodes

* Each inode corresponds to a file and contains its metadata

So, things like read/write permissions are stored with *file,* not in directory

Allows multiple names (directory entries) for a file

Inode maintains a multi-level tree structure to find storage blocks for files

* Great for little and large files

* Asymmetric tree with fixed sized blocks

![image-20230706003947296](https://p.ipic.vip/gmetbs.png)

For small files, there are 12 pointers direct to data blocks. It is sufficient for files up to 48 KB. 

4KB blocks can actually hold 1024 pointers. So Dbl. Indirect pointer can hold  blocks with size of 4MB. Thripl. indirect pointer can hold blocks with size of 4GB.

## Fast File System

Same inode structure as in BSD 4.1

Optimization for Performance and Reliability:

* Distribute inodes among different tracks to be closer to data

* Uses bitmap allocation in place of freelist

* Attempt to allocate files contiguously

* 10% reserved disk space

* Skip-sector positioning (mentioned later) 

The UNIX BSD 4.2 (FFS) distributed the header information (inodes) closer to the data blocks

* Often, inode for file stored in same “cylinder group” as parent directory of the file 
* makes an “ls” of that directory run very fast

File system volume divided into set of block groups

* Close set of tracks

Data blocks, metadata, and free space interleaved within block group

* Avoid huge seeks between user data and system structure

Put directory and its files in common block group

<img src="https://p.ipic.vip/drpe3f.png" alt="image-20230706022239753" style="zoom:50%;" />

**First-Free Allocation**: FFS uses a first-free allocation strategy, assigning data to the first available block.**Expanding Files**: If a file needs to grow, FFS first tries to find **successive** free blocks. If none are available, it chooses a new range of blocks on the disk. Finally, it can even go to another block group.

**Block Clustering**: In this approach, FFS leaves a few unallocated spaces at the start of **a group of blocks** (block cluster), placing larger, sequential blocks towards the end.

> When a file is stored on disk, it's broken up into these blocks. If a file doesn't fully use up a block, that remaining space becomes unusable for other files because blocks cannot be shared between files. This is called internal fragmentation, and it can waste a lot of space if not managed well.
>
> To mitigate this issue, FFS uses block clustering. This involves grouping a number of blocks together into a larger block, often known as a "cluster", and treating that larger block as a single entity.

**Sequential Layout for Large Files**: For large files, FFS stores blocks in sequential order on the disk, reducing the movement of the disk head and boosting read/write performance.

**Reserve Space**: reserve 10% free space in the block group.

### Attack of the Rotational Delay

**Issue**: Read one block, do processing, and read next block. In meantime, disk has continued turning: missed next block! Need 1 revolution/block!

**Solution1**: Skip sector positioning (“interleaving”)

Place the blocks from one file on every other block of a track: give time for processing to overlap rotation

![Screenshot 2023-07-13 at 2.01.02 PM](https://p.ipic.vip/h216ig.png)

Can be done by OS or in modern drives by the disk controller

**Solution 2**: Read ahead: read next block right after first, even if application hasn’t asked for it yet

This can be done either by OS (read ahead) 

By disk itself (track buffers) - many disk controllers have internal RAM that allows them to read a complete track

## Linux Ext2/3 Disk Layout

Disk us divided into block groups. It provides locality and each group has two block-sized bitmaps (free blocks/inodes). Block sizes are settable at format time: 1K, 2K, 4K, 8K…

The actual inode strcture is similar to 4.2 BSD.

Example: create a file1.dat under /dir1/ in Ext3

<img src="https://p.ipic.vip/9mj5ec.png" alt="image-20230713143347825" style="zoom:50%;" />

Ext3 is Ext2 with Journaling. There are several degrees of protection with comparable overhead.

### Hard Links

Hard link: 

* Mapping from name to file number in the directory structure

* First hard link to a file is made when file created

* Create extra hard links to a file with the `link()` system call

* Remove links with `unlink()` system call

When can file contents be deleted?

* When there are no more hard links to the file

* Inode maintains reference count for this purpose

### Soft Links

Soft link or Symbolic Link or Shortcut

* Directory entry contains the path and name of the file

* Map one name to another name

Contrast these two different types of directory entries:

* Normal directory entry: `<file name, file #>`

* Symbolic link: `<file name, dest. file name>`
* Unix can create soft links with `symlink` syscall

## Windows NTFS

Variable length extents rather than fixed blocks.

Everything (almost) is a sequence of \<attribute:value\> pairs (Meta-data and data)

Each entry in MFT contains metadata and:

* File’s data directly (for small files)

* A list of *extents* (start block, size) for file’s data

* For big files: pointers to other MFT entries with *more* extent lists

### Master File Table

* Database with Flexible 1KB entries for metadata/data

* Variable-sized attribute records (data or metadata)

* Extend with variable depth tree (non-resident)

### Extents – variable length contiguous regions

* Block pointers cover runs of blocks

  NTFS uses a system where a block pointer in the MFT record points to a run of blocks that constitute an extent. This is a very efficient way to represent the location of file data, especially for large files that occupy many contiguous blocks.

* Similar approach in Linux (ext4)

* File create can provide hint as to size of file

  When a new file is created, NTFS allows for a hint to be provided regarding the expected size of the file. This allows the file system to potentially allocate an appropriate number of contiguous blocks, optimizing the layout of the file on disk for better performance.

NTFS Small files: data stored with metadata

<img src="https://p.ipic.vip/c2xs5j.png" alt="image-20230713151816777" style="zoom:50%;" />

NTFS Medium File: Extents for File Data

![image-20230713151911932](https://p.ipic.vip/35mq4z.png)

NTFS Large File: Pointers to Other MFT Records

<img src="https://p.ipic.vip/s9dl0t.png" alt="image-20230713151941931" style="zoom:50%;" />

NTFS Huge, Fragmented File

<img src="https://p.ipic.vip/9j3qt3.png" alt="image-20230713152126265" style="zoom:50%;" />

### NTFS Directories

* Directories implemented as B Trees

* File's number identifies its entry in MFT

* MFT entry always has a file name attribute

  Human readable name, file number of parent dir

* Hard link? Multiple file name attributes in MFT entry

# Memory Mapped Files

Traditional I/O involves explicit transfers between buffers in process address space to/from regions of a file

* This involves multiple copies into caches in memory, plus system calls

What if we could “map” the file directly into an empty region of our address space. We implicitly “page it in” when we read it. We write it and “eventually” page it out

Executable files are treated this way when we exec the process!!

## Advantages

- Memory-mapped files allow for multiple processes to share read-only access to a common file. As a straightforward example, the C standard library (`glibc.so`) is mapped into all processes running C programs. As such, only one copy of the file needs to be loaded into physical memory, even if there are thousands of programs running.
- In some cases, memory-mapped files simplify the logic of a program by using memory-mapped I/O. Rather than using `fseek()` multiple times to jump to random file locations, the data can be accessed directly by using an index into an array.
- Memory-mapped files provide more efficient access for initial reads. When `read()` is used to access a file, the file contents are first copied from disk into the kernel’s [buffer cache](https://w3.cs.jmu.edu/kirkpams/OpenCSF/Books/csf/html/Glossary.html#term-buffer-cache). Then, the data must be copied again into the process’s user-mode memory for access. Memory-mapped files bypass the buffer cache, and the data is copied directly into the user-mode portion of memory.
- If the region is set up to be writable, memory-mapped files provide extremely fast IPC data exchange. That is, when one process writes to the region, that data is immediately accessible by the other process without having to invoke a system call. Note that setting up the regions in both processes is an expensive operation in terms of execution time; however, once the region is set up, data is exchanged immediately. [[1\]](https://w3.cs.jmu.edu/kirkpams/OpenCSF/Books/csf/html/MMap.html#f17)
- In contrast to message-passing forms of IPC (such as [pipes](https://w3.cs.jmu.edu/kirkpams/OpenCSF/Books/csf/html/Glossary.html#term-pipe)), memory-mapped files create persistent IPC. Once the data is written to the shared region, it can be repeatedly accessed by other processes. Moreover, the data will eventually be written back to the file on disk for long-term storage.

## Interface

![image-20230713154518503](https://p.ipic.vip/ri9ylu.png)

```c
#include <sys/mman.h> /* also stdio.h, stdlib.h, string.h, fcntl.h, unistd.h */

int something = 162;

int main (int argc, char *argv[]) {
  int myfd;
  char *mfile;

  printf("Data  at: %16lx\n", (long unsigned int) &something);
  printf("Heap at : %16lx\n", (long unsigned int) malloc(1));
  printf("Stack at: %16lx\n", (long unsigned int) &mfile);

  /* Open the file */
  myfd = open(argv[1], O_RDWR | O_CREAT);
  if (myfd < 0) { perror("open failed!");exit(1); }

  /* map the file */
  mfile = mmap(0, 10000, PROT_READ|PROT_WRITE, MAP_FILE|MAP_SHARED, myfd, 0);
  if (mfile == MAP_FAILED) {perror("mmap failed"); exit(1);}

  printf("mmap at : %16lx\n", (long unsigned int) mfile);

  puts(mfile);
  strcpy(mfile+20,"Let's write over it");
  close(myfd);
  return 0;
}
```

# Buffer Cache

Kernel *must* copy disk blocks to main memory to access their contents and write them back if modified

* Could be data blocks, inodes, directory contents, etc.

* Possibly dirty (modified and not written back)

**Key Idea**: Exploit locality by caching disk data in memory

* Name translations: Mapping from paths to inodes

* Disk blocks: Mapping from block address to disk content 

**Buffer Cache**: Memory used to cache kernel resources, including disk blocks and name translations

* Can contain “dirty” blocks (with modifications not on disk)

## Discussion

Implemented entirely in OS software

* Unlike memory caches and TLB

Blocks go through transitional states between free and in-use

* Being read from disk, being written to disk

* Other processes can run, etc.

Blocks are used for a variety of purposes

* inodes, data for dirs and files, freemap

* OS maintains pointers into them

Termination – e.g., process exit – open, read, write

### Replacemen policy

* Advantages:
  * Works very well for name translation
  * Works well in general as long as memory is big enough to accommodate a host’s working set of files.

* Disadvantages:

  * Fails when some application scans through file system, thereby flushing the cache with data used only once

    Example: `find . –exec grep foo {} \;`

Some systems allow applications to request other policies. Example, ‘Use Once’:File system can discard blocks as soon as they are used

### Cache Size

How much memory should the OS allocate to the buffer cache vs virtual memory?

* Too much memory to the file system cache Þ won’t be able to run many applications

* Too little memory to file system cache Þ many applications may run slowly (disk caching not effective)

* Solution: adjust boundary dynamically so that the disk access rates for paging and file access are balanced

### File System Prefetching

Read Ahead Prefetching: fetch sequential blocks early

* Key Idea: exploit fact that most common file access is sequential by prefetching subsequent disk blocks ahead of current read request

* Elevator algorithm can efficiently interleave prefetches from concurrent applications

How much to prefetch?

* Too much prefetching imposes delays on requests by other applications

* Too little prefetching causes many seeks (and rotational delays) among concurrent (which indicates a great number of processes) file requests

### Delayed Writes

Buffer cache is a writeback cache (writes are termed “Delayed Writes”)

`write()` copies data from user space to kernel buffer cache

* Quick return to user space

`read()` is fulfilled by the cache, so reads see the results of writes

* Even if the data has not reached disk

When does data from a `write` syscall finally reach disk?

* When the buffer cache is full (e.g., we need to evict something)

* When the buffer cache is flushed periodically (in case we crash)

Advantages:

* Performance advantage: return to user quickly without writing to disk!

* Disk scheduler can efficiently order lots of requests

  Elevator Algorithm can rearrange writes to avoid random seeks

* Delay block allocation: 

  May be able to allocate multiple blocks at same time for file, keep them contiguous

* Some files never actually make it all the way to disk

  Many short-lived files!

# "ilities"

**Availability**: the probability that the system can accept and process requests

* Measured in “nines” of probability: e.g. 99.9% probability is “3-nines of availability”

* Key idea here is independence of failures

**Durability**: the ability of a system to recover data despite faults

* This idea is fault tolerance applied to data

* Doesn’t necessarily imply availability: information on pyramids was very durable, but could not be accessed until discovery of Rosetta Stone

**Reliability**: the ability of a system or component to perform its required functions under stated conditions for a specified period of time (IEEE definition)

* Usually stronger than simply availability: means that the system is not only “up”, but also working correctly

* Includes availability, security, fault tolerance/durability

* Must make sure data survives system crashes, disk crashes, other problems

Disk blocks contain Reed-Solomon **error correcting codes** (ECC) to deal with small defects in disk drive

* Can allow recovery of data from small media defects 

Make sure writes **survive in short term**

* Either abandon delayed writes or

* Use special, battery-backed RAM (called non-volatile RAM or NVRAM) for dirty blocks in buffer cache

Make sure that data **survives in long term**

* Need to replicate! More than one copy of data!

* Important element: independence of failure

  Could put copies on one disk, but if disk head fails…

  Could put copies on different disks, but if server fails…

  Could put copies on different servers, but if building is struck by lightning…. 

  Could put copies on servers in different continents…

## RAID 1: Disk Mirroring/Shadowing

![image-20230713165521685](https://p.ipic.vip/5kvvfn.png)

Each disk is fully duplicated onto its “shadow”

* For high I/O rate (Reads may be optimized - Can have two independent reads to same data), high availability environments

* Most expensive solution: 100% capacity overhead

## RAID 5+: High I/O Rate Parity

<img src="https://p.ipic.vip/qth5bi.png" alt="image-20230713165705733" style="zoom:50%;" />

Data stripped across multiple disks

* Successive blocks stored on successive (non-parity) disks

* Increased bandwidth over single disk

Parity block (in green) constructed by XORing data bocks in stripe
$$
P0=D0\oplus D1\oplus D2 \oplus D3
$$
Can destroy any one disk and still reconstruct data

**RAID 6 and Other Erasure Codes**

In general: RAIDX is an “erasure code”

* Must have ability to know which disks are bad

* Treat missing disk as an “Erasure”

Today, disks so big that: RAID 5 not sufficient!

* Time to repair disk sooooo long, another disk might fail in process!

* “RAID 6” – allow 2 disks in replication stripe to fail

* Requires more complex erasure code, such as EVENODD code

## More General Reliability Solutions

**Use Transactions for atomic updates**: Ensure that multiple related updates are performed atomically (i.e., if a crash occurs in the middle, the state of the systems reflects either all or none of the updates)

Most modern file systems use transactions internally to update filesystem structures and metadata. Many applications implement their own transactions

**Provide Redundancy for media failures**

* Redundant representation on media (Error Correcting Codes)

* Replication across media (e.g., RAID disk array)

### Transactions

Closely related to critical sections for manipulating shared data structures

They extend concept of atomic update from memory to stable storage

* Multiple updates to various persistent data structures should be atomic.

Many ad-hoc approaches

* FFS carefully ordered the sequence of updates so that if a crash occurred while manipulating directory or inodes the disk scan on reboot would detect and recover the error (fsck)

* Applications use temporary files and rename 

**Definition**: a *transaction* is an atomic sequence of reads and writes that takes the system from consistent state to another.

* Begin a transaction – get transaction id

* Do a bunch of updates

  >  If any fail along the way, roll-back
  >
  > Or, if any conflicts with other transactions, roll-back

* Commit the transaction

We achieve better reliability through use of log. Changes are treated as transactions 

A transaction is committed once it is written to the log

* Data are forced to disk for reliability

  > On memory it will lose once crashed

* The process can be accelerated with NVRAM

  > This is like having a special pen that writes both on your assignment and on the log at the same time, making the whole process faster.

Although File system may not be updated immediately, data are preserved in the log.

**Difference** between “Log Structured” and “Journaled”

* In a Log Structured filesystem, data stays in log form

* In a Journaled filesystem, Log used for recovery

**Journaling File Systems**

Don’t modify data structures on disk directly

Write each update as transaction recorded in a log

* Commonly called a journal or intention list

* Also maintained on disk (allocate blocks for it when formatting)

Once changes are in the log, they can be safely applied to file system 

* e.g. modify inode pointers and directory mapping

Garbage collection: once a change is applied, remove its entry from the log

**Log Structured File Systems**

The log is the storage.

<img src="https://p.ipic.vip/t8yjuw.png" alt="image-20230713232056026" style="zoom:50%;" />

<img src="https://p.ipic.vip/5h47fm.png" alt="image-20230713232111927" style="zoom:50%;" />

One continuous sequence of blocks that wrap around whole disk. Inodes put into log when changed and point to new data in the log

Large, important portion of the log is cached in memory. Relies on Buffer Cache to make reading fast

There is a gabage collection process to obtain free space.
