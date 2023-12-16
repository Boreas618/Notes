# Linking

Linking can be performed at

* **Compile time**: when the source code is translated into machine code
* **Load time**: when the program is loaded into memory and executed by the *loader*
* **Run time**: by application programs.

## Compiler Drivers

Most compilation systems provide a **compiler driver** that invokes the language preprocessor, compiler, assembler, and linker, as needed on behalf of the user. 

<img src="https://p.ipic.vip/zgt5zk.png" alt="Screenshot 2023-11-30 at 10.46.39 PM" style="zoom:50%;" />

**Components**:

* **Preprocessor**: `cpp [*other arguments*] main.c /tmp/main.i`
* **C compiler**: `cc1 /tmp/main.i -Og [*other arguments*] -o /tmp/main.s`
* **Assembler**: `as [*other arguments*] -o /tmp/main.o /tmp/main.s`
* **Linker**: `ld -o prog [*system object files and args*] /tmp/main.o /tmp/sum.o`

## Static Linking

**What do linkers do?**

* **Step 1: Symbol resolution**

  Programs define and **reference** symbols (global variables and functions). Symbol definitions are stored in object file(by assembler) in symbol table. Symbol table is an array of structs. Each entry includes name, size and location of symbol.

* **Step 2: Relocation**

  Merges separate code and data sections into single sections. Relocate symbols from their relative locations in the `.o` files to their final absolute memory locations in the executable. Update all references to these symbols to reflect their new positions.

## Object Files

**Three Kinds of Object Files (Modules)**

* Relocatable object file (`.o` file)
* Executable object file (`a.out` file)
* Shared object file(`.so` file)

An **object module** is a sequence of bytes, and an **object file** is an object module stored on disk in a file. We can use them interchangeably.

Windows uses the **Portable Executable (PE) format**. Mac OS-X uses the **Mach-O format**. Modern x86-64 Linux and Unix systems use **Executable and Linkable Format (ELF)**.

## Relocatable Object Files

<img src="https://p.ipic.vip/zty9oh.png" alt="Screenshot 2023-11-30 at 11.01.15 PM" style="zoom:50%;" />

### ELF header

Begins with a 16-byte sequence that describes the word size and byte ordering of the system.

The rest includes:

* The size of the ELF header
* The object file type (e.g., relocatable, executable, or shared)
* The machine type (e.g., x86-64)
* The file offset of the section header table,
* The size and number of entries in the section header table.

```c
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              EXEC (Executable file)
  Machine:                           RISC-V
  Version:                           0x1
  Entry point address:               0x1011a
  Start of program headers:          64 (bytes into file)
  Start of section headers:          138000 (bytes into file)
  Flags:                             0x5, RVC, double-float ABI
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         3
  Size of section headers:           64 (bytes)
  Number of section headers:         24
  Section header string table index: 23
```

* `.text`: The machine code of the compiled program.
* `.rodata`: Read-only data such as the format strings in printf statements, and jump tables for switch statements.
* `.data`: Initialized global and static C variables.
* `.bss`: Uninitialized global and static C variables, along with any global or static variables that are initialized to zero. 
* `.symtab`: A symbol table with information about functions and global variables that are defined and referenced in the program. In fact, every relocatable object file has a symbol table in `.symtab`.
* `.rel.text`: A list of locations in the `.text` section that will need to be modified when the linker combines this object file with others. In general, any instruction that calls an external function or references a global variable will need to be modified. On the other hand, instructions that call local functions do not need to be modified. Note that relocation information is not needed in executable object files, and is usually omitted unless the user explicitly instructs the linker to include it.
* `.rel.data`: Relocation information for any global variables that are referenced or defined by the module. In general, any initialized global variable whose initial value is the address of a global variable or externally defined function will need to be modified.
* `.debug`: A debugging symbol table with entries for local variables and typedefs defined in the program, global variables defined and referenced in the program, and the original C source file. It is only present if the compiler driver is invoked with the -g option.
* `.line`: A mapping between line numbers in the original C source program and machine code instructions in the .text section. It is only present if the compiler driver is invoked with the -g option.
* `.strtab`: A string table for the symbol tables in the `.symtab` and `.debug` sections and for the section names in the section headers. A string table is a sequence of null-terminated character strings.

## Symbols and Symbol Tables

### Linker Symbols

