---
title: "Notes (TILC 02/05): Arithmetic"
---

# 2 Arithmetic

Thus far, everything we've seen with lambda calculus has been a symbol
manipulation game, and not obviously meaningful. So we're going to put some
meaning into it by coming up with a way to do arithmetic.

First, we're going to need numbers. Specifically, we're going to need to
find a coherent way to represent the natural numbers

```
0, 1, 2, 3, ...
```

as lambda expressions. The way we're going to do this is by implementing of
Peano numbers in lambda calculus.

[N.B. I will find/write a good explanation of Peano numbers, link it
here, and rework this to be clearer]

First, we're going to pick an expression for `0`, which we'll call
`Zero`, and then a successor function `succ`, so that

```
0 = Zero
1 = succ Zero
2 = succ (succ Zero)
...
```

and so on. This is the standard Peano definition of the natural numbers.

But there's a neat trick we can do in our lambda calculus implementation
of Peano numbers. Notice how the Peano numbers are defined as a zero and
then layers of a succesor function on top of zero.
We know how to express functions in lambda calculus, so what if we rewrote
the above definition of Peano numbers as functions of two parameters `s` and
`z`, which will stand in for whatever expressions we pick for the successor
function and the zero, respectively. We'll call these lambda functions that
return numbers `n_`, where `n` is the number they return:

Let's start with `0_`. We know that all the `n_` functions are
functions of `s`, (`succ`) and `z` (`Zero`), so the
heads of all our `n_` will be the same: `\s z`.

For `0_` we throw away the successor and just return the zero:

```
0_ = \s z. z
```

For `1_` we apply the successor once:

```
1_ = \s z. s z
```

For `2_` we apply the succesor twice:

```
2_ = \s z. s (s z)
```

And so on, giving us the following functions:

```
0_ = \s z. z
1_ = \s z. s z
2_ = \s z. s (s z)
3_ = \s z. s (s (s z))
...
```

Now here's the trick. The above functions are already lambda expressions. So
while we could find other expressions that would work as numbers, we can
just use the above `n_` functions as numbers directly.

If we do that then we already found an expression for our zero: `\sz.z`.
Which is just a function that throws away its argument and returns our old friend
the identity function.

One way to visualize this structure is to imagine `n_` as the set of functions
that apply another function to an argument `n` times. `Zero` is "apply zero
times" or "don't apply", one is "apply once", two is "apply twice" and so on.

Now we just need a `succ` function. `succ` takes a number and returns
that number plus one. But if our numbers are functions that apply another
function to an argument some number of times, then `succ` takes
a number `n`, a function `f` and an argument `x` then applies `f` to x
an `(n+1)` number of times. In other words, `succ` is like an "apply once
more"function.

Here's what that function looks like:

    succ = \n f x. f (n f x)

Let's see what happens when we apply `succ` to a number:

```
succ 0_ = (\n f x. f (n f x)) (\s z. z) =>
\f x. f ((\s z.z) f x) =>
\f x. f x
```

Through alpha equivalence `\f x. f x` is the same as `\s z. s z`
which is `1_`.

Applying `succ` to `1_`

```
succ 1_ = (\n f x. f (n f x)) (\s z. s z) =>
\fx. f ((\s z. s z) f x) =>
\f x. f (f x) =>
2_
```

## 2.1 Addition

Once we understand `succ`, `add` is pretty easy . Whereas `succ` is
"apply `f` to `x`, `n`-times", `add` is "apply `f` to `x`, `n`-times, then
`m`-times, for a total of `(n + m)` applications."

```
add = \m n f x. m f (n f x)
```

Let's see this in action:

```
add 1_ 2_ = (\n m f x. m f (n f x)) (\s z. s z) (\s z. s (s z)) =>
\f x. (\s z. s z) f ((\s z. s (s z)) f x) =>
\f x. (\z. f z) ((\s z. s (s z)) f x) =>
\f x. f (\s z. s (s z) f x) =>
\f x. f (f (f x)) =>
3_
```

A useful pattern we can notice is that `add` can also be expressed as:

```
add = \m n f x. m f (n f x) => \m n. \f x.  m f (n f x) =>
\m n. m (\f x. n f x) => \m n. m (\n f x. n f x) n =>
\m n. m succ n
```

This might seem less mysterious if we do the equivalence in reverse.

```
\m n. m succ n => \m n. m (\n f x. f (n f x)) n =>
\m n. m (\f x. f (n f x)) => \m n. \f x. m f (n f x) =>
\m n f x. m f (n f x)
=> add
```

## 2.2 Multiplication

`multiply` is even easier, we just feed one number into another:

```
multiply n m = \n m t. n (m t)
```

If we multiply `2_` by `2_`:

```
multiply 2_ 2_ =>
(\n m t. n (m t)) (\s z.s (s z)) (\s z. s (s z)) =>
\t. (\s z. s (s z)) (\s z. s (s z) t) =>
\t. (\s z. s (s z)) (\z. t (t z)) =>
\t. \z. (\z. t (t z)) ((\z. t (t z)) z) =>
\t z. (\z. t (t z)) ((\z. t (t z)) z) =>
\t z. (\z. t (t z)) (t (t z)) =>
\t z. t (t (t (t z))) =>
\s z. s (s (s (s z))) =>
 4_
```

Okay, maybe that was a little tough to read. Maybe a little substitution
will make things cleaner?

```
multiply 2_ 2_ =>
(\n m t. n (m t)) 2_ 2_ =>
\t. 2_ (2_ t) =>
\t. 2_ (\x. t (t x)) =>
\t. \z. (\x. t (t x)) ((\x. t (t x)) z) =>
\t. \z. (\x. t (t x)) (t (t z)) =>
\t. \z.t (t (t (t z))) =>
\t z. t (t (t (t z))) =>
4_
```

I don't think that helped much. There is, clearly, a reason why we do not write
programs directly in lambda calculus.
