# File Sytem Concepts

Two types of files:

* Low-level, Byte-Stream Files

* Structure Files （**Example**: ASCII character files）

  UNIX file manager does not distinguish text files from other byte streams

A **structured sequential file** is a named sequence of logical records, indexed by the nonnegative integers.

## File Organization

* **Pile Files**: Data are collected in the order they arrive. Records may have different fields, or similar

  fields in different orders. Each field should be **self-described**,including a field name as well as a value. There is no structure to the pile file.

* **Sequential Files**: A fixed format is used for records. All records are of the **same length**, consisting of the same number of **fixed-length fields in a particular order**. The key field **uniquely identifies** the record. Records are stored in key sequence.

  Typically used in batch applications (i.e. processing all of records of the file). Not optimal for manipulating single records (because the applications should go over the file to find the target records).

  Improve performance by keeping a log file or transaction file which accumulates single operations.

* **Indexed Sequential File**: Records are organized in sequence based on a key field.

  An index to the file to support random access: the index is searched to find **highest key value** that is equal or less than the desired key value.

  An **overflow file** to store the additions temporarily.

  <img src="https://p.ipic.vip/a755dn.png" alt="Screenshot 2023-12-11 at 1.57.38 AM" style="zoom:50%;" />

* **Indexed File**: Uses multiple indexes for different fields. One index for each field that may be the subject of a search. Records are accessed only through their indexes. Variable-length records can be employed.

  Indexed files are used mostly in applications where timeliness of information is critical and where data are rarely processed exhaustively.

* **Direct or Hashed File**: A key field is required in each record .

  <img src="https://p.ipic.vip/bhhlcn.png" alt="Screenshot 2023-12-11 at 2.02.38 AM" style="zoom:33%;" />

## File Access

* **Sequential access**: A read operation reads the next portion of the file and automatically advances a file pointer. A write operation appends t**o the end of the file** and advances to the end of the newly written material.
* **Random access**: use block number as the parameter.

## File Directories

Disks are split into one or more **partition**s. Partitions are also known as **minidisk**s in the IBM world or **volume**s in the PC and Macintosh arenas. Each partition contains information about files within it. This information is kept in entries in a **device directory** or volume table of contents.

<img src="https://p.ipic.vip/z9j38f.png" alt="Screenshot 2023-12-11 at 2.28.35 AM" style="zoom:33%;" />

**MBR (Master Boot Record)**: The sector 0 of the disk; used to boot the computer.

**Partition table**: contained at the end of the MBR; the table gives the starting and ending addresses of each partition; one of the partition in the table is marked as **active**.

The first block of the active partition is called the **boot block**, which loads the OS contained in that partition.

**Superblock** is read into memory when the computer is booted, and it contains all the key parameters about the file system such as file system type, the number of blocks in the file system, and other key administrative information.

Typically, an interactive user or a process has associated with a current directory, often referred to as **working directory**.

### Logical Structure of a Direcrtory

* **Single-Level Directory**: the simplest directory structure is a list of entries, one for each file.

  There's only one root directory. Cannot group files. Cannot use the same name for two different files.

* **Two-level Directory**: one separate directory for each user, and a master directory.

* **Tree-Structured Directories**: 

  > UNIX: `/usr/ast/mailbox`
  >
  > Windows: `\usr\ast\mailbox`

* **Acyclic-Graph Directories**: Two techniques: hard linking and symbolic linking (or soft linking)

  For hard linking, the disk addresses of files should not be present in directory entries. Instead, disk blocks are listed in a little data structure associated with the file itself.

  **Limitations of hard linking**: Cannot create a hard link to a directory; Cannot hard link to files in other disk partitions (because inode numbers are only unique within a partition).

  For symbolic linking, user B links to one of user C’s files by having the system create a new file of type LINK, and entering that file in B’s directory. The new file contains just the path name of the file to which it is linked.

  **Limitations of symbolic linking**: Time overhead of parsing the address; An extra inode.

### Directory Implementation

