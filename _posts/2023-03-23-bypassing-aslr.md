---
layout: post
title: Bypassing ASLR Damn Heroes and their defenses
date: 2023-03-23 21:28:16
description: Tutorial on bypassing ASLR
tags: memory hacking aslr reverse
categories: blog
---

![Shield Hero](/assets/img/shield_hero.png "Shield Hero"){:class="featured-image"} Shield Hero
Damn heroes and their defenses.
---

All of the tutorials we've done thus far have been done without randomizing memory locations. As we move towards younger code, we move towards code that has a few more security mechanisms in them. Not to worry, all mechanics can be broken, given enough time, and the time has come for us to break ASLR.

## History

**Address space layout randomization (ASLR)** was developed as a security mechanism to prevent the exploitation of functions in memory. It randomizes the address space positions of the stack, heap, and libraries.

![ASLR](/assets/img/aslr-osx.png "ASLR"){:class="featured-image"}

- Linux PaX project was the first to design, publish, and implement ASLR into the Linux kernel in July 2001.
- The first mainstream OS to support ASLR was OpenBSD 3.4 in 2003.
- Windows integrated ASLR into their OS starting with Vista in January 2007.

Integrating ASLR into Vista added a 1 in 256 chance the correct address could be selected. They enabled it only for executables and dynamic link libraries specially linked to be ASLR-enabled. For compatibility, it was not enabled by default for other programs. ASLR can be turned on by default via editing the registry entry:
`HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management\MoveImages`


Alternatively, it can be enabled by installing the **Enhanced Mitigation Experience Toolkit (EMET).**

---

## Bypassing ASLR

When we played with Bypassing NX/DEP, we only had to determine the base address for the function `[system()]` in `libc`, and the location of the string, `"/bin/sh"`. When we turn on `randomize_va_space` (enabled by default in most Linux OS'), it will randomize the location of the `libc` base address.

![aslr-ex1](/assets/img/aslr-ex1.png "aslr-ex1"){:class="featured-image"}


**Trick:** Only the `libc` base address is randomized. The offset of each function from the base address is not random at all. If we can determine the base address for `libc`, we can determine any function address by providing the offset of the random `libc` address.

## Understanding Position Independent Code (PIC)
Position Independent Code (PIC) enables the sharing of .text segments among multiple processes. The shared library’s .text segment points to a specific table in the `.data` segment instead of providing an absolute virtual address. This is a table that holds global function's absolute virtual addresses and their global symbols.

The dynamic linker, as part of its relocation, appends this table. While relocation happens, only the `.data` segment is modified. The .text segment stays untouched.

There are two ways a dynamic linker can relocate global symbols:

### Procedure Linkage Table (PLT):
Used to call external procedures/functions whose address isn't known at the time of linking. This is resolved by the dynamic linker at runtime.

### Global Offset Table (GOT):
Similarly used to resolve addresses. Both PLT and GOT, along with other relocation information, are explained in greater length in related

There are two ways a dynamic linker can relocate global symbols:

1. **Procedure Linkage Table (PLT):**  
   Used to call external procedures/functions whose address isn't known at the time of linking. This is resolved by the dynamic linker at runtime.

2. **Global Offset Table (GOT):**  
   Similarly used to resolve addresses. Both PLT and GOT, along with other relocation information, are explained in greater length in this article.



## Let’s Get Coding!

This code is from sploitfun.

```c
#include <stdio.h>
#include <string.h>

/*
 * Even though shell() function isn't invoked directly, it's needed here since
 * 'system@PLT' and 'exit@PLT' stub code should be present in the executable to
 * successfully exploit it.
 */
void shell() {
    system("/bin/sh");
    exit(0);
}

int main(int argc, char *argv[]) {
    int i = 0;
    char buf[256];
    strcpy(buf, argv[1]);
    printf("%s\n", buf);
    return 0;
}
```

As I'm taking apart this binary after seeking to `main`, I realize I don't see the function call for the function `shell()`. But that's because `main` doesn't call it. I'd like to learn how to find all the functions in a binary.  

At first, I ran the `afl` command but didn’t get any results. This is because `r2` needs to analyze the binary first. So use `aaa`.

```bash
$ aaa
$ afl
```
![aslr-ex2](/assets/img/aslr-ex2.png "aslr-ex2"){:class="featured-image"}

All the functions that are native to the binary start with sym. So the function we created, yet never called in main, is named sym.shell, respectively. Again, remember I use the "s" for "seek."

Let's analyze sym.shell now.
To analyze `sym.shell`, you can use the following steps with Radare2:

```bash
> s sym.shell
> pdf
```

The s command is used to `seek` to the address of the function, and pdf will print the disassembly of the function.

Let's analyze sym.shell now.
![aslr-ex3](/assets/img/aslr-ex3.png "aslr-ex3"){:class="featured-image"}


## References

- [Bypassing ASLR Part I by sploitfun](https://sploitfun.wordpress.com/2015/05/08/bypassing-aslr-part-i/)
- [Wiki: Address Space Layout Randomization (ASLR)](https://en.wikipedia.org/wiki/Address_space_layout_randomization)
- [What is PLT?](https://reverseengineering.stackexchange.com/questions/1992/what-is-plt-got)
- [Radare2 Cheat Sheet by Zach Grace](https://zachgrace.com/cheat_sheets/radare2.html)
- [Jump Over ASLR: Attacking Branch Predictors to Bypass ASLR](https://chris-magistrado-c74h.squarespace.com/s/Jump-Over-ASLR-Attacking-Branch-Predictors-to-Bypass-ASLR.pdf)
- [Wikipedia: Position Independent Code](https://en.wikipedia.org/wiki/Position-independent_code)
- [Radare2 YouTube Channel](https://www.youtube.com/channel/UClcE-kVhqyiHCcjYwcpfj9w)
- [Linux Internals: Dynamic Linking Wizardry on 0x00sec](https://0x00sec.org/t/linux-internals-dynamic-linking-wizardry/1082)
- [Linux Internals: The Art of Symbol Resolution on 0x00sec](https://0x00sec.org/t/linux-internals-the-art-of-symbol-resolution/1488)
- [Reverse Engineering 101: PLT and GOT – The Key to Code Sharing and Dynamic Libraries](https://www.technovelty.org/linux/plt-and-got-the-key-to-code-sharing-and-dynamic-libraries.html)
- [System Overlord: GOT and PLT for Pwning](https://systemoverlord.com/2017/03/19/got-and-plt-for-pwning.html)
- [Wikipedia: Process Environment Block (PEB)](https://en.wikipedia.org/wiki/Process_Environment_Block)
- [Microsoft: PEB Structure](https://msdn.microsoft.com/en-us/library/windows/desktop/aa813706(v=vs.85).aspx)
- [Wikipedia: Win32 Thread Information Block (TIB)](https://en.wikipedia.org/wiki/Win32_Thread_Information_Block)
- [WehnTrust Documentation](https://wehntrust.codeplex.com/SourceControl/latest#documentation/README.txt)
- [Security Stack Exchange: Does Your Program Have NX/XD and/or ASLR Enabled?](https://security.stackexchange.com/questions/58528/how-can-i-check-if-a-mac-application-has-nx-or-aslr-enabled)
