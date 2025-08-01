---
title: The Behavior of Modulo operation
date: 2021-03-16T23:02:50+08:00
katex: true
---

The modulo operator can be found in almost every programming language. However, the
behavior of how they are defined depends on the language.

<!--more-->

## The Modulo Operation

Given two numbers $a$ and $b$, the quotient $q$ and the remainder $r$ of $a$ divided by
$b$ satisfy the following conditions:

$$
\begin{aligned}
& q \in Z \cr
& a = bq + r \cr
& |r| < |b| \cr
\end{aligned}
$$

Obviously, $r$ should be a positive number if $a$ and $b$ are positive. That is, $r$ is the remainder of the
Euclidean division.

However, it will leave a sign ambiguity if $a$ or $b$ is nonpositive. In mathematics, the
result of the modulo operation is an equivalence class, and any number of the class may be
chosen as representative. For example,

$$
14 \bmod -3
$$

may yield 2 or -2. Either of them is correct. In a real programming language, we know, it
can't yield an ambiguous result for a given expression. Which should be chosen?

## Variants of Modulo Definitions

Many programming languages like C/C++ use truncated division which returns the integer nearest
$q$ (real quotient) between zero and $q$:

$$
\text{trunc}\left(\frac{a}{b}\right) =
\begin{cases}
\lfloor a/b\rfloor & a/b \ge 0\cr
\lceil a/b \rceil & \text{else}\cr
\end{cases}
$$

So the remainder will be

$$
r_{\text{trunc}} = a - b \times \text{trunc}\left(\frac{a}{b}\right)
$$

The programming languages like Python have a floored division which returns the
floor function of the real quotient:

$$
\left\lfloor \frac{a}{b} \right\rfloor
$$

So the remainder will be

$$
r_{\text{floor}} = a - b\times \left\lfloor \frac{a}{b}\right\rfloor
$$

IEEE 754 defines the rounding division:

$$
\text{round}\left( \frac{a}{b} \right) = \begin{cases}
\lfloor a/b \rfloor & a/b - \lfloor a/b \rfloor < 0.5\cr
\lceil a/b \rceil \text{ bitand } -2 & a/b - \lfloor a/b \rfloor = 0.5\cr
\lceil a/b \rceil & a/b - \lfloor a/b \rfloor > 0.5
\end{cases}
$$

We have

$$
r_{\text{round}} = a - b \times \text{round}\left( \frac{a}{b} \right)
$$

In C89/C++98, the quotient is rounded in implementation-defined direction. Since C99/
C++11, the quotient is truncated towards zero. So modern C/C++ uses truncated modulo
(the first case above).

In Go, the modulo operator is truncated. In Python, the floored modulo is used (the second
case above).

In Haskell, `mod` is a floored modulo, `rem` is a truncated modulo. [Wikipedia](https://en.wikipedia.org/wiki/Modulo_operation) gives a
full list.

The truncated modulo and the floored modulo are common. I will just cover them.

## The Properties of The Modulo Operations

When $a/b \ge 0$, the truncated modulo and the floored modulo are the same.

$$
r_{\text{trunc}} = r_{\text{floor}}
$$

We just need to study the negative case. Let the real quotient $q$ be

$$
q = \frac{a}{b} = d + \delta
$$

where $d$ and $\delta$ is the integer and fractional parts of $q$, respectively. For example,
$-4.2=-4 + -0.2 = d + \delta$.

Then

$$
\begin{aligned}
r_{\text{trunc}} & = a - b\left\lceil \frac{a}{b} \right\rceil = a-bd\cr
r_{\text{floor}} & = a - b\left\lfloor \frac{a}{b} \right\rfloor = a-bd+b
\end{aligned}
$$

Generally, we always have two available remainder (the positive one and the negative one).
$r_{\text{floor}}$ is the positive one and $r_{\text{trunc}}$ is negative for
$a<0\land b>0$. $r_{\text{floor}}$ is the negative one and $r_{\text{trunc}}$ is positive
for $a>0\land b<0$.

That is, the sign of $r_{\text{floor}}$ is same as the divisor. The sign of
$r_{\text{trunc}}$ is same as the dividend.

## References

- [Modulo operation - Wikipedia](https://en.wikipedia.org/wiki/Modulo_operation)
