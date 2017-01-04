---
layout: post
published: true
title: 'JavaScript: Command pattern'
subtitle: Undo and redo!
---
Happy new year! I've started reading the [Game Programming Patterns](http://gameprogrammingpatterns.com/) book. It's greate! Well written, funny, and extremely practical. I'd like to share a simple implementation in *JavaScript* of the *Command* patterns, the first introduced pattern in the book.

## Definition of *Command pattern*

That's a good question! I highly recommend you to read [this chapter](http://gameprogrammingpatterns.com/command.html) from the book. Basically, this pattern is useful to refactor code and create a layer of *abstraction* between direct input and executed actions. This can be extended by using lists, like command queues or command histories.

## Practical example

That's a good question! It can be implemented in a web-application editor based on JavaScript (like an [online paint](http://www.onemotion.com/flash/sketch-paint/)).

In this section, I'll try to explain how to code a **undo-redo command framework** for a webpage. Our ~~generic and~~ simple program will be a [stack calculator](https://en.wikipedia.org/wiki/Reverse_Polish_notation) (does that thing still exists?). These are the supported buttons and their associated actions:

- **Press "[Number]"**: Add a new element to the stack with the value *[Number]*.
- **Press "+"**: Remove the top 2 values from the stack, sum them, and add the result to the stack.
- **Press "-"**: Remove the top 2 values from the stack, substract them, and add the result to the stack.
- **Press "*"**: Remove the top 2 values from the stack, multiply them, and add the result to the stack.
- **Press "/"**: Remove the top 2 values from the stack, divide them, and add the result to the stack.
- **Press "CLS"**: Restart the calculator.

All the previos actions can be *undone* and *re-undone*. We'll need a global list to store the history of actions performed. Basically, everytime a new action is executed, it'll be added to the global list as a command object.

```
... [Command] [Command] [Command] [Command] [Command] ...
                  |         |        |
               Previous  Current    Next

```

Therefore, the next commands are required to interact with the commands queue:

- **Press "Undo"**: Undo the last action, if available.
- **Press "Redo"**: Redo the next action, if available.

