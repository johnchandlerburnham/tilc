---
title: "Notes (TILC 05/05): Projects for the Reader"
---

# 5 Projects for the reader

[**N.B.**: I have not tested any of this code. I plan on writing a Lambda
Calculus interpreter at some point and when I do, I will test this code
and add a link back here.]

## 1 "less than" and "greater than"

See [section 3.4](#equality-and-inequalities)

## 2 Positive and Negative Integers

Let `(\x. x np nn)` be the pair of natural numbers `np`, and `nn`, the integer
`n` will then be the difference `np - nn`. In other words, `np` is the
positive component of `n` and `nn` is the negative component.

However, in this representation there will be many possible expressions
that signify the same integer. Let us then define the case where either
`np` or `nn` equals `0` as the canonical or simplified representation.

To make things easier let's define some aliases for handling pairs:

```
pair = \a b. (\x . x a b)
fst = \p. p true
snd = \p. p false
```

`pair` makes a pair out of two elements.
`fst` returns the first element of a pair.
`snd` returns the second element of a pair.

We can then define a function `simplifyInt` as:

```
simplifyInt = (\x. ( \ p n. (gte p n)
                            (pair (n pred p) 0)
                            (pair 0 (p pred n))
                   )
                   (fst x)
                   (snd x)
              )

```

If `p` is greater than or equal to `n`, the number is positive, so we apply
`pred` `n` times to `p`, yielding an integer `(p - n, 0)`. Otherwise,
we apply `pred` `n` times to `p`, yielding an integer `(0, n - p)`.

In either case, the integer is now in "lowest terms", so to speak.

## 3 Addition and Subtraction of Integers

Adding two integers `(p, n)` and `(q, m)` is the same as adding their
components and simplifying:

```
addInt = \ x y. simplifyInt
                (pair (add (fst x) (fst y))
                      (add (snd x) (snd y))
                )
```

Subtracting is the same as adding with sign-flip function, called `negate`:

```
negateInt  = \ x . pair (snd x) (fst x)
subtractInt = \ x y. addInt x (negateInt y)
```

## 4 Division of positive integers recursively

The positive integers are the natural numbers, so we'll have our
`div` function return a natural number.

If in `div x d` either `x` or `d` are negative, we'll return 0 to signify
an error.

```
loop = \ test f next start. Y (\r s. test s ((f s) (r (next s)))) start

divUnsafe = \ x d. loop (\s . (lth s d)) 0) (\s. succ) (\s . d pred s) x

div = \x d. (\x d . (and (gte x 0) (gte d 0) (divUnsafe x d) 0))
            (fst (simplify x))
            (fst (simplify d))
```

## 5 Factorial

```
factorial = \n . loop (\m. isZero m 1) mult pred n
```

## 6 Rational Numbers

Our rational number representation will be a pair of integers: a numerator
`x` and a divisor `y`:

```
pair x y
```

## 7 Addition, Subtraction, Multiplication, and Division of Rationals

First we're going to need a way to multiply integers,
if we have two integers `x = (p - n)`, and `y = (q - m)`, then `x`
multiplied by `y` is

```
(p - n) (q - m) = (pq + nm) - (nq + pm)
```

Since according to our integer representation, `p`, `n`, `q`, and `m` are
all natural numbers, we can do this:

```
multiplyInt = \x y. pair (add (multiply (fst x) (fst y))
                              (multiply (snd x) (snd y))
                         )
                         (add (multiply (snd x) (fst y))
                              (multiply (fst x) (snd y))
                         )
```

For adding rationals, since `a/b + c/d = (ad + cb)/bd`, and `a,b,c,d` are
all integers:

```
addR = \x y. pair (addInt (multiplyInt (fst x) (snd y))
                          (multiplyInt (fst y) (snd x))
                  )
                  (multiplyInt (snd x) (snd y))
```

For subtraction,  `a/b - c/d = (ad - cb)/bd`:

```
subtractR = \x y. pair (subtractInt (multiplyInt (fst x) (snd y))
                                    (multiplyInt (fst y) (snd x))
                  )
                  (multiplyInt (snd x) (snd y))
```

Multiplication, `(a/b) * (c/d) = (ac)/(bd)`:

```
multiplyR = \x y. pair (multiplyInt (fst x) (fst y))
                       (multiplyInt (snd x) (snd y))
```

Division, `(a/b) / (c/d) = (ad)/(cb)`:

```
divideR = \x y. pair (multiplyInt (fst x) (snd y))
                     (multiplyInt (snd x) (fst y))
```

## 8 Lists of Numbers

A list can be thought of as either a `nil` element or a `cons` of head and
a tail, where the head is an expression and the tail is a list.

If `x,y,z` are list elments then lists of lengths 0 to 3 are, respectively:

```
0-list = nil
1-list = cons x nil
2-list = cons x (cons y nil))
3-list = cons x (cons y (cons z nil))
```

Let's turn the above into abstractions on the variables `x,y,z,c,n` where
`c` stands for `cons` and `n` stands for `nil`:

```
0-list = \ c n. n
1-list = \ x c n. c x n
2-list = \ x y c n. c x (c y n)
3 list = \ x y z c n. c x (c y (c z n))
```

If we pass in elements to the above list constructors (so that `x_, y_, z_` are
now representing expressions rather than variables):

```
0-list = \ c n. n
1-list = \ c n. c x_ n
2-list = \ c n. c x_ (c y_ n)
3 list = \ c n. c x_ (c y_ (c z_ n))
```


This suggests a nice list encoding as the right-fold of some function `c`
over whatever we want our list elements to be, (with `n` as the base argument).

We already have our `nil` from the above as `\ c n . n`, now we need our
`cons` function.


At first glance the function 2-list looks like a decent definition for `cons`,
since it combines two elements into a list.

```
2-list = \ x y c n. c x (c y n)
```

But our definition of `cons` is that it combines an element (the head) and a list
(the tail) into a list, not two elements.

Watch what happens if we pass an element `x_` and a list into `2-list`

```
\ c n . c x_ (c (\ c n. c a_ n) n)
```

In Haskell list notation, the above is `[x, [a]]`, not `[x, a]`, which
is what we want.

But since a list is a function of `c` and `n`, we can modify 2-list
slightly like so:

```
cons = \ x y c n. c x (y c n)
```

Now if we call `cons` with `x_` and `\ c n. c a_ n` we get

```
cons x_ (\ c n. c a_ n) =>
\ c n. c x_ ((\c n. c a_ n) c n)
\ c n. c x_ (c a_ n)
```

Which does what we want. I like to think of cons as folding some
function `c` over the tail with `n` and then adding one more fold layer
of `c` with the head.

## 9 List Head

Since our list is a fold of a function `c` over the elements
of a list, we call our list with `true`, or `\ x y. x` to throw away the tail.
As an example:


```
(\ c n. c x_ (c y_ n)) true =>
true x_ (c y_ n) =>
x_
```

But if the list is nil `\ c n. n` we want to return `nil`, so our `head`
function is actually:

```
head = \ l. l true nil
```

## 10 List Length

We fold `(\ x. succ)` over the list with nil as `0_`:

```
length = \ l. l (\ x. succ) 0_
```

As an example,

```
length (\ c n. c x_ (c y_ n)) =>
(\ c n. c x_ (c y_ n)) succ 0_ =>
(\ x. succ) x_ ((\ x. succ) y_ 0_)) =>
(succ (succ 0_)) =>
2_
```
We add the extra abstraction over `succ` to throw away the list elements.

## 11 Turing Machine

So the rest of this document is fairly self-contained, but this question is
introducting a pretty big conceptual dependency. Namely, what precisely is
a Turing Machine?

I'll cover this in a future post and link back here.

Also, if I'm going to write a Turing Machine in Lambda Calculus, I absolutely
want to write proper executable code. Writing massive walls of pseudocode that
don't do anything is no fun. Code is meant to run!

So I'm going to go write a Lambda Calculus interpreter, vivify all the dead
notation in this document, and then come back to this question.
