# THE KERNEL ABSTRACTION

A central role of operating system: protection

## The process concept

The operating system keeps track of the various processes on the computer using a data structure called the **process control block.** The process control block stores all the information the operating system needs about a particular process: where it is stored in memory, where its executable image is on disk, which user asked it to start executing, what privileges the process has, and so forth.

**Each thread of a multi-thread program have its own stack and PC.** They do share the data and code from the process.

## Dual-mode operation

In user-mode, the processor checks each instruction before executing it to verify that the instruction is permitted to be performed by that process.

In kernel-mode, there’s no verification.

Hardware needed to do the protection:

- Privileged instructions
- Memory Protection
- Timer interrupts

### Privileged instructions

Process isolation is only possible if there is a way to **limit programs running in user-mode from directly changing their privilege level.**

Other than trapping into the system kernel at the system call locations, an application process cannot be allowed to change its privilege level.

**cannot be allowed to change the set of memory locations it can access**

**cannot disable processor interrupts**

**Privileged instructions:** avaliable only in kernel-mode

When a process attempts to access the memory it is not allowed to access, exception happens. Transfer control to exception handler.

Usually, the operating system kernel simply halts the process on privilege violations, as it often means that the application’s code has encountered a bug.

**Memory protection**

Early computers: base and bounds

Can only be changed by privileged instructions

Cost: two extra comparisons for each instruction

Gain: memory protection

Only needed to check against base and bounds in user-mode

Unable to provide:

- Expandable heap and stack
- Memory sharing
- Non-relative memory address
- Memory fragmentation

Modern solution: virtual address

**Timer interrupts**

A hardware timer: interrupt the processor after a certain delay.

After resetting the timer, the operating system will resume execution of the process.

## Safe control transfer

Three reasons for transferring from user-mode to kernel-mode:

- Exceptions

can be used to set breakpoints

- Interrupts

**Hardware Interrupt**

timer events/ the completion of I/O requests

An alternative to interrupts: polling. The kernel loop and check if an event has occured and is needed to be handled

Interprocessor interrupts: coorindate actions across the multiprocessor

- System calls

**Software Interrupt**

Most processors implement system calls using a special trap instruction.

Applications trap only to a pre-defined address

### Safe mode switch

A common sequence for entering the kernel and returning from the kernel

- Limited entry points

The user program cannot jump arbitrarily.

- Automic changes to processor state

mode, program counter, stack and memory protection all changed at the same time.

- Transparent, restartable execution

**Interrupt vector**

A special register pointing to an area of kernel memory called the interrupt vector. The interrupt vector is an array of pointers, with each pointer pointing to the first instruction of a handler procedure.

x86: 0-31 hardware exceptions    32-255 interrupts     entry 64 points to the system call trap handler

**Interrupt stack**

A privileged hardware register pointing to a region of kernel memory called the interrupt stack.

Procedure:

- Save some of the interrupted process’s registers onto the interrupt stack(done by hardware)
- call the kernel handler
- Save the remaining registers(done by the handler)
- do the handler work

Procedure of returning from the interrupt, exception or trap:

- pop the registers stored by the handler
- hardware restore the registers it saved into the interrupt stack

---

Modern operating system allocates a kernel interrupt stack for every user-level process and user-level thread.

**Interrupt masking**

Interrupts arrive asynchronously may cause confusion when one interrupt handler is executing and another interrupt comes.

The hardware provides a privileged instruction to temporarily defer delivery of interrupt until it is safe to do so. On the x86 and several other processors, this instruction is called **disable interrupts.** The interrupt is only deferred(masked) and not ignored. The instruction are previleged.

**Hardware support for saving and restoring registers**

When a context switch occurs the x86 hardware:

- If in user-mode, pushes the interrupted process’s stack pointer onto the kernel’s exception stack, and switches to the kernel stack.
- Pushes the interrupted process’s instruction pointer
- Pushes the x86 *processor status word*.

Once the handler starts running, it can use the `pushad` instruction to save the remaining registers onto the stack.

`pushad`saves the x86 integer registers; because the kernel does not typically do ﬂoating point operations, those do not need to be saved unless the kernel switches to a new process.

---

`popad`pop an array of integer register values off the stack into the registers

`iret` instruction that loads a stack pointer, instruction pointer and processor status word off the stack into the appropriate processor registers.

**Putting it all together: Mode switch on the x86**

The x86 is segmented. Pointers come in 2 parts: a segment, such as code, data or stack, and an offset within that segment. 

The current user-level instruction is based on a combination of the code segment(`cs` register plus the instruction pointer `eip`)

The current stack position is based on the stack segment `ss` and the stack pointer within the stack segment `esp`.

