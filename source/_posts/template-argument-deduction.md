---
title: C++ 模板参数推导
date: 2020-10-19 22:14:18
updated: 2020-10-19
tags:
- C++
- template
---

实例化函数模板时, 模板实参必须是已知的, 但不必显式指定. 编译器会从函数模板的实参中推导缺失的模板实参.

<!-- more -->

## 函数模板参数的推导

对于下列的函数模板及其调用

```c++
template <typename T>
void f(P param);

// ...

f(A);
```

其中 P 和 T 有关, 例如 P 可能为 `const T&` 或 `T&&` 等. 有下列的推导规则:

- A 的引用属性被忽略.
- P 是非引用时, A 的 cv 限定符被忽略.
- 如果 P 是无 cv 限定符的转发引用 (即 `T&&`), 且 A 是左值时, T 被推导为左值引用.
- 如果 A 是数组或函数, P 是值时, 数组和函数退化为指针. P 是引用时, 不退化为指针.

这里 cv 限定符指的是 `const` 和 `volatile`.

使用下面的代码验证

```c++
#include <cstdio>

template <typename T>
void f(T param) {
    std::puts(__PRETTY_FUNCTION__);
}

int main() {
    int i = 0;
    f(i);
}
```

这个例子中 `P = T`, `A = int`. 结果是

```bash
void f(T) [T = int]
```

注意 `__PRETTY_FUNCTION__` 是编译器扩展, 在 gcc 和 clang 中支持, 在 MSVC 中可以使用 `__FUNCSIG__`.

下面是一个经过实验的表格

|     P      |          A          |           T           |
| :--------: | :-----------------: | :-------------------: |
|     T      |        `int`        |         `int`         |
|     T      |       `int*`        |        `int*`         |
|     T      |       `int&`        |         `int`         |
|     T      |     `const int`     |         `int`         |
|     T      |    `const int *`    |     `const int *`     |
|     T      |    `int * const`    |        `int *`        |
|     T      |    `const int &`    |         `int`         |
|     T      | `const int * const` |     `const int *`     |
|     T      |     `char [2]`      |       `char *`        |
|     T      |  `const char [12]`  |    `const char *`     |
|     T      |    `void (int)`     |    `void (*)(int)`    |
| `const T`  |        `int`        |         `int`         |
| `const T`  |       `int *`       |        `int *`        |
| `const T`  |       `int &`       |         `int`         |
| `const T`  |     `const int`     |         `int`         |
| `const T`  |    `const int *`    |     `const int *`     |
| `const T`  |    `const int &`    |         `int`         |
| `const T`  | `const int * const` |     `const int *`     |
| `const T`  |     `char [2]`      |       `char *`        |
| `const T`  |  `const char [12]`  |    `const char *`     |
| `const T`  |    `void (int)`     |    `void (*)(int)`    |
|    `T&`    |        `int`        |         `int`         |
|    `T&`    |       `int *`       |        `int *`        |
|    `T&`    |       `int &`       |         `int`         |
|    `T&`    |     `const int`     |      `const int`      |
|    `T&`    |    `const int *`    |     `const int *`     |
|    `T&`    |    `const int &`    |      `const int`      |
|    `T&`    | `const int * const` |  `const int * const`  |
|    `T&`    |     `char [2]`      |      `char [2]`       |
|    `T&`    |  `const char [12]`  |   `const char [12]`   |
|    `T&`    |    `void (int)`     |     `void (int)`      |
|   `T&&`    |        `int`        |        `int &`        |
|   `T&&`    |       `int *`       |       `int *&`        |
|   `T&&`    |       `int &`       |        `int &`        |
|   `T&&`    |     `const int`     |     `const int &`     |
|   `T&&`    |    `const int *`    |    `const int *&`     |
|   `T&&`    |    `const int &`    |     `const int &`     |
|   `T&&`    | `const int * const` | `const int * const &` |
|   `T&&`    |     `char [2]`      |     `char (&)[2]`     |
|   `T&&`    |  `const char [12]`  | `const char (&)[12]`  |
|   `T&&`    |    `void (int)`     |    `void (&)(int)`    |
|   `T&&`    |      `int &&`       |         `int`         |
| `const T&` |      `int &&`       |         `int`         |

## 类模板参数推导 _(Class template argument deduction, CTAD)_

C++17 将模板参数推导扩展到了仅给出类模板名称的对象构造. 例如

```c++
#include <type_traits>
#include <utility>

int main() {
    std::pair p{0, 0.0};
    static_assert(std::is_same_v<decltype(p), std::pair<int, double>>);
}
```

p 会被推导为 `std::pair<int, double>`. `static_assert` 验证了这点.

## 推导指引

如果构造函数是模板函数, CTAD 不会工作. 因此 C++17 也允许使用推导指引 _(deduction guide)_. 它指示编译器如何从类模板参数中推导构造函数模板的参数.

```c++
#include <iterator>

// declaration of the template
template <typename T>
struct container {
    container(T t) {}
    template <typename Iter>
    container(Iter beg, Iter end);
};
// additional deduction guide
template <typename Iter>
container(Iter b, Iter e)
    -> container<typename std::iterator_traits<Iter>::value_type>;
```

注意推导指引不能放在类中构造函数的定义后面, 因为这会被认为是尾置返回类型继而引发编译错误.

## References

- [Class template argument deduction (CTAD) (since C++17) - cppreference.com](https://en.cppreference.com/w/cpp/language/class_template_argument_deduction)
- [How to Use Class Template Argument Deduction | C++ Team Blog](https://devblogs.microsoft.com/cppblog/how-to-use-class-template-argument-deduction/)

