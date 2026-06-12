---
title: "Pragmas"
author: "Austin Chen"
date: "2023-03-16"
categories: [byte-wise]
toc: true
# draft: true
---

![](thumbnail.jpg)

> `#pragma` is more than just a header guard.

`#pragma` is a directive specifying extra information to the compiler. With it you can print debug messages, change compilation warnings and optimizations and much more.

You might be familiar with `pragma once`, which lets the compiler know to only read the file once. It is usually found on top of header files. This is so called "un-official" replacement for include guard for preventing the same header file to be included multiple times.

```c
// header file
#ifndef HEAD_FILE_H_
#deine HEAD_FILE_H_
// contents of header file

#endif // HEAD_FILE_H_
```

The conventional method requires that the programmer does provide a unique macro for each header file. Pragma on the other hand is trivial.

Include errors, resulting in some form of re-definition, is caused by the header being included by other header files, which are then in turn included in a source file - thus multiple inclusions of the same header file.

`#pragma` in itself is part of the C and C++ standard, but `#pragma once` is part of the compiler implementation, therefore it is not as universal as `#ifndef` include guards, which are based on the preprocessor standard. Still it is widely adopted in most modern compilers.

The syntax for a pragma might look something like this:
```c
#pragma PREFIX option value(s)
```

Pragma can source its rules and options from various packages. Those defined by the standard use a prefix `STDC`, while those used by GNU use `GCC` prefix.

## Option stacking

It is important to note, that a pragma directive applies for the whole file from file from that point. Everything before it is unaffected. This is why, when applying certain options just for a small section, it is useful to save current options and restore them back afte the said section.

This is why you should use `push` and `pop` options before and after applying a progma. Push and pop can be used as a value to an option or as part of the option as well. Here are a few examples

```c
#pragma GCC diagnostic push
#pragma GCC diagnostic error "-Wformat" // treat this warning as error

// section affected by
void foo()
{
    ...
}

#pragma GCC diagnostic pop
```

```c
#define MACRO 1
#pragma GCC push_macro("MACRO") // save MACRO value to stack
#undef MACRO // undefine macro MACRO

// section affected by
void foo()
{
    ...
}

#pragma GCC pop_macro("MACRO") // pop value of macro MACRO back
```

## Code optimization

For most scenarios, I stand by a rule to use the same optimization options for both `Debug` and `Release` builds. This prevents unnessary errors and code breakage when switching builds types, where optimizer might do a too-good job and it ends up breaking the executaable. Remember, build type is not just about optimization level.

For my builds, I use option `Og` for `gcc` compiler. This is, by my estimations, the est compromise between optimized code and debug-ability. However it still isn't perfect. While it does prevent inlining of regular functions, some lambdas and trivial class methods do get inlined. Sometimes whole `if-else` blocks get optimized to a point, that you cannot put a breakpoint anywhere useful. Even if you do, some variables might get optimized and debugger will not be able to give you any information on their value.

```c
#pragma GCC push_options // save current compiler options
#pragma GCC optimize ("-O0") // set optimization level to zero

void foo(int a)
{
    int cpy = a;
    return cpy + 5;
}

#pragma GCC pop_options // restore saved compiler options
```

In the example above, such trivial function might get optimized away. If not, stepping through it might prove useless, as variable `cpy` might get optimized so that debugger cannot see its value. Setting optimization level to zero disables any funny business from the compiler and forces it to generate expected instructions.


