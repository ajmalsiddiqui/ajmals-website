---
title: "Calling Rust From C And x86 Assembly"
date: 2024-02-28T21:20:50-05:00
lastmod: 2024-02-28T21:20:50-05:00
author: Mohammed Ajmal Siddiqui
authorlink: /
categories:
  - rust
  - assembly
  - miscellaneous
tags:
  - rust
  - assembly
  - misc
  - programming-languages
showcase: true
draft: false
---

I was curious about how FFIs (Foreign Function Interfaces) work, and how that relates to compiling code into an executable, so I decided to play with trying to call Rust code from... x86 assembly. Why? I don't know, but it was fun and I learnt a thing or two along the way.

<!--more-->

I was already quite familiar with most of the related concepts, and the trivial x86 assembly I'd have to write felt well within the tolerance levels of a reasonably curious programmer, so I was expecting this to be straightforward.

It wasn't.

There are a lot of subtleties and nitty-gritty details that come with trying to manipulate build toolchains at such a low level, and I ran into some head-scratchers along the way. This post is a distillation of some of the things I learned while trying to make Rust, C and x86 assembly play nice together.

This post assumes a Linux environment. You'll need basic familiarity with C and Rust, and you should know (at a high level) what compilers, linkers, object files, etc are. You should know what assembly language is, and what it looks like. I'll provide enough explanation for the minimal assembly that we're going to write today, but any familiarity with x86 assembly is a bonus. Quick "tooltips" with explanations about relevant things are scattered throughout the post.

> Details of the exact versions of rustc, gcc, nasm, etc used here are at the very end of the post.

## Calling Rust from C

Our eventual goal is to call Rust from assembly, but let's start with something a little more tame - trying to call a Rust function from C code. Admittedly, this is the opposite of what you'd usually see in the wild in polyglot codebases that result in a single executable. Normally, you'd write FFI wrappers in Rust that call C functions, often from a library.

> FFI stands for Foreign Function Interface. This is a technique used to call functions written in a language other than the one you're using. Often, this is used by languages like Python or Rust to call C code.

### Code

Here's the Rust code:

```rust
// hello.rs

#[no_mangle]
extern "C" fn hello() {
    println!("Hello from Rust!");
}
```

And here's some C code that calls the `hello` Rust function:

```c
// main.c
extern void hello();

void main() {
    hello();
}
```

The code is trivial, but a couple of things warrant explanation.

### Preventing Name Mangling

By default, the Rust compiler mangles the names of the symbols in the generated object files (I believe it does so to satisfy the linker, which requires that all symbol names be globally unique). This is fine when the rust compiler is solely responsible for building all of your code since it can keep track of the mangled names, but is a problem for us since we wish to call a Rust function from C or assembly so we'd like to know in advance what the name of the function in the generated object file will be. The `#[no_mangle]` compiler directive asks the compiler to not mangle the name of the function after it, thus letting us refer to the function as `hello` in C or assembly. You can see the difference using the `nm` tool.

The `nm` tool lists the symbols in an object file, and you can use that to see the difference in action. When run with the `#[no_mangle]` directive, here is what our `hello` function is like (see the third entry):

```
❯ nm 2>/dev/null libhello.a| grep hello
hello.hello.a04ca2dc-cgu.0.rcgu.o:
hello.hello.a04ca2dc-cgu.1.rcgu.o:
0000000000000000 T hello
hello.3wtkpwhk4eo6pr7y.rcgu.o:
```

If you remove the `extern "C"` directive and compile the code again, you no longer see a symbol associated with `hello` (the 2 things in the output are names of object files present inside the ar archive that represents this static lib):

```
❯ nm 2>/dev/null libhello.a| grep hello
hello.hello.a04ca2dc-cgu.0.rcgu.o:
hello.3wtkpwhk4eo6pr7y.rcgu.o:
```

### Specifying The Calling Convention

The `extern "C"` directive is less trivial. It has to do with the fact that the Rust does not have a defined calling convention, so any C or assembly code that links with it does not know where the Rust function expects to see it's arguments or store the return value. However, using the `extern "C"` directive instructs the compiler to use the C calling convention instead of the default (and undefined) rust calling convention, which allows C code to call it.