Note that the previous commands will not add new elements to the command queue (should you undo an undo? Isn't that a redo? :)).

## Framework

Let's create a framework to manage any kind of application, and, after that, let's create the stack calculator. Here's the framework:

```js
// Create a global manager with an auto-executable anonymous function.
var Commander = (function(scope) {
  'use strict';

  // Manager object instance.
  var obj = {};
  
    // -- Private --

  // Command list.
  var _commands = {};
  // Commands list for undo/redo.
  var _list = [];
  // Current command index on `list`.
  var _current = 0;

  var _currentCommand = function() {
    var name = _list[_current];
    return _commands[name];
  };
  
    // -- Public --
  
  // Add a new command to the list (name shortcut). `name` must be a string, and `command`
  // must be an object which responds to `execute` and `undo`.
  obj.addCommand = function(name, command) {
    _commands[name] = command;
  };
  
  // Execute a command by its name.
  obj.execute = function(name) {
    var cmd = _commands[name];
    
    // Remove the actions after `current` command in the stack. Of course,
    // if we undo 3 times, we can redo 3 times. However, if we undo 3 times,
    // and execute a new action, we can no longer those 3 old commands.
    // This action will not affect the list if the `current` points to
    // the last element.
    _list = _list.slice(0, _current);
    
    // Execute the command, add it to the top of the stack and increment
    // the current index.
    cmd.execute();
    _list.push(name);
    _current += 1;
  };
  
  // Check if the the next command can be `undone`
  obj.canUndo = function() {
    return _current > 0;
  };
  
  // Check if the next command can be `redone`.
  obj.canRedo = function() {
    return _current < _list.length;
  };
  
  // Undo the previous command executed.
  obj.undo = function() {
    // Stop if there are no elements on the list.
    if(!obj.canUndo()) return false;
    
    // Move back, and undo the previous command.
    _current -= 1;
    _currentCommand().undo();
    return true;
  };

  // Redo the next command.
  obj.redo = function() {
    // Stop if there are no elements on the list.
    if(!obj.canRedo()) return false;
    
    // Move forward, and execute the next command.
    _currentCommand().execute();
    _current += 1;
    return true;
  };
  
  // Restart commands list.
  obj.restart = function() {
    _list = [];
    _current = 0;
  };

  // Return the instance.
  return obj;
})(window);

```

Cool, isn't it? Now, don't be afraid; it's very simple to use. To add new commands to our `Commander`, use the `addCommand` method. It recieves a `name` (string) and an execution object (`execute` and `undo` functions). For example:

```js
Commander.addCommand('say-hi', {
  execute: function() {
    console.log('Hello!');
  },
  undo: function() {
    console.log("Un-hello!");
  }
});
```

To actually execute a command, use the `execute` method:

```js
Commander.execute('say-hi'); // Prints "Hello!"
```

To undo the previous command, use the `undo` method:

```js
Commander.undo(); // Prints "Un-hello!"
```

To redo the next command, use the `redo` method:

```js
Commander.redo(); // Prints "Hello!"
```

We may use this now as a base to implement our application!

## Stack calculator

> **Note:** To speed up the process, I'll be using [jQuery](https://jquery.com/) to query objects and setup events.

Let's create a HTML view to show our calculator, with some buttons and elements.

```html
<!-- Current stack -->
<div id="stack">
</div>

<!-- Number buttons -->
<div class="numbers">
  <button id="btn-0">0</button>
  <button id="btn-1">1</button>
  <button id="btn-2">2</button>
  <button id="btn-3">3</button>
  <button id="btn-4">4</button>
  <button id="btn-5">5</button>
  <button id="btn-6">6</button>
  <button id="btn-7">7</button>
  <button id="btn-8">8</button>
  <button id="btn-9">9</button>
</div>

<!-- Operator buttons -->
<div class="operators">
  <button id="btn-add">+</button>
  <button id="btn-substract">-</button>
  <button id="btn-multiply">*</button>
  <button id="btn-divide">/</button>
</div>

<!-- Other -->
<div class="other">
  <button id="btn-clear">Clear</button>
  <button id="btn-undo">Undo</button>
  <button id="btn-redo">Redo</button>
</div>
```

Now, we need some javascript code. We need some *helper* functions to interact with the stack calculator:

- **Push** a new number.
- **Pop** the last number.
- **Reset** the calculator.

The numbers will be stored as `<span>` elements in the `#stack` container.

```js
function push(n) {
  $('#stack').append(
    $('<span>', { text: String(n) })
  )
};

function pop() {
  var value = $('#stack').find('span').first().remove().text();
  return parseInt(value) || 0;
}

function reset() {
  $('#stack').empty();
}
```

Now, let's add the commands for each number. This is straightforward:

```js
Commander.addCommand('0', {
  execute: function() { push(0); },
  undo: function() { pop(); }
});

Commander.addCommand('1', {
  execute: function() { push(1); },
  undo: function() { pop(); }
});

// ...

Commander.addCommand('9', {
  execute: function() { push(9); },
  undo: function() { pop(); }
});

```

Of course, this code can be refactored to look more professional (I'll let you do the hard work for me this time).

Now, the operators. Since we need to keep track of the stack elements before doing any operation, we need to use *"closures"*, which can be achieved with... guess what? more anonymous functions!

```js
Commander.addCommand('+', (function() {
  var prev = [];

  return {
    execute: function() {
      var n1 = pop(), n2 = pop();
      prev.push(n1);
      prev.push(n2);
      
      push(n1 + n2);
    },
    undo: function() {
      pop();
      var n1 = prev.pop(), n2 = prev.pop();
      push(n2);
      push(n1);
    }
  };
})());

Commander.addCommand('-', (function() {
  var prev = [];

  return {
    execute: function() {
      var n1 = pop(), n2 = pop();
      prev.push(n1);
      prev.push(n2);

      push(n1 - n2);
    },
    undo: function() {
      pop();
      var n1 = prev.pop(), n2 = prev.pop();
      push(n2);
      push(n1);
    }
  };
})());

Commander.addCommand('*', (function() {
  var prev = [];

  return {
    execute: function() {
      var n1 = pop(), n2 = pop();
      prev.push(n1);
      prev.push(n2);

      push(n1 * n2);
    },
    undo: function() {
      pop();
      var n1 = prev.pop(), n2 = prev.pop();
      push(n2);
      push(n1);
    }
  };
})());

Commander.addCommand('/', (function() {
  var prev = [];

  return {
    execute: function() {
      var n1 = pop(), n2 = pop();
      prev.push(n1);
      prev.push(n2);

      push(n1 / n2);
    },
    undo: function() {
      pop();
      var n1 = prev.pop(), n2 = prev.pop();
      push(prev.pop());
      push(prev.pop());
    }
  };
})());
```

They can be refactored, too. Now, let's bind our buttons to those commands:

```js
$("#btn-0").click(function() { Commander.execute("0"); });
$("#btn-1").click(function() { Commander.execute("1"); });
// ...
$("#btn-9").click(function() { Commander.execute("9"); });

$("#btn-add").click(function() { Commander.execute("+"); });
$("#btn-substract").click(function() { Commander.execute("-"); });
$("#btn-multiply").click(function() { Commander.execute("*"); });
$("#btn-divide").click(function() { Commander.execute("/"); });
```

And finally, we'll map the *Clear*, *Undo* and *Redo* buttons to some actions:

```js
$("#btn-clear").click(function() {
  reset();
  Commander.restart();
});

$("#btn-undo").click(function() {
  Commander.undo();
});

$("#btn-redo").click(function() {
  Commander.redo();
});
```

And we're done! Here's a live example on [this JSFiddle](https://jsfiddle.net/887qen0u/16/). Oh, and the code is clean and refactored on this one!

Happy coding!