* **Linear List**: a linear list of file names with pointers to the data blocks.

  How to reuse a directory entry whose corresponding file has been just deleted?

  1. Mark the entry as unused (by assigning it a special name, or with a used-unused bit in each entry)
  2. Attach it to a list of free directory entries.
  3. To copy the last entry in the directory into the freed location, and to decrease the length of the directory.

* **Hash Table**: a linear list stores the directory entries, but a hash data structure is also used.

## File Protection

**Access Rights**: None $\rightarrow$ Knowledge $\rightarrow$ Execution $\rightarrow$ Reading $\rightarrow$ Appending $\rightarrow$ Updating $\rightarrow$ Changing protection $\rightarrow$ Deletion

**Domain**: A domain is a set of `(object,rights)` pairs. In UNIX, the domain of a process is defined by its UID and GID.

**Access Control Lists**: Each object is associated with an (ordered) list containing all the domains that may access the object and how.

**Capabilities**: Each process is associated with a list of objects that may be accessed, along with an indication of which operations are permitted on each.

To condense the length of the access control lists, many systems recognize three classification of users in connection with each file:

* Owner: the user who created the file

- Group or work group: A set of users who are sharing the file and need similar access
- Universe: all other users in the system constitute the universe

## Record Blocking

For I/O to be performed, records must be organized as blocks.

Given the size of block, there are three methods of blocking:

- **Fixed blocking**: Fixed-length records are used, and an integral number of records are stored in a block.
- **Variable-length spanned blocking**: Variable-length records are used, and are packed into blocks with no unused space. Some records must span two blocks, with the continuation indicated by a pointer to the successor block.
- **Variable-length unspanned blocking**: Variable-length records are used, but spanning is not employed. There is wasted space in most blocks because the remainder of a block is not enough to hold the next record.

## Secondary Storage Management

On secondary storage, a file consists of a collection of blocks.

Three file allocaiton methods:

- Contiguous allocation
- Chained allocation
- Indexed allocation

### Contiguous allocation

With contiguous allocation, a single contiguous set of blocks is allocated to a file at the time of file creation.

Only a single entry in the file allocation table. Starting block and length of the file. This is a preallocation strategy, using variable-size portions. 

Accessing a file that has been allocated contiguously is easy for both sequential access and direct access.

Contiguous allocation is feasible and in fact widely used in one situation: On CD-ROM.

<img src="https://p.ipic.vip/wuegcm.png" alt="Screenshot 2023-12-16 at 5.15.29 AM" style="zoom:50%;" />

### Chained Allocation

With chained allocation, allocation is on an individual block basis.

No external fragmentation, and no size declaration problem.

**Disadvantage 1**: It can be used effectively only for sequential-access files. To find the i-th block of a file, we must start at the beginning of the file, and follow the pointers until we get to the ith block.

**Disadvantage 2**: Consolidation is required to improve the performance of sequential access. The blocks of a file is scattered all over the disk. If several blocks of a file is to be brought in,it is required to access different parts of the disk (multiple disk seeks).

**Disadvantage 3**:The pointers require some disk space.

**Disadvantage4**: Reliability.

<img src="https://p.ipic.vip/tbkfaq.png" alt="Screenshot 2023-12-16 at 5.20.47 AM" style="zoom:50%;" />

> **FAT: File Allocation Table**
>
> A section of disk at the beginning of each partition is set aside to contain the table.
>
> - The table has one entry for each disk block, and is indexed by block number.
> - The directory entry contains the block number of the first block of the file.
> - The table entry indexed by that block number then contains the block number of the next block in the file
> - This chain continues until the last block, which has a special end-of-file value as the table entry.
> - The unused blocks are indicated by a 0 table value
>
> **Trade-offs**: The disk head must move to the start of the partition to read the FAT and find the location of the block in question. Then move to the location of the block itself. The benefit is that random access time is improved. Because the disk head can find the location of any block by reading the information in the FAT.

### Indexed Allocation

Chained allocation solves the external fragmentation and size-declaration problems of contiguous allcoation. However, it can not support efficient direct access. Indexed allocation solves this problem by bringing all the pointers together into one location: the index block

