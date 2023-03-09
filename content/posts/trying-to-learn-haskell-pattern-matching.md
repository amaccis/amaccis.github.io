---
title: "Trying to learn Haskell - Pattern matching"
date: 2023-03-09
draft: false
tags: ["English", "Haskell", "Python"]
---

The first time I read about **Haskell** pattern matching, **Python** slicing syntax came to my mind.

If you are not familiar with Python slicing, it's easier than you think. Imagine you have a list of _n_ items, and you want to get a subset of that list, let's say you want all 
the items of the original list except for the first one. Python solves the problem in a simple and elegant way. Look the example below
```python
numbers = [4,8,15,16,23,42]
print(numbers[1:])
```
I practically selected a subset of the list by specifying that the second item (with key 1) is the starting point and that the last item is the ending point
(without specifying anything after the colon).

Ok and what about getting a new subset of the previous original list but this time picking the first item and excluding the last one?
```python
print(numbers[:-1])
```
In the same way, I selected the subset by specifying the starting point (key 0 is implied if you do not write anything) and the ending point (-1 represents the second to last item).

And that's the point, with Python you can slice lists (actually not only lists, but it's not important now) using the colon like a delimiter, by specifying where you start and/or 
where you end slicing.

It's not the same with Haskell, but now that you got the gist, let's see how pattern matching works.

Imagine you have a list of numbers, and you want to get another list by multiplying the first number of the list by every and each other number.
Let's say we have something like this
```
{2,3,4,5,6,7}
```
and we need to write a function able to return
```
{6,8,10,12,14}
```

With the help of [list comprehensions](../trying-to-learn-haskell-list-comprehensions), we can write the function below
```haskell
handleList :: (Num a) => [a] -> [a]
handleList [] = []
handleList (x:xs) = [ x*a | a <- xs ]
```
that, used in this way
```haskell
handleList [2,3,4,5,6,7]
```
returns exactly what we were expecting
```haskell
[6,8,10,12,14]
```

Let's talk about the function above. Leaving aside the first line, the type signature (that basically says that the function takes a list of `a` as input and returns a list of 
`a` as output) and the second line, the edge case (that basically says that in case of empty list as input the function returns an empty list), we can focus on the last line.
The function knows that it needs to slice the input list, using the first item `x` and the other items `xs` in different ways.

The powerful thing is that you can have also something like `x:y:xs` if you are interested in handling differently the first item, the second one and the others, or even something 
like `x:_:xs` if you are interested in handling the first item and the others except for the second one, that has to be ignored.

The function `handleList`, updated like this
```haskell
handleList :: (Num a) => [a] -> [a]
handleList [] = []
handleList (x:_:xs) = [ x*a | a <- xs ]
```
and used with the same input
```haskell
handleList [2,3,4,5,6,7]
```
this time returns 
```haskell
[8,10,12,14]
```

Again, it's stylish, isn't it?