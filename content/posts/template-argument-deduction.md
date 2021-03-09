---
title: C++ 模板参数推导
date: 2020-10-19 22:14:18
updated: 2020-10-23
tags:
  - c++
  - template
---

实例化函数模板时, 模板实参必须是已知的, 但不必显式指定. 编译器会从函数模板的实参中推导缺失的模板实参. 另外模板参数推导也可以在 auto 说明符的上下文中从初始化器推导变量的类型.

<!--more-->

## 函数模板参数的推导

对于下列的函数模板及其调用

```c++
template <typename T>
void f(P param);

// ...

f(A);
```

其中 P 和 T 有关, 例如 P 可能为 const T& 或 T&& 等. 有下列的推导规则:

- A 的引用属性被忽略.
- P 是非引用时, A 的 cv 限定符被忽略.
- 如果 P 是无 cv 限定符的转发引用 (即 T&&), 且 A 是左值时, T 被推导为左值引用.
- 如果 A 是数组或函数, P 是值时, 数组和函数退化为指针. P 是引用时, 不退化为指针.

这里 cv 限定符指的是 const 和 volatile.

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

这个例子中 P = T, A = int. 结果是

```bash
void f(T) [T = int]
```

注意 `__PRETTY_FUNCTION__` 是编译器扩展, 在 gcc 和 clang 中支持, 在 MSVC 中可以使用 `__FUNCSIG__`.

下面是一个经过实验的表格

|    P     |         A          |          T           |
| :------: | :----------------: | :------------------: |
|    T     |        int         |         int          |
|    T     |       int\*        |        int\*         |
|    T     |        int&        |         int          |
|    T     |     const int      |         int          |
|    T     |    const int \*    |     const int \*     |
|    T     |    int \* const    |        int \*        |
|    T     |    const int &     |         int          |
|    T     | const int \* const |     const int \*     |
|    T     |      char [2]      |       char \*        |
|    T     |  const char [12]   |    const char \*     |
|    T     |     void (int)     |    void (\*)(int)    |
| const T  |        int         |         int          |
| const T  |       int \*       |        int \*        |
| const T  |       int &        |         int          |
| const T  |     const int      |         int          |
| const T  |    const int \*    |     const int \*     |
| const T  |    const int &     |         int          |
| const T  | const int \* const |     const int \*     |
| const T  |      char [2]      |       char \*        |
| const T  |  const char [12]   |    const char \*     |
| const T  |     void (int)     |    void (\*)(int)    |
|    T&    |        int         |         int          |
|    T&    |       int \*       |        int \*        |
|    T&    |       int &        |         int          |
|    T&    |     const int      |      const int       |
|    T&    |    const int \*    |     const int \*     |
|    T&    |    const int &     |      const int       |
|    T&    | const int \* const |  const int \* const  |
|    T&    |      char [2]      |       char [2]       |
|    T&    |  const char [12]   |   const char [12]    |
|    T&    |     void (int)     |      void (int)      |
|   T&&    |        int         |        int &         |
|   T&&    |       int \*       |       int \*&        |
|   T&&    |       int &        |        int &         |
|   T&&    |     const int      |     const int &      |
|   T&&    |    const int \*    |    const int \*&     |
|   T&&    |    const int &     |     const int &      |
|   T&&    | const int \* const | const int \* const & |
|   T&&    |      char [2]      |     char (&)[2]      |
|   T&&    |  const char [12]   |  const char (&)[12]  |
|   T&&    |     void (int)     |    void (&)(int)     |
|   T&&    |       int &&       |         int          |
| const T& |       int &&       |         int          |

## auto 类型推导

C++11 中, 对于包含 auto 的变量声明, auto 被一个虚构的模板参数 U 替换, 实参 A 是初始化表达式, 按照模板参数推导的规则从 P 和 A 推导出 U 后, 将 U 代入 P 得到实际的变量类型.

例如

```c++
const auto& x = 1 + 2;
```

用 U 替换 auto, 得到

```c++
const U& x = 1 + 2;
```

这里 P 是 const U&, A 是 1 + 2, A 的类型为 int, 使用模板参数推导的规则, U 是 int, 因此变量 x 的类型为 const int &.

auto 类型推导基本与模板参数推导相同, 不同的一点是如果 A 是拷贝列表初始化, 则 U 被替换为 `std::initializer_list<U>`.

## 函数返回类型推导

C++14 中, 对于返回 auto 的函数, auto 被一个虚构的模板参数 U 替换, 参数 A 是 return 语句的表达式, 如果 return 语句没有操作数, A 为 void(). 使用上面的规则推导 U, 然后获得实际的返回类型.

例如

```c++
auto f() {
    return 42;
}
```

其中 A 是 42, 类型为 int. 因此 U 被推导为 int, 函数返回类型也为 int.

如果返回语句是 `return;` 或没有返回语句, A 被认为是 void(). 因此, 下面的函数

```c++
auto f() {
    return;
}
```

的返回类型为 void.

对于下面的函数

```c++
// C++14
#include <type_traits>

const auto f() {
    return 42;
}

static_assert(std::is_same<decltype(f()), const int>::value, "error");
```

如果按照模板参数推导的规则, 函数 f 的返回值应该是 const int. 但是上面的 static_cast 会失败:

```bash
error: static_assert failed due to requirement 'std::is_same<int, const int>::value' "error"
```

看起来 f 的返回类型实际上是 int. 这是由于非引用类型的函数返回值是纯右值, 而非类非数组的纯右值不能被 cv 限定, 因此虽然 f 的返回类型是 const int, 但它的 const 属性会被立刻剥离.

如果有多条 return 语句, 上述规则会对每条 return 语句执行, 如果它们的实际返回类型不同, 会引发编译错误.

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

p 会被推导为 `std::pair<int, double>`. static_assert 验证了这点.

## 推导指引

如果构造函数是模板函数, CTAD 不会工作. 因此 C++17 也允许使用推导指引 _(deduction guide)_. 它指示编译器如何从构造函数模板的参数中推导类模板参数.

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
- Effective Modern C++
- [Value categories - cppreference.com](https://en.cppreference.com/w/cpp/language/value_category)
