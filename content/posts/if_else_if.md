---
title: "'else if' is not real?"
date: 2025-09-04
draft: false
ShowToc: true
author: Herocharge
---

It's always interesting to see how different languages implement their `if` statements.
Python has `if, elif, else`, Ruby has `if, elsif, else`, and the most common ones are `if, else if, else` in languages like C++, Java, and Go.

But is `else if` actually a thing?

Let's take the example of C.

In C, we can have if statements like this:
```c
if(<condition>){
    // statements;
}
else {
    //statements;
}
```


We can exclude the curly braces if there is only one statement inside.
Something like:

```c
if(<condition>)
    // statement;
else
    // statement;
```

Well, the whole `if else` block is considered a statement and what if we have that as the else block?

```c
if(<condition>)
    // statement;
else
    if(<condition>) {
        // statements;
    }
    else {
        // statements;
    }
```

Wait a minute, C doesn't really care about the indentations very strictly. So we can rewrite it to:

```c
if(<condition>)
    // statement;
else if(<condition>) {
    // statements;
}
else {
    // statements;
}
```


And we get our `else if` block. So one doesn't really need to implement a special `else if` syntax if the language already allows for single statements without curly braces and has leniant indentation rules. All you need is `if else`.

Which brings us to Go, it has `else if` but it doesn't allow `else` statements without curly braces.

I don't know if any of my observations reflect what's actually going on under the hood but it's fun to think about. 

Cheers :)