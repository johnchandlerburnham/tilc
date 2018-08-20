---
title: "Notes (TILC 01/05): Definition"
---

# 1 Definition

**Lambda Calculus**: The text says "the smallest universal programming
language of [sic] the world [, consisting] of a single transformation rule
(variable substitution) and a single function definition scheme." I'm inferring
that by "smallest" the author means "simplest," but I would be interested to
see the claim that "lambda calculus is the simplest possible programming
language" developed further. What does it mean for a programming language
to be "simple"?

[NB. I will investigate this claim later and link back here]

Lamda Calculus consists of *expressions*:

**expression**: an expression is a *name*, a *function* or an *application*.
Any expression can be optionally surrounded by parentheses for clarity.

**name**: Also called a *variable*. Represents values, denoted
by single lettters:

```
a, b, c ...
```

**function**: Also called an *abstraction*, Represents functions (in the
mathematical sense).  Notation is of the form

```
\N. E
```

where `N` is a name and `E` is an expression. `N` is called the *head* and `E`
is called the *body* of the function.

**application**: Represents the concept of apply a function to an argument.
Notation is of the form

```
F N
```

where `F` and `N` are expressions. Application
associates to the left, so

```
F M N
```

is equivalent to

```
((F M) N)
```

**evaluation**: In an application expresion

```
F N
```

where `F` is a function, if the *name* in the head of `F` occurs anywhere in
the body of `F`, replace each occurence of that name with the expression `N`,
and return the body.

Example:

```
(\x. x) y
```

evaluates to simply `y`, and

```
(\x. x y) y
```
evaluates to `y y`.

**the identity function**:

```
id = \x. x
```
returns whatever it is applied to.

## 1.1 Free and bound variables

**bound variable**: The name in the head of a function is called a *bound*
variable, because the function binds it as a paramter.

**free variables**: Any names in the body of a function are not bound by the
function and therefore are *free*.

The formal definition of free vs. bound in this text is confusing.  The
definition is overly detailed and obscures the idea of a function *binding* a
variable.

The principle is that a variable is bound if and only if it is in the body of a
function that *binds* it. Any variables not so bound are free. A variable
cannot be both bound and free in the scope of a single function, and we
really only care about single functions, since we evaluate lambda expressions
one abstraction at a time.

The example given of `y` being both "bound" and "free" in

    (\x. x y) (\y. y)

is right but misleading.  `y` is "free" in the
scope of the first function, but bound in the second.  Despite all appearences,
though, these are not the same `y`.

`\y. y` is the identity function. So is `\x. x` or `\n. n`,
and any other expression of the same form which replaces the `y`'s with any
other letter. These are all equivalent ways to write the same function. And
therefore we can substitute any of them for any other. This property of not
caring which letters of the alphabet we use as long as the form is the same is
called alpha equivalence.

In the case of the example in the text, through alpha equivalence,

    (\x. x y) (\y. y) => (\x. x y) (\n. n)

And presto, `y` is not bound at all in this expression, according to the
definition given in the text.

The reason this is important is because scope matters. A statement might be
true at one scope and false in another, and if we're not careful to keep
a clear idea of what scope we're operating at, we can start badly confusing
ourselves and others.

As an example, let's add an abstraction over `y` to the above:

    \y. (\x. x y) (\y. y)

Now `y` is not free in the expression at all. But we would be badly mistaken if
when we applied this abstraction to an expression `N` we did this:

    (\y. (\x. x y) (\y. y)) N => (\x. x N) (\y. N)

Again, the `y` in `(\y.y)` is not the same `y` as the one in the head
of the outer abstraction. Instead, we have to do this:

    (\y. (\x. x y) (\y. y)) N => (\x. x N)(\y. y)

Which makes sense, since

    (\x. x y)(\y. y) => (\x. x y)(\n. n)

**name-shadowing**: When, in nested abstractions, a variable in an inner
abstraction has the same name as a variable in an outer abstraction.

## 1.2 Substitutions

The description of substitution in the text is a little haphazard, and I think
the formalism in the previous section would have been better served in this
one.

Here's what I would have included
(see [Wikipedia](https://en.wikipedia.org/wiki/Lambda_calculus)):

**alpha equivalence**: Renaming the bound (formal) variables in the expression.
Used to avoid name collisions. Let `x` and `y` be names and let `M[x]` be an
expression `M` containing some `x` terms. Then in the following function,

    (\x.M[x]) => (\y.M[x:=y])

where `M[x:=y]` denotes replacing each instance of `x` in `M` with `y`.

**beta reduction**: Substituting the bound variable by the argument expression
in the body of the abstraction. Let `x` be a name and let `M` and `E` be
expressions. Then in the following application,

    ((\x.M) E) => M[x:=E]

where `M[x:=E]` denotes replacing each instance of `x` in `M` with `E`.

Name shadowing can make both alpha equivalence and beta reduction more
complicated, so the above definitions only really apply without exception
when the variable names in all the lambda abstraction heads are different.
