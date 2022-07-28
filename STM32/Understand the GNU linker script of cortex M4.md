## 

Introduction: The goal of this article is to provide a brief introduction about the GNU linker script of EFM32 Arm Cortex M4 devices. With this article, you should able to understand how the GNU

Jul 9, 2021•Knowledge

DETAILS

**Introduction:**

The goal of this article is to provide a brief introduction about the GNU linker script of EFM32 Arm Cortex M4 devices. With this article, you should able to understand how the GNU linker creates the executable file from the object files.

We will take the GNU linker script of EFM32GG11 _efm32gg11b.ld_ provided by the SDK as example, you can get the linker file in the folder below after installing the gecko SDK.

```text
.\sdks\gecko_sdk_suite\v2.x\platform\Device\SiliconLabs\EFM32GG11B\Source\GCC
```

**Overview of the Linker Script**

The Linker script consist of few parts below.

-   Memory layout
-   Entry point definition
-   Section definitions

**Memory layout**

The linker's default configuration permits allocation of all available memory, include the flash and embedded SRAM. If user need to specify the memory region that will be used by the linker and avoid using some memory region in their project, they can override the default configuration by using the _MEMORY_ command.

The _MEMORY_ command describes the location and size of memory blocks in the target. And for each linker script, it can only has at most one use of the _MEMROY_ command, however, user can define as may memory regions within it.

The syntax of _MEMORY_ command is below.

```
MEMORY
  {
    name (attr) : ORIGIN = origin, LENGTH = len
    ...
  }
```

_name_: defines the name of the regions that referred by the GNU linker.

_attr_: defines the attributes of a particular memory region. What the valid attribute should be made up of the options below (e.g. rwx).