- **Global symbols**: Symbols defined by module *m* that can be referenced by other modules. E.g., non-static C functions and non-static global variables.

- **External symbols**: Referenced by module *m* but defined by other module

- **Local symbols**: Defined and referenced **exclusively** by module *m*. E.g., C functions and global variables defined with the static attribute.

  **Local symbols are not local program variables. Linkers have no idea of local variables.** The symbol table in .`symtab` does not contain any symbols that correspond to local nonstatic program variables. 

  Functions that we want to be private are defined with `static`, while functions that we want to make global are not.

  > **Local non-static C variables**: stored on the stack
  >
  > **Local static C variables**: stored in either `.bss`, or `.data`

  ```c
  // Compiler allocates space in `.data` for each definition of x
  int f() {
    static int x = 0;
    return x;
  }
  
  int g() {
    static int x = 1;
    return x;
  }
  ```

### Symbol Tables

Symbol tables are built by assemblers, using symbols exported by the compiler into the assembly-language .s file. An ELF symbol table is contained in the .symtab section. 

An entry in the symbol table appears as follows:

```
Num:    Value          Size Type    Bind   Vis      Ndx Name
433: 00000000000101ac    72 FUNC    GLOBAL DEFAULT    1 main
```

- **Num**: This is the byte offset into the string table, leading to the symbol's name, represented as a null-terminated string.

- **Value**: This represents the address of the symbol. In relocatable modules, it's an offset from the start of the section where the symbol is defined. In executable object files, it indicates an absolute run-time address.

- **Size**: This denotes the object's size in bytes. The type is typically categorized as either data or function. Additionally, the symbol table may include entries for individual sections and the original source file's pathname.

- **Type**: Indicates whether the symbol represents a function, data, or other categories.

- **Bind**: This field specifies the symbol's binding attribute, indicating whether it is a local or global symbol.

- **Vis**: Indicates the symbol's visibility, such as default or hidden, determining how the symbol can be accessed by other modules.

- **Ndx**: The index of the section to which the symbol is associated. Besides index, there are three special pseudosections: **ABS** is for symbols that should not be relocated. **UNDEF** is for undefined symbols—that is, symbols that are referenced in this object module but defined elsewhere. **COMMON** is for uninitialized data objects that are not yet allocated.

  > **COMMON**: Uninitialized global variables
  >
  > `.bss`: Uninitialized static variables, and global or static variables that are initialized to zero

- **Name**: This is the actual name of the symbol as it appears in the code, providing an identifiable reference for programmers.

## Symbol Resolution

### How Linker Resolves Dupilcate Symbol Definitions?

**Strong symbols**: procedures(functions) and initialized globals

**Weak symbols**: uninitialized globals

**Linker’s Symbol Rules**: 

- Multiple strong symbols are not allowed. Otherwise: Linker error.
- Given a strong symbol and multiple weak symbols, choose the strong symbol. References to the weak symbol resolve to the strong symbol
- If there are multiple weak symbols, pick an arbitrary one. Can override this with `gcc -fno-common`. This can lead to bugs since 

> In some cases the linker allows multiple modules to define global symbols with the same name.
>
> When the compiler is translating some module and encounters a weak global symbol, say, `x`, it does not know if other modules also define `x`, and if so, it cannot predict which of the multiple instances of `x` the linker might choose. So the compiler defers the decision to the linker by assigning `x` to COMMON. 
>
> On the other hand, if `x` is initialized to zero, then it is a strong symbol (and thus must be unique by rule 2), so the compiler can confidently assign it to` .bss`. Similarly, static symbols are unique by construction, so the compiler can confidently assign them to either `.data` or `.bss`.

> **Illustration of a Bug Due to Uninitialized Global Variables**:
>
> ```c
> /* File: foo5.c */
> #include <stdio.h>
> 
> // Declaration of global variables
> int y = 15212;
> int x = 15213;
> 
> void f(void); // Forward declaration of function f
> 
> int main() {
>     f(); // Call to function f
>     // Printing the values of x and y in hexadecimal format
>     printf("x = 0x%x, y = 0x%x\n", x, y);
>     return 0; 
> }
> 
> /* File: bar5.c */
> // Redefining x as a double
> double x;
> 
> // Implementation of function f
> void f() {
>     x = -0.0; // Assigning a double value to x
> }
> 
> ```
>
> **Explanation**: In this code, the global variable `x` is defined in two different source files (`foo5.c` and `bar5.c`) with different data types (`int` in `foo5.c` and `double` in `bar5.c`). The memory addresses for `x` and `y` are `0x601020` and `0x601024`, respectively. When the linker resolves the symbol `x`, it might choose the `int x` from `foo5.c`. Since `x` in `bar5.c` is treated as a `double` and assigned a value, this operation might inadvertently overwrite the adjacent memory where `y` resides, leading to undefined behavior. This issue exemplifies the risks of using uninitialized global variables across multiple modules.

