---
title: How new is implemented
date: 2021-04-21T23:41:50+08:00
tags: [c++]
---

You may be asked what's the difference between `new` and `malloc`. In fact, they are completely different. However, in a sense, they are almost no difference.

<!--more-->

We wrote this code:

```c++
int main() {
    auto ptr = new int;
    delete ptr;
}
```

then compiled with clang 12:

```asm
main:                                   # @main
        push    rbp
        mov     rbp, rsp
        sub     rsp, 32
        mov     dword ptr [rbp - 4], 0
        mov     edi, 4
        call    operator new(unsigned long)
        mov     qword ptr [rbp - 16], rax
        mov     rax, qword ptr [rbp - 16]
        mov     qword ptr [rbp - 24], rax       # 8-byte Spill
        cmp     rax, 0
        je      .LBB1_2
        mov     rdi, qword ptr [rbp - 24]       # 8-byte Reload
        call    operator delete(void*)
.LBB1_2:
        mov     eax, dword ptr [rbp - 4]
        add     rsp, 32
        pop     rbp
        ret
```

The `new` and `delete` expressions are replaced by the `operator new` and `operator delete` functions. These functions don't exist in C++ standard library. So, they should be implementation-specific.

Let's see libstdc++ source for `operator new`

```c++
// gcc/libstdc++-v3/libsupc++/new_op.cc
_GLIBCXX_WEAK_DEFINITION void *
operator new (std::size_t sz) _GLIBCXX_THROW (std::bad_alloc)
{
  void *p;

  /* malloc (0) is unpredictable; avoid it.  */
  if (__builtin_expect (sz == 0, false))
    sz = 1;

  while ((p = malloc (sz)) == 0)
    {
      new_handler handler = std::get_new_handler ();
      if (! handler)
	_GLIBCXX_THROW_OR_ABORT(bad_alloc());
      handler ();
    }

  return p;
}

// gcc/libstdc++-v3/libsupc++/new_opv.cc
_GLIBCXX_WEAK_DEFINITION void*
operator new[] (std::size_t sz) _GLIBCXX_THROW (std::bad_alloc)
{
  return ::operator new(sz);
}
```

The `operator new` function simply calls `malloc`. If failed, `new_handler` will be used to solve it. If `new_handler` is a null pointer, `std::bad_alloc` will be raised. No matter what method, the new expression would succeed to allocate memory if it returns.

If `new_handler` really does nothing, new expression will block.

Therefore, the following program will block.

```c++
#include <new>

void handler() {
}

int main() {
    auto new_handler = std::get_new_handler();
    std::set_new_handler(handler);
    auto ptr = new int[10'000'000'000];
    delete[] ptr;
}
```

Another unexpected thing is `new[]` just simply call `new`. It's amazing. Maybe we can mix `new` and `new[]`? They just do the same thing.

Let's see how `delete` works.

```c++
// gcc/libstdc++-v3/libsupc++/del_op.cc
_GLIBCXX_WEAK_DEFINITION void
operator delete(void* ptr) noexcept
{
  std::free(ptr);
}

// gcc/libstdc++-v3/libsupc++/del_opv.cc
_GLIBCXX_WEAK_DEFINITION void
operator delete[] (void *ptr) _GLIBCXX_USE_NOEXCEPT
{
  ::operator delete (ptr);
}
```

`delete` and `delete[]` are wrappers of `free`. They also do the same thing!

That means we might be able to mix `new`, `new[]`, `delete` and `delete[]`.

For example

```c++
int main() {
    auto ptr = new int[10'000];
    delete ptr;
}
```

would not leak memory. We can use valgrind to verify it:

```shell
$valgrind ./a.out
==233263== Memcheck, a memory error detector
==233263== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
==233263== Using Valgrind-3.16.1 and LibVEX; rerun with -h for copyright info
==233263== Command: ./a.out
==233263==
==233263== Mismatched free() / delete / delete []
==233263==    at 0x483FEAB: operator delete(void*) (vg_replace_malloc.c:584)
==233263==    by 0x10918A: main (in /home/lcdh/work/experimental/test/a.out)
==233263==  Address 0x4d96c80 is 0 bytes inside a block of size 40,000 alloc'd
==233263==    at 0x483F50F: operator new[](unsigned long) (vg_replace_malloc.c:431)
==233263==    by 0x109168: main (in /home/lcdh/work/experimental/test/a.out)
==233263==
==233263==
==233263== HEAP SUMMARY:
==233263==     in use at exit: 0 bytes in 0 blocks
==233263==   total heap usage: 2 allocs, 2 frees, 112,704 bytes allocated
==233263==
==233263== All heap blocks were freed -- no leaks are possible
==233263==
==233263== For lists of detected and suppressed errors, rerun with: -s
==233263== ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 0 from 0)
```

Really no leaks! Don't mind the error at the end. It just complains Mismatched `free() / delete / delete []`.

The implementation of libcxx is almost the same as libstdc++.

In conclusion, we can mix `new`, `new[]`, `delete` and `delete[]`. They call `malloc` and `free` to allocate and free memory. However, don't really mix them. Don't assume anything about implementation defined-behavior!
