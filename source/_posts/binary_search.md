---
title: 二分搜索及其变种
date: 2020-09-23 02:52:13
updated: 2020-11-27
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
    size_t left = 0;
    size_t right = arr.size();
    while (left < right) {
        size_t mid = left + (right - left) / 2;
        if (arr[mid] < target) {
            left = mid + 1;
        } else if (arr[mid] > target) {
            right = mid;
        } else {
            return mid;
        }
    }
    return {};
}
```

## 查找最左边的元素

如果数组中有多个目标值, 要求找到最左边的目标值. 如下 C++ 代码 (版本 2)

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

上面的代码可能不太好理解, 这就是二分搜索的难点所在, 只要修改一些细微之处, 就可以搜索满足各种条件的值.

下面简单的讨论一下版本 2 算法的属性. 大致分为三种情况:

- 数组中有一个目标值.
- 数组中有多个目标值.
- 数组中没有目标值.

首先, 我们假设数组中有一个目标值. 那么初始条件 `arr[0] <= target`.

- 如果 `arr[0] == target`, 那么 `arr[mid] >= target` 始终成立, `right = mid` 始终被执行, 即 `right` 不断的向 0 移动, 最后返回 0.
- 如果 `arr[0] < target`, 当 `left` 或 `right` 第一次遇到 `target` 并指向它后, 另一方都会向它移动直到它们相等并退出.

这时, 算法总会找到目标值.

现在假设数组中有多个目标值, 那么初始条件 `arr[0] <= target`.

- 如果 `arr[0] == target`, 那么 `arr[mid] >= target` 始终成立, `right` 不断的向 0 移动, 即使中间满足 `arr[right] = target`, `right` 也不会停止移动. 直到相等退出循环.
- 如果 `arr[left] < target`, 不管 `right` 指向什么值, 只要 `left = mid + 1` 被执行且执行之前 `arr[left] == target` 不满足, 执行之后 `arr[left] == target` 满足, 那么 `left` 之后就不会移动, `right` 只能一直向 `left` 移动直到相等.

因此, 如果数组中有多个目标值, 最后指向的位置始终是最左边的那个目标值.

现在假设数组中没有目标值, 那么初始条件 `arr[0] > target` 或 `arr[0] < target` 且数组中没有目标值.

如果 `arr[0] > target`, `right = mid` 始终被执行, 直到返回. 因此返回的是索引 0. 而且 `arr[0]` 是大于 `target` 的最小元素.

如果 `arr[0] < target` 即数组中只有大于 `target` 和小于 `target` 的元素. `left` 和 `right` 都在相向移动. 那么它们最后会移动到哪呢?

假设我们在第 x 次循环时 `left == right` 并退出循环, 那么第 `x-1` 次循环时, `left < right` 且, `left` 和 `right` 分别指向小于和大于 `target` 的元素. 如果执行 `left = mid + 1`, 这意味着下列布局:

```
+-----+---+---+-----+
| ... | 2 | 4 | ... |
+-----+---+---+-----+
        |   |
       mid right
```

如图, 假设我们要找 3, 最后 left 和 right 都会指向 4, 即大于 3 的最小元素的位置, 如果 4 有多个, 那么它也会找到最左边的 4.

如果执行的是 `right = mid`, 这意味着布局如下:

```
      left
        |
+-----+---+---+-----+
| ... | 2 | 4 | ... |
+-----+---+---+-----+
        |
       mid
```

但是, 执行 `right = mid` 的前提是 `arr[mid] >= target`, 上图的布局又不满足这个条件, 因此 `right = mid` 不可能被执行.

因此, 如果不存在目标值, 算法会找到大于它的最小元素, 这个位置也是如果插入目标值, 它将会在的位置. 如果还不存在, 返回 `arr.size()` 即尾后元素索引.

总之, 版本 2 的二分搜索总是找到相同目标值的最左边那个元素, 如果不存在, 它会找到如果要插入目标值, 目标值应该在的位置, 这个位置现在保存的是大于目标值的最小元素, 如果这个位置还不存在, 返回 `arr.size()`. 同时, 我们还简单的证明了如果目标值不存在且 `arr[0] < target`, 那么最后执行的分支总是 `left = mid + 1`.

## 查找最右边的元素

```c++
template <typename Container>
size_t binary_search_v3(const Container& arr,
                        const typename Container::value_type& target) {
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
    return left == 0 ? arr.size() : left - 1;
}
```

版本 3 和版本 2 的差别是 `arr[mid] == target` 时执行的分支由 `right = mid` 改为了 `left = mid + 1`. 另外最后返回 `left - 1`.

版本 3 的二分搜索总是找到相同目标值的最右边那个元素, 如果不存在, 它会找到小于目标值的最大元素, 如果还不存在, 返回 `arr.size()`. 如果目标值不存在且 `arr[0] < target`, 那么最后执行的分支总是 `left = mid + 1`.

注意到在每次迭代中, 版本 2 和版本 3 都比版本 1 少一次 if 判断. 这使得每一次的循环更快. 相对地, 总循环数更多了.

另外还有两种变种:

- 总是查找大于目标值的最小元素. 这使用版本 3 的返回值再加 1 即可找到.
- 总是查找小于目标值的最大元素. 这使用版本 2 的返回值再减 1 即可找到.

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

`upper_bound` 返回 `[first, last)` 中大于目标值的 `value` 的第一个元素:

```c++
template <class _Compare, class _ForwardIterator, class _Tp>
_LIBCPP_CONSTEXPR_AFTER_CXX17 _ForwardIterator
__upper_bound(_ForwardIterator __first, _ForwardIterator __last, const _Tp& __value_, _Compare __comp)
{
    typedef typename iterator_traits<_ForwardIterator>::difference_type difference_type;
    difference_type __len = _VSTD::distance(__first, __last);
    while (__len != 0)
    {
        difference_type __l2 = _VSTD::__half_positive(__len);
        _ForwardIterator __m = __first;
        _VSTD::advance(__m, __l2);
        if (__comp(__value_, *__m))
            __len = __l2;
        else
        {
            __first = ++__m;
            __len -= __l2 + 1;
        }
    }
    return __first;
}
```

基本思想与版本 3 相同, 但是返回值不减 1.

## References

- [Binary search algorithm - Wikipedia](https://en.wikipedia.org/wiki/Binary_search_algorithm)
- [c++ template class; function with arbitrary container type, how to define it? - Stack Overflow](https://stackoverflow.com/questions/7728478/c-template-class-function-with-arbitrary-container-type-how-to-define-it)
