The control flow of the processor: Each transition from one address of a certain instruciton to another address of instruction is called **flow of control**, or **control flow** of the processor.

# Interrupts

**Interrupt** is an abrupt **change** in the control flow in response to some change in the processor’s state.

In the case, when the processor detects that the event has occured, it makes an indirect procedure call through a jump table called an **interrupt vector table**, to an operating system subroutine (the **interrupt handler**) that is specially designed to process this particular kind of event. When the exception handler finishes processing, one of three things happens, depending on the type of event that caused the exception:

* returns to $I_{\text{curr}}$
* returns to $I_{\text{next}}$
* Aborts the interrupted program

## Interrupt Handling

Interrupts are denoted with **interrupt number**s. Assigned by processor or kernel.

At system boot time, the operating system allocates and initializes a jump table called an **interrupt vector table**.

The interrupt number is an index into the interrupt vector table, whose starting address is contained in a special CPU register called the **interrupt vector table base register**.

When interrupts occur, control is being transfered from a user program to the kernel and certain data (like CPU registers' values, return address, etc.) are saved (pushed) onto the kernel's stack. This ensures that the interrupt handling routine has all the necessary information it needs, and after handling the interrupt, the system can restore the user program's state and resume its execution. This is because the kernel needs this information to handle the interrupt, and it also helps to isolate the interrupt handling routine from potentially unsafe user code.

After the handler has processed the event, it optionally returns to the interrupted program by executing a special “return from interrupt” instruction.

## Classes of Interrupts

| Class                | Cause                           | Async/sync | Return behavior           |
| -------------------- | ------------------------------- | ---------- | ------------------------- |
| (narrowly) Interrupt | Signal from I/O device / Signal | Async      | Next instruction          |
| Trap                 | Intentional exception           | Sync       | Next instruction          |
| Fault                | Potentially recoverable error   | Sync       | Might current instruction |
| Abort                | Nonrecoverable error            | Sync       | Never returns             |

* **Interrupts**

  Occur asynchronously as a result of signals from I/O devices that are external to the processor.

  I/O devices such as network adapters, disk controllers, and timer chips trigger interrupts by signaling a pin on the processor chip and placing onto the system bus the exception number that identifies the device that caused the interrupt.

  **After the current instruction finishes executing**, the processor notices that the interrupt pin has gone high, reads the exception number from the system bus, and then calls the appropriate interrupt handler.

  The interrupts can be further classified into maskable Interrupt and non-maskable interrupt. They can also be called as external/hardware interrupt.

* **Traps and System calls**

  Traps provide a procedure-like interface between user program and the kernel, known as system call.

  `syscall n` Executing `syscall` instruction causes a trap to an exception handler that decodes the argument and calls the appropriate kernel routine.

  **Distinguish between the calling and executing of system calls.** The former is in user mode and the latter is in kernel mode.

* **Faults**

  If the handler is able to correct the error condition, it returns control to the faulting instruction, thereby re-executing it. Otherwise, the handler returns to an `abort`routine in the kernel that terminates the application program that caused the fault.

* **Aborts**

  Aborts result from unrecoverable fatal errors.

## Interrupts in Linux x86-64 Systems

There are up to 256 different interrupt types. Numbers in the range from 0 to 31 correspond to exceptions that are defined by the Intel architects and thus are identical for any x86-64 system. Numbers in the range from 32 to 255 correspond to interrupts and traps that are defined by the operating system. Entry 64 points to the system call trap handler.

| Exception number | Description              | Exception class   |
| ---------------- | ------------------------ | ----------------- |
| 0                | Divide error             | Fault             |
| 13               | General protection fault | Fault             |
| 14               | Page fault               | Fault             |
| 18               | Machine check            | Abort             |
| 32-255           | OS-defined exceptions    | Interrupt or trap |

Each system call has a unique integer number that corresponds to an offset in a jump table in the kernel. (Notice that this jump table is not the same as the exception table. The **interrupt vector table** deals with unexpected conditions in the kernel's operation and helps it recover or handle such situations gracefully. The **system call jump table** serves as an index to quickly and efficiently route system call requests from user space programs to the appropriate kernel functions.)

![Screenshot 2023-07-13 at 11.57.46 PM](https://p.ipic.vip/trbk4b.png)

All arguments to Linux system calls are passed through general purpose registers rather than the stack. By convention, `%rax` contains the syscall number, with up to six arguments in `%rdi`, `%rsi`, `%rdx`,  `%r10`, `%r8` and `%r9`. On return from the system, registers `%r11`and `%rcx`are destroyed, and `%rax` contains the return value. A negative return value between -4095 and -1 indicates an error corresponding to negative `errno`.

### System Call Error Handling

When Unix-like systems' system-level functions encounter an error, they typically return $-1$ and set the global integer variable `errno` to indicate what went wrong.

Example for error checking:

```c
if((pid = fork()) < 0) {
	fprintf(stderr, "fork error: %s\n", sterror(errno));
	exit(0);
}
```

The `strerror` function returns a text string that describes the error associated with a particular value of `errno`.

error-handling wrapper:

```c
pid_t Fork(void){
	pid_t pid;

	if((pid = fork()) < 0)
		unix_error("Fork error");
	return pid;
}
```

# Dual-mode operation

In user-mode, the processor checks each instruction before executing it to verify that the instruction is permitted to be performed by that process.

In kernel-mode, there’s no verification.

Hardware needed to do the protection:

* **Privileged Instructions**: Protects system control.
* **Memory Protection**: Guards data access.
* **Timer Interrupts**: Balances processor usage.

## Privileged instructions

Process isolation is only possible if there is a way to **limit programs running in user-mode from directly changing their privilege level. **Other than trapping into the system kernel at the system call locations, an application process **cannot be allowed to change its privilege level**.

**Cannot disable processor interrupts**

When a process attempts to access the memory it is not allowed to access, exception happens. Transfer control to exception handler.

Usually, the operating system kernel simply halts the process on privilege violations, as it often means that the application’s code has encountered a bug.

## Memory protection

Discuss later in the memory part of this wiki.

## Timer interrupts

A hardware timer: interrupt the processor after a certain delay.

After resetting the timer, the operating system will resume execution of the process.

# Safe control transfer

Reasons for control transfer: the four classes of interrupts.

## Safe mode switch

A common sequence for entering the kernel and returning from the kernel

* Limited entry points

  The user program cannot jump arbitrarily.

* Automic changes to processor state

  mode, program counter, stack and memory protection all changed at the same time.

* Transparent, restartable execution

**Interrupt handler stack pointer**: A **privileged hardware register** pointing to a region of **kernel memory** called the interrupt handler stack.

Procedure:

* Save some of the interrupted process’s registers onto the interrupt stack (done by hardware)
* Call the kernel handler
* Save the remaining registers(done by the handler)
* Do the handler work

Procedure of returning from the interrupt, exception or trap:

* pop the registers stored by the handler
* hardware restore the registers it saved into the interrupt stack

> When a ***software*** interrupt occurs, the user's register information is saved on the **kernel stack**.
>
> When a ***hardware*** interrupt is triggered, the user's register information is saved on the **interrupt handler stack**.
>
> When a thread is rescheduled, the user's register information is saved on the **user stack**.
>
> Each thread corresponds to one user stack and one kernel stack, and each processor corresponds to one interrupt handling stack.
>
> This applies to Linux x86 and x86_64

> Linux has a per-process kernel stack that is used for system calls, faults, and other kernel-level operations. However, when it comes to handling hardware exceptions (interrupts), Linux uses a single, per-processor interrupt stack.

**Interrupt masking**

Interrupts arrive asynchronously may cause confusion when one interrupt handler is executing and another interrupt comes.

The hardware provides a privileged instruction to temporarily defer delivery of interrupt until it is safe to do so. On the x86 and several other processors, this instruction is called **disable interrupts.** The interrupt is only deferred(masked) and not ignored. The instruction are previleged.

**Hardware support for saving and restoring registers**

Once the handler starts running, it can use the `pushad` instruction to save the remaining registers onto the stack.

`pushad` saves the x86 integer registers; because the kernel does not typically do ﬂoating point operations, those do not need to be saved unless the kernel switches to a new process.

`popad ` pop an array of integer register values off the stack into the registers.

`iret` instruction loads a stack pointer, instruction pointer and processor status word off the stack into the appropriate processor registers.

**Putting it all together: Mode switch on the x86**

The x86 is segmented. Pointers come in 2 parts: a segment, such as code, data or stack, and an offset within that segment.

The current user-level instruction is based on a combination of the code segment(`cs` register plus the instruction pointer `eip`)

> 1. **Code Segment (CS)**: This segment contains the actual executable code of the program.
> 2. **Data Segment (DS)**: This segment contains static data such as global variables.
> 3. **Stack Segment (SS)**: This segment contains the program's execution stack, which includes local variables and function call information.
> 4. **Extra Segment (ES)**: This segment is generally used for extra data and is sometimes used by certain instructions that need to access data in a different segment.
> 5. **FS, GS**: Additional segments that can be used for various purposes depending on the specific needs of a program.

The detailed picture of mode switch:

1. Save three key values. The hardware **internally** saves the value of the stack pointer (the x86 `esp` and `ss` registers), the execution ﬂags (the x86 `eflags` register), and the instruction pointer (the x86 `eip` and `cs` registers).
2. Switch onto the interrupt handler stack. The hardware then switches the stack pointer to the base of the kernel handler stack. The hardware switches to a new stack if the Interrupt Stack Table (IST) feature is used. The new stack's address is found in the IST, which is a part of the Task State Segment (TSS).
3. Push the three key values onto the new stack. The hardware then stores the internally saved values onto the stack.
4. Optionally save error code. Certain types of interrupts such as page faults **generate an error code** to provide more information about the event; for these exceptions, the hardware pushes this code as the last item on the stack. For other types of events, the software interrupt handler typically pushes a dummy value onto the stack so that the stack format is identical in both cases.
5. Invoke the interrupt handler. Finally, the hardware changes the program counter to the address of the interrupt handler procedure through interrupt vector table.

6. In the interrupt handler process, `pushad` pushes the rest of the registers, **including the current stack pointer**, onto the stack. x86 `pushad` pushes the contents of all general purpose registers onto the stack.

At this point the kernel’s interrupt handler stack holds

1. the stack pointer, execution ﬂags, and program counter saved by the hardware
2. an error code or dummy value
3. a copy of all of the general registers (including the stack pointer but not the instruction pointer or eﬂags register)

To prevent an inﬁnite loop, the handler modiﬁes the program counter stored at the base on the stack to point to the instruction immediately after the one causing the mode switch.

### System calls

Trap instructions are instructions used in computer processors to cause the operating system to **switch from user mode to kernel mode.**

Issue a system call by **executing the trap instruction** to transfer control to the operating system kernel

To issue a system call:

| Architecture | Method  |
| ------------ | ------- |
| x86          | int     |
| x86-64       | syscall |

The system call handler will implement each system call. It runs in kernel mode. When a system call is made, the arguments of it should be carefully validated by the system call handler.

A pair of stubs are two short procedures that mediate between two environments, in this case between the user program and the kernel.

<img src="https://p.ipic.vip/srrxqv.png" alt="Screenshot 2023-07-14 at 1.17.33 AM" style="zoom:50%;" />

The syscall function takes care of marshalling the arguments passed by the user program into a format that can be understood by the system call handler, and handles any necessary validation of the arguments.

**x86** The system call calling convention is arbitrary, so here we pass arguments on the user stack, with a code indicating the type of system call in the register `%eax`. The return value comes back in `%eax` so there is no work to do on the return.

The kernel stub has four tasks:

* **Locate system call arguments:** the arguments are stored on the process’s user stack. We should convert the virtual addresses of the arguments to physical addresses.
* **Validate parameters**: you cannot trust the processes.
* **Copy before check:** the kernel copies system call parameters into kernel memory before performing the necessary checks.
* **Copy back any results**

In turn, the system call handler pops any saved registers (except `%eax`) and uses the `iret` instruction to return back to the user stub immediately after the trap, allowing the user stub to return to the user program.

### Starting a new process

The kernel allocates and initializes the process control block, allocates memory for the process, copies the program from disk into the newly allocated memory, and allocates **both** a **user-level** stack for normal execution and a **kernel-level** stack for handling system calls, interrupts and exceptions.

Arguments of a **program** are stored in the higher address.

When we create the new process, we allocate it a kernel stack, and we reserve room at the bottom of the kernel stack for the initial values of its registers, program counter, stack pointer, and processor status word. To start the new program, we can then switch to the new stack and **jump to the end of the interrupt handler**. When the handler executes `popad` and `iret`, the processor “returns” to the start of the user program.

`Exit` is a system call that terminates the process.

### Upcalls

We call virtualized interrupts and exceptions **upcalls.** In UNIX, they are called **signals**, and in Windows they are called **asynchronous events**.

Applications have the option to run UNIX signal handlers either on the process’s normal execution stack or on a special signal stack allocated by the user process in user memory. Running signal handlers on the normal stack can reduce the ﬂexibility of the signal handler in manipulating the stack, e.g., to cause a language-level exception to be raised.

A UNIX signal handler automatically **mask**s further delivery of that type of signal until the handler returns. The program can mask other signals, either all together or individually, as needed.

**Handling signals in user-level program**

When the timer interrupt occurs, 

* the hardware generates an interrupt request (IRQ) signal to the processor, which causes the processor to switch from user mode to kernel mode.

* The kernel interrupt handler saves the current state of the user-level computation onto the kernel interrupt handler stack.

* The kernel then copies the saved state from the kernel stack to a **user-level buffer**, which is a special area of memory reserved for handling signals and interrupts. This buffer contains the saved state of the user-level program that was interrupted by the timer interrupt.

* The kernel then sets the program counter register to the singal handler and the stack pointer register to signal stack. The signal stack is a special area of memory reserved ofr handling signals.

  > The signal handler is running on user mode and might change the registers. For this reason, we should backup the state of program.

* The kernel then returns from the interrupt handler, and the `reti` instruction resumes execution at the signal handler, rather than the original program counter. The signal handler is a user-level function that is responsible for handling the timer interrupt.

* When the signal handler has finished executing, it returns control to the kernel.

* The kernel copies the processor state from the signal handler back into kernel memory.

* The kernel then returns to the interrupted user-level program, using the saved state from the user-level buffer to restore the program's original state.