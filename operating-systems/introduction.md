# OS Introduction

## Features

* Concurrency 并发

  Difference between **concurrency** and **parallelism**: the former is about more than one events occur within a certain time range and the latter is about at the same time.

* Sharing 共享

  **Mutal exclusion** and **Simultaneous access**.

* Virtual 虚拟

  The feeling of illusion. Like the virtual memory mechanism makes the program have the illusion that the program can access the whole memory space.

* Asynchronism 异步

  Asynchronism allows a system to initiate a task without waiting for it to complete. Once the task is initiated, the system can proceed to other tasks, returning to the original task once it is completed.

## Interfaces

* Command Interfaces
  * Online Command Interface: user input commands. Preemptive/RT
  * Offline Command Interface: batch processing command interface. Batch processing system
* Programming Interfaces (Generalized Instruction)

## **Kernel Components**

* Timer Management
* Interrupt Mechanism
* Atomic Operation
* System Data Structures

## Evaluation Criteria of operating systems

*   Reliability

    Avalibility

    MTTF (mean time to faliure)

    MTTR (mean time to repair)
*   Security

    Enforcement: how the operating system ensures that only permitted actions are allowed
*   Portability

    A portable abstraction doesn’t change as the hardware changes

    AMI (abstract machine interface): the interface provided by operating systems to applications

    API (applicaiton programming interface): a key part of AMI

    HAL (hardware abstraction layer): an operating system implement independent of the hardware details
*   Performance

    Overhead: the added resource cost of implementing an abstraction

    One way to measure eﬃciency (or inversely, overhead) is the degree to which the abstraction impedes application performance.

    Throughput: the rate at which a group of tasks can be completed

    Predictability: whther the system’s response time or other performance metric is consistent over time

    A proprietary system vs. an open system

Modern operating system: time-sharing

# OS Booting

Use Linux as an example.

## **BIOS/UEFI Initialization Phase**

**BIOS (Basic Input/Output System)** and **UEFI (Unified Extensible Firmware Interface)** serve as the cornerstone firmware interfaces in computer systems, playing the pivotal role of bootstrapping the computer by initializing all essential hardware components.

Upon powering up the computer, the firmware (either BIOS or UEFI) begins its process, with the **POST (Power-On Self-Test)** being the first step. POST is a diagnostic testing sequence responsible for conducting preliminary hardware checks. It ensures that all critical components like memory, processor, and storage devices are functioning properly. Once POST is successfully completed, the system then looks for a bootloader to initiate the operating system loading process.

For systems with a BIOS, the firmware checks the **Master Boot Record (MBR)** on the primary storage device. The MBR contains a partition table which identifies the active partition. The BIOS then reads the **Partition Boot Record (PBR)** of the active partition, which houses the bootloader necessary for loading the operating system.

In contrast, modern systems with UEFI don't rely on the MBR. Instead, they look for the **EFI System Partition (ESP)** on the storage device. The ESP is a special partition that stores UEFI bootloaders and other utility applications necessary for the boot process, including the bootloader for the operating system.

## **Bootloader**

Its primary role is to load the system's kernel into memory so that the system can start. In the Linux world, the bootloader is the first software that runs when a system is powered on. In the context of Linux, there are several bootloaders, with GRUB and LILO being among the most popular.

### **GRUB (Grand Unified Bootloader)**

GRUB is a versatile bootloader with multi-platform support. It can boot various operating systems, and its configuration is both powerful and flexible. Its stages are as follows:

#### **MBR Stage**:

- **GRUB Stage 1**: If your system is set up using the MBR (Master Boot Record) partitioning scheme, GRUB's first stage is stored within the MBR. This is limited to a mere 512 bytes. Due to its small size, its primary task is to find and load the next stage of the bootloader. It does this by either:
  - Loading **Stage 1.5**: Found in the first 32KB following the MBR on the storage device. This space is generally unallocated and free. Stage 1.5 knows the filesystem layout and can directly load Stage 2 from it.
  - Directly loading **Stage 2**: In some setups where Stage 1.5 is not present.

#### **UEFI Stage**:

Modern systems, especially those post-2010, often come with UEFI (Unified Extensible Firmware Interface) instead of the legacy BIOS system. UEFI has many advantages over BIOS, such as:

- Faster boot times
- Better security features
- Native support for larger drives
- A GUI interface and mouse support

