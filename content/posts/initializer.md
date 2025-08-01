---
title: How Compilers Initialize C style array
date: 2021-04-13T16:39:06+08:00
tags: [c++]
---

In C++11, you can initialize all members of a C-style array to zero.

<!--more-->

For example, you can

```c++
int a[10] = {0}; // all elements 0
// or
int a[10] = {}; // all elements 0
```

But how the compilers implement it?

Assume we have this code:

```c++
void f() {
    int a[10] = {};
}
```

With clang 11.0.1, we get this asm code:

```asm
f():                                  # @f()
        push    rbp
        mov     rbp, rsp
        sub     rsp, 48
        xor     esi, esi
        lea     rax, [rbp - 48]
        mov     rdi, rax
        mov     edx, 40
        call    memset
        add     rsp, 48
        pop     rbp
        ret
```

As you see, clang use `memset` to initialize the array.

With gcc 10.3, we get this asm code:

```asm
f():
        push    rbp
        mov     rbp, rsp
        mov     QWORD PTR [rbp-48], 0
        mov     QWORD PTR [rbp-40], 0
        mov     QWORD PTR [rbp-32], 0
        mov     QWORD PTR [rbp-24], 0
        mov     QWORD PTR [rbp-16], 0
        nop
        pop     rbp
        ret
```

It looks like gcc expanded the function call. To get what gcc really do, we use `int a[128]`, then get

```asm
f():
        push    rbp
        mov     rbp, rsp
        sub     rsp, 392
        lea     rdx, [rbp-512]
        mov     eax, 0
        mov     ecx, 64
        mov     rdi, rdx
        rep stosq
        nop
        leave
        ret
```

What `rep stosq` means?

From Intel's documentation, the `stosq` instruction is described as follows:

> Store RAX at address RDI or EDI.

And `rep`

> Repeats a string instruction the number of times specified in the count register or until the indicated condition of
> the ZF flag is no longer met. The REP (repeat), REPE (repeat while equal), REPNE (repeat while not equal), REPZ
> (repeat while zero), and REPNZ (repeat while not zero) mnemonics are prefixes that can be added to one of the
> string instructions.
>
> ...
>
> | Repeat Prefix | Termination Condition |
> | :-----------: | :-------------------: |
> |      REP      |   RCX or (E)CX = 0    |

In the above example, the CPU will fill 8 bytes of 0 each time and repeats 64 times.

An interesting point is when the array is too large, gcc would call `memset` rather than `stosq` instruction.

## References

- [Intel® 64 and IA-32 Architectures Software Developer’s Manual Volume 2 (2A, 2B, 2C & 2D): Instruction Set Reference, A-Z (PDF, 22MiB)](https://software.intel.com/content/dam/develop/external/us/en/documents-tps/325383-sdm-vol-2abcd.pdf)
