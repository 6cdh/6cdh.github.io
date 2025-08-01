---
title: C++ 中的 virtual 函数和 RTTI
date: 2020-09-23 02:52:13
tags:
  - c++
updated: 2020-11-21
---

C++ 中的 virtual 说明符指定非静态成员函数为虚函数, 并支持基类和派生类的运行时调用. 只要在基类和派生类中重载虚函数, 当使用基类的引用或指针指向派生类的对象时, 对该指针或引用调用虚函数依然可以调用派生类的虚函数, 判断要调用哪个虚函数是在运行时实现的. RTTI (Run-Time Type Information) 是一种机制, 允许在运行时判断对象的类型信息. 同样是运行时, 两者的实现实际上是相似的. 这很有趣, 本文会深入一下它们的原理.

<!--more-->

## Virtual 函数

如下的 C++ 代码说明了虚函数的用法.

```c++
#include <iostream>

struct Base {
    virtual void vf() { std::puts("Base::vf()"); }
};

struct Derived : Base {
    void vf() override { std::puts("Derived::vf()"); }
};

int main() {
    Derived derived_obj;
    Base &base_ref = derived_obj;
    base_ref.vf();
}
```

这个例子创建了一个 Derived 类的对象 derived_obj, 并使用一个指向 Base 类的引用 base_ref 来指向它. Base 类和 Child 类都重写了虚函数 vf(), 最后对 base_ref 调用 vf().

编译并执行, 会产生如下输出:

```shell
Derived::vf()
```

显然, 在运行时决定了要调用 base_ref 指向的实际对象的虚函数.

虚函数 Child::vf() 中的 override 说明符指定虚函数 Child::vf() 重载了 Base::vf().

下面再说一下 RTTI.

## RTTI

RTTI 包含三部分:

- dynamic_cast 操作符. 它用于在继承的层次结构中向上 (向基类的方向) 或向下 (向派生类的方向) 或横向 () 安全的转换指针和引用.
- typeid 操作符. 它用于查询对象的类型.
- type_info 类. 它是一个特定于实现 _(Implementation-specific)_ 的类, 也就是依赖于标准库和编译器的实现而不同. 该类包含类的名称和比较两个类型是否相等或相对大小的方法.

一个例子:

```c++
struct Base {
    int m_base{};
    virtual void vf() { std::puts("Base::vf()"); }
};

struct Derived : Base {
    int m_derived{};
    void vf() override { std::puts("Derived::vf()"); }
};

int main() {
    Base base_obj;
    Derived derived_obj;

    // 输出 base_obj 和 derived_obj 的类型名称
    std::puts(typeid(base_obj).name());
    std::puts(typeid(derived_obj).name());

    // 用 base_ref 指向 derived_obj
    Base &base_ref = dynamic_cast<Base &>(derived_obj);

    // 输出 base_ref 指向的对象的类型名称
    std::puts(typeid(base_ref).name());

    base_ref.vf();
}
```

typeid 操作符返回 type_info 对象, type_info 类的 name() 方法原型如下:

```c++
const char *name() const noexcept;
```

编译并执行产生如下输出:

```shell
4Base
7Derived
7Derived
Derived::vf()
```

可以看出 type_info 的 name() 方法返回的名字的格式是类名的长度加类名.

## 从汇编看运行时的实现

将 C++ 代码编译成汇编语言是一个探索 C++ 机制的有用技术, 可以看到一些有趣的事实. 下面在 64 位平台上使用 clang 10.0.1 和 flag `-Og` 对上面 RTTI 一节的代码进行编译, 得到如下的 Intel 汇编代码:

