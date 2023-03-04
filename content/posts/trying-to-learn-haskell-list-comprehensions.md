---
title: "Trying to learn Haskell - List comprehensions"
date: 2023-02-25
draft: false
tags: ["English", "Haskell", "PHP", "Python"]
---

Every time I tried to learn **functional programming**, I hated it and I did not achieve any appreciable result. 
Probably it will go the same way this time too, anyway I thought to give functional programming, and in particular **Haskell**, one more chance.
For this reason I've just started reading a beautiful book: **[Learn You a Haskell for Great Good!](http://learnyouahaskell.com/)**.

I thought I may write down all the interesting (or weird) things I find out, trying, if possible, to compare them with other things I already know.

So, with no particular order, let's start with **list comprehensions** even if it's not a peculiarity neither of functional programming nor Haskell, but I find them very 
stylish. I mean, I have to start somewhere.

Imagine you need to create a list by taking all the natural numbers less than or equal to 10 and multiplying each one of them by 2. If you are familiar 
with set theory you know that, using roster notation, you're aiming to get this set
```
S = { 2, 4, 6, 8, 10, 12, 14, 16, 18, 20 }
```
that, using the set-builder notation, you can write in this way  
```
S = { 2x | x ∈ ℕ, x ≤ 10 }
```
or even better in this way
```
S = { 2x | x ∈ (1,2,3...,10) }
```
But, why set-builder notation? It might seem a little off-topic, but have faith, it will come in handy in a few seconds.

Going back to our set, if you are writing **PHP** code, you probably will choose something like that
```php
$s = [];
foreach (range(1,10) as $x) {
  $s[] = 2 * $x;
}
```
or, if you are writing **Python** code, something like that
```python
s = []
for x in range(1,10):
  s.append(2 * x)
```
because, truth be told, they are very easily readable. Anyway you might prefer something more compact for a lot of reasons.

In PHP, a more compact way to get the same result could be this one
```php
$s = array_map(fn($x) => 2 * $x, range(1,10));
```
but, as you can see, you need to use array maps and arrow functions.

Some languages, like Python, Haskell and others, provide a syntax construct called list comprehension that allow you to get the same result in a very compact and elegant way.

Let's take a look at Python list comprehensions. In order to get the result you want, you can write this
```python
s = [2*x for x in range(1,10)]
```
where, as you can see, syntactically, you practically embed a loop into a list.

That is good but, in my opinion Haskell does even a better job in terms of syntax. Take a look at that
```haskell
s = [2*x | x <- [1..10]]
```
that produces exactly the result you are expecting
```haskell
[2,4,6,8,10,12,14,16,18,20]
```
The syntax above, looks familiar? I mean, more or less it's the copy of the set-builder notation, isn't it?
Before the vertical pipe (|) you put the output of your list comprehension, and after you put the conditions, also known as predicates.
In short, you draw from the list *[1..10]* to get all your *x* items you need to multiply by 2 to fill in your list.

Apart from this basic example, list comprehensions allow you to do a bunch of things in a context of filtering, transforming and combining lists. 
You can set multiple predicates (separated by commas), and you can also draw from several lists.

Let's say that you want to use all possible combinations of numbers in three simple lists to get a list of tuples. In Haskell, you can write something like that 
```haskell
s = [(x,y,z) | x <- [1,2], y <- [3,4], z <- [5,6]]
```
to get the list of tuples I've just mentioned
```haskell
[(1,3,5),(1,3,6),(1,4,5),(1,4,6),(2,3,5),(2,3,6),(2,4,5),(2,4,6)]
```

Or, let's say that, playing with strings (of course you can use lists of strings too!), you want to mix a little the Ubuntu releases names
```haskell
adjectives = ["Warty", "Hoary", "Breezy", "Dapper", "Edgy"]
animals = ["Warthog", "Hedgehog", "Badger", "Drake", "Eft"]
releases = [adjective ++ " " ++ animal | adjective <- adjectives, animal <- animals]
```
to get something funny like that
```haskell
["Warty Warthog","Warty Hedgehog","Warty Badger","Warty Drake","Warty Eft","Hoary Warthog","Hoary Hedgehog","Hoary Badger","Hoary Drake","Hoary Eft","Breezy Warthog",
"Breezy Hedgehog","Breezy Badger","Breezy Drake","Breezy Eft","Dapper Warthog","Dapper Hedgehog","Dapper Badger","Dapper Drake","Dapper Eft","Edgy Warthog","Edgy Hedgehog",
"Edgy Badger","Edgy Drake","Edgy Eft"]
```

The interesting (and powerful) thing is that you can also create nested list comprehensions if you have particular needs, mostly if you are handling lists that contain lists.
Imagine you have a list of names (first and last names) and you only want to extract the initials of these names to create another list, a list of initials precisely. 

Well, in Haskell, strings are lists too, they are lists of characters. That's why, in order to get something like that
```haskell
["AD","FP","ZB"]
```
the code below should not be a surprise
```haskell
names = ["Arthur Dent", "Ford Prefect", "Zaphod Beeblebrox"]
initials = [ [ initial | initial <- name, initial `elem` ['A'..'Z'] ] | name <- names ]
```
Please, note that the function *elem* (called **infix function** when it's written, with backticks, between parameters) is used to tell if an item exists in a list 
and the used list *['A'..'Z']* contains all the capital letters.
