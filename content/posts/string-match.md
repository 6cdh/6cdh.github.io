---
title: 标准库中的字符串搜索
date: 2020-11-21
updated: 2020-11-21
katex: true
tags:
  - c++
  - stl
---

给定文本 T(ext) 和模式 P(attern), 字符串搜索问题在 T 中寻找 P 出现的位置.

<!--more-->

## 朴素字符串搜索 _(Naïve string search)_

最简单, 直观的算法, 依次匹配 T 和 P 中的每个字符, 直到找到.

```c++
auto naive_match(const std::string& T, const std::string& P) {
    if (T.size() < P.size()) {
        return T.end();
    }
    if (P.empty()) {
        return T.begin();
    }
    for (size_t i = 0; i < T.size() - P.size() + 1; ++i) {
        size_t j = 0;
        while (T[i + j] == P[j]) {
            ++j;
            if (j == P.size()) {
                return T.begin() + i;
            }
        }
    }
    return T.end();
}
```

假设 T 的长度为 t, P 的长度为 p. 朴素字符串搜索算法的时间复杂度最好是 O(t + p). 最坏是 O((t - p + 1)p) = O(tp). 空间复杂度是 O(1).

## 优化的朴素字符串搜索 _(Optimized Naïve string-search algorithm)_

libcxx 和 MS STL 使用了优化的朴素字符串搜索.

标准库提供的字符串搜索算法:

```c++
#include <iostream>
#include <string>

int main() {
    std::string str("aabaa");
    auto index = str.find("baa");
    if (index != std::string::npos) {
        std::cout << "Index: " << index << '\n';
    } else {
        std::puts("Not found.");
    }
}
```

将输出

```shell
Index: 2
```

`string::find` 函数实际上是 `std::basic_string<CharT, Traits, Allocator>::find`, 原型(之一, 有多个重载)为:

```c++
size_type find(const basic_string& str, size_type pos = 0) const noexcept;
```

作用是从 pos 开始搜索第一个等于 str 的子串的索引. 如果不存在, 返回 `std::basic_string<CharT, Traits, Allocator>::npos`, npos 的定义实际上是

```c++
static const size_type npos = -1;
```

libcxx 中 find 的实现:

```c++
// llvm-project/libcxx/include/string

_LIBCPP_INLINE_VISIBILITY
size_type find(const basic_string& __str, size_type __pos = 0) const _NOEXCEPT;

// ...

template<class _CharT, class _Traits, class _Allocator>
inline
typename basic_string<_CharT, _Traits, _Allocator>::size_type
basic_string<_CharT, _Traits, _Allocator>::find(const basic_string& __str,
                                                size_type __pos) const _NOEXCEPT
{
    return __str_find<value_type, size_type, traits_type, npos>
        (data(), size(), __str.data(), __pos, __str.size());
}
```

可以看到它实际上调用了 `__str_find` 函数, 这个函数的源码如下

```c++
// llvm-project/libcxx/include/__string

template<class _CharT, class _SizeT, class _Traits, _SizeT __npos>
inline _SizeT _LIBCPP_CONSTEXPR_AFTER_CXX11 _LIBCPP_INLINE_VISIBILITY
__str_find(const _CharT *__p, _SizeT __sz,
       const _CharT* __s, _SizeT __pos, _SizeT __n) _NOEXCEPT
{
    if (__pos > __sz)
        return __npos;

    if (__n == 0) // There is nothing to search, just return __pos.
        return __pos;

    const _CharT *__r = __search_substring<_CharT, _Traits>(
        __p + __pos, __p + __sz, __s, __s + __n);

    if (__r == __p + __sz)
        return __npos;
    return static_cast<_SizeT>(__r - __p);
}
```

`__str_find` 的参数很多, 分别是

- `__p` - 指针, 指向 T.
- `__sz` - T 的长度.
- `__s` - 指针, 指向 P.
- `__pos` - 索引, 表示开始匹配的位置.
- `__n` - P 的长度.

这个函数进行了两次判断, 分别排除了 `__pos` 过大和 P 为空的情况, 然后把字符串搜索的逻辑转换到了 `__search_substring` 函数中:

```c++
// llvm-project/libcxx/include/__string

template <class _CharT, class _Traits>
inline _LIBCPP_CONSTEXPR_AFTER_CXX11 const _CharT *
__search_substring(const _CharT *__first1, const _CharT *__last1,
                   const _CharT *__first2, const _CharT *__last2) {
  // Take advantage of knowing source and pattern lengths.
  // Stop short when source is smaller than pattern.
  const ptrdiff_t __len2 = __last2 - __first2;
  if (__len2 == 0)
    return __first1;

  ptrdiff_t __len1 = __last1 - __first1;
  if (__len1 < __len2)
    return __last1;

  // First element of __first2 is loop invariant.
  _CharT __f2 = *__first2;
  while (true) {
    __len1 = __last1 - __first1;
    // Check whether __first1 still has at least __len2 bytes.
    if (__len1 < __len2)
      return __last1;

    // Find __f2 the first byte matching in __first1.
    __first1 = _Traits::find(__first1, __len1 - __len2 + 1, __f2);
    if (__first1 == 0)
      return __last1;

    // It is faster to compare from the first byte of __first1 even if we
    // already know that it matches the first byte of __first2: this is because
    // __first2 is most likely aligned, as it is user's "pattern" string, and
    // __first1 + 1 is most likely not aligned, as the match is in the middle of
    // the string.
    if (_Traits::compare(__first1, __first2, __len2) == 0)
      return __first1;

    ++__first1;
  }
}
```

