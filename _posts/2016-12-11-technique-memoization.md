---
layout: post
published: true
title: 'Technique: Memoization'
subtitle: 'Compute it once, use it everyday'
---
So, you've an amazing program that computes, for example, a factorial of a number. We know that the recursive algorithm for that operation is the following:

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

Amazing! You can now use your little and readable function for computing factorials. The problem is... It's not optimal, by far. For example, `factorial(5)` is computed as `6*5*4*3*2*1`. Everytime you call that function with the same input, it'll always return the same output; this is known as a [deterministic algorithm](https://en.wikipedia.org/wiki/Deterministic_algorithm).

Think about it! If you know what's the value of `factorial(5)`, why shouldn't I use it to calculate `factorial(6)` as `6*factorial(5)` instead of `6*5*4*3*2*1`? Here's why we use [memoization](https://en.wikipedia.org/wiki/Memoization): we want to store those values in memory, so we only have to compute them **once** and we can retrieve them later.

## But... what's the difference between dynamic programming and memoization?

Good point! The different is pretty simple:

- Memoization is a **technique** used to optimize algorithms that will be constantly used to compute similar or overlapped values (for example, factorials). Using this, you compute it once, store it, and then retrieve it if needed; this means that the first time will take much more time that the next ones. This can be applied to almost any deterministic algorithm
- Dynamic programming is a method for solving complex problems with overlapping solutions by breaking them into smaller problems, and storing each solution on a cell of a given table. The next time that subproblem is found, the value will be retrieved and reused. This reduces computation time by increasing memory usage.

## How do I use it?

The best way to use this techniq is by using store structures with high access speeds, like [hashes tables](https://en.wikipedia.org/wiki/Hash_table) (with *O(1)*) or [binary search trees](https://en.wikipedia.org/wiki/Binary_search_tree) (with *O(log n)*). The quickest way to get started is by using dictionaries or maps, which are present in almost every programming language.

The concept is pretty simple; create a *memory object* which will store all previous computations of each input. If the value is found, it'll be retrieved instead of being re-computed. Otherwise, the value must be computed for the first time. The pseudoce it's the following:

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

## Can you show me some examples?

Sure. Here are some examples for the factorial function for different languages.

### C++

````cpp
#include <iostream>
#include <map>

int factorial(int n) {
  static std::map<int, int> memory = std::map<int, int>();

  auto found = memory.find(n);
  if(found == memory.end()) {
    // Not found
    return memory[n] = n <= 1 ? 1 : (n * factorial(n - 1));
  }
  else {
    // Found
    return found->second;
  }
}

int main() {
  std::cout << factorial(5) << std::endl;
}
````

### Haxe

````haxe
class Factorial {
  private static var _memory: Map<Int, Int>;
  
  public static function factorial(n: Int): Int {
    if(_memory == null) _memory = new Map<Int, Int>();
    
    var val = _memory[n];
    if(val == null) {
      // Not found
      return _memory[n] = n <= 1 ? 1 : (n * factorial(n - 1));
    }
    else {
      // Found
      return val;
    }
  }
  
  public static function main() {
    trace(factorial(5));
  }
}
````

### Javascript

````javascript
var factorial = (function() {
  var memory = {};

  return function(n) {
    var val = memory[n];
    if(val === undefined) {
      // Not found
      return memory[n] = n <= 1 ? 1 : (n * factorial(n - 1));
    }
    else {
      // Found
      return val;
    }
  }
})();

console.log(factorial(5));

````

### Python

````python
memory = {}
def factorial(n):
  if n not in memory:
    memory[n] = 1 if n <= 1 else (n*factorial(n-1))
  
  return memory[n]

print(factorial(5))
````

### Ruby

````ruby
def factorial(n)
  @memory ||= {}
  @memory[n] ||= n <= 1 ? 1 : (n * factorial(n - 1))
end

puts factorial(5)
````

## Can I *memoize* anything?

Almost. You can also create wrappers or modules to refactor code and be able to memoize anything. For example, in ruby, you may wrap everything in a lambda with a local memory variable:

````ruby
def memoize(function)
  memory = {}
  ->(*args) do
    args = args.first if args.size == 1
    memory[args] ||= function.call(*args)
  end
end

factorial = memoize ->(x) { x <= 1 ? 1 : x * factorial.call(x - 1) }
puts factorial.call(5)
````

You can apply this to almost any deterministic function!

````ruby
fibonnaci = memoize ->(n) do
  return n if n <= 1
  fibonnaci.call(n - 1) + fibonnaci.call(n - 2)
end
puts fibonnaci.call(6)

rectangle_area = memoize ->(x, y) { x * y }
puts rectangle_area.call(5, 4)
````

Easy peasy!

## In summary

- Memoization is a technique used to optimize algorithms which is used to store already computed values, then retrieve them if required instead of re-calculating them.
- It can be implemented easily on almost every language easily.
- Dynamic programming and memoization are not the same.

Happy coding!