File allocation table contains a separate one-level index for each file. The index has one entry for each portion allocatedto the file. 

Indexed allocation does suffer from wasted space. The pointer overhead of the index block is generally greater than the pointer overhead of chained allocation. An entire index block must be allocated even if the file only occupies one or two block.

<img src="https://p.ipic.vip/x8eh0n.png" alt="Screenshot 2023-12-16 at 5.43.10 AM" style="zoom: 33%;" />

## Free Space Management

* Bit tables (Bitmap, or Bit Vector)
* LinkedList
* FreeBlockList
* Grouping
* Chained free portions

### Bitmap

We cannot afford to traverse the bitmap. Solution: maintain auxiliary data structures that summarize the contents of subranges of the bit table.

### Linked List

This scheme is not efficient. To traverse the list, we must read each block, which requires substantial I/O time on HDDs.

### Free Block List

To store the free block numbers in a reserved region on disk. If a 32-bit block number is used, then the space penalty is 4 bytes for every 512-byte block.

### Grouping

We could store the addresses of n free blocks in the first free block. The first $n-1$ of these blocks are actually free. The last block contains the addresses of another $n$ free blocks, and so on.

The addresses of a large number of free blocks can be found quickly.

### Chained Free Portions

The free portions may be chained together by using a pointer and length value in each free portion.

This method is suited to all of the file allocation methods:

* If allocation is a block at a time, simply choose the free block at the head of the chain and adjust the first pointer or length value.
* If allocation is by variable-length portion, a first-fit algorithm may be used.

## File System Consistency

This problem is especially critical if the blocks that are not written out includes file allocation table, disk allocation table, etc.

### Block Consistency

To check for block consistency, two tables are constructed.

Each table contains a counter for each block, initially set to 0.

- The counters in the first table keep track of how many times each block is present in a certain file.

* The counters in the second table record how often each block is present in the free list (or the bitmap of free blocks)

If the file system is consistent, each block will have a 1 either in the first table or in the second table.

**Missing block**: A block does not present in either table. **Solution**: add the block to the free list.

**Duplicate block**: A block occurs twice or more times in the freelist. **Solution**: rebuild the free list.

**Duplicate data block**: The same data block is present in two or more files. **Solution**: allocate a free block, copy the contents of the data block into it, and insert the copy into one of the files.

<img src="https://p.ipic.vip/hl8o2f.png" alt="Screenshot 2023-12-16 at 6.10.52 AM" style="zoom:50%;" />

### File Consistency

 It counts file usages per directory by recursively traversing the file system tree. For every i-node in each directory, it increments the file's usage count, considering hard links. Symbolic links are excluded from this count.

After the check, it generates a list indexed by i-node numbers to determine how many directories contain each file. It then compares these numbers with the link counts stored in the i-nodes. In a consistent file system, these counts match. However, two types of errors can occur: 

1. If the link count is higher than the number of directory entries, it's a non-serious error that should be fixed by setting the i-node's link count to the correct value.

2. The more serious error occurs when the i-node indicates one link, but multiple directory entries are linked to the file. Removing one directory entry will reduce the i-node count to zero, marking the file system as unused and releasing its blocks. This can lead to problems. To resolve this, set the i-node's link count to the actual number of directory entries.

## Unix File System

Four types of files are distinguished:

- **Ordinary**: Files that contain information entered in them by a user, an user program, or a system utility program

- **Directory**: Contains a list of file names plus pointers to associated i-nodes (index nodes).

  Directories are actually ordinary files with special write protection privileges so that only the file system can write into them, while read access is available to user programs.

- **Special**: Used to access peripheral devices. Each I/O device is associated with a special file.

- **Named**: Name pipes, discussed in Section 6.7

<img src="https://p.ipic.vip/fc7ut2.png" alt="Screenshot 2023-12-16 at 6.19.15 AM" style="zoom:50%;" />

The length of a block is 1 Kbyte in UNIX system, and each block can hold a total of 256 block addresses.

<img src="https://p.ipic.vip/3skzkj.png" alt="Screenshot 2023-12-16 at 6.20.27 AM" style="zoom:50%;" />

