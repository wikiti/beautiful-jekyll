---
layout: post
published: false
title: 'JavaScript: Command pattern'
---
I've started reading the [Game Programming Patterns](http://gameprogrammingpatterns.com/) book. It's greate! Well written, funny, and extremely practical. I'd like to share a simple implementation in *JavaScript* of the *Command* patterns, the first introduced pattern in the book.

## Definition of *Command pattern*

That's a good question! I highly recommend you to read [this chapter](http://gameprogrammingpatterns.com/command.html) from the book. Basically, this pattern is useful to refactor code and create a layer of *abstraction* between direct input and executed actions. This can be extended by using lists, like command queues or command histories.

## Practical example

That's a good question! It can be implemented in a web-application editor based on JavaScript (like an [online paint](http://www.onemotion.com/flash/sketch-paint/)).

In this section, I'll try to explain how to code a **undo-redo command framework** for a webpage. Our ~~generic and~~ simple program will be a stack calculator. These are the supported keys and their associated actions:

- **Press Space**: Add a new element to the stack.
- **Press A**: Add 1 to the top value of the stack.
- **Press S**: Substract 1 from the top value of the stack.
- **Press M**: Remove the top 2 values from the stack, multiply them, and add the result to the stack.
- **Press D**: Remove the top 2 values from the stack, divide them, and add the result to the stack.
- **Press R**: Restart the stack.

All the previos actions can be *undone* and *re-undone*. We'll need a global list to store the history of actions performed. Basically, everytime a new action is executed, it'll be added to the global list as a command object.

-- IMAGE GOES HERE --

Therefore, the next commands are required to interact with the commands queue:

- **Press Cntrl+Z**: Undo the last action, if available.
- **Press Cntrl+Y**: Redo the next action, if available.

Note that the previous commands will not add new elements to the command queue.

## Implementation

...