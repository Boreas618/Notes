# OS Introduction

**Features**:

* Concurrency 并发

  Difference between **concurrency** and **parallelism**: the former is about more than one events occur within a certain time range and the latter is about at the same time.

* Sharing 共享

  **Mutal exclusion** and **Simultaneous access**.

* Virtual 虚拟

  The feeling of illusion. Like the virtual memory mechanism makes the program have the illusion that the program can access the whole memory space.

* Asynchronism 异步

**Interfaces**:

* Command Interfaces
  * Online Command Interface: user input commands. Preemptive/RT
  * Offline Command Interface: batch processing command interface. Batch processing system
* Programming Interfaces (Generalized Instruction)

**Kernel Components**:

* Timer Management
* Interrupt Mechanism
* Atomic Operation
* System Data Structures

**Evaluation Criteria of operating systems:**

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

**BIOS (Basic Input/Output System)** and **UEFI (Unified Extensible Firmware Interface)** serve as the cornerstone firmware interfaces in computer systems, performing the pivotal role of bootstrapping the computer by initializing all essential hardware components. 

Upon powering up the computer, the **POST (Power-On Self-Test)** process is the first to run. POST is responsible for conducting preliminary hardware checks, ensuring that all critical components like memory, processor, and storage devices are functioning appropriately. Immediately after POST, the BIOS or UEFI takes over.

In systems with a BIOS, it seeks out the **Master Boot Record (MBR)** on the primary storage device. The MBR contains a small amount of executable code called the boot loader, which is essential for loading the operating system. 

Modern systems with UEFI look for a special partition, aptly named the UEFI partition. This partition stores necessary boot and service applications, including the boot loader for the operating system. 

MBR contains the partition table which tells the activie partitions from inactive partitions. The inactive partition contains the operating system. Then the control is transfered to active partition and PBR is read (the first sector in active partition)

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
- **System Structures**: Initializing essential data structures required for managing processes, memory, and other system resources.
- **RAM Disk Identification**: After setting up the basic structures, the kernel searches for an initial RAM disk, often referred to as Initramfs.

**3. Initramfs (Initial RAM FileSystem)**: Initramfs plays a vital role in the early boot process. Essentially, it's a simple in-memory file system. This temporary root file system provides the kernel with a set of preliminary tools and utilities. By using Initramfs, the Linux kernel becomes more modular and can boot without relying heavily on specific device drivers or intricate configurations. Once the necessary drivers are loaded and the real root file system is mounted, Initramfs is typically discarded.

**4. Execution**: With system initialization complete, the kernel is ready to launch its first userspace process. This process is crucial for managing subsequent system processes and is known as the `init` process. While `init` has been the traditional first process in UNIX-like systems, modern Linux distributions have widely adopted `systemd` as the `init` system due to its more efficient and flexible service management capabilities.