> The calling convention is a contract between functions about how arguments and return values are passed between them. These are very low level implementation details - which CPU registers or what offsets on the stack frame are used to store arguments, etc. The calling convention will depend on both the processor architecture and compiler since the former determines what registers are available, and the latter decides how to use them. The calling convention is undefined for the Rust compiler (i.e. it is an implementation detail that can change without notice), but fortunately for us, the C calling convention for x86 is defined. Find some of the gory details [here](https://en.wikipedia.org/wiki/X86_calling_conventions#List_of_x86_calling_conventions).

The astute reader will notice that the Rust function in question (i.e. `hello`) does not take any arguments, and thus posit that the calling convention shouldn't matter in this context. And said reader would be correct! If you remove the `extern "C"` directive from the Rust code and follow the above steps, things still work. However, I wouldn't ever advise omitting the `extern "C"` directive even for functions that don't take arguments or return things, at least for consistency and to clearly indicate the potential use of FFI here.

### Baking An Executable

Now let's compile these two into a single executable. You're likely familiar with using `gcc` or `rustc` (or `cargo` which calls `rustc` under the hood) to compile your source code directly into an executable binary, but there are actually many stages in the compilation process. Since we're doing something non-trivial by mixing in two languages with different compilers, we'll have to work through some of these stages manually.

The trick is to compile each of the parts into object files, and then link them all together using the linker. This is what compilers do internally as well.

>I had some trouble with getting a single object file to work on the Rust side, so I compiled it into a static library instead. A static library is just an ar archive that contains a bunch of object files. You can take it apart using `ar x libhello.a` if you're curious. The C code is compiled into a usual object file.

```sh
# Compiles hello.rs into a static lib called libhello.a
rustc --crate-type staticlib hello.rs

# The -c flag tells gcc to generate an object file instead of an executable
# The resulting object file is called main.o
gcc -c main.c

# Link the object file and static lib into an executable
gcc -o main main.o libhello.a
```

> The order of arguments to gcc in the last command matters! Here's what happens if you switch main.o and libhello.a:
> ```
> gcc -o main libhello.a main.o
> /usr/bin/ld: main.o: in function `main':
> main.c:(.text+0xa): undefined reference to `hello'
> collect2: error: ld returned 1 exit status
> ```
> The error comes from gcc which calls ld (the linker) internally. The arguments to ld must be such that the thing referencing a symbol (in this case `main.o`, which references our rust function `hello`) must come before the thing defining it (in this case, the `libhello.o` static library. See [this stackoverflow answer](https://stackoverflow.com/questions/45135/why-does-the-order-in-which-libraries-are-linked-sometimes-cause-errors-in-gcc) for more details.

This results in an executable called `main` which can be run normally:

```sh
❯ ./main 
Hello from Rust!
```

## Calling Rust From x86 Assembler

Now that we understand how to call Rust code from C, we can tackle calling it from assembly. We'll only encounter a couple of speedbumps here, the first of which is the blank stare you're about to have if you're not familiar with x86 assembly.

Without further ado, you may now feast your eyes:

```asm
; main.asm

section .text
	global _start

_start:
    ; Declare an external function called hello
	extern hello

    ; Call the hello function (i.e. jump to the symbol called hello)
	call hello

    ; Exit cleanly
	mov rbx, 0 ; return code 0
	mov rax, 1 ; syscall 1: SYS_EXIT
	int 0x80 ; interrupt 0x80 means make syscall
```

That was likely at least a little underwhelming if you generally consider raw assembly an Eldritch horror beyond mortal comprehension. This is exceptionally terse, simple code! Regardless, an explanation is due (skip to the next section if you understand the above snippet).

The `section .text` directive tells the assembler that the instructions that follow should be placed in the `.text` section of the compiled executable. It starts by declaring the `_start` symbol, which is a convention for where code execution must start. We then inform the assembler of an external function called `hello` via the `extern hello` directive (this is no different from the `extern` directive in our C code from the previous section). Finally, we call the `hello` function.

The last three lines are likely the most cryptic here. This is how we cleanly exit a program in x86 assembly. We're invoking the exit syscall on Linux with the argument 0, which tells the OS that this program exited cleanly. If you care about the hairy details, here they are: we place the syscall number (in this case, 1, which corresponds to the exit syscall on Linux) in the `rax` register, the argument to the syscall (in this case, 0, which indicates success) in `rbx`, and finally raise interrupt number 0x80, which transfers control to the kernel asking it to execute the syscall.

> Terrible things will come to pass if you don't do the dance required to correctly exit from the program! In the absence of the exit syscall, the kernel will attempt to continue executing instructions from the memory after the last instruction (in this case, `call hello`) has completed. Since the memory likely contains garbage instead of coherent instructions (likely in non-executable memory regions), the kernel will barf out a segfault!

### Baking Another Executable

We'll be using nasm (netwide assembler) to transform our handrolled assembly into an object file. We can then try to link the object file with `libhello.a` like we did before.

Compile assembler with confidence:

```sh
# Compiles hello.rs into a static lib called libhello.a
rustc --crate-type staticlib hello.rs

# Compile the assembly program into an object file
# We tell nasm that our object file should be a 64 bit ELF binary
nasm -f elf64 main.asm
```

> The -f elf64 flag tells nasm that we want an object file in the ELF format. ELF stands for Executable and Linkable Format and is a standard format for executables and object files on Linux.

And then watch the whole thing blow up:

```sh
❯ gcc -o main main.o libhello.a
/usr/bin/ld: main.o: in function `_start':
main.asm:(.text+0x0): multiple definition of `_start'; /usr/lib/gcc/x86_64-redhat-linux/13/../../../../lib64/crt1.o:(.text+0x0): first defined here
/usr/bin/ld: /usr/lib/gcc/x86_64-redhat-linux/13/../../../../lib64/crt1.o: in function `_start':
(.text+0x1b): undefined reference to `main'
collect2: error: ld returned 1 exit status
```

That's surprising. Multiple definitions of `_start`? We only have one definition in our assembly code. And then there's an undefined reference to `main` even though we never reference anything called `main` anywhere.

### How `main` Works

Resolving this error is easy, but requires understanding how a usual compiled executable works. We're used to our programs executing the main function as the entrypoint, but that is not where the execution actually starts. Execution starts from a special symbol called `_start`. That is why we defined exactly this symbol in our assembly code.

> If you define a start symbol with any label other than `_start`, your code will compile. The linker will throw a benign looking warning. And then it will blow up in miserable ways (spoiler: segfault).

But consider what happens when you compile a normal C program. You define a `main` function that does what you want it to do, but where's the `_start` function, and what does that do? You'll likely guess that `_start` in a normal C program does some initialization work and then calls `main`, which is a good guess. A hint about where the `_start` function for normal C programs is defined lies in the error log above. It says that the `_start` function was first defined in an object file called `crt1.o`. This file obviously doesn't come from our code. It refers to the C runtime, which defines a `_start` function that eventually calls a function called `main`. This is why the entrypoint of a C program must always be named `main`. This then explains our second error, the one about an undefined reference to `main`. We never defined a main function anywhere, so naturally this symbol is undefined. As you might've guessed, the C runtime is linked against by default when baking executables with gcc. This makes perfect sense for vanilla C programs that define a main and rely on the C runtime to set things up for them, but trips us up when we try to take the reins and write code that is designed to be the very first thing that executes.

The fix is trivial - you can ask gcc to not link to the C runtime by supplying the `-nostartfiles` argument.

```sh
gcc -o main main.o libhello.a -nostartfiles
```

Having done that, we finally have a binary called main that, when executed, calls a Rust library function! \o/

```sh
❯ ./main 
Hello from Rust!
```

## Conclusion

I still don't know what might be a good use case for calling a Rust library function from assembly. But getting this trivial example to work taught me a lot of the nitty-gritty details of compiler toolchains and what makes an executable run. So I hope the learning outcomes of this post were worth it.

## Toolchain Details

```sh
$ rustc --version
rustc 1.76.0 (07dca489a 2024-02-04)

$ gcc -v
Using built-in specs.
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/libexec/gcc/x86_64-redhat-linux/13/lto-wrapper
OFFLOAD_TARGET_NAMES=nvptx-none
OFFLOAD_TARGET_DEFAULT=1
Target: x86_64-redhat-linux
Configured with: ../configure --enable-bootstrap --enable-languages=c,c++,fortran,objc,obj-c++,ada,go,d,m2,lto --prefix=/usr --mandir=/usr/share/man --infodir=/usr/share/info --with-bugurl=http://bugzilla.redhat.com/bugzilla --enable-shared --enable-threads=posix --enable-checking=release --enable-multilib --with-system-zlib --enable-__cxa_atexit --disable-libunwind-exceptions --enable-gnu-unique-object --enable-linker-build-id --with-gcc-major-version-only --enable-libstdcxx-backtrace --with-libstdcxx-zoneinfo=/usr/share/zoneinfo --with-linker-hash-style=gnu --enable-plugin --enable-initfini-array --with-isl=/builddir/build/BUILD/gcc-13.2.1-20231205/obj-x86_64-redhat-linux/isl-install --enable-offload-targets=nvptx-none --without-cuda-driver --enable-offload-defaulted --enable-gnu-indirect-function --enable-cet --with-tune=generic --with-arch_32=i686 --build=x86_64-redhat-linux --with-build-config=bootstrap-lto --enable-link-serialization=1
Thread model: posix
Supported LTO compression algorithms: zlib zstd
gcc version 13.2.1 20231205 (Red Hat 13.2.1-6) (GCC)

$ ld -v
GNU ld version 2.40-13.fc39

$ nasm -v
NASM version 2.16.01 compiled on Jul 20 2023
```
