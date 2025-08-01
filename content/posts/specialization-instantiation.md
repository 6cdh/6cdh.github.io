---
title: 模板特化与实例化
date: 2021-01-16 10:42:26
updated: 2021-01-16
tags:
  - c++
  - template metaprogramming
---

模板特化和实例化有时挺让人迷惑.

<!--more-->

## 模板特化与实例化的语法

首先, 我们定义一个普通的模板类 (也称 primary template).

```c++
template <class T>
class A {};
```

模板的特化分为全特化和部分特化. 模板的显式 (全) 特化 _(explicit (full) specialization)_ 使用形如

```plaintext
template <> declaration
```

的语法:

```c++
template <>
class A<int> {};
```

模板的部分特化 _(partial specialization)_ 使用形如

```plaintext
template <parameter-list>
class-key class-head-name <argument-list> declaration
```

的语法:

```c++
template <class T>
class A<T*> {};
```

模板的实例化分为显式实例化和隐式实例化. 显式实例化 _(explicit instantiation)_ 也有特殊的语法:

```plaintext
template class-key template-name <argument-list>;
```

例如:

```c++
template class A<long>;
```

注意显式实例化不能有定义, 且会实例化模板类中未使用的成员.

关于隐式实例化, [cppreference]() 给出了一些解释:

> When code refers to a template in context that requires a completely defined type, or when the completeness of the type affects the code, and this particular type has not been explicitly instantiated, implicit instantiation occurs. For example, when an object of this type is constructed, but not when a pointer to this type is constructed.
>
> This applies to the members of the class template: unless the member is used in the program, it is not instantiated, and does not require a definition.

这有些难懂, 看看下面的例子:

```c++
template <class T>
class A {}; // primary template

template <>
class A<int> {}; // explicit specialization

template <class T>
class A<T*> {}; // partial specialization

template class A<long>; // explicit instantiation

A<char> ac; // Implicit instantiation from primary template 'A<char>'
A<int> ai; // No instantiation for explicit specialization 'A<int>'
A<int*> aip; // Implicit instantiation from partial specialization 'A<int*>'
A<long> al; // No instantiation for explicit instantiation 'A<long>'
```

## 使用显式实例化减少编译时间

不过既然有隐式实例化, 为什么还要有显式实例化呢? 特别是显式实例化的语法和显式特化的语法还很像.

显式实例化在创建使用模板并进行分发的库时很有用, 未实例化的模板定义不会放在可重定位的对象文件 _(relocatable object file)_ 中.

另外, 显式实例化还可以减少编译时间.

假如我们有三个文件:

```c++
// a.h
#pragma once

template <unsigned N>
struct fac {
    static int value() { return fac<N - 1>::value() + fac<N - 2>::value(); }
};

template <>
struct fac<0> {
    static int value() { return 1; }
};

template <>
struct fac<1> {
    static int value() { return 1; }
};

constexpr int N = 10000;

// a.cc
#include "a.h"

#include <iostream>

#ifdef EXPLICIT_INSTANTIATION
extern template struct fac<N>;
#endif

extern int fb();

int main() {
    std::cout << fac<N>::value() + fb();
}

// b.cc
#include "a.h"

#ifdef EXPLICIT_INSTANTIATION
template struct fac<N>;
#endif

int fb() {
    return fac<N>::value();
}
```

在 a.h 中, 我们定义了一个模板 fac, 然后在 a.cc 和 b.cc 中分别使用 fac<10000>, fac<10000> 会在这两个翻译单元中分别实例化一次, 每一次都会花费很长时间. 另外, 我们添加了一些由宏 `EXPLICIT_INSTANTIATION` 控制的显式实例化的代码, 在 b.cc 中对 fac<10000> 进行显式实例化, 在 a.cc 中使用 extern 声明 fac<10000> 的外部链接. 这样, 在定义了 `EXPLICIT_INSTANTIATION` 宏再编译时, 我们期望模板仅实例化一次并节省大量编译时间.

我们首先看看不使用显式实例化即实例化两次 fac<10000> 的编译时间:

```bash
$time clang++ a.cc b.cc -ftemplate-depth=10000
clang++ a.cc b.cc -ftemplate-depth=10000  5.04s user 0.26s system 99% cpu 5.319 total
```

消耗了 5.04s.

然后看看使用了显式实例化的代码的编译时间:

```bash
$time clang++ a.cc b.cc -ftemplate-depth=10000 -DEXPLICIT_INSTANTIATION
clang++ a.cc b.cc -ftemplate-depth=10000 -DEXPLICIT_INSTANTIATION  3.31s user 0.24s system 99% cpu 3.554 total
```

仅消耗 3.3s!

使用 gcc 也可以分别得到 10.7s 和 6.5s 的结果.

## References

- [Class template - cppreference.com](https://en.cppreference.com/w/cpp/language/class_template)
- [explicit (full) template specialization - cppreference.com](https://en.cppreference.com/w/cpp/language/template_specialization)
- [Partial template specialization - cppreference.com](https://en.cppreference.com/w/cpp/language/partial_specialization)
- [Explicit Instantiation - Microsoft Docs](https://docs.microsoft.com/en-us/cpp/cpp/explicit-instantiation?view=msvc-160)
- [Build Throughput Series Template Metaprogramming Fundamentals - C++ Team Blog](https://devblogs.microsoft.com/cppblog/build-throughput-series-template-metaprogramming-fundamentals/)