```asm
main:                                   # @main
        push    rbx
        sub     rsp, 32
        lea     rdi, [rsp + 16]
        call    Base::Base() [base object constructor]
        mov     rbx, rsp
        mov     rdi, rbx
        call    Derived::Derived() [base object constructor]
        mov     rax, qword ptr [rsp + 16]
        mov     rdi, qword ptr [rax - 8]
        call    std::type_info::name() const
        mov     rdi, rax
        call    puts
        mov     rax, qword ptr [rsp]
        mov     rdi, qword ptr [rax - 8]
        call    std::type_info::name() const
        mov     rdi, rax
        call    puts
        mov     rax, qword ptr [rsp]
        mov     rdi, qword ptr [rax - 8]
        call    std::type_info::name() const
        mov     rdi, rax
        call    puts
        mov     rax, qword ptr [rsp]
        mov     rdi, rbx
        call    qword ptr [rax]
        xor     eax, eax
        add     rsp, 32
        pop     rbx
        ret
Base::Base() [base object constructor]:                           # @Base::Base() [base object constructor]
        mov     qword ptr [rdi], offset vtable for Base+16
        mov     dword ptr [rdi + 8], 0
        ret
Derived::Derived() [base object constructor]:                        # @Derived::Derived() [base object constructor]
        push    rbx
        mov     rbx, rdi
        call    Base::Base() [base object constructor]
        mov     qword ptr [rbx], offset vtable for Derived+16
        mov     dword ptr [rbx + 12], 0
        pop     rbx
        ret
std::type_info::name() const:                # @std::type_info::name() const
        mov     rcx, qword ptr [rdi + 8]
        lea     rax, [rcx + 1]
        cmp     byte ptr [rcx], 42
        cmovne  rax, rcx
        ret
Base::vf():                          # @Base::vf()
        push    rax
        mov     edi, offset .L.str
        call    puts
        pop     rax
        ret
Derived::vf():                       # @Derived::vf()
        push    rax
        mov     edi, offset .L.str.1
        call    puts
        pop     rax
        ret
vtable for Base:
        .quad   0
        .quad   typeinfo for Base
        .quad   Base::vf()

typeinfo name for Base:
        .asciz  "4Base"

typeinfo for Base:
        .quad   vtable for __cxxabiv1::__class_type_info+16
        .quad   typeinfo name for Base

.L.str:
        .asciz  "Base::vf()"

vtable for Derived:
        .quad   0
        .quad   typeinfo for Derived
        .quad   Derived::vf()

typeinfo name for Derived:
        .asciz  "7Derived"

typeinfo for Derived:
        .quad   vtable for __cxxabiv1::__si_class_type_info+16
        .quad   typeinfo name for Derived
        .quad   typeinfo for Base

.L.str.1:
        .asciz  "Derived::vf()"
```