Avoid global variables if you can. Otherwise:

- Use `static` if you can
- Initialize if you define a global variable
- Use `extern` if you reference an external global variable

### Linking with Static Libraries

All compilation systems provide a mechanism for packaging related object modules into a single file called a **static library**.

Standard I/O, string manipulation, and integer math functions such as `atoi`, `printf`, `scanf`, `strcpy`, and `rand`. They are available to every C program in the `libc.a` library.

At link time, the linker will only copy the object modules that are referenced by the program, which reduces the size of the executable on disk and in memory.

To compose an archive (`.a` file, static library),

```shell
$ gcc -c addvec.c multvec.c
$ ar rcs libvector.a addvec.o multvec.o
```

To use the archive (`.a` file, static library),

```shell
$ gcc -c main2.c
$ gcc -static -o prog2c main2.o ./libvector.a
# The -static argument tells the compiler driver that the linker should build a fully linked executable object file that can be loaded into memory and run without any further linking at load time.
```

or

```shell
$ gcc -c main2.c
$ gcc -static -o prog2c main2.o -L. -lvector
# The -lvector argument is a shorthand for libvector.a, and the -L. argument tells the linker to look for libvector.a in the current directory.
```

The whole picture is:

<img src="https://p.ipic.vip/ui9pde.png" alt="Screenshot 2023-12-01 at 1.48.44 AM" style="zoom:50%;" />

### How Linkers Use Static Libraries to Resolve References

The sequence of the object files, source files and libraries matters. The algorithm resolves the symbols in the files that appears right to the current file. The general rule for libraries is to place them at the end of the command line. Luckliy, the files can be repeated in the command line:

```shell
$ gcc foo.c libx.a liby.a libx.a
```

## Relocation

