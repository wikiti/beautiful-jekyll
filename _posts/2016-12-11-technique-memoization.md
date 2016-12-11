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

## But... what's the difference between dynamic programming and memoization?

Good point!

...

## How to use it

The best way to use this techniq is by using store structures with high access speeds, like [hashes tables](https://en.wikipedia.org/wiki/Hash_table) (with *O(1)*) or [binary search tree](https://en.wikipedia.org/wiki/Binary_search_tree) (with *O(log n)*). The quickest way to get started is by using dictionaries or maps, which are present in almost every programming language.

The concept is pretty simple; create a *memory object* which will store all previous computations. If the value is found, it'll be retrieved instead of being re-computed. Otherwise, the value must be computed for the first time. The pseudoce it's the following:

````
memory := {}

function memo(input):
  if memory[input]
    return memory[input]
  else
    // do whatever you want with input, and store it
    output := input * 2
    memory[input] = output
    return output
````

For example, the factorial function, which is recursive, can be optimized with memoization:

````
memory := {}

function factorial(n):
  if memory[n]
    return memory[n]
  else
    value := 0
    if n <= 1
      value := 1
    else
      value := n * factorial(n - 1)
    
    memory[n] := value
    return value
````

## Examples in different languages

- C++

````cpp
````

- Haxe

````haxe
````

- Java

````java
````

- Javascript

````javascript
````

- Python

````pythom
````

- Ruby

````ruby
````

- Rust

````rust
````


