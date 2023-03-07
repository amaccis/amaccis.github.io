---
title: "Trying to learn Haskell - Recursion"
date: 2023-03-04
draft: false
tags: ["English", "Haskell", "PHP"]
---

It's a fact that we, bad programmers, avoid the recursion as much as we can. Maybe it's a practice you absorb when you program with imperative languages or, 
somehow you're pushed to think about optimization (iterative algorithms are often faster than recursive ones) or you tell yourself that recursion produces code hard to read, 
to debug and to maintain, but the fact is that whenever you can apply loops (almost always) you do it.

That's a pity, because a little at a time, you risk of getting rusty with the ability of thinking recursively. And what is thinking recursively but a strategy for breaking down 
a problem into smaller sub-problems easier to solve?

Clearly this kind of approach changes if you write code using languages like **Haskell** because rather than specifying how to compute something, you need to declare what 
something is.

So thanks to Haskell we can talk about Tool! And in particular about the title track of their beautiful album Lateralus.
```
black 
then 
white are
all I see 
in my infancy 
red and yellow then came to be
```
It is well known that Maynard James Keen is a genius, but this post is not about that (although it could!), so let's count the syllables of the lines above. 
1 1 2 3 5 8: Fibonacci sequence. Using Fibonacci sequence to talk about recursion is trivial, but it is the only way I could think for quoting Tool, and also it's a good way 
to realize, that recursive functions can turn out to be more readable than iterative ones.

What we know about Fibonacci sequence is
```
Fib(0) = 0
Fib(1) = 1
Fib(n) = Fib(n - 1) + Fib(n - 2) for n > 1
```

Let's focus on writing a function that takes as input a number _n_ and returns the value of _Fib(n)_. Just to keep the things simple, we can take for granted that the input 
will be always a number greater than or equal to zero, freeing us from input validation.

In our ardour of avoiding recursion, we, bad programmers, are perfectly able to write something like this
```php
function fib(int $number): int 
{
	$sequence = [0, 1];
    for ($i = 2; $i <= $number; $i++) {
        $previousNumber = $sequence[$i-1];
        $previousPreviousNumber = $sequence[$i-2];
        $sequence[$i] = $previousNumber + $previousPreviousNumber;
    }
    return $sequence[$number];
}
```
also because **PHP** (in this case) or other imperative languages, allow us to do it. 

It's not bad, and it works, but, the code above is really that readable? Maybe... an array containing the first two numbers represents the sequence, is filled with the following 
numbers through a loop that stops only when the target number is reached and at the end that number is taken from the sequence.

But what about thinking recursively? I mean, again using **PHP**. Take a look at the function below
```php
function fib(int $number): int 
{
    switch ($number) {
        case 0:
            return 0;
        case 1:
            return 1;
        default:
            return fib($number-1) + fib($number-2);    
    }
}
```
It isolates the two base cases (_Fib(0)_ and _Fib(1)_) and for the other numbers, applies the recursion. This solution is far more readable than the previous one, isn't it?
Of course, it could have been written with `if()/else if()/else` statements too, but it would be less comparable to Haskell.

So, let's move on to Haskell!
```haskell
fib :: Int -> Int 
fib 0 = 0
fib 1 = 1
fib n = fib (n-1) + fib (n-2)
```
Leaving aside the type signature (that simply tells that the function _fib_ takes an integer as input and returns an integer), it's amazing how stylish the syntax is. 
Unmistakable, straight to the point.

Another straightforward example is n!, that is factorial numbers. What we know is:
```
n! = 1 for n = 0
n! = n*(n-1)! for n ≥ 1
```

Now, in Haskell, we can simply write something like that
```haskell
factorial :: Int -> Int
factorial 0 = 1
factorial n = n * factorial (n-1)
```
Could it be more readable? I doubt it.