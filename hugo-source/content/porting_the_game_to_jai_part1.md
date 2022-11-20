+++
title = "Porting 58000 lines of D and C++ to jai, Part 1: Preperations"
date = 2022-11-12T14:53:00+01:00
header_img = ""
toc = true
tags = ["programming", "game", "jai"]
categories = []
draft = true
+++

In this series of blogposts, I document my experience of porting [the game that I am currently working on](https://www.spaced-game.com) to the jai programming language. In part 0, the state of the codebase looked like this:

    -------------------------------------------------------------------------------
    Language                     files          blank        comment           code
    -------------------------------------------------------------------------------
    D                               27          10201           3104          32396
    C++                              4           2343            411           7807
    C/C++ Header                     1            533             75           1750
    -------------------------------------------------------------------------------
    SUM:                            32          13077           3590          41953
    -------------------------------------------------------------------------------

Before starting to write actual jai code, I wanted to do away with some arbitrary differences in the current codebase vs. the jai way of things, to make the actual porting process less error prone. Various concepts of jai have already found their way into my codebase, so overall the changes won't be too big, but details still differ.

## Methods vs. Functions

The containers in my game along with some other data structures have methods on them. In jai, there are only free functions. I converted all methods of `Dynamic_Array`, `Slice`, `String` and various special array types like a `Bounded_Dynamic_Array` with constant memory footprint to functions. This is the first time I REALLY missed an IDE (I'm working with Sublime Text 4), because even with macros this took hours. Eventually I stopped because this was the most tedious of the tasks while yielding the least benefit in porting: I would have to touch all the call sites anyway when calling, and would also have to reimplement any function or method, so I wasn't really gaining anything.

## Allocators

<!-- The allocators in my game are very similar to jai's allocators, but there are differences. -->
In jai, there are two types of allocators. There are normal allocators that encapsulate the usual `malloc`, `realloc` and `free` functionality and can be changed via the context, and then there's the special temp allocator, which is a stack-based allocator that is to be periodically reset by the programmer. This temp allocator offers the normal allocator API, but has extended functions to reset it, which are currently not designed to be swapped out for another implementation.
The temp allocator was one of the ideas that I adapted early on, because it removes a very large part of the tedium that comes with manual memory management. In my case, I used a single api for both types of allocators, that allows to `malloc`, `realloc`, `free`, `free_all`, `get_mark` and `free_to_mark`. You can query an allocator if it's temporary or not, and will receive an error if you call an unsupported operation (for example a get_mark for a non-temporary allocator). This allowed me to also implement     ??

- All my containers and some other structs have methods. In jai, there are only free functions.
- The layout of my dynamic-array and array-view (non-owning array) types are incompatible with jai's equivalents.
- My allocators expose a differnt API. They are a superset of jai's allocators.
- My string type remembers 

