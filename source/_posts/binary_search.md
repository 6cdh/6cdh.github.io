---
title: 二分搜索及其变种
date: 2020-09-23 02:52:13
updated: 2020-10-07
---

二分搜索 _(binary search)_ 也称折半搜索 _(half-interval search)_, 用于在有序数组上搜索给定值的位置. 这是一个常见的搜索算法, 似乎没什么难度. 然而, 在解决 [Leetcode 35 (Search Insert Position)](https://leetcode.com/problems/search-insert-position/) 时, 我意识到对二分搜索及其变种的理解还不够, 因此记录一下.

<!--more-->

## 基本的二分搜索

搜索算法对于一个给定的非降序输入数组和一个给定的目标值, 搜索目标值所在的位置. 二分搜索首先搜索数组中点. 如果中点值大于目标值, 说明目标值在数组左半边, 这样就将输入数组缩小了一半. 如果中点的值小于目标值, 说明目标值在数组右半边, 否则, 中点值等于目标值, 返回. 重复以上步骤.

根据上面的思路得到了基本的二分搜索的 C++ 代码, 称为 `binary_search_v1` (version 1).

```c++
template <typename Container>
int binary_search_v1(const Container& arr,
                     const typename Container::value_type& target) {
    int left = 0;
    int right = arr.size() - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (arr[mid] < target) {
            left = mid + 1;
        } else if (arr[mid] > target) {
            right = mid - 1;
        } else {
            return mid;
        }
    }
    return -1;
}
```

注意上面的代码的正确性依赖于它使用的索引是有符号类型 (int). 对于过大的输入数组, 该算法可能不能正常工作. 因为 C++ 中容器大小的类型通常是 `std::size_t`, 而 `std::size_t` 是无符号类型, 特别是对于 64 位系统, `std::size_t` 通常是 64 位, 使用 32 位有符号的 int 保存它会导致溢出.

`binary_search_v1` 可以用于伪代码以表示二分搜索的思想. 也可以进行略微改写以使它支持无符号类型并防止溢出:

```c++
#include <optional>

template <typename Container>
std::optional<size_t> binary_search_v1_plus(
    const Container& arr, const typename Container::value_type& target) {
    if (target < arr.front() || arr.back() < target) {
        return {};
    }
    size_t left = 0;
    size_t right = arr.size() - 1;
    while (left <= right) {
        size_t mid = left + (right - left) / 2;
        if (arr[mid] < target) {
            left = mid + 1;
        } else if (arr[mid] > target) {
            right = mid - 1;
        } else {
            return mid;
        }
    }
    return {};
}
```

## 查找最左边的元素

如果数组中有多个目标值, 要求找到等于目标值的最左边的元素. 如下 C++ 代码 (版本 2)

```c++
template <typename Container>
size_t binary_search_v2(const Container& arr,
                        const typename Container::value_type& target) {
    size_t left = 0;
    size_t right = arr.size();
    while (left < right) {
        size_t mid = left + (right - left) / 2;
        if (arr[mid] < target) {
            left = mid + 1;
        } else {
            right = mid;
        }
    }
    return left;
}
```

类似的, 有找到最右边元素的版本. 如下 C++ 代码 (版本 3)

```c++
template <typename Container>
size_t binary_search_v3(const Container& arr,
                        const typename Container::value_type& target) {
    if (target < arr.front()) {
        return -1;
    }
    if (arr.back() < target) {
        return arr.size();
    }
    size_t left = 0;
    size_t right = arr.size();
    while (left < right) {
        size_t mid = left + (right - left) / 2;
        if (arr[mid] > target) {
            right = mid;
        } else {
            left = mid + 1;
        }
    }
    return left - 1;
}
```

注意到在每次迭代中, 版本 2 和版本 3 都比版本 1 少一次 if 判断. 这使得每一次的循环更快. 相对地, 平均的循环数更多了.

版本 2 中不使用 `optional` 的原因是如果输入数组中不存在目标值, 版本 2 会返回如果插入目标值, 它应该在的位置. 有时这是很有用的.

版本 3 中不使用 `optional` 的原因是如果输入数组中不存在目标值, 版本 3 会返回如果插入目标值, 它应该在的位置减 1, 这一般是小于目标值的最大元素的位置.

另外还有两种变种:

- 查找大于目标值的最小元素. 这使用版本 3 的返回值再加 1 即可找到.
- 查找小于目标值的最大元素. 这使用版本 2 的返回值再减 1 即可找到.

## Libcxx 的实现

C++ 标准库中实现的 `binary_search` 原型如下:

```c++
template<class ForwardIt, class T, class Compare>
bool binary_search(ForwardIt first, ForwardIt last, const T& value, Compare comp);
```

libcxx 中的实现如下:

```c++
template <class _Compare, class _ForwardIterator, class _Tp>
inline _LIBCPP_INLINE_VISIBILITY _LIBCPP_CONSTEXPR_AFTER_CXX17
bool
__binary_search(_ForwardIterator __first, _ForwardIterator __last, const _Tp& __value_, _Compare __comp)
{
    __first = __lower_bound<_Compare>(__first, __last, __value_, __comp);
    return __first != __last && !__comp(__value_, *__first);
}

template <class _ForwardIterator, class _Tp, class _Compare>
_LIBCPP_NODISCARD_EXT inline
_LIBCPP_INLINE_VISIBILITY _LIBCPP_CONSTEXPR_AFTER_CXX17
bool
binary_search(_ForwardIterator __first, _ForwardIterator __last, const _Tp& __value_, _Compare __comp)
{
    typedef typename __comp_ref_type<_Compare>::type _Comp_ref;
    return __binary_search<_Comp_ref>(__first, __last, __value_, __comp);
}

template <class _Compare, class _ForwardIterator, class _Tp>
_LIBCPP_CONSTEXPR_AFTER_CXX17 _ForwardIterator
__lower_bound(_ForwardIterator __first, _ForwardIterator __last, const _Tp& __value_, _Compare __comp)
{
    typedef typename iterator_traits<_ForwardIterator>::difference_type difference_type;
    difference_type __len = _VSTD::distance(__first, __last);
    while (__len != 0)
    {
        difference_type __l2 = _VSTD::__half_positive(__len);
        _ForwardIterator __m = __first;
        _VSTD::advance(__m, __l2);
        if (__comp(*__m, __value_))
        {
            __first = ++__m;
            __len -= __l2 + 1;
        }
        else
            __len = __l2;
    }
    return __first;
}
```

可以看到 `binary_search` 实际上调用了 `lower_bound`, 这个函数返回范围 `[first, last)` 中不小于目标值 `value` 的第一个元素. 基本思想与版本 2 相同.

## References

- [Binary search algorithm - Wikipedia](https://en.wikipedia.org/wiki/Binary_search_algorithm)
- [c++ template class; function with arbitrary container type, how to define it? - Stack Overflow](https://stackoverflow.com/questions/7728478/c-template-class-function-with-arbitrary-container-type-how-to-define-it)
