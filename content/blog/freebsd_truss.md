---
date: "2011-12-30T17:53:00-07:00"
title: "In assembly language we truss"
slug: "in-assembly-language-we-truss"
---

[Assembly language programming](http://en.wikipedia.org/wiki/Assembly_language) is almost a lost art. Piecing together [opcode](http://en.wikipedia.org/wiki/Opcode) mnemonics from the [x86](http://en.wikipedia.org/wiki/X86_instruction_listings) instruction set. Utilizing the interrupts, system calls, or api your operating system offers. Shaving off [bytes](http://en.wikipedia.org/wiki/Byte) and [clock cycles](http://en.wikipedia.org/wiki/Cycles_per_instruction) for efficiency, elegance and bragging rights.

Debugging, loving and loathing.

The "pros" for assembly language have remained unchanged. Even if the code is poorly written it will produce a better optimized binary than any high level language compiler could spit out. No wasted resources. Few constraints from the operating system. Direct interaction with hardware. Blazingly fast software.

The use of assembly language has migrated off major platforms to the small niche of device drivers and embedded microprocessors. Development in assembly is slow and error prone. As computational power grew, so did the ability to timely execute robust applications, evolving to our current complexity. The need to quickly develop these applications spurred the growth of high level languages such as [COBOL](http://en.wikipedia.org/wiki/COBOL), [BASIC](http://en.wikipedia.org/wiki/BASIC), [PASCAL](http://en.wikipedia.org/wiki/Pascal_(programming_language)), and [C](http://en.wikipedia.org/wiki/C_(programming_language)). The latter is the only one of these legacy languages which have survived and flourished. C remains the foundation of open source operating systems and software, and has provided a framework for most popular scripting languages such as [SHELL](http://en.wikipedia.org/wiki/Shell_script), [PERL](http://en.wikipedia.org/wiki/Perl) and [PYTHON](http://en.wikipedia.org/wiki/Python_(programming_language)).

So why bother? The old adage goes "If you are fluent in assembly, then no program is closed source".

The problem with sorting through software compiled from high level languages is the amount of bloat and library garbage you must sift through to view desired routines. Modern tools such as [IDA](http://www.hex-rays.com/products/ida/index.shtml), [OllyDbg](http://www.ollydbg.de), and [Edb](http://codef00.com/projects#debugger) make this job manageable. None of these tools are available on my operating system of choice. [ALD](http://ald.sourceforge.net/) runs on the i386 build of FreeBSD and is available in the ports tree. It has not been updated since 2004 and its work flow is less than ideal. It is still easier to work with than trying to use [gdb](http://www.gnu.org/software/gdb/) on [nasm](http://www.nasm.us/) assembled static binaries.

This does not leave us with much. [FreeBSD](http://www.freebsd.org/) is an operating system written in C, and is open source. The is no widespread need for binary debuggers on this platform.

If you are familiar with [strace](http://en.wikipedia.org/wiki/Strace) for GNU/Linux systems, you will quickly pick up [truss(1)](http://www.freebsd.org/cgi/man.cgi?query=truss&amp;apropos=0&amp;sektion=0&amp;manpath=FreeBSD+8.2-RELEASE&amp;arch=default&amp;format=html). Both these utilities trace the calls made to the kernel during a programs execution. The following is the truss output of our [example program found here](https://github.com/eholzbach/assembly/blob/master/wipe.asm), making three passes overwriting auth.log before unlinking it from the file system:

```bash
devel:~/asm% truss ./wipe auth.log
open("auth.log",O_RDWR,027757764434) = 3 (0x3)
lseek(3,0x0,SEEK_END) = 32 (0x20)
lseek(3,0x0,SEEK_SET) = 0 (0x0)
write(3,"\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"...,8192) = 8192 (0x2000)
fsync(0x3,0x4,0x3,0x80491f8,0x2000,0x5) = 0 (0x0)
lseek(3,0x0,SEEK_SET) = 0 (0x0)
write(3,"\M^?\M^?\M^?\M^?\M^?\M^?\M^?\M^?"...,8192) = 8192 (0x2000)
fsync(0x3,0x4,0x3,0x80491f8,0x2000,0x5) = 0 (0x0)
lseek(3,0x0,SEEK_SET) = 0 (0x0)
write(3,"\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"...,8192) = 8192 (0x2000)
fsync(0x3,0x4,0x3,0x80491f8,0x2000,0x5) = 0 (0x0)
lseek(3,0x0,SEEK_SET) = 0 (0x0)
close(3) = 0 (0x0)
unlink("auth.log") = 0 (0x0)
process exit, rval = 10
```

It is also handy to get a summery of calls made, errors flagged, and total system time used:
```bash
devel:~/asm% truss -c ./wipe auth.log
syscall seconds calls errors
lseek 0.000196957 5 0
open 0.000084799 1 0
close 0.000044560 1 0
unlink 0.000122718 1 0
write 0.000219196 3 0

0.000668230 11 0
```
While this is no substitution for a real debugging instance, this makes it trivial to track down a crash or point of error, assuming you are familiar with your code. I tend to include error checking for almost ever routine and phase it out as development progresses. Without the use of a real debugger this will drastically reduce the time it takes to locate a bug. Lets make the following diff to our example program to produce an error:

```bash
30c30
< push ecx
 --
> push eax</pre></blockquote>
```
and execute it with truss:
```bash
devel:~/asm% truss ./wipe auth.log
open("(null)",O_RDWR,027757764460) ERR#14 'Bad address'
fail
write(1,"fail\n",5) = 5 (0x5)
process exit, rval = 4
```
We can clearly see the program did not make it very far, failing to open the file we are trying to wipe. So lets take a look at our code:

```
_start:
pop eax
pop eax
pop ecx
jecxz .sparms
pop eax
or eax,eax
jne .sparms

push dword 2
push eax
mov eax,5
push eax
int 0x80
jc .fail
```

The program starts by pulling in command line arguments. The first being the argument count, the second is the name of the program itself, and the third is auth.log. This ascii pointer has been popped off the stack into ecx. A quick review of the next routine shows we have mistakenly pushed eax to the stack instead of ecx which contains the ascii file pointer, hence the bad address error and the inability to open our file. An easy methodology for resolution. There are also the [ktrace](http://www.freebsd.org/cgi/man.cgi?query=ktrace&amp;sektion=1) and [kdump](http://www.freebsd.org/cgi/man.cgi?query=kdump&amp;sektion=1) utilities which may help resolve work flow issues. While none of this is a replacement for an actual debugger, you can usually find your error with truss before you even need to curse at your debugger. This allows plenty of time to `more /usr/src/sys/kern/syscalls.master` followed by `man 2 stupidsyscall` followed by cursing at the standard C library.

Disassembling small static binaries is very straightforward. Objdump is included in the base and provides the expected results, though if you write in intel syntax be aware it produces slightly different syntax. Here is the same snippet of our program:

```bash
devel:~/asm% objdump -d wipe

wipe: file format elf32-i386-freebsd

Disassembly of section .text:

08048080 <.text>:
8048080: 58 pop %eax
8048081: 58 pop %eax
8048082: 59 pop %ecx
8048083: e3 70 jecxz 0x80480f5
8048085: 58 pop %eax
8048086: 09 c0 or %eax,%eax
8048088: 75 6b jne 0x80480f5
804808a: 6a 02 push $0x2
804808c: 51 push %ecx
804808d: b8 05 00 00 00 mov $0x5,%eax
8048092: 50 push %eax
8048093: cd 80 int $0x80
8048095: 72 45 jb 0x80480dc
```

As you can see all it takes to start producing a readable disassembly of your program is some human friendly address locations and the swap from at&amp;t syntax to my preferred intel syntax.

How often will you use this in the real world? Never, unless your job title is "malware reverse engineer". How much fun is it to tinker with? Boatloads.
