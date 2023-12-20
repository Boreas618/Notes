# Case Study: Pintos Booting

Pintos is a simple operating system framework for the 80x86 architecture. It supports kernel threads, loading and running user programs, and a file system, but it implements all of these in a very simple way.

## Loading

The loader is in `threads/loader.S`

**The PC BIOS loads the loader from the first sector of the first hard disk (MBR master boot record) into memory.** MBR is comprised of partition table and boot loader.

**The loader finds the kernel by reading the partition table** on each hard disk and finding the bootable partition of the type used for a Pintos kernel. Then the kernel is loaded into the memory and execute.

PC conventions reserve 64 bytes of the MBR for the partition table, and Pintos uses about 128 additional bytes for kernel command-line arguments. This leaves **a little over 300 bytes for the loader's own code**.

## PC Bootstrap

**Bootstrapping**: the process of loading the operating system into memory for running after a PC is powered on.

Bootloader can be stored in memory, floopy disk and partitioned computer mass storage devices like fixed disks or removable drives.

IA32 bootloaders generally have to fit within 512 bytes in memory for a partition or floppy disk bootloader. For a bootloader in the Master Boot Record (MBR), it has to fit in an even smaller 436 bytes. The BIOS and bootloader should be written in assembly.

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

The 384KB area from `0x000A0000` through `0x000FFFFF` was reserved by the hardware for special uses such as video display buffers and firmware held in non-volatile memory. The most important part of this reserved area is the BIOS.

Nowadays, the PC architects still preserved the original layout for the low 1MB of physical address space in order to ensure *backward compatibility* with existing software.

Modern PCs therefore have a "hole" in physical memory from 0x000A0000 to 0x00100000, dividing RAM into "low" or "conventional memory" (the first 640KB) and "extended memory" (everything else).

In addition, **some space at the very top of the PC's 32-bit physical address space**, above all physical RAM, **is now commonly reserved by the BIOS for use by 32-bit PCI devices.**

## Bootloader

Floppy and hard disks for PCs are divided into 512-byte regions called sectors.

If the disk is bootable, the first sector is called the **boot sector**, since this is where the boot loader code resides.

When the BIOS finds a bootable floppy or hard disk, it loads the 512-byte boot sector into memory at physical addresses `0x7c00` through `0x7dff`, and then uses a `jmp`instruction to set the` CS:IP` to `0000:7c00`, passing control to the boot loader.

IA32 bootloaders have the unenviable task of running in **real mode** (also known as 16-bit mode). In this mode, **the segment registers are utilized** to compute memory addresses using the following formula: **`address = 16 * segment + offset`**. In later chapters, we will see more details about addressing with segment registers.

The code segment `CS` is used for instruction execution. For instance, if the BIOS jumps to `0x0000:7c00`, the corresponding physical address would be `16 * 0 + 7c00 = 7c00`. Other segment registers include **SS for the stack segment, DS for the data segment, and ES for data movement**.

It should be noted that each segment is 64KiB in size. Since bootloaders often need to load kernels larger than 64KiB, they must carefully utilize the segment registers.

## Physical Memory Map

| Memory Range (0x)       | Owner    | Contents                                                     |
| ----------------------- | -------- | ------------------------------------------------------------ |
| `00000000` -`000003ff`  | CPU      | Real mode interrupt table.                                   |
| `00000400` - `000005ff` | BIOS     | Miscellaneous data area.                                     |
| `00000600` - `00007bff` | --       | ---                                                          |
| `00007c00` - `00007dff` | Pintos   | Loader.                                                      |
| `0000e000` - `0000efff` | Pintos   | Stack for loader; kernel stack and `struct thread` for initial kernel thread. |
| `0000f000` - `0000ffff` | Pintos   | Page directory for startup code.                             |
| `00010000` - `00020000` | Pintos   | Page tables for startup code.                                |
| `00020000` - `0009ffff` | Pintos   | Kernel code, data, and uninitialized data segments.          |
| `000a0000` - `000bffff` | Video    | VGA display memory.                                          |
| `000c0000` - `000effff` | Hardware | Reserved for expansion card RAM and ROM.                     |
| `000f0000` - `000fffff` | BIOS     | ROM BIOS.                                                    |
| `00100000` - `03ffffff` | Pintos   | Dynamic memory allocation.                                   |

## Low-level Kernel Initialization

The entry point for the kernel is the `start()` function in `threads/start.S`. This function's primary role is transitioning the CPU from 16-bit "real mode" to 32-bit "protected mode," which modern 80x86 operating systems use.

Key steps in this process include:

1. **Determining Memory Size**: The code first queries the BIOS for the PC's memory size, storing this data in `init_ram_pages`.

2. **Enabling the A20 Line**: This step is crucial for accessing memory beyond 1 MB.

   > The Intel 8086, Intel 8088, and Intel 80186 processors had 20 address lines, numbered A0 to A19; with these, the processor can access 220 bytes, or 1 MB. Internal address registers of such processors only had 16 bits. To access a 20-bit address space, an external memory reference was made up of a 16-bit offset address added to a 16-bit segment number, shifted 4 bits to the left so as to produce a 20-bit physical address. The resulting address is equal to segment × 16 + offset.

3. **Creating a Basic Page Table**: This table maps the first 64 MB of virtual memory to their corresponding physical addresses. It also maps the same physical memory starting at `LOADER_PHYS_BASE` (default: 3 GB).

4. **Final Preparations for Protected Mode**: The startup code enables protected mode, sets up paging, and prepares the segment registers. Interrupts are disabled at this stage as the system isn't ready to handle them yet.

5. **Initial Kernel Functions**: The process then proceeds to call `pintos_init()`.

## High-level Kernel Initialization

In `pintos_init()`:

1. Call `bss_init()`. In most C implementations, whenever you declare a variable outside a function without providing an initializer, that variable goes into the BSS.

2. Call `read_command_line()` to break the kernel command line into arguments, then `parse_options()` into read any options at the beginning of the command line.

3. Call `thread_init()` initializes the thread system.

4. Initialize the console and print a startup message to the console.

5. Initialize the kernel’s memory system.

   * `palloc_init()` sets up the kernel page allocator, which doles out memory one or more pages at a time

   - `malloc_init()` sets up the allocator that handles allocations of arbitrary-size blocks of memory
   - `paging_init()` sets up a page table for the kernel

6. Initializes the interrupt system.

   - `intr_init()` sets up the CPU's *interrupt descriptor table* (IDT) to ready it for interrupt handling

   - `timer_init()` and `kbd_init()` prepare for handling timer interrupts and keyboard interrupts, respectively.

   - `input_init()` sets up to merge serial and keyboard input into one stream.

7. Start the scheduler with `thread_start()`, which creates the idle thread and enables interrupts.

8. `serial_init_queue()` switch to interrupt-driven serial port I/O mode.

9. `timer_calibrate()` calibrates the timer for accurate short delays.

10. If the file system is compiled in, we **initialize the IDE disks** with `ide_init()`, then **the file system** with `filesys_init()`.

11. Boot complete.

12. `run_actions()`parses and executes actions specified on the kernel command line.

13. Finally, if `-q` was specified on the kernel command line, we call `shutdown_power_off()` to terminate the machine simulator. Otherwise, `pintos_init()`calls `thread_exit()`, which allows any other running threads to continue running.