For systems with UEFI: The UEFI firmware loads the EFI binary that GRUB provides. This binary is typically stored on the EFI system partition, a small, separate partition that contains boot-related files.

#### **Stage 2**:

Regardless of whether the system uses MBR or UEFI, Stage 2 is common:

- **Configuration Loading**: At this stage, GRUB looks for its configuration file, usually located at `/boot/grub/grub.cfg`. This file contains the menu entries and various boot options.
- **Menu Presentation**: The user sees the familiar GRUB menu, where they can choose which operating system or kernel version to boot. If no choice is made within a specified timeout, the default entry is booted.
- **Kernel Loading**: Upon the user's selection or after the timeout, GRUB loads the selected Linux kernel and initial ramdisk (if any) into memory. Once loaded, control is transferred to the kernel, and the actual OS starts booting.

### **LILO (LInux LOader)**

Though not as popular as GRUB in recent distributions, LILO has its historical significance. Unlike GRUB, which is more versatile and can understand filesystems, LILO relies on block mapping to boot the kernel. This means each time you change the configuration, LILO needs to be rewritten to the MBR. The simplicity of LILO, however, made it a preferred choice for many Linux enthusiasts in the earlier days.

## **Kernel Loading Phase**

This process can be broken down into several stages:

**1. Decompression**: Most modern operating systems store their kernel image in a compressed format to reduce storage space requirements. The Linux kernel image is commonly referred to as `vmlinuz`. Before the kernel can be executed, this compressed image must be decompressed.

**2. Initialization**: Once decompressed, the kernel proceeds with the initialization tasks. This involves:

- **Hardware Configuration**: Setting up necessary hardware components and establishing communication protocols.
  1. `setup_arch()`: This function does much of the architecture-specific setup, such as querying the BIOS for hardware parameters, setting up memory regions, and other architecture-specific details.
  2. `boot_cpu_init()`: Marks the current CPU (the boot CPU) as online.
  3. `page_address_init()`: If the system supports high memory configurations, this function initializes the high memory area.
  4. `early_trap_init()`: Sets up any early trap handlers.
  5. `setup_per_cpu_areas()`: Initializes per-CPU areas.
  6. `setup_traps()`: This function installs trap (exception) handlers, such as those for handling hardware interruptions.
  7. `init_IRQ()`: Initialization of the Interrupt Controller and the architecture-specific interrupt handling mechanisms.
  8. `sched_init()`: Initializes the scheduler.
  9. `timekeeping_init()`: Initializes the timekeeping system.
- **System Structures**: Initializing essential data structures required for managing processes, memory, and other system resources.
  1. `mm_init()`: Initializes the memory management infrastructure.
     - `mem_init()`: Clears the free page list and then adds free pages to it.
     - `kmem_cache_init()`: Initializes the memory allocator (SLAB allocator).
     - `percpu_init_late()`: Completes per-CPU area initialization.
     - `vfs_caches_init()`: Initializes the caches used by the Virtual File System.
  2. `cred_init()`: Initializes the credential subsystem.
  3. `fork_init()`: Allocates memory required for the process descriptor array.
  4. `proc_caches_init()`: Initializes caches for processes and files.
  5. `buffer_init()`: Buffer head structures are initialized, which are used by block devices.
  6. `key_init()`: Initializes the key management system.
  7. `security_init()`: Calls into the Linux Security Modules (LSM) framework for any security-related initializations.
