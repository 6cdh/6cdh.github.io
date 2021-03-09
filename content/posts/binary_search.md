---
title: 二分搜索及其变种
date: 2020-09-23 02:52:13
katex: true
tags:
  - c++
  - binary search
updated: 2021-01-17
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

为了证明该程序是正确的, 我们需要证明以下 4 个属性:

1. 程序的初始条件是正确的.
2. 循环过程中, 始终维持某个不变式成立.
3. 程序会终止.
4. 程序终止时会产生正确的 (我们想要的) 结果.

显然, 只要这些属性满足, 程序一定是正确的.

为了方便, 我们令 $a$ 表示我们需要搜索的有序数组, $x=\text{target}, L=0, R=\text{size}(a), m=\text{mid}$, 这样, 区间 $[L, R)$ 也可以表示我们需要搜索的有序数组. 在每次循环时, $\text{left, right}$ 的值分别保存给 $l, r$, 每轮循环结束时, $\text{left, right}$ 的值发生变化.

然后定义一个不变式 $P: (\text{right}-\text{left} < r-l)\land ([L, \text{left}) < x)\land (x \le[\text{right}, R))$. 注意 $\land$ 表示逻辑与, 不变式 $P$ 保证三个条件成立:

1. $\text{right}-\text{left} < r-l$ 表示每轮循环都会导致 $[\text{left, right})$ 缩小.
2. $[L, \text{left}) < x$ 表示区间 $[L, \text{left})$ 的元素都比 $x$ 小
3. $x \le[\text{right}, R)$ 表示区间 $[\text{right}, R)$ 的元素都不比 $x$ 小.

下面我们证明属性 1 即证明 $L, R$ 满足 $[L, L) < x\land [R, R)$. 显然 $[L, L)$ 和 $[R, R)$ 为空, 因此属性 1 成立.

然后是属性 2. 令 $m=l+\lfloor (r-l)/2 \rfloor$, 并且对于前面的循环, 始终有 $P$ 成立.

如果 $\text{arr[mid] < target}$, 那么 $\text{left}=m+1, \text{right}=r$, 我们需要证明

$$
\begin{aligned}
& \text{right - left} < r - l\cr
& = r - (m + 1) < r - l\cr
& = m + 1 > l\cr
& = l + \lfloor (r-l)/2 \rfloor + 1 > l\cr
& = \lfloor (r-l)/2 \rfloor > -1\cr
\end{aligned}
$$

因为循环条件保证了 $r > l$, 因此, $\lfloor (r-l)/2\rfloor \ge 0 > -1$. 此外, 显然 $[L, m+1) < x$ 成立, $[r, R)$ 也成立 (前面的循环保证了它成立). 因此, $P$ 成立.

如果 $\text{arr[mid]} \ge \text{target}$, 那么 $\text{left}=l, \text{right}=m$, 我们需要证明

$$
\begin{aligned}
& \text{right - left} < r - l\cr
& = m - l < r - l\cr
& = m < r\cr
& = l+\lfloor (r-l)/2\rfloor < r\cr
& = \lfloor (r-l)/2\rfloor < r - l
\end{aligned}
$$

我们可以轻松证明最后一行成立. 另外, $[L, l)<x$ 成立, $x\le [m, R)$ 也成立. 因此 $P$ 成立. 属性 2 得证.

下面我们证明属性 3. 令整数函数(值为整数) $t=\text{right - left}$, 显然, 每次循环都会导致 $t$ 减小, 且直到循环终止前, $t$ 都大于 0. 因此, 程序一定会终止.

最后证明属性 4. 我们现在知道最后一次循环结束时 $P$ 成立, 在最后一次循环后 $\text{left}\ge \text{right}$ 然后退出循环. 由于 $P$ 成立, 所以 $[L, \text{left})<x\le [\text{right}, R)$ 成立. 如果 $\text{left>right}$, 那么 $P$ 就不会成立了, 所以 $\text{left}$ 一定和 $\text{right}$ 相等, 因此 $[L, \text{left})<x\le a[\text{left}]=a[\text{right}]$. 另外, $[\text{right}, R)$ 可能为空, 这意味着 $\text{right}$ 和 $R$ 相等. 此时数组中的所有元素都小于 x, 简单的返回 $\text{left}$ 即可.

综上, 我们知道最后 $\text{left}$ 指向的元素一定是第一个大于等于目标值的元素, 这个元素可能不存在, 那么 $\text{left}$ 指向数组的尾后元素.

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

我们创建版本 3 的代码的不变式 $P:(\text{right}-\text{left} < r-l)\land ([L, \text{left}) \le x)\land (x <[\text{right}, R))$.

我们可以轻松的证明属性 1, 2, 3. 需要注意的是属性 4:

$$
\begin{aligned}
& P \land \text{left}\ge \text{right}\cr
& = [L, \text{left})\le x < a[\text{left}]=a[\text{right}]
\end{aligned}
$$

因此, $\text{left}$ 最后指向的元素是第一个大于目标值的元素, 为了获得最后一个等于目标值的元素, 我们将 $\text{left}$ 减 1. 这个元素可能不存在, 此时仍然返回尾后元素.

注意到在每次迭代中, 版本 2 和版本 3 都比版本 1 少一次 if 判断. 这使得每一次的循环更快. 相对地, 总循环数更多了.

另外还有两种变种:

- 总是查找大于目标值的最小元素. 这使用版本 3 的返回值再加 1 即可找到.
- 总是查找小于目标值的最大元素. 这使用版本 2 的返回值再减 1 即可找到.

## 搜索满足任意条件的元素

我们知道, 版本 2 的代码总能找到大于等于目标值的第一个元素, 版本 3 的代码总能找到大于目标值的第一个元素, 现在我们可以搜索满足 >= 和 > 谓词的元素, 也就可以搜索满足 <= 和 < 谓词的元素, 通过自定义谓词, 我们可以搜索满足任意条件的元素. 许多编程语言的标准库都支持搜索自定义条件的元素的二分搜索.

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

基本思想与版本 3 相同.

## References

- [Binary search algorithm - Wikipedia](https://en.wikipedia.org/wiki/Binary_search_algorithm)
- [c++ template class; function with arbitrary container type, how to define it? - Stack Overflow](https://stackoverflow.com/questions/7728478/c-template-class-function-with-arbitrary-container-type-how-to-define-it)