![Screenshot 2023-02-23 at 6.15.10 PM.png](https://p.ipic.vip/7mypa5.png)

**Relocation Entries**

One moudle contains only one function.

![Screenshot 2023-02-23 at 6.28.47 PM.png](https://p.ipic.vip/aaupke.png)

The address is initially set to 0 and waiting to be updated later.

`sum-0x4`: The address is PC relative and you have to -0x4 becasue the PC has been changed to the address of the next instruction.

![Screenshot 2023-02-23 at 6.32.46 PM.png](https://p.ipic.vip/ghwfa2.png)

`e8 05 00 00 00` : `05` is the offset of `<sum>`from `4004e3`.

![Screenshot 2023-02-23 at 6.37.45 PM.png](https://p.ipic.vip/uyk0g2.png)

`brk` is matained by kernel

The loader copies code and data from executable file to memory as a collection of segments. It starts running the (usually) first instruction at the entry point specified in the ELF file. The instruction is a special loader called `_start` written in assembly for wahtever CPU architecture you’re using. This does super basic housekeeping then calls the C function `_libc_start_main` which will set the environment variables, argc and argv, call the main function…

The `.init` section contains `_init` function which is called by `_start` before `_libc_start_main`. It can be used to initialize profiling.

**Packaging Commonly Used Functions**

Old-fashioned Solution: Static Libraries

- Concatenate related relocatable object files into a single file with an index
- Enhance linker so that it tries to resolve unresolved external references by looking for symbols in one or more archives.
- If an archive member file resolves reference, link it into the executable

![Screenshot 2023-02-23 at 6.48.24 PM.png](https://p.ipic.vip/be4jjl.png)

Linking with Static Libraries

![Screenshot 2023-02-23 at 7.21.02 PM.png](https://p.ipic.vip/6s8icl.png)

`vector.h` contains the declaration of the function addvec. The declaration is a weak symbol. 

**Linker’s algorithm for resolving external references**

Scan `.o` files and `.a` files in the command line order

During the scan, keep a list of the current unresolved references.

As each new `.o` or `.a` file, **obj**, is encountered, try to resolve each unresolved reference in the list against the symbols defined in **obj.**

If any entries in the unresolved list at end of scan, then error.

**Problem:**

Command line order matters!

Moral: put libraries at the end of the command line.

**Modern Solution: Shared Libraries**

Disadvantages:

- There is duplication in the stored executables, as every function requires `libc(printf)`.
- Duplication in the running executables
- Minor bug fixes of system libraries require each application to explicitly relink

Shared Libraries:

- Object files that contain code and data that are loaded and linked into an application dynamically, at euther load-time or run-time
- Also called: dynamic link libraries, DLLs, .so files

---

Shared libraries can linked at compile time, load time, and runtime.

**Note that even if a shared library is linked at load time, a dynamic linker is still needed.**

- Compile time linking:

  ```c
  gcc main.c -L/path/to/lib -lmylibrary -o myprogram
  ```

- Load-time linking:

  ```c
  gcc -c -fPIC library.c
  gcc -shared -o liblibrary.so library.o
  gcc main.c -L. -llibrary -o program
  ```

- Runtime linking

  ```c
  gcc -c -fPIC library.c
  gcc -shared -o liblibrary.so library.o
  gcc main.c -o program
  ```

  At runtime, the program loads the **`liblibrary.so`**library dynamically using the **`dlopen()`**function and resolves its symbols using the **`dlsym()`**function.

![Screenshot 2023-02-23 at 7.39.07 PM.png](https://p.ipic.vip/mx553r.png)

syscall: execve, prog21 cannot be executed directly. 

The loadeer (execve) calls the dynamic linker

**Dynamic Libraries: Linking**

- Linker creates an executable that can be linked with the library at load time
- Copies relocation and symbol table information from library
- Creates a `.interp` section with the location of the dynamic linker

**Dynamic Libraries: Load Time**

- Loader checks for `.interp` section
- Loader runs the dynamic linker
- Dynamic linker relocates( actually puts) the text and data sections of the shared libraries into memory
- Dynamic linker relocates references to any symbols referenced in shared libraries

**Dynamic Libraries: Runtime (Recommend time-saving)**

- The dynamic loader resolves the GOT and PLT information if applicable.

---

To build a shared library `[libvector.so](http://libvector.so)`:

```dart
$ gcc -shared -fpic -o libvector.so addvec.so multvec.so
```

- The `-fpic` flag directs the compiler to generate position-idependent code.
- The `-shared` flag directs the compiler to create a shared object file.

We can link it to our program: 

```dart
$ gcc -o prog21 main2.c ./libvector.so
```

The loader loads the partially linked executable `prog21`. It notice that the executable contains a `.interp` section, which contains the path name of the dynamic linker. The loader then loads and runs the dynamic linker. The dynamic linker then finishes the linkingh task by performing the following relocations:

- Relocating the text and data of `[libc.so](http://libc.so)` into some memory segment.
- Relocating the text and data of `[libvector.so](http://libvector.so)` into another memory segment.
- Relocating any references in `prog21` to symbols defined by `[libvec.so](http://libvec.so)` and `libvector.so`

**Loading and Linking Shared Libraries from Applications**

```dart
#include <dlfcn.h>

void *dlopen(const char *filename, int flag);

//Returns pointer to handle if OK, NULL on error
```

The external symbols in `filename` are resolved using libraries previously opened with the `RTLD_GLOBAL` flag. If the current executable was compiled with `-rdynamic` flag, then its global symbols are also avaliable for symbol resolution. The flag argument can be either `RTLD_NOW` or `RTLD_LAZY`.

```dart
#include <dlfcn.h>

void *dlsym(void *handle, char *symbol);
//Returns pointer to symbol if OK, NULL on error

int dlclose(void *handle);
//Returns 0 if OK, -1 on error

const char *dlerror(void);
//Returns error message if previous call to dlopen, dlsym, or dlclose failed
```

An example program:

![Untitled](https://p.ipic.vip/1wz41u.png)

To compile our program

```dart
$ gcc -rdynamic -o prog2r dll.c -ldl
```

**`-ldl`**specifies the linker flag to link with the **`dl`**library, which is required for dynamic loading of libraries.

**Position-Independent Code(PIC)**

Code that can be loaded without needing any relocations is known as position-independent code(PIC).(Codes that won’t use absolute address, relative address instead) Users direct GNU compilation systems to generate PIC code with the `-fpic` option to GCC. Shared libraries must always be compiled with this option.

**Global variables**

Global offset table:

![Untitled](/Users/sunyi/Downloads/ics/CSAPP 0cb817a3ebec4e5088555c3d1b89d5df/Untitled 63.png)

| GOT[0] and GOT[1] | Information for thr dynamic linker |
| ----------------- | ---------------------------------- |
| GOT[2]            | Pointer to the dynamic linker      |
| Remaining entries | Function addresses to resolve      |

The GOT contains an 8-byte entry for each global data object that is referenced by the object module. The compiler also generates a relocation record for each entry in the GOT. At load time, the dynamic linker relocates each GOT entry so that it contains the absolute address of the object.

**Function calls**

Lazy binding: bind each procedure address the first time the procedure is actually called.

Means you don’t have to resolve functions that aren’t actually called.

Procedure Linkage Table: 

| PLT[0]         | Points to dynamic linker for this shared library |
| -------------- | ------------------------------------------------ |
| PLT[1]         | System startup                                   |
| PLT[2] onwards | Function called by user code                     |

Each PLT entry has a corresponding GOT entry.

**PLT entries**

- Jump to corresponding GOT entry
- Push ID for function onto the stack
- Jump to PLT[0]

PLT[0] Pass the function ID through to dynamic linker and jump into dynamic linker.

**Live demo about PLT and GOT**

```c
#include <stdio.h>

int main(){
    printf("Hello, world!\n");
    printf("Hello, world! *2\n");
    return 0;
}
```

And we use the readelf to find a section named `.got.plt`. The **`.got.plt`**section provides a global offset table (GOT) for the PLT to store the addresses of these symbols.

The PLT mechanism resolves the address of a shared library function during runtime. It loads the address from the `.got.plt` section into a register and jumps to it to execute the function. The `.got.plt` entry is updated with the actual address of the function so that subsequent calls can skip the PLT code.

```nasm
callq 0x4003f0 <printf@plt>
```

Jumps to the `.got.plt` table.

Disassemble `printf@plt`:

```nasm
jmpq *0x200c22(%rip)
pushq $0x0
jmpq 0x4003e0
```

Here, `0x200c22(%rip)`is in the got table and we can read the value there. The value is the address of the instruction `pushq $0x0` here. Thus we call the dynamic linker with the function identifier `0x4003e0` pushed into the stack. The dynamic linker would update the value in `0x200c22(%rip)` with the actual address of the function `printf`. So the next time we call `printf` we will directly go for it.

---

IDEA on system design: **make common case fast**

**Bind plt with got — first time, remain times**

---

**Library interpositioning**

- Compile-Time Interpositioning

  ```nasm
  gcc -DCOMPILETIME -c malloc.c
  gcc -I. -o intc int.c mymalloc.o
  ```

  ![Untitled](https://p.ipic.vip/agstnm.png)

- Link-Time Interpositioning

  ```nasm
  gcc -DLINKETIME -c mymalloc.c
  gcc -c int.c
  gcc -Wl,--wrap,malloc -Wl,--wrap,free -o intl int.o mymalloc.o
  ```

  The Linux static linker supports link-time interpositioning with the `--wrap f` flag. The flag tells the linker to resolve references to symbol `f`as `__wrap_f`, and to resolve references to symbol `__real_f` as `f`.

  The `Wl,option` flag passes option to the linker. Each comma in `option` is replaced with a space.

  ![Untitled](https://p.ipic.vip/kn862r.jpg)

  Pay attention to the code at line 4 and 5.

- Run-Time Interpositioning

  The fascinating mechanism is based on the dynamic linker’s `LD_PRELOAD` environment variable.

  If the `LD_PRELOAD` environment variable is set to a list of shared library pathnames, then when you load and execute a program, the dynamic linker will search the `LD_PRELOAD` libraries first, before any other shared libraries, when it resolves undefined references.

  ```nasm
  gcc -DRUNTIME -shared -fpic -o mymalloc.so mymalloc.c -ldl
  ```

  ```nasm
  LD_PRELOAD="./mymalloc.so" ./intr
  ```

  ![Untitled](/Users/sunyi/Notes/introduction-to-computer-system/CSAPP 0cb817a3ebec4e5088555c3d1b89d5df/Untitled 65.png)