显然, `__search_substring` 是真正字符串搜索逻辑发生的函数. first1 和 last1 分别指向 T 的首尾, first2 和 last2 分别指向 P 的首尾, P 的第一个字符为 `__f2`. 每次在区间 `[first1, last1 - len(P)]` 内搜索 `__f2` 出现的位置, 并令 first1 指向该位置. 然后比较 first1 和 first2 之后的字符. 匹配则返回 first1, 否则递增 first1, 继续搜索.

`__search_substring` 调用了两个函数: `_Traits::find` 和 `_Traits::compare`. 它们的源码如下:

```c++
template <class _CharT>
inline
_LIBCPP_CONSTEXPR_AFTER_CXX14 const _CharT*
char_traits<_CharT>::find(const char_type* __s, size_t __n, const char_type& __a)
{
    for (; __n; --__n)
    {
        if (eq(*__s, __a))
            return __s;
        ++__s;
    }
    return 0;
}

inline _LIBCPP_CONSTEXPR_AFTER_CXX14
int
char_traits<char>::compare(const char_type* __s1, const char_type* __s2, size_t __n) _NOEXCEPT
{
    if (__n == 0)
        return 0;
#if __has_feature(cxx_constexpr_string_builtins)
    return __builtin_memcmp(__s1, __s2, __n);
#elif _LIBCPP_STD_VER <= 14
    return memcmp(__s1, __s2, __n);
#else
    for (; __n; --__n, ++__s1, ++__s2)
    {
        if (lt(*__s1, *__s2))
            return -1;
        if (lt(*__s2, *__s1))
            return 1;
    }
    return 0;
#endif
}
```

MS STL 的实现类似:

```c++
template <class _Traits>
constexpr size_t _Traits_find(_In_reads_(_Hay_size) const _Traits_ptr_t<_Traits> _Haystack, const size_t _Hay_size,
    const size_t _Start_at, _In_reads_(_Needle_size) const _Traits_ptr_t<_Traits> _Needle,
    const size_t _Needle_size) noexcept {
    // search [_Haystack, _Haystack + _Hay_size) for [_Needle, _Needle + _Needle_size), at/after _Start_at
    if (_Needle_size > _Hay_size || _Start_at > _Hay_size - _Needle_size) {
        // xpos cannot exist, report failure
        // N4659 24.3.2.7.2 [string.find]/1 says:
        // 1. _Start_at <= xpos
        // 2. xpos + _Needle_size <= _Hay_size;
        // therefore:
        // 3. _Needle_size <= _Hay_size (by 2) (checked above)
        // 4. _Start_at + _Needle_size <= _Hay_size (substitute 1 into 2)
        // 5. _Start_at <= _Hay_size - _Needle_size (4, move _Needle_size to other side) (also checked above)
        return static_cast<size_t>(-1);
    }

    if (_Needle_size == 0) { // empty string always matches if xpos is possible
        return _Start_at;
    }

    const auto _Possible_matches_end = _Haystack + (_Hay_size - _Needle_size) + 1;
    for (auto _Match_try = _Haystack + _Start_at;; ++_Match_try) {
        _Match_try = _Traits::find(_Match_try, static_cast<size_t>(_Possible_matches_end - _Match_try), *_Needle);
        if (!_Match_try) { // didn't find first character; report failure
            return static_cast<size_t>(-1);
        }

        if (_Traits::compare(_Match_try, _Needle, _Needle_size) == 0) { // found match
            return static_cast<size_t>(_Match_try - _Haystack);
        }
    }
}
```

了解了标准库的思路, 我们可以自己实现它:

```c++
size_t find_in_range(const std::string& T, char ch, size_t beg, size_t end) {
    const auto posptr = memchr(T.c_str() + beg, ch, end - beg);
    if (posptr == nullptr) {
        return end;
    }
    return static_cast<const char*>(posptr) - T.c_str();
}

bool equal(const std::string& T, const std::string& P, size_t pos) {
    return memcmp(T.c_str() + pos, P.c_str(), P.size()) == 0;
}

auto match_std(const std::string& T, const std::string& P) {
    if (P.size() > T.size()) {
        return T.end();
    }
    if (P.empty()) {
        return T.begin();
    }
    const auto end = T.size() - P.size() + 1;
    for (size_t match_try = 0;; ++match_try) {
        match_try = find_in_range(T, P[0], match_try, end);
        if (match_try == end) {
            return T.end();
        }
        if (equal(T, P, match_try)) {
            return T.begin() + match_try;
        }
    }
}
```

对 memchr 和 memcmp 的调用大大加快了代码的速度. 优化的朴素字符串搜索时间复杂度为 $O(tp)$.

libcxx 的相关讨论给出了 [benchmark](https://reviews.llvm.org/D27068) 的结果.

## strstr/两路字符串搜索 _(Two-way string-search algorithm)_

glibc 中的 strstr 使用了两路字符串搜索. 需要 O(p) 时间预处理, 执行时间是 O(t + p), 消耗 O(1) 的空间.

由于代码很长很复杂, 这里不再给出源码. 可以点击[链接](https://github.com/bminor/glibc/blob/master/string/strstr.c#L76)进入 Github 查看.

[Wikipedia](https://en.wikipedia.org/wiki/Two-way_string-matching_algorithm)相关页面描述了算法的基本思路.

被 C 标准库采用, 两路字符串搜索是非常快的. 它可以被视为组合了 KMP(Knuth-Morris-Pratt) 算法和 BM(Boyer-Moore) 算法.

## BM 算法

TODO

## KMP 算法

TODO

## References

- [⚙ D27068 Improve string::find](https://reviews.llvm.org/D27068)
- [String-searching algorithm - Wikipedia](https://en.wikipedia.org/wiki/String-searching_algorithm)
- [Two-way string-matching algorithm - Wikipedia](https://en.wikipedia.org/wiki/Two-way_string-matching_algorithm)
