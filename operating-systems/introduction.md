# Introduction

> 1. Booting

[Case Study: Pintos Booting](./case-study-pintos-booting.md)

> 2. Process and Thread

[Process and Thread](./process-and-thread.md)

[Programming: Process and Thread](./programming-process-and-thread.md)

[Case Study: Linux Task Model](./linux-task.md)

> 3. Processor Scheduling

[Scheduling](./scheduling.md)

[Case Study: Linux Scheduling](./linux-scheduling.md)

> 4. Synchronization and Mutual Exclusion

[Synchronization and Mutual Exclusion](./synchronization-and-mutual-exclusion.md)

[Case Study: Linux Futex](./linux-futex.md)

## Features

* **Concurrency**

  > Difference between **concurrency** and **parallelism**: the former is about more than one events occur within a certain time range and the latter is about at the same time.

* **Sharing**: Mutal exclusion and Simultaneous access.

* **Virtualization**: The feeling of illusion. For instance, the virtual memory mechanism makes the program have the illusion that the program can access the whole memory space.

* **Asynchronism**: Asynchronism allows a system to initiate a task without waiting for it to complete. Once the task is initiated, the system can proceed to other tasks, returning to the original task once it is completed.

## OS Booting Process Explained Using a Linux PC

### BIOS/UEFI Initialization Phase

The BIOS (Basic Input/Output System) or UEFI (Unified Extensible Firmware Interface) acts as the fundamental bridge between a computer's hardware and its operating system. When the computer is powered on, the BIOS or UEFI is the first to activate. 

During this phase, the BIOS or UEFI conducts a power-on self-test (POST) to ensure that key components such as memory, processor, and storage devices are functioning correctly.

Following the POST, the BIOS or UEFI identifies the appropriate boot device and accesses its boot record. The processes for BIOS and UEFI at this stage diverge:

* **BIOS**: This system searches for a bootable device (like a hard disk or optical disk) and, upon finding one, reads its master boot record (MBR). The MBR, occupying the first 512 bytes of the boot device, includes a partition table and the initial stage of the bootloader. The partition table highlights the active partition. After reading the MBR, control shifts to the **first stage bootloader**.
* **UEFI**: It looks for the ESP (EFI System Partition) on the storage device, a designated partition that houses the UEFI bootloader and essential tools required during startup.

### Bootloader Phase

The bootloader's primary function is to load the operating system's kernel into the computer's memory.

As previously mentioned, the control is passed to the **first stage bootloader** after the MBR is read. This bootloader locates the active partition using the MBR's partition table and then reads its partition boot record. This record directs to the **second stage bootloader**, which is more sophisticated than the first. The second stage is tasked with loading the operating system kernel.

The second stage bootloader is not limited by the MBR's 512-byte size, allowing it to offer more advanced features, such as a graphical user interface for user interaction. For instance, GRUB (a commonly used bootloader in Linux systems) loads the selected Linux kernel and, if present, the initial ramdisk into memory. Subsequently, control is transferred to the kernel, initiating the operating system's booting process.

## **Kernel Loading Phase in Operating Systems**

The kernel loading phase marks the point where the operating system's kernel takes control, initiating the kernel initialization. This critical process involves several key steps and components:

1. **Decompression**: Modern operating systems typically compress their kernel image to save on storage. For instance, the Linux kernel image, known as `vmlinuz`, is compressed. Before the kernel can run, it must first decompress this image.

2. **Initialization**: Post-decompression, the kernel embarks on a series of initialization tasks, which can be categorized as follows:

   > **Hardware Configuration**: This phase sets up and configures hardware components and communication protocols. Key functions include:
   >
   > 1. `setup_arch()`: Manages architecture-specific setup like BIOS queries for hardware parameters, memory region setup, etc.
   > 2. `boot_cpu_init()`: Activates the current CPU as the boot CPU.
   > 3. `page_address_init()`: For systems with high memory, this function initializes the high memory area.
   > 4. `early_trap_init()`: Establishes early trap handlers.
   > 5. `setup_per_cpu_areas()`: Prepares per-CPU areas.
   > 6. `setup_traps()`: Installs trap handlers for hardware interruptions.
   > 7. `init_IRQ()`: Initializes the Interrupt Controller and related mechanisms.
   > 8. `sched_init()`: Sets up the scheduler.
   > 9. `timekeeping_init()`: Starts the timekeeping system.

   > **System Structures**: This involves initializing key data structures for process, memory, and resource management, including:
   >
   > 1. `mm_init()`: Establishes the memory management framework.
   >    - `mem_init()`: Prepares the free page list.
   >    - `kmem_cache_init()`: Sets up the SLAB allocator.
   >    - `percpu_init_late()`: Completes per-CPU area setup.
   >    - `vfs_caches_init()`: Initializes Virtual File System caches.
   > 2. `cred_init()`: Sets up the credential subsystem.
   > 3. `fork_init()`: Allocates memory for process descriptors.
   > 4. `proc_caches_init()`: Prepares caches for processes and files.
   > 5. `buffer_init()`: Initializes buffer head structures for block devices.
   > 6. `key_init()`: Starts the key management system.
   > 7. `security_init()`: Integrates with the Linux Security Modules for security setups.

   > **RAM Disk Identification**: After basic setups, the kernel locates the initial RAM disk, or Initramfs.
   >
   > 1. `early_initrd()`: Identifies the initrd memory location.
   > 2. `populate_rootfs()`: Extracts initramfs to a temporary filesystem if `CONFIG_BLK_DEV_INITRD` is set.
   > 3. `free_initrd_mem()`: Frees initrd memory after its contents are transferred or not needed.

3. **Initramfs (Initial RAM FileSystem)**: This in-memory file system is crucial in the early boot stage, providing essential tools and utilities. It allows for a more modular kernel boot, independent of specific device drivers or configurations. Once the actual root file system is mounted, Initramfs is typically discarded.

4. **Execution**: After system initialization, the kernel launches its first userspace process, `init`. This process is pivotal in managing other system processes. Although `init` has been traditional in UNIX-like systems, many modern Linux distributions now use `systemd` for its enhanced service management capabilities.
