---
layout: post
title: "Javascript Garbage Collection"
meta: "A brief introduction to JavaScript garbage collection. All you need to know to have a better understanding of what's happening under the covers."
categories: Node JavaScript
slug: "javascript-garbage-collection"
---
# Introduction
I was recently interviewed by telephone for a contract position. It went very badly, not least because I was placed on speakerphone and could hear myself being broadcast to the room at the other end of the line with a slight delay - a terrible situation which, for me at least, makes it impossible to concentrate and string a sentence together.

What made things even worse was that this was a technical interview, and I was already on the back foot due to the echo. What floored me completely was a question about JavaScript garbage collection. If I had my wits about me I would have replied to the effect that I don't have the details of this process at the front of my mind as 99.9% of the time it's not something that one needs to consider in any great detail (that being, after all, the point of garbage collection). In the event of a memory leak in one of my Node.js applications I would refresh myself by reviewing garbage collection on the internet and take things from there. Unfortunately, I didn't have the presence of mind to think of that and completely screwed the up interview.

With this in mind, I decided that my next post would be a refresher on JavaScript garbage collection, so here we go. 

# Do you remember when you had to manage your own memory?
My first job as a professional programmer involved writing simulations of gas flow and pressure in pipeline networks. For better or worse I decided to create these simulations in C. How we used to laugh after discovering that the reason our simple 5-line routine caused a catastrophic failure was because we accidentally wrote a value into the 11th element of an array with only 10 available spaces! Forgive me if this is obvious to you, but there are plenty of coders out there who've never used `alloc` or `malloc` in their lives. If you don't know what I'm talking about get yourself a copy of [Kernighan & Ritchie][1]{:target="_blank"} and marvel at the steam-powered code cavemen used to write. 

# So what is Garbage Collection?
Having to manage memory yourself in languages like C isn't necessarily better or worse than languages like JavaScript with managed memory. It may require more planning and computation cost to allocate memory yourself but managed memory has its downsides, not least that it can easily run out. This is why managed memory requires garbage collection. Generally speaking it is cheap (in terms of computation and, therefore, time) to allcate memory and relatively expensive to reclaim it when the memory pool is exhausted.

JavaScript has the simple types of numbers, strings, booleans, `null` and `undefined`. All other values are objects. Simple, or `primitive`, types are manipulated by value because they consist of nothing more than a small, fixed number of bytes that can be easily managed by the JavaScript interpreter. Objects, however, are reference types. Objects can contain an arbitrary number of elements or properties which means that they can become very large. As a consequence it doesn't make sense to manipulate objects by value as this could involve copying and comparing large amounts of memory. Thus, object variables do not contain their values directly but rather hold a reference to the actual value (what C programmers would know as a `pointer`).

JavaScript, [as any fule kno][2]{:target="_blank"}, uses garbage collection to reclaim the memory occupied by primitive types and objects that are no longer in use. This means that the programmer is free from having to explicitly allocate and deallocate memory the way that languages like C require. This raises the question of how the garbage collector knows when it is safe to reclaim memory. 

# Where do we put the garbage to be collected?
In languages like C variables are stored in two places: the stack and the heap (as they are in most other programming languages). The stack is a continuous region of memory used to allocate a local context for each executing function. The stack grows and shrinks fast as functions push and pop local variables to and from it and is usually created at compile time. Variables are allocated and freed here automatically. The heap, on the other hand, is a much larger region of memory in which storage is allocated dynamically. Unlike the stack, the heap requires memory management. JavaScript, on the other hand, doesn't allow for user memory management and doesn't separate memory into stack/heap. In JavaScript there's just the heap. There are many different spaces in the heap which are used depending upon the type, size and age of objects or variables. Essentially, however, it has two main segments: the **New Space** and the **Old Space**.

The reason for creating a New Space and an Old Space is that objects tend to die young. As the names suggest, New Space is where new allocations happen. It is relatively small and things change quickly in New Space. It's easy to allocate space in New Space - we simply have an allocation pointer which is incremented whenever we reserve space for a new object. New Space is small, and when we reach the end of it a *scavenge* (minor garbage collection cycle) is triggered. The scavenge either removes dead objects or promotes marked objects to Old Space. Old Space is where objects in the New Space that survived two garbage collections are promoted to. They are know as the **Old Generation**. Allocation in Old Space is fast but garbage collection is expensive so it isn't performed here very often. 

# How does the Garbage Collector know what to collect?
The garbage collector must never reclaim variables or objects that are still in use and should only reclaim values that cannot be reached from a root node (global objects, DOM elements or local variables). In order to do this the garbage collector uses an algorithm known as `mark & sweep`. In the marking phase, all live objects on the heap are discovered and marked. This is done by recursively traversing the list of all variables and marking any values referenced by those variables.

There are 3 marking states: 
 - White: an object which has not yet been discovered by the garbage collector.
 - Grey: An object which has been discovered by the garbage collector but not all of its neighbours have been processed yet.
 - Black: An object which has been discovered and all of its neighbours have been fully processed.

When the marking algorithm is complete all live objects are marked black and all dead objects are left white. Memory can now be released by either sweeping or compacting. The sweeping algorithm adds the space used by unmarked (white) objects to a list of available memory (known as free space). The compacting algorithm moves fragmented objects into contiguous areas on free space. This free space, like the heap itself, is broken into small chunks called pages to make processing more efficient. The reason garbage collection is expensive on the Old Space is that the JavaScript engine has to "stop the world" in order for the process to run properly. This means that program execution stops while garbage collection is in progress, otherwise the garbage collector would be in a state of flux and unable to definitively determine which variables need to be marked and which do not. 

# Where do I go from here?
This description has necessarily been highly simplified but it covers the main points you would need to know on a daily basis. Here are a few rules of thumb to bear in mind when coding so as to write memory-friendly code: 
 - One of the reasons why JavaScripts global scope is considered so dangerous is that all global variables are considered to be live all the time. Unless you explicitly set a global variable to 'null' once it's been used it will never be garbage collected and will degrade application performance.
 - This is another good reason for developing in `strict` mode. Amongst other things, `strict` mode highlights where local variables have been declared without using the `var` keyword which results in them being considered global variables (see the previous point).
 - Beware objects that reference each other. If A references B and B references A then they will never be garbage collected even if no other objects or variables reference A or B. Note that JavaScript ES6 introduced what are know as weak references (references that don't stop an object being garbage collected) in [WeakMap][3]{:target="_blank"} and [WeakSet][4]{:target="_blank"} in recognition of this issue.
 - The easiest way to minimise garbage collection is to avoid creating objects in the first place. Note that this includes functions since functions are objects in JavaScript.
 - If you can't avoid creating objects then try re-using them. This is know as object pooling and it is the process of creating a pool (usually an array) of objects once, calling a factory method to get an object from the pool when it is needed and then returning it to the pool once it has finished being used.
 - If you're using event listeners make sure you unbind them when they are no longer required.

It's worth understanding garbage collection in principle and using good practice such as the tips described above as part of your everyday routine but don't worry too much about optimising code for memory management unless or until it actually becomes a problem. Chances are you'll be spending an awful lot of effort to achieve minimal gain because you won't know what to optimise until you look at what is causing the problem.

In short, read this and then forget about it until you have a performance or memory problem. In that case, give this post a quick glance, refresh your memory, and go fix the problem!

 [1]: https://en.wikipedia.org/wiki/The_C_Programming_Language
 [2]: https://en.wikipedia.org/wiki/Nigel_Molesworth
 [3]: https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/WeakMap
 [4]: https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/WeakSet