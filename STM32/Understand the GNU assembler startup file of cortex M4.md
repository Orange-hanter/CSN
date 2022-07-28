# Understand the GNU assembler startup file of cortex M4

Introduction: The goal of this article is to provide a brief introduction about the GNU assemble startup file of EFM32 and EFR32 Arm Cortex M4 devices. With this article, you can understand how the

**Introduction:**

The goal of this article is to provide a brief introduction about the GNU assemble startup file of EFM32 and EFR32 Arm Cortex M4 devices. With this article, you can understand how the Cortex M4 processor starts.

We will take the GNU assembler startup file of EFM32GG11 _startup_efm32gg11b.S_ as example, you can get the startup file in the folder below after installing the gecko SDK.

```
.\sdks\gecko_sdk_suite\v2.x\platform\Device\SiliconLabs\EFM32GG11B\Source\GCC
```

Bear in mind, the GNU Assembler (known as GAS) and ARM assembler are two different syntaxes for assembly language source code, however, the organization of the startup code are similar.

For more information about the GNU assembler and ARM assemble, please refer to the online documentation here.  
[https://ftp.gnu.org/old-gnu/Manuals/gas-2.9.1/html_chapter/as_toc.html](https://ftp.gnu.org/old-gnu/Manuals/gas-2.9.1/html_chapter/as_toc.html)  
[http://www.keil.com/support/man/docs/armasm/](http://www.keil.com/support/man/docs/armasm/)

**Overview of the Startup code:**

The startup code consist of few parts below.

-   Architecture and syntax
-   Declaration of the Stack area
-   Declaration of the Heap area
-   Vector table
-   Assembler code of Reset handler
-   Definition of interrupt handler

**Architecture and syntax**

Generally, it will specify the instruction set syntax. There are two slightly different syntaxes are support for ARM and THUMB instructions, **divided** and **unified**.

```
    .syntax     unified
    .arch       armv7e-m
```

The _".arch"_ instruction used to select the target architecture. What the architecture of Cortex-M4 is **ARMv7E-M**.

**Declaration of the Stack area**

The stack is a contiguous area of memory that may be used for storage of local variables and for passing additional arguments to subroutines when there are insufficient argument registers available.

The assembly code below declares the stack area, and the _".align 3"_ makes the starting of this area on a multiple of 8-byte (2^3 = 8) boundary.

If didn’t define the stack size with the macro _“__STACK_SIZE”_, it will declare a constant Stack_Size of value 0x00000400 with the directive _“.equ”_.

_".globl"_ directive makes the symbol _“__StackTop”_ and _“__StackLimit”_ visible to GNU linker.

Then emit Stack_Size bytes with the directive _“.space Stack_Size”_, it will fill the section with zero by default if didn’t specify the **fill** parameter for the _“.space”_ directive.

Then set the size associated with the symbol name __StackLimit, and the size in bytes is computed from the expression _“. - __StackLimit”_.

```C
    .section    .stack
    .align      3
#ifdef __STACK_SIZE
    .equ        Stack_Size, __STACK_SIZE
#else
    .equ        Stack_Size, 0x00000400
#endif
    .globl      __StackTop
    .globl      __StackLimit
__StackLimit:
    .space      Stack_Size
    .size       __StackLimit, . - __StackLimit
__StackTop:
    .size       __StackTop, . - __StackTop
```

The linker file will set the stack top to end of RAM, and stack limit move down by size of stack_dummy section.

```C
  __StackTop = ORIGIN(RAM) + LENGTH(RAM);
  __StackLimit = __StackTop - SIZEOF(.stack_dummy);
  PROVIDE(__stack = __StackTop);
```

**Declaration of the Heap area**

The heap is a pool of memory that are managed by the processor itself (for example, with the C malloc function). It is typically used for the creation of dynamic data objects.

The assembly code below declares the heap area which are similar as the stack declaration. The labels _“__HeapBase”_ and _“__HeapLimit”_ indicate the starting and the ending of the heap area respectively.

```C
    .section    .heap
    .align      3
#ifdef __HEAP_SIZE
    .equ        Heap_Size, __HEAP_SIZE
#else
    .equ        Heap_Size, 0x00000C00
#endif
    .globl      __HeapBase
    .globl      __HeapLimit
__HeapBase:
    .if Heap_Size
    .space      Heap_Size
    .endif
    .size       __HeapBase, . - __HeapBase
__HeapLimit:
    .size       __HeapLimit, . - __HeapLimit
```

What the __HeapBase and __HeapLimit are defined in the linker file.

**Vector table**

The vector table contains the reset value of the stack pointer, and the start addresses for all exception and interrupt handlers. Figure below shows the order of the Cortex-M4 exception and interrupt vectors in the vector table.

![](https://community.silabs.com/servlet/rtaImage?eid=ka01M000000gFpY&feoid=00N1M00000FHjri&refid=0EM1M000001gpaM)

In general, the vector table is fixed at address 0x00000000 on system reset. The privileged software can write to the VTOR register to relocate the vector table start address to a different memory location, in the range 0x00000080 to 0x3FFFFF80.

Once an exception or interrupt is triggered, the processor automatically jumps to the corresponding address in the vector table which contains an address to the relevant exception or interrupt handlers (ISR).

For more information about each exception handler of the Cortex-M4, the reader is referred to the ARM Cortex-M4 Technical Reference Manual.

```c
    .section    .vectors
    .align      2
    .globl      __Vectors
__Vectors:
    .long       __StackTop                 /* Top of Stack */
    .long       Reset_Handler              /* Reset Handler */
    .long       NMI_Handler                /* NMI Handler */
    .long       HardFault_Handler          /* Hard Fault Handler */
    .long       MemManage_Handler          /* MPU Fault Handler */
    .long       BusFault_Handler           /* Bus Fault Handler */
    .long       UsageFault_Handler         /* Usage Fault Handler */
    .long       Default_Handler            /* Reserved */
    .long       Default_Handler            /* Reserved */
    .long       Default_Handler            /* Reserved */
    .long       Default_Handler            /* Reserved */
    .long       SVC_Handler                /* SVCall Handler */
    .long       DebugMon_Handler           /* Debug Monitor Handler */
    .long       Default_Handler            /* Reserved */
    .long       PendSV_Handler             /* PendSV Handler */
    .long       SysTick_Handler            /* SysTick Handler */

    /* External interrupts */
    .long       EMU_IRQHandler             /* 0 - EMU */
    .long       WDOG0_IRQHandler           /* 1 - WDOG0 */
    .long       LDMA_IRQHandler            /* 2 - LDMA */
    /* ------------------- */
    .size       __Vectors, . - __Vectors
```

The assembly code above declared the vectors section with a multiple of 4-bytes (2^2 = 4) boundary, and then places all of the exception and interrupt handlers into the vector section with the directive _“.long”_ since all of the handlers are 32-bit values.

As shown in the figure above, the first 32-bit of the vector table is the value for initializing the stack pointer. And the default value here is _“__StackTop”_ that correspond to the end of RAM. So it will always initialize the stack pointer as the “__StackTop” after resetting.

At the ending, set the size of the __Vectors area, and calculate the size by subtracting the address of __Vectors from the current address.

**Assembler code of Reset handler**

After defining the vector table, we will discuss about the reset handler in the text segment.

Reset is invoked on power up or a warm reset. The exception model treats reset as a special form of exception. When reset is asserted, the operation of the processor stops, potentially at any point in an instruction.

When reset is deasserted, execution restarts from the address provided by the reset entry in the vector table, which is _“Reset_Handler”_ in our example.

_“.type”_ directive set the symbol _“Reset_Handler”_ to be a function symbol.

```C
    .text
    .thumb
    .thumb_func
    .align      2
    .globl      Reset_Handler
    .type       Reset_Handler, %function
Reset_Handler:
#ifndef __NO_SYSTEM_INIT
    ldr     r0, =SystemInit
    blx     r0
#endif
```

If didn’t define the macro __NO_SYSTEM_INIT, it will load the address of _“SystemInit”_ function to R0, and then branch to the label _“SystemInit”_ which defined in the _“system_efm32gg11b.c”_ for system initialization (e.g. set the Vector Table Offset) before the main() routine and any data has been initialized.

After resetting the system, it will branch to the function _“ _start”_. In fact, the function is provided by the C standard library, in a file called [crt0.o](https://en.wikipedia.org/wiki/Crt0), which include a set of execution startup routines to perform initialization and then call program’s _main_ function.

```C
#ifndef __START
#define __START _start
#endif
    bl      __START

    .pool
    .size   Reset_Handler, . - Reset_Handler
```

The source code of crt0.s for ARM Cortex-M can be found on the link below with the path newlib/libc/sys/arm/crt0.S, and you can find the call to main() around line 400.

[https://developer.arm.com/open-source/gnu-toolchain/gnu-rm](https://developer.arm.com/open-source/gnu-toolchain/gnu-rm)

In fact, before branching to the main function, there are much assembler code used for copying data from read only memory to RAM, and clearing the BSS sections.

**Definition of interrupt handler**

During the code execution, there might be exception or interrupt occurs, and the processor will start executing the exception or interrupt handler.

This _“.weak”_ directive sets the weak attribute on the symbol _“Default_Handler”_, if the symbol does not already exist in the source code, it will be created. And by default, the handler is just an endless loop by the directive _“b .”_

```C
    .align  1
    .thumb_func
    .weak   Default_Handler
    .type   Default_Handler, %function
Default_Handler:
    b       .
    .size   Default_Handler, . - Default_Handler
```

And then use the commands _“.macro”_ and _“.endm”_ to define a macro to define the default handlers for all of the exception and interrupt except **Reset**.

Define a macro called _“def_irq_handler”_ with an argument “handler_name”, the macro will set the symbol “\handler_name” as weak, and then set the value of the symbol “\handler_name” to “Default_Handler”.

```C
    .macro  def_irq_handler	handler_name
    .weak   \handler_name
    .set    \handler_name, Default_Handler
    .endm

    def_irq_handler     NMI_Handler
    def_irq_handler     HardFault_Handler
```

For e.g. look at the NMI exception handler above, with the definition of the macro, “def_irq_handler NMI_Handler” is equivalent to the assembly input below:

```C
    .week	NVI_Handler
    .set	NVI_Handler, Default_Handler
```

So the default handlers of all of the exception and interrupt handlers except Reset will be set as “Default_Handler” which is an endless loop.

**Reference:**

GNU Assembler Manual  
[https://ftp.gnu.org/old-gnu/Manuals/gas-2.9.1/html_chapter/as_toc.html](https://ftp.gnu.org/old-gnu/Manuals/gas-2.9.1/html_chapter/as_toc.html)

Using ld (The GNU linker)  
[https://ftp.gnu.org/old-gnu/Manuals/ld-2.9.1/html_mono/ld.html](https://ftp.gnu.org/old-gnu/Manuals/ld-2.9.1/html_mono/ld.html)

Cortex-M4 Device Generic User Guide  
[http://infocenter.arm.com/help/topic/com.arm.doc.dui0553a/DUI0553A_cortex_m4_dgug.pdf](http://infocenter.arm.com/help/topic/com.arm.doc.dui0553a/DUI0553A_cortex_m4_dgug.pdf)

Jul 9, 2021