1. Save three key values. The hardware **internally** saves the value of the stack pointer (the x86 `esp` and `ss` registers), the execution ﬂags (the x86 `eflags` register), and the instruction pointer (the x86 `eip` and `cs` registers).
2. Switch onto the kernel exception stack. The hardware then switches the stack pointer to the base of the kernel exception stack, speciﬁed in a special hardware register.
3. Push the three key values onto the new stack. The hardware then stores the internally saved values onto the stack.
4. Optionally save error code. Certain types of exceptions such as page faults **generate an error code** to provide more information about the event; for these exceptions, the hardware pushes this code as the last item on the stack. For other types of events, the software interrupt handler typically pushes a dummy value onto the stack so that the stack format is identical in both cases.
5. Invoke the interrupt handler. Finally, the hardware changes the program counter to the address of the interrupt handler procedure, speciﬁed via a special register in the processor that is accessible only to the kernel. This register contains a pointer to an array of exception handler addresses in memory. The type of interrupt is mapped to an index in this array, and the program counter is set to the value at this index.

In the interrupt handler process,

`pushad` pushes the rest of the registers, **including the current stack pointer**, onto the stack. x86 `pushad` pushes the contents of all general purpose registers onto the stack.

At this point the kernel’s exception stack holds 

1. the stack pointer, execution ﬂags, and program counter saved by the hardware
2. an error code or dummy value
3. a copy of all of the general registers (including the stack pointer but not the instruction pointer or eﬂags register)

To prevent an inﬁnite loop, the exception handler modiﬁes the program counter stored at the base on the stack to point to the instruction immediately after the one causing the mode switch.

The program counter for the instruction after the trap is saved on the kernel’s interrupt stack.

### System calls

Trap instructions are instructions used in computer processors to cause the operating system to **switch from user mode to kernel mode.**

Issue a system call by **executing the trap instruction** to transfer control to the operating system kernel

To issue a system call:

| X86    | int     |
| ------ | ------- |
| X86-64 | syscall |

The system call handler will implement each system call. It runs in kernel mode. When a system call is made, the arguments of it should be carefully validated by the system call handler.

A pair of stubs are two short procedures that mediate between two environments, in this case between the user program and the kernel.

![Untitled](https://p.ipic.vip/3c1k6z.png)

The syscall function takes care of marshalling the arguments passed by the user program into a format that can be understood by the system call handler, and handles any necessary validation of the arguments.

********X86******** The system call calling convention is arbitrary, so here we pass arguments on the user stack, with a code indicating the type of system call in the register `%eax`. The return value comes back in `%eax` so there is no work to do on the return.

---

The kernel stub has four tasks:

- **Locate system call arguments:** the arguments are stored on the process’s user stack. We should convert the virtual addresses of the arguments to physical addresses.
- **Validate parameters**: you cannot trust the processes.
- **Copy before check:** the kernel copies system call parameters into kernel memory before performing the necessary checks.
- **Copy back any results**

In turn, the system call handler pops any saved registers (except %eax) and uses the iret instruction to return back to the user stub immediately after the trap, allowing the user stub to return to the user program.

### Staring a new process

The kernel allocates and initializes the process control block, allocates memory for the process, copies the program from disk into the newly allocated memory, and allocates **both** a **user-level** stack for normal execution and a **kernel-level** stack for handling system calls, interrupts and exceptions.

Arguments of a **program** are stored in the higher address.

When we create the new process, we allocate it a kernel stack, and we reserve room at the bottom of the kernel stack for the initial values of its registers, program counter, stack pointer, and processor status word. To start the new program, we can then switch to the new stack and **jump to the end of the interrupt handler**. When the handler executes `popad` and `iret`, the processor “returns” to the start of the user program.

`Exit` is a system call that terminates the process.

### Upcalls

We call virtualized interrupts and exceptions **upcalls.** In UNIX, they are called **signals**, and in Windows they are called **asynchronous events**.

Each process deﬁnes its own handlers for each signal type, much as the kernel deﬁnes its own interrupt vector table. If a process does not deﬁne a handler for a speciﬁc signal, then the kernel calls a default handler instead.

Applications have the option to run UNIX signal handlers either on the process’s normal execution stack or on a special signal stack allocated by the user process in user memory. Running signal handlers on the normal stack can reduce the ﬂexibility of the signal handler in manipulating the stack, e.g., to cause a language-level exception to be raised.

A UNIX signal handler automatically masks further delivery of that type of signal until the handler returns. The program can mask other signals, either all together or individually, as needed.

---

**Handling signals in user-level program**

When the timer interrupt occurs, the hardware generates an interrupt request(IRQ) signal to the processor, which causes the processor to switch from user mode to kernel mode. 

The kernel interrupt handler saves the current state of the user-level computation onto the kernel stack.

The kernel then copies the saved state from the kernel stack to a **user-level buffer**, which is a special area of memory reserved for handling signals and interrupts. This buffer contains the saved state of the user-level program that was interrupted by the timer interrupt.

The kernel then sets the program counter register to the singal handler and the stack pointer register to signal stack. The signal stack is a special area of memory reserved ofr handling signals.

The kernel then returns from the interrupt handler, and the `reti` instruction resumes execution at the signal handler, rather than the original program counter. The signal handler is a user-level function that is responsible for handling the timer interrupt.

When the signal handler has finished executing, it returns control to the kernel.

The kernel copies the processor state from the signal handler back into kernel memory.

The kernel then returns to the interrupted user-level program, using the saved state from the user-level buffer to restore the program's original state.

## Case Study: Booting an operating system kernel

The BIOS reads bootloader from flash RAM or disk.

## Case Study: Virtual machines

**host operating system**

**guest operating system**

