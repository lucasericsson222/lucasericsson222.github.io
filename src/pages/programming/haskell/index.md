---
layout: '/src/layouts/BaseLayout.astro'
title: "Haskell. Confusing"
pubDate: 2024-03-24
description: 'Learning about Haskell from the ground up'
author: 'kasisuli'
---

<h1><span class="tokipona" lang="tok"> lawa pakala</span> Haskell</h1>

So, my first experience with Haskell was through this research article: 

I found it by:
- exploring https://webring.xxiivv.com/
- clicking on beespace
- going to https://lunabee.space/extraspace/port.html
- clicking `enter the nexus` 
- and then the green "`﹝`" looking symbol. 
- Then, "`enter the archives`", 
- press "`...`" 
- then "`stacks cart`" 
- and "`urn:issn:2159-8118`" (now we are at https://lunabee.space/stacks/typepapers.html). 
- Finally, I was looking at the first most paper https://kar.kent.ac.uk/98022/.


And here it is, the first Haskell code I saw:

```hs
type Inverse a = a %1 -> () 
-- recall () is the unit type 1 of Haskell

divide :: a %1 -> Inverse a %1 -> ()
divide x u = u x
```

Firstly, this code is bizarre in a few ways, and also a rough way to start off a language.

Like, what is `%1` what is `divide x u = u x`.

Well, it turns out that `%1` is the syntax for a linear type... and here the rabbit hole continues.


https://hackage.haskell.org/package/linear-base

These take you here: https://arxiv.org/abs/1710.09756
which introduces this notation for a function that uses linear types `⊸`. `%1 ->` is an alternate notation.

Linear types have some interesting properties: 
when the result is consumed once, the input is consumed once. 

Now, what does consumed mean?
It appears to enforce a couple of rules:

> * Returning x unmodified
> * Passing x to a linear function
> * Pattern-matching on x and using each argument exactly once in the same fashion.
> * Calling it as a function and using the result exactly once in the same fashion.
>
> -- https://ghc.gitlab.haskell.org/ghc/doc/users_guide/exts/linear_types.html

// elaborate more here

Since all this is described in Haskell, I decided to take some time and learn it.

## Learning Haskell

What's a monad? "A monad is a monoid in the category of endofunctors" yeah right.

Totally understand it now.

After further reading:
Haskell types are a Category.

# Functors

Functors are mappings between categories.
So, a functor in haskell would be mapping the Hask set to another Hask set, without modifying morphism or structure.

Some a functor would take:

```
A --> B
 \    |
  \   V
   \> C
```
and produce
```
f(A) --> f(B)
   \       |
    \      |
     \     v
      \> f(C)
```
Which has the same structure but each inside was changed. 
This ends up being any sort of mapping function. 

For example, in python, something like:
```py
a = [0, 1, 2]
addone
    = lambda item: item + 1
f = lambda x: map(x, addone)

print(f(a)) # produces [1, 2, 3]
```
this `f` would be a functor because it is a mapping between python types to other python types, but it doesn't change the order of the list.

In Haskell:
```hs
addone :: Int -> Int
addone x = x + 1

let a = [0, 1, 2]
in map a addone
```

but it appears that in Haskell a functor isn't the mapping but a type that has a mapping or inherits one? // need to figure out more about this here.

In Haskell, a functor is defined as
```hs
class Functor f where
    fmap :: (a -> b) -> f a -> f b
    (<$) :: a -> f b -> f a
```
So types can inherit the class functor. `f` is a container in this case, and `(a -> b)` is the mapping, with `f a -> f b` showing how it's only changing the item in the container.

# References
[Youtube: Haskell for Imperative Programmers #36 - Category Theory (Functors, Applicatives, Monads)](https://youtu.be/Jsmt4uaL1O8)