使用 [godbolt.org](https://godbolt.org/z/q17x4K) 可以在线编译并查看结果.

为了防止看不懂汇编, 这里将它转换成类 C++ 的伪代码. 如下:

```c++
int main() {
    // 分配 4 * 8 = 32 的空间
    // +--------+ <<------ stack + 32
    // |        |
    // +--------+ <<------ stack + 24
    // |        |
    // +--------+ <<------ stack + 16
    // |        |
    // +--------+ <<------ stack + 8
    // |        |
    // +--------+ <<------ stack 或者说 rsp
    char *stack[32];
    char *rax, *rdi;

    // 在 stack + 16 处构造 Base 对象 base_obj
    Base::Base(stack + 16);
    // 在 stack 处构造 Derived 对象 derived_obj
    Derived::Derived(stack);
    // 现在栈的内容如下
    // +----------------------------+ <<------ stack + 32
    // |   4 bytes  |  m_base = 0   |
    // +----------------------------+ <<------ stack + 24
    // |   &vtable_for_Base + 16    |
    // +----------------------------+ <<------ stack + 16
    // | m_derived = 0 | m_base = 0 |
    // +----------------------------+ <<------ stack + 8
    // |  &vtable_for_Derived + 16  |
    // +----------------------------+ <<------ stack

    rax = *(long *)(stack + 16); // rax = vtable_for_Base + 16;
    rax = std::type_info::name(*(long *)(rax - 8)); // 传进去的实参实际上是 *(vtable_for_Base + 8)
    std::puts(rax);

    rax = *(long *)(stack); // rax = vtable_for_Derived + 16;
    rax = std::type_info::name(*(long *)(rax - 8)); // 传进去的实参实际上是 *(vtable_for_Derived + 8)
    std::puts(rax);

    char *base_ref = stack;
    rax = *(long *)(base_ref); // rax = vtable_for_Derived + 16;
    rax = std::type_info::name(*(long *)(rax - 8)); // 传进去的实参实际上是 *(vtable_for_Derived + 8)
    std::puts(rax);

    rax = *(long *)stack; // rax = vtable_for_Derived + 16
    function vf = *(long *)rax; // vf = *(vtable_for_Derived + 16)
    vf(base_ref);
}

Base::Base(char *rdi) {
    // 将 rdi 指向的空间初始化为
    // +----------------------+ <<------ rdi + 16
    // | 4 bytes  | m_base = 0|
    // +----------------------+ <<------ rdi + 8
    // |&vtable_for_Base + 16 |
    // +----------------------+ <<------ rdi
    *(long *)rdi = vtable_for_Base + 16;
    *(int *)(rdi + 1) = 0;
}

Derived::Derived(char *rdi) {
    // 将 rdi 指向的空间初始化为
    // +----------------------------+ <<------ rdi + 16
    // | m_derived = 0 | m_base = 0 |
    // +----------------------------+ <<------ rdi + 8
    // |  &vtable_for_Derived + 16  |
    // +----------------------------+ <<------ rdi
    Base::Base(rdi);
    *(long *)rdi = vtable_for_Derived + 16;
    *(int *)(rdi + 12) = 0;
}

const char *std::type_info::name(char *rdi) const {
    // rdi 等于 typeinfo_for_Base 或 typeinfo_for_Derived
    // rcx = "4Base" 或 "7Derived"
    char *rcx = *(rdi + 8);
    // rax = "Base" 或 "Derived"
    char *rax = rcx + 1;
    // if rax[0] != '*'
    if (*rcx != 42) {
        // rax = "4Base" 或 "7Derived"
        rax = rcx;
    }
    return rax;
}

Base::vf(char *rdi) {
    puts("Base::vf()");
}

Derived::vf(char *rdi) {
    puts("Derived::vf()");
}

long vtable_for_Base[3] = {
    0,
    typeinfo_for_Base,
    Base::vf
};
char *typeinfo_name_for_Base = "4Base";
long typeinfo_for_Base[2] = {
    vtable_for___cxxabiv1::__class_type_info + 16,
    typeinfo_name_for_Base
};

long vtable_for_Derived[3] = {
    0,
    typeinfo_for_Derived,
    Derived::vf
};
char *typeinfo_name_for_Derived = "7Derived";
long typeinfo_for_Derived[3] = {
    vtable_for___cxxabiv1::__si_class_type_info + 16,
    typeinfo_name_for_Derived,
    typeinfo_for_Base
};
```

如果类 Base 没有虚函数, 即

```c++
struct Base {
    int m_base;
};
```

那么 sizeof(Base) 会返回 4, 即它的成员 m_base 的长度. 而增加了虚函数 vf() 的 Base 类会在成员变量前面加上一个指向 `vtable_for_{classname} + 16` 的指针, 即指向虚函数表的指针, 其中`{classname}` 是类的名字, 此时 Base 类的大小增加到了 12, 由于结构体对齐, Base 的大小会进一步增长到 16, sizeof(Base) 得到的结果也是 16.

vtable 是 virtual method table (VMT) 或 virtual function table 的缩写, 用于运行时选择虚函数, 访问虚基类对象和 RTTI.

- 其第一个元素是 offset to top (8 bytes). 上面的例子中为 0.
- 第二个元素是 `typeinfo_for_{classname}` 的地址 (8 bytes), 用于 RTTI.
- 从第三个元素开始都是类的虚函数的指针, 这些指针按照声明的顺序排序.

在 `Base &base_ref = dynamic_cast<Base &>(derived_obj);` 执行后, 对 base_ref 调用虚函数 vf() 时, 会直接解引用它的第一个指针, 即所谓的虚函数表的指针, 从而得到要调用的虚函数的地址, 如此实现了虚函数的运行时选择和调用.

`typeinfo_for_{classname}` 中第二个元素是类型名称, 用于 RTTI 在运行时进行一些操作.

## 总结

虚函数和 RTTI 虽然看起来没什么关系, 但由于其运行时的特性, 都是在 vtable 中实现的. 如果对 vtable 实现的细节感兴趣, 可以参考 [Itanium ABI](https://itanium-cxx-abi.github.io/cxx-abi/abi.html#vtable) 的相关约定.

## References

- [virtual function specifier - cppreference.com](https://en.cppreference.com/w/cpp/language/virtual)
- [Run-Time Type Information | Microsoft Docs](https://docs.microsoft.com/en-us/cpp/cpp/run-time-type-information?view=vs-2019)
- [Virtual Functions | Microsoft Docs](https://docs.microsoft.com/en-us/cpp/cpp/virtual-functions?view=vs-2019)