![](https://community.silabs.com/servlet/rtaImage?eid=ka01M000000gFmj&feoid=00N1M00000FHjri&refid=0EM1M000001gppr)

_ORIGIN_: specify the start address of the memory region in the physical memory.

_LENGTH_: specify the size of the memory region in bytes.

For example, the default GNU linker script of EFM32GG11B820F2048GL192 _efm32gg11b.ld_ defined two memory regions by using the _MEMROY_ command. The first region named as FLASH start at physical address 0x0 with 2048kB size. And the second region named as RAM start as physical address 0x20000000 with 512kB size. The memory regions defined here corresponding to the whole Flash and RAM memory of the specified part number, read “**Section definitions**” in this article for how to use the memory regions.

```
MEMORY
{
	FLASH (rx) : ORIGIN = 0x0, LENGTH = 0x200000 /* 2048k */
	RAM (rwx) : ORIGIN = 0x20000000, LENGTH = 0x80000 /* 512k */
}
```

![](https://community.silabs.com/servlet/rtaImage?eid=ka01M000000gFmj&feoid=00N1M00000FHjri&refid=0EM1M000001gpps)

We do have some other KBAs discuss how to create memory regions by using the _MEMORY_ command, and then locate functions or entire file at the specific memory regions. Please refer to link below to get more information.

[/article/how-to-locate-an-entire-file-at-specific-memory-region](https://community.silabs.com/s/article/how-to-locate-an-entire-file-at-specific-memory-region)

[/article/using-a-linker-script-in-gcc-to-locate-functions-at-specific-memory-regions](https://community.silabs.com/s/article/using-a-linker-script-in-gcc-to-locate-functions-at-specific-memory-regions)

**Entry point definition**

The _ENTRY_ command is used for defining the first executable instruction. The syntax of _ENTRY_ command is:

```
ENTRY(symbol)
```

What the argument of the command is a symbol name.

For example, the default GNU linker script of EFM32GG11 _efm32gg11b.ld_ defined the first executable instruction of the target is the ‘Reset_Handler’.

```
ENTRY(Reset_Handler)
```

The symbol ‘Reset_Handler’ must be defined in code. Refer to the KBA [here](https://community.silabs.com/s/article/understand-the-gnu-assembler-startup-file-of-cortex-m4) for more information about the definition of the ‘Reset_Handler’.

**Section definitions**

The _SECTIONS_ command controls how to map the input sections into output sections, and also the order of the output sections in memory. It can only has at most one _SECTIONS_ command in a linker script file, however, many statements can be included within the _SECTIONS_ command.

The most frequently used statement in the _SECTIONS_ command is the _section definition_, which specifies the properties of an output section: its location, alignment, contents, fill pattern, and target memory region.

The full syntax of a _SECTIONS_ command is:

```
SECTIONS {
...
secname start BLOCK(align) (NOLOAD) : AT ( ldadr )
  { contents } >region :phdr =fill
...
}
```

_secname_: The name of the output section.

_start_: Specify the address that the output section will be loaded at.

_BLOCK(align)_: Advance the location counter . prior to the beginning of the section, so the section will begin at the specified alignment.

_(NOLOAD)_: Mark a section to not be loaded at run time.

_AT ( ldadr )_: Specify the load address of the section to 'ldadr'. If don't use the _AT_ keyword, the default load address of the section is same as the relocation address.

_>region_: Assign this section to a defined region of memory.

The ‘secname’ and ‘contents’ are required for a section definition, others are optional.

Take the .text section definition in the default linker file as an example to illustrate how to define a section.

```
  .text :
  {
    KEEP(*(.vectors))
    __Vectors_End = .;
    __Vectors_Size = __Vectors_End - __Vectors;
    __end__ = .;

    *(.text*)

    KEEP(*(.init))
    KEEP(*(.fini))

    /* .ctors */
    *crtbegin.o(.ctors)
    *crtbegin?.o(.ctors)
    *(EXCLUDE_FILE(*crtend?.o *crtend.o) .ctors)
    *(SORT(.ctors.*))
    *(.ctors)

    /* .dtors */
    *crtbegin.o(.dtors)
    *crtbegin?.o(.dtors)
    *(EXCLUDE_FILE(*crtend?.o *crtend.o) .dtors)
    *(SORT(.dtors.*))
    *(.dtors)

    *(.rodata*)

    KEEP(*(.eh_frame*))
  } > FLASH
```

.text is the name of the section, and the _KEEP((*(.vectors))_ is used for marking the ‘.vectors’ input section not be eliminated, the similar method also applied to ‘.init’ ‘.fini’ and the ‘.eh_frame’ input sections. The special linker variable dot ‘.’ always contains the current output location counter, so it can get the end address of the vectors by using the dot ‘.’ variable following the KEEP((*(.vectors)), and calculate the size of the ‘.vectors’ input section.

And then place the ‘.ctors’ and ‘.dtors’ input section from the crtgebin.o and crtbegin?.o files into the .text output section.

The '.ctors' section is set aside for a list of constructors (also called initialization routines) that include functions to initialize data in the program when the program is started. And the '.dtors' is set aside for a list of destructors (also called termination routines) that should be called when the program terminates. For more information about the ‘.ctors’ and ‘.dtors’ sections, please refer to the link below.

[https://gcc.gnu.org/onlinedocs/gccint/Initialization.html](https://gcc.gnu.org/onlinedocs/gccint/Initialization.html)

> FLASH assign the .text output section to the FLASH memory region that defined previously.

**Reference:**

Using ld (The GNU linker)  
[https://ftp.gnu.org/old-gnu/Manuals/ld-2.9.1/html_mono/ld.html](https://ftp.gnu.org/old-gnu/Manuals/ld-2.9.1/html_mono/ld.html)

Understand the GNU assembler startup file of cortex M4

[/article/understand-the-gnu-assembler-startup-file-of-cortex-m4](https://community.silabs.com/s/article/understand-the-gnu-assembler-startup-file-of-cortex-m4)



List of sourses:
https://community.silabs.com/s/article/understand-the-gnu-linker-script-of-cortex-m4?language=en_US
https://interrupt.memfault.com/blog/get-the-most-out-of-the-linker-map-file
