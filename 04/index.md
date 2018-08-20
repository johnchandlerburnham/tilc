---
title: "Notes (TILC 04/05): Recursion"
---

# 4 Recursion

**recursion**: A pattern where a function calls itself.

The basic idea behind recursion in lambda calculus is that we want to build an
expression that regenerates itself as we reduce it. What I mean by that is
if a function `F` is going to apply itself inside itself, it's reduction needs
to somehow build a new copy of `F`. We're going to want to end up with some
function that transforms `F` into a sequence of repeated applications
of `F`:

```
F (F (F ....)))
```

Imagine we want to apply an expression `F` to itself once. If `F` is the
identity function `id`:

```
id id => (\x.x) (\x.x) => \x.x
```

Let's say we want a function that does this self-application, that we'll
call `M`:

```
M = \f. f f
```

Now supppose we apply `M` to `M`:

```
M(M) = (\f. f f) (\f. f f) =>
(\f. f f) (\f. f f) => \dots
```

This is an infinite loop! `M(M)` regenerates itself perfectly, so that any
reduction just goes `M(M) => M(M) ...` . `M(M)` is in fact the
classic case of a non-terminating lambda expression, and is usually called
&Omega;).

Infinite loops are cool, but what we really want is to modify `M(M)` so that
we add an application of a function `R` at every loop:

```
M (\f. R (f f)) = (\f. f f) (\f. R (f f)) =>
(\f. R (f f)) (\f. R (f f)) =>
R ((\f. R (f f)) (\f. R (f f)) =>
R ( R ((\f. R (f f)) (\f. R (f f)))) =>
...
```

When we abstract over `R`, this becomes the famous `Y` combinator:

```
Y = \r. (\f. f f) (\f. r (f f)) => \r. (\f. r (f f)) (\f. r (f f))
```

**combinator**: A lambda abstraction with no free variables.

---

Another way to think about the `Y` combinator is as a *fixed-point* combinator.

**fixed-point**. If `x = (f x)`, `x` is a *fixed-point* of the function `f`.

Suppose we have a function `fix` such that`(fix f)` that returns a fixed-point of
`f`. Then, by the definition of a fixed-point,

```
(fix f) = f (fix f)
```

This can be expanded indefinitely as

```
fix f => f (fix f) => f (f (fix f)) => ...
```

Which is precisely the same as the `Y` combinator:

```
Y f => f (Y f) = f (f (Y f))
```

so we can say that `Y` is the lambda expression that implements the function
`fix` for lambda functions.

---

We know know how to recurse a function `R` an infinite number of times:

```
Y R => R (Y R) => R (R (Y R)) => ...
```

But suppose we want to only recurse `R` a finite number of times:

```
Y R => R (Y R) => R (R (Y R)) => R ( R ( ... (R B)) ...)
```

We're going to have to bottom-out at some base-case `B`.

But how do we structure our function `R` so that it generates a base case, if
`YR => R(Y R)`? Won't that generate more copies of `R` no matter what we do?

Watch what happens if we apply `Y` to `0_`:

```
Y 0_ => Y (\s z. z) -> (\s z. z) Y (\s z. z) -> (\s z. z) -> 0_
```

`0_` throws away its first argument, and because it throws away the `Y`,
the recursion stops. So we if build an `R` that at some point throws
away the `Y` combinator, we'll lose our means of producing new copies of `R`.

Let's look at the example in the text, which is supposed to sum
all the integers between `n` and `0_`:

```
R = (\r n. isZero n 0_ (n succ (r (pred n))) )
sum n = Y R n
```

Let's do some reductions on this, starting with `sum 0_`:

```
sum n = Y R 0_ =>
Y (\r n. isZero n 0_ (n succ (r (pred n)))) 0_ =>
(\rn.isZero n 0_ (n succ (r (pred n)))) (Y R) 0_ =>
(isZero 0_ 0_ (0_ succ ((Y R) (pred 0_)))) =>
true 0_ (0_ succ ((Y R) (pred 0_)))) =>
0_
```


Now `sum 1`:

```
Y R 1 =>
Y (\rn.isZero n 0_ (n succ (r (pred n)))) 1_ =>
(\rn.isZero n 0_ (n succ (r (pred n)))) (Y R) 1_ =>
(isZero 1_ 0_ (1_ succ ((Y R) (pred 1)))) =>
false 0_ (1_ succ ((Y R) (pred 1)))) =>
1_ succ ((Y R) (pred 1))) =>
1_ succ (Y R 0) =>
1_ succ 0 =>
1_
```

And `sum 2_` for good measure:

```
Y R 2_ =>
Y (\rn.isZero n 0_ (n succ (r (pred n)))) 2_ =>
(\rn.isZero n 0_ (n succ (r (pred n)))) (Y R) 2_ =>
(isZero 2_ 0_ (2_ succ ((Y R) (pred 2_)))) =>
false 0_ (2_ succ ((Y R) (pred 2_)))) =>
2_ succ ((Y R) (pred 2_))) =>
2_ succ (Y R 1_) =>
2_ succ (1_ succ 0_) =>
3_
```

Let's break this down into the general case:

```
loop = \ test f next start. Y (\r s. test s (f (r (next s)))) start
```

First we `test` our state `s` and return either `true` or `false`. In
the example in the text, `test` is `isZero` and `s` is a number `n`.

If our test returns `true` we terminate by returning our `base`, (`0_` in the
above example) otherwise we recurse
and return `f (r (next s))`.
The function `f` is what we're actually interested in evaluating over our
recursion. In the above the function `f` is `add n ` or `(n succ m)`.
The `r` is the stand-in for the rest of the recursion that will
be generated with a `Y R` application. And the `next` is `pred` in
the above example, because we want to count down from `n` to `0_` by
1s.

Supposing we want to loop a function `f` over
the numbers from `m_` to `0_` (technically the term is *fold* `f` over
the numbers `m_` to `0_`, but I don't want to get into folds just now).

All we have to do is fill apply to loop to the corresponding variables:

```
loop (\n. isZero n base) f pred m_ =>
  Y (\r n. isZero n base
          (f (r (pred n)))
    )
  m_
=>
  (\r n. isZero n base (f (r (pred n))))
  ( Y (\r n. isZero base
            (f (r (pred n)))
      )
  )
  m_
=>
(\r n. isZero base
  ( f
    ( (Y (\r n. isZero base (f (r (pred n)))))
      (pred m_))
  )
)
=>
...
```

In this way we can build a wide range of finite recursive structures.
