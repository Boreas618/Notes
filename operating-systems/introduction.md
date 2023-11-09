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

# OS Booting

When a computer is powered on, it undergoes a series of steps to initialize the hardware components and load the operating system. This process is known as OS booting. In this guide, we will use Linux as an example to explain this process in detail.

## BIOS/UEFI Initialization Phase

BIOS (Basic Input/Output System) and UEFI (Unified Extensible Firmware Interface) are firmware interfaces that play a critical role in the boot process. They initialize the necessary hardware components and set the stage for the operating system to take over. Here's a step-by-step breakdown of the process:

### Power-On and Initiation of Firmware

Upon powering on the computer, the installed firmware—either BIOS or UEFI—initiates. This firmware acts as a bridge between the computer's hardware and the operating system, facilitating communication between the two.

### POST (Power-On Self-Test)

The firmware conducts the POST, a diagnostic procedure that checks the status and functionality of critical hardware components such as the memory, processor, and storage devices. This step ensures that the system is free of hardware issues before proceeding to the boot process.

### Boot Device Selection and Boot Record Reading

* BIOS Systems

  1. **Boot Device Selection**: BIOS identifies the boot device from a predefined list stored in the CMOS memory, which retains information about the system's hardware configuration.

  2. **Master Boot Record (MBR) Reading**: After the BIOS completes the POST and identifies the bootable device, it reads the MBR located on the primary storage device. The MBR contains a partition table that indicates the active partition where the operating system resides, as well as the first stage of the bootloader.
  3. **Partition Boot Record (PBR) Reading**: Following the MBR reading, the first stage of the bootloader initiates and accesses the PBR of the active partition. The PBR houses the second stage of the bootloader, which is necessary for initiating the OS loading process. This stage of the bootloader is more complex and is responsible for loading the essential files to start the operating system.

* UEFI Systems
  1. **EFI System Partition (ESP) Identification**: UEFI systems identify the ESP on the storage device, a special partition that contains UEFI bootloaders and utility applications essential for the boot process, including the OS bootloader.

## Bootloader Phase

The bootloader is the first software to run when a system is powered on. Its main function is to load the system's kernel into memory, initiating the startup process. In Linux, popular bootloaders include GRUB and LILO.

### GRUB (Grand Unified Bootloader)

GRUB is a versatile bootloader with support for multiple platforms and operating systems. It offers a powerful and flexible configuration. Here's a breakdown of its stages:

#### 1. Initial Stage (MBR or UEFI)

- MBR Stage

  - Stage 1

    Stored within the MBR, this stage is limited to 512 bytes and primarily locates and loads the next stage of the bootloader, either by:

    - Loading **Stage 1.5**: Found in the first 32KB following the MBR. This stage understands the filesystem layout and can directly load Stage 2.
    - Directly loading **Stage 2**: In setups where Stage 1.5 is not present.
  
- UEFI Stage

  - For UEFI systems, the firmware loads the EFI binary provided by GRUB, typically stored on the ESP. UEFI offers several advantages over BIOS, including faster boot times, enhanced security features, support for larger drives, and a GUI interface with mouse support.

#### 2. Main Bootloader Stage (Stage 2)

- **Configuration Loading**: GRUB locates its configuration file, usually found at `/boot/grub/grub.cfg`, which contains menu entries and various boot options.
- **Menu Presentation**: The GRUB menu is displayed, allowing users to select an operating system or kernel version to boot. If no selection is made within a specified timeout, the default entry is booted.
- **Kernel Loading**: GRUB loads the selected Linux kernel and initial ramdisk (if any) into memory. Control is then handed over to the kernel, and the OS begins to boot.

By following these steps, both BIOS and UEFI systems successfully initialize the hardware components and facilitate a smooth transition to the operating system, ensuring a successful boot process.

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

## PC Bootstrap

**The process of loading the operating system into memory for running after a PC is powered on is commonly known as *bootstrapping.***

Bootloader can be stored in memory, floopy disk and partitioned computer mass storage devices like fixed disks or removable drives.

IA32 bootloaders generally have to fit within 512 bytes in memory for a partition or floppy disk bootloader. For a bootloader in the Master Boot Record (MBR), it has to fit in an even smaller 436 bytes.

The BIOS and bootloader should be written in assembly.

### The PC’s Physical Address Space

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

## Bootloader

Floppy and hard disks for PCs are divided into 512-byte regions called sectors.

If the disk is bootable, the first sector is called *the boot sector*, since this is where the boot loader code resides.

When the BIOS finds a bootable floppy or hard disk, it loads the 512-byte boot sector into memory at physical addresses 0x7c00 through 0x7dff, and then uses a `jmp`instruction to set the CS:IP to `0000:7c00`, passing control to the boot loader.

IA32 bootloaders have the unenviable task of running in **real-addressing mode** (also known as "real mode"). In this mode, **the segment registers are utilized** to compute memory addresses using the following formula: **`address = 16 * segment + offset`**.

The code segment CS is used for instruction execution. For instance, if the BIOS jumps to `0x0000:7c00`, the corresponding physical address would be `16 * 0 + 7c00 = 7c00`.

Other segment registers include **SS for the stack segment, DS for the data segment, and ES for data movement**.

It should be noted that each segment is 64KiB in size. Since bootloaders often need to load kernels larger than 64KiB, they must carefully utilize the segment registers.

## Physical Memory Map

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
