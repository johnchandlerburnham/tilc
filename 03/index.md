---
title: "Notes (TILC 03/05): Conditionals"
---

# 3 Conditionals

```
true = \x y.x
false = \x y.y
```

Why these expressions? Probably because it's convenient to have
`false` be the same expression as `0_`:

```
false => \x y. y => \s z. z => 0_
```

This matches the common convention found in imperative programming languages.
We could choose to make `true` equal zero instead, and then change all the
logic functions by negating their inputs, so `(and a b)` in the usual way
becomes

```
(and (not a) (not b))
```

But I'm not sure if there's any tangible
benefit to doing this other than being contrarian.

## 3.1  Logical operations

```
and = \x y. x y (\x y. y) => \x y. x y false
```

The first parameter of `and` returns the second parameter, if it is `true`
and `false` if it is `false`. The only way `And` returns `true` is if both
parameters are `true`.

```
or = \x y. x (\x y. x) y = \x y. x true y
```

Same general idea as `And`. The first parameter selects `true` if `true`
and `y` if `false`.

```
not = \x. x false true
```

Straightforward, it's just flipping the order of the parameter.

## 3.2 A conditional test

```
isZero = \x. x false not false
```

If x is `0_`:

```
isZero 0_ => (\x. x false not false) 0_ =>
0_ false not false =>
not false =>
true
```

If x is `1_`:

```
isZero 1_ => (\x. x false not false) 1_ =>
1_ false not false =>
(\x y. y) not false =>
(\y. y) false =>
false
```

If x is `2_`:

```
isZero 2_ => (\x. x false not false) 2_ =>
2_ false not false =>
(\x y. y) ((\x y. y) not) false =>
(\y. y) false =>
false
```

It doesn't matter how many times we apply `false` to `not`
because `false` applied to any expression is always the Identity
function.

## 3.3 The predecessor function

The predecessor function `pred` is like the opposite of the successor
function `succ`. Instead of adding one to a number, it subtracts one from a
number (the words increment and decrement are sometimes used in other contexts
to describe adding or subtracting one, respectively).

Since numbers are functions that take other functions (higher-order functions),
and apply them a certain number of times, predecessor can be thought of as
an "apply one less time than this number" function.

Recall that `succ` is

```
succ = \n f x. f (n f x)
```

Naively, we might want something that looks like this

```
pred n = \n f x. (invert f) (n f x)
```

where `(invert f)` is the inverse function of `f`:

```
(invert f) (f x) = x
```
But we have a problem: Not every function is invertible.

Take `false` for example:

```
false x = (\x y. y) x = (\y. y) = id
```

Since `false` of any argument `x` is the `identity` function, and since
a function can only have one ouput for any given input, there's no
way for us to build an expression for `(invert false).` And that means
that we can build an `invert` function that is *total*, in that it is
defined over all possible inputs.

But it turns out it is possible to build a total `pred` function,
but we have to try another method. Instead of starting from `n` and working
backwards, we'll start from `0_` and build forwards. If we apply
`succ` `n`-times to `0_`, we get `n`. So if we
apply `succ` only `(n-1)`-times to `0_`., /e'll get `n - 1` which is
`pred n.`

We'll start by building an expression that holds a pair of numbers, `a_`
and `b_`. Suppose we just put them next to each other:

```
a_ b_
```

This might look fine at first, but watch what happens if we give them concrete
values:

```
0_ 0_ => (\s z. z) (\s z. z)
=> \z.z
```

The numbers evaluate against each other! Let's add an abstraction to stop the
application:

```
\x. a_ b_
```

Okay, that stops the application, but only temporarily. If we ever
apply this expression to anything, it'll just reduce back again and throw away
the values in the pair.

What we want is some way to access the two individual values in the pair.

```
\x. x a_ b_
```

We already have functions which will return only their first or second arguments:
`true` and `false`. We can use these to select which element
of the pair we want:

```
(\x. x a_ b_) true  = a_
(\x. x a_ b_) false = b_
```

Great, we've got a pair! Now we need a function which we'll call `phi` to turn
a pair of numbers `(a, b)` into `(a + 1, a)`

```
phi = \p z. z (succ (p true)) (p true)
```

For example,

```
(\p z. z (succ (p true)) (p true)) (\x. x 2_ 1_) =>
(\z. z (succ ((\x. x 2_ 1_) T)) ((\x. x 2_ 1_) T))  =>
(\z. z (succ ((\x. x 2_ 1_) T)) ((\x. x 2_ 1_) T))  =>

```

Now we just need to apply `phi` to `\x. x 0_ 0_`,
`n`-times, and the first element will be equal to `n` and the second will
be `n - 1`:

```
\n. n phi (\z. z 0_ 0_)
```

But since this function is supposed to be the predecessor, we really just want the
second element:

```
pred n = \n. n phi (\z. z 0_ 0_) F
```


## 3.4 Equality and inequalities

A number, `x` is greater than or equal to a number `y`  if applying predecessor
`x`-times to `y` is 0:

```
gte x y = \x y. isZero (x pred y)
````

The reason this function is "greater than or equal to" and not just "equals"
is that the predecessor of `0` is `0`.

But we can get an equals function by simply doing:

```
eq x y = \x y. and (gte x y) (gte y x)
```

Then, not equal is:

```
neq x y = \x y. not (eq x y)
```

Greater than:
```
gth x y = \x y. and (gte x y) (not x y)
```

Less than or equal to:

```
lte x y = \x y. not (gth x y)
```

Less than:

```
lth x y = \x y. not (gte x y)
```
