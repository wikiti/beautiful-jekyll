---
layout: post
published: false
title: 'Technique: Memoization'
---
## What is it?

So, you've an amazing program that computes, for example, a factorial of a number. We know that the recursive algortihm for that operation is the following:

````
function factorial(n):
  if n <= 1
    return 1
  else
    return n * factorial(n - 1)
````

For example, implementing it on Haxe, it'll result in the following code:

````haxe
public function factorial(n: Int): Int {
  return n <= 1 ? 1 : (n * factorial(n - 1));
}
````

Amazing! You can now use your little and readable function for computing factorials. The problem is... It's not optimal. For example, `factorial(5)` is computed as `6*5*4*3*2*1`. Everytime you call that function with an unique input, it'll always return the same output; this is known as a [deterministic algortihm](https://en.wikipedia.org/wiki/Deterministic_algorithm).

Think about it! If you know what's the value of `factorial(5)`, why shouldn't I use it to calculate `factorial(6)` as `6*factorial(5)`? Here's where we use [memoization](https://en.wikipedia.org/wiki/Memoization): we want to store those values in memory, so we only have to compute them **once**.


## How to use it

## Examples in different languages

## Resources

- [Wikipedia: Memoization](https://en.wikipedia.org/wiki/Memoization)
- [Wikipedia: Deterministic algorithm]
- [StackOverflow: Difference between memoization and dynamic programming](http://stackoverflow.com/questions/6184869/what-is-difference-between-memoization-and-dynamic-programming)