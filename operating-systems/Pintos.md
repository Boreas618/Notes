# Loading

The loader is in `threads/loader.S`

**The PC BIOS loads the loader from the first sector of the first hard disk (MBR master boot record) into memory.** 

**The loader finds the kernel by reading the partition table** on each hard disk and finding the bootable partition of the type used for a Pintos kernel. Then the kernel is loaded into the memory and execute.

PC conventions reserve 64 bytes of the MBR for the partition table, and Pintos uses about 128 additional bytes for kernel command-line arguments. This leaves **a little over 300 bytes for the loader's own code**.

# Low-level Kernel Initialization

The kernel’s entry point is `start()` in `threads/start.S`. The job of this code is to switch the CPU from legacy 16-bit **"real mode"** into the 32-bit **"protected mode"** used by all modern 80x86 operating systems.

The startup code's first task is actually to **obtain the machine's memory size**, by asking the BIOS for the PC's memory size. The memory size is stored in `init_ram_pages` in pages.

The first part of CPU initialization is to **enable the A20 line**, that is, the CPU's address line numbered 20. For historical reasons, PCs boot with this address line fixed at 0, which means that attempts to access memory beyond the first 1 MB (2 raised to the 20th power) will fail. Pintos wants to access more memory than this, so we have to enable it.

Next, the loader **creates a basic page table.**

This page table maps the 64 MB at the base of virtual memory (starting at virtual address 0) directly to the identical physical addresses.

It also maps the same physical memory starting at virtual address `LOADER_PHYS_BASE`, which defaults to `0xc0000000` (3 GB).

We initialize the page table, then enable protected mode and paging, and set up the segment registers by loading the CPU's control registers. In protected mode, we disable interrupts because we are not yet able to handle them.

Then we call `pintos_init()`.

# High-level Kernel Initialization

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

# PC BOOTSTRAP

**The process of loading the operating system into memory for running after a PC is powered on is commonly known as *bootstrapping.***

- Bootloader can be stored in memory, floopy disk and partitioned computer mass storage devices like fixed disks or removable drives.

  IA32 bootloaders generally have to fit within 512 bytes in memory for a partition or floppy disk bootloader. For a bootloader in the Master Boot Record (MBR), it has to fit in an even smaller 436 bytes.

The BIOS and bootloader should be written in assembly.

## The PC’s physical address space

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

# BOOTLOADER

Floppy and hard disks for PCs are divided into 512-byte regions called sectors.

If the disk is bootable, the first sector is called *the boot sector*, since this is where the boot loader code resides.

When the BIOS finds a bootable floppy or hard disk, it loads the 512-byte boot sector into memory at physical addresses 0x7c00 through 0x7dff, and then uses a `jmp`instruction to set the CS:IP to `0000:7c00`, passing control to the boot loader.

IA32 bootloaders have the unenviable task of running in **real-addressing mode** (also known as "real mode"). In this mode, **the segment registers are utilized** to compute memory addresses using the following formula: **`address = 16 * segment + offset`**.

The code segment CS is used for instruction execution. For instance, if the BIOS jumps to `0x0000:7c00`, the corresponding physical address would be `16 * 0 + 7c00 = 7c00`.

Other segment registers include **SS for the stack segment, DS for the data segment, and ES for data movement**.

It should be noted that each segment is 64KiB in size. Since bootloaders often need to load kernels larger than 64KiB, they must carefully utilize the segment registers.

# PHYSICAL MEMORY MAP

![Screenshot 2023-04-18 at 3.30.58 AM.png](https://p.ipic.vip/gw7a6d.png)