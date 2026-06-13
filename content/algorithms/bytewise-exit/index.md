---
title: "exit(0)"
author: "Austin Chen"
date: "2023-03-19"
categories: [byte-wise,c,c++]
toc: true
---

![](https://media.istockphoto.com/id/685520492/photo/outdoors.jpg?s=612x612&w=0&k=20&c=ag3ZFqVpkx59ZRq0uB-vUFHEOBQwCYxlGNQtrWidppo=)

> Does `exit(0)` introduce memory leak?

## Motivation

The other day when I was reading someone else's code, I found the author uses a lot of `exit(EXIT_FAILURE)` which makes me wonder why I rarely use this and will it not cause memory leak since the function never returns to the callee instead it executes the `exit(EXIT_FAILURE)` whenever a error checking does not go through.

## Back to Basics

Before going any further, let's dive into the first-hand materials and see how they describe for the `exit()`.

> **Linux Programmer's Manual**
>
> The `exit()` function causes **normal process termination** and the least significant byte of status (i.e., status & 0xFF) is returned to the parent (see wait(2)). All functions registered with `atexit(3)` and `on_exit(3)` are called, in the reverse order of their registration. (It is possible  for one of these functions to use `atexit(3)` or `on_exit(3)` to register an additional function to be executed **during exit processing**; the new registration is added to the front of the list of functions that remain to be called.) If one of these functions does not return (e.g., it calls `_exit(2)`, or kills itself with a signal), then **none of the remaining functions is called**, and further exit processing (in particular, flushing of stdio(3) streams) is abandoned. If a function has been registered multiple times using `atexit(3)` or `on_exit(3)`, then it is called as many times as it was registered.

> **C23 §7.24.4.4 The exit function**
>
> The exit function causes **normal program termination** to occur. No functions registered by the [`at_quick_exit`](https://en.cppreference.com/w/cpp/utility/program/at_quick_exit) function are called. If a program calls the exit function more than once, or calls the `quick_exit` function in addition to the exit function, the behavior is undefined.

> First, all functions registered by the `atexit` function are called, in the reverse order of their registration, except that a function is called after any previously registered functions that had already been called at the time it was registered. If, during the call to any such function, a call to the `longjmp` function is made that would terminate the call to the registered function, the behavior is undefined.

> Next, all **open streams with unwritten buffered data are flushed**, all **open streams are closed**, and all files created by the **tmpfile function are removed**.

> Finally, control is returned to the host environment. If the value of status is zero or `EXIT_SUCCESS`, an implementation-defined form of the status successful termination is returned. If the value of status is `EXIT_FAILURE`, an implementation-defined form of the status unsuccessful termination is returned. Otherwise the status returned is implementation-defined.

In the above,

- `exit()` invokes the **normal process termination**, also suggesting the local automatic objects are not destroyed because the **stack is not unwound**.
- with `exit()`, all functions registered with `atexit()` are called.
- with `exit()`, all openning streams are flushed and closed, and files created by `tmpfile` are removed.

Also, when it comes to the **destructor in C++** [^1],

- with `exit()`, **non-local objects with static storage duration** are destroyed in the reverse order of the completion of their constructor.
- with `exit()`, for each **local object, `obj`, with static storage duration**, `obj` is destroyed as if a function calling the destructor of `obj` were registered with `std::atexit` at the completion of the constructor of `obj`.


## Memory Leak

Before jumping into any conclusion, let's refresh a bit on the definition of **memory leak**. To my way of thinking, essentially, there are two definitions of **memory leak** that are in common usage among programmers.

This **first** commonly seen definition of memory leak is,

> Memory was allocated and **was not subsequently freed** before the program is terminated.

However, many programmers argue that certain types of memory leaks that fits this definition don't actually pose any sort of problem, and therefore should not be considered **true memory leaks**.

A more useful definition of memory leak is,

> Memory was allocated and **cannot be subsequently freed** because the program lost track of any pointers to the allocated memory block.

> [!note] What Valgrind Defines as Memory Leak
> **Valgrind** uses the stricter definition of the term memory leak. This is the type of leak which can potentially cause significant heap depletion, especially for **long lived** processes. The **still reachable** category within Valgrind's leak report refers to allocations that fit only the first definition of memory leak. These blocks were not freed, but they could have been freed (if the programmer had wanted to) because the program still was keeping track of pointers to those memory blocks.[^2]

## Conclusion

The answer is quite obviously up to this point, the `exit()` invokes a termination of the current process, the OS collects all the memory allocated to that process, so strictly speakning, the `exit()` does not pose any memory leaks in this case.

> [!warning] Warning
> Just to add, in the vast majority of cases the OS will free the memory - as is the case with normal flavors of Windows, Linux, Solaris, etc. However it is important to note that in specialized environments such as various Real-Time Operating Systems the memory may not be freed when the program is terminated.

[^1]: [`std::exit`](https://en.cppreference.com/w/cpp/utility/program/exit)
[^2]: [Still Reachable Leak detected by Valgrind](https://stackoverflow.com/questions/3840582/still-reachable-leak-detected-by-valgrind)