- **RAM Disk Identification**: After setting up the basic structures, the kernel searches for an initial RAM disk, often referred to as Initramfs.
  1. `early_initrd()`: If the kernel was booted with an initial RAM disk (initrd), this function identifies its location in memory.
  2. `populate_rootfs()`: If `CONFIG_BLK_DEV_INITRD` is set, the initial RAM disk (initramfs) is extracted into a temporary filesystem. This temporary filesystem becomes the root filesystem until the real root filesystem can be mounted.
  3. `free_initrd_mem()`: Once the contents of the initrd are no longer needed (because they have been copied to the temporary root filesystem or because they weren't used), its memory is freed.

**3. Initramfs (Initial RAM FileSystem)**: Initramfs plays a vital role in the early boot process. Essentially, it's a simple in-memory file system. This temporary root file system provides the kernel with a set of preliminary tools and utilities. By using Initramfs, the Linux kernel becomes more modular and can boot without relying heavily on specific device drivers or intricate configurations. Once the necessary drivers are loaded and the real root file system is mounted, Initramfs is typically discarded.

**4. Execution**: With system initialization complete, the kernel is ready to launch its first userspace process. This process is crucial for managing subsequent system processes and is known as the `init` process. While `init` has been the traditional first process in UNIX-like systems, modern Linux distributions have widely adopted `systemd` as the `init` system due to its more efficient and flexible service management capabilities.

# Pintos

## Loading

The loader is in `threads/loader.S`

**The PC BIOS loads the loader from the first sector of the first hard disk (MBR master boot record) into memory.** 

MBR is comprised of partition table and boot loader.

**The loader finds the kernel by reading the partition table** on each hard disk and finding the bootable partition of the type used for a Pintos kernel. Then the kernel is loaded into the memory and execute.

PC conventions reserve 64 bytes of the MBR for the partition table, and Pintos uses about 128 additional bytes for kernel command-line arguments. This leaves **a little over 300 bytes for the loader's own code**.

## Low-level Kernel Initialization

The kernel’s entry point is `start()` in `threads/start.S`. The job of this code is to switch the CPU from legacy 16-bit **"real mode"** into the 32-bit **"protected mode"** used by all modern 80x86 operating systems.

The startup code's first task is actually to **obtain the machine's memory size**, by asking the BIOS for the PC's memory size. The memory size is stored in `init_ram_pages` in pages.

The first part of CPU initialization is to **enable the A20 line**, that is, the CPU's address line numbered 20. For historical reasons, PCs boot with this address line fixed at 0, which means that attempts to access memory beyond the first 1 MB (2 raised to the 20th power) will fail. Pintos wants to access more memory than this, so we have to enable it.

Next, the loader **creates a basic page table.**

This page table maps the 64 MB at the base of virtual memory (starting at virtual address 0) directly to the identical physical addresses.

It also maps the same physical memory starting at virtual address `LOADER_PHYS_BASE`, which defaults to `0xc0000000` (3 GB).

We initialize the page table, then enable protected mode and paging, and set up the segment registers by loading the CPU's control registers. In protected mode, we disable interrupts because we are not yet able to handle them.

Then we call `pintos_init()`.

## High-level Kernel Initialization

In `pintos_init()`

1. call `bss_init()`. In most C implementations, whenever you declare a variable outside a function without providing an initializer, that variable goes into the BSS.

2. call `read_command_line()` to break the kernel command line into arguments, then `parse_options()` into read any options at the beginning of the command line.

3. call `thread_init()` initializes the thread system.

4. initialize the console and print a startup message to the console.

5. initialize the kernel’s memory system.

   - **`palloc_init()`** sets up the kernel page allocator, which doles out memory one or more pages at a time
   - **`malloc_init()`** sets up the allocator that handles allocations of arbitrary-size blocks of memory
   - **`paging_init()`** sets up a page table for the kernel

6. initializes the interrupt system.

   - **`intr_init()`** sets up the CPU's *interrupt descriptor table* (IDT) to ready it for interrupt handling

   - **`timer_init()`** and **`kbd_init()`** prepare for handling timer interrupts and keyboard interrupts, respectively.

   - **`input_init()`** sets up to merge serial and keyboard input into one stream.

     In projects 2 and later, we also prepare to handle interrupts caused by user programs using **`exception_init()`** and **`syscall_init()`**.

7. start the scheduler with `thread_start()`, which creates the idle thread and enables interrupts.

8. `serial_init_queue()` switch to interrupt-driven serial port I/O mode.

9. `timer_calibrate()` calibrates the timer for accurate short delays.

10. If the file system is compiled in, as it will starting in project 2, we **initialize the IDE disks** with **`ide_init()`**, then **the file system** with **`filesys_init()`**.

11. Boot complete.

12. `run_actions()`parses and executes actions specified on the kernel command line.

13. Finally, if **`-q`** was specified on the kernel command line, we call `shutdown_power_off()` to terminate the machine simulator. **Otherwise**, `pintos_init()`calls **`thread_exit()`**, which allows any other running threads to continue running.

## PC BOOTSTRAP

**The process of loading the operating system into memory for running after a PC is powered on is commonly known as *bootstrapping.***

Bootloader can be stored in memory, floopy disk and partitioned computer mass storage devices like fixed disks or removable drives.

IA32 bootloaders generally have to fit within 512 bytes in memory for a partition or floppy disk bootloader. For a bootloader in the Master Boot Record (MBR), it has to fit in an even smaller 436 bytes.

The BIOS and bootloader should be written in assembly.

### The PC’s physical address space

```
	+------------------+  <- 0xFFFFFFFF (4GB)
	|      32-bit      |
	|  memory mapped   |
	|     devices      |
	|                  |
	/\/\/\/\/\/\/\/\/\/\
	/\/\/\/\/\/\/\/\/\/\
	|                  |
	|      Unused      |
	|                  |
	+------------------+  <- depends on amount of RAM
	|                  |
	|                  |
	| Extended Memory  |
	|                  |
	|                  |
	+------------------+  <- 0x00100000 (1MB)
	|     BIOS ROM     |
	+------------------+  <- 0x000F0000 (960KB)
	|  16-bit devices, |
	|  expansion ROMs  |
	+------------------+  <- 0x000C0000 (768KB)
	|   VGA Display    |
	+------------------+  <- 0x000A0000 (640KB)
	|                  |
	|    Low Memory    |
	|                  |
	+------------------+  <- 0x00000000
```

This is the scenario for the first PCs, which were based on the 16-bit Intel 8088 processor. They were only capable of addressing 1MB of physcial memory.

**The 640KB area marked "Low Memory" was the only random-access memory (RAM) that an early PC could use.**

The 384KB area from 0x000A0000 through 0x000FFFFF was reserved by the hardware for special uses such as video display buffers and firmware held in non-volatile memory. The most important part of this reserved area is the BIOS.

Nowadays, the PC architects still preserved the original layout for the low 1MB of physical address space in order to ensure *backward compatibility* with existing software.

Modern PCs therefore have a "hole" in physical memory from 0x000A0000 to 0x00100000, dividing RAM into "low" or "conventional memory" (the first 640KB) and "extended memory" (everything else).

In addition, **some space at the very top of the PC's 32-bit physical address space**, above all physical RAM, **is now commonly reserved by the BIOS for use by 32-bit PCI devices.**

## BOOTLOADER

Floppy and hard disks for PCs are divided into 512-byte regions called sectors.

If the disk is bootable, the first sector is called *the boot sector*, since this is where the boot loader code resides.

When the BIOS finds a bootable floppy or hard disk, it loads the 512-byte boot sector into memory at physical addresses 0x7c00 through 0x7dff, and then uses a `jmp`instruction to set the CS:IP to `0000:7c00`, passing control to the boot loader.

IA32 bootloaders have the unenviable task of running in **real-addressing mode** (also known as "real mode"). In this mode, **the segment registers are utilized** to compute memory addresses using the following formula: **`address = 16 * segment + offset`**.

The code segment CS is used for instruction execution. For instance, if the BIOS jumps to `0x0000:7c00`, the corresponding physical address would be `16 * 0 + 7c00 = 7c00`.

Other segment registers include **SS for the stack segment, DS for the data segment, and ES for data movement**.

It should be noted that each segment is 64KiB in size. Since bootloaders often need to load kernels larger than 64KiB, they must carefully utilize the segment registers.

## PHYSICAL MEMORY MAP

| Memory Range (0x)  | Owner    | Contents                                                     |
| ------------------ | -------- | ------------------------------------------------------------ |
| 00000000--000003ff | CPU      | Real mode interrupt table.                                   |
| 00000400--000005ff | BIOS     | Miscellaneous data area.                                     |
| 00000600--00007bff | --       | ---                                                          |
| 00007c00--00007dff | Pintos   | Loader.                                                      |
| 0000e000--0000efff | Pintos   | Stack for loader; kernel stack and `struct thread` for initial kernel thread. |
| 0000f000--0000ffff | Pintos   | Page directory for startup code.                             |
| 00010000--00020000 | Pintos   | Page tables for startup code.                                |
| 00020000--0009ffff | Pintos   | Kernel code, data, and uninitialized data segments.          |
| 000a0000--000bffff | Video    | VGA display memory.                                          |
| 000c0000--000effff | Hardware | Reserved for expansion card RAM and ROM.                     |
| 000f0000--000fffff | BIOS     | ROM BIOS.                                                    |
| 00100000--03ffffff | Pintos   | Dynamic memory allocation.                                   |
