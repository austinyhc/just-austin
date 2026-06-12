---
title: "Declaring a String"
author: "Austin Chen"
date: "2022-11-18"
categories: [byte-wise,c,string]
toc: true
---

![](https://media.istockphoto.com/id/953073756/video/moving-stock-market-ticker.jpg?s=640x640&k=20&c=7UxEFeJXdl7dhPuXqxAWUtqXpmqaQh5nqTds7PZxpBs=)

> Demystify `char*` vs `char[]`

## Motivation

Let me start with some codes. This is a classic leetcode problem, [20. Valid Parentheses](https://leetcode.com/problems/valid-parentheses/). The strategy is simple. It reuses the same chunk of memory of input, `s`, as the stack for checking the closing parentheses. However, this is not what we are focusing on. Take some time and try to spot the problem.

> 💡 **Hint**
> 1. The execution complains **`SEGFAULT`**.
> 2. `GDB` tells the error happens at **line 7**.

```c
bool isValid(char *s) {

    char *reuse_ptr = s;

    for (char *iter=s; *iter; iter++) {
        switch (*iter) {
            case '(': *reuse_ptr = ')'; reuse_ptr++; continue;
            case '[': *reuse_ptr = ']'; reuse_ptr++; continue;
            case '{': *reuse_ptr = '}'; reuse_ptr++; continue;
            default: {
                if (0 == reuse_ptr-s) return false;

                char expected = *--reuse_ptr;
                if (*iter != expected) return false;
            }
        }
    }
    return 0 == reuse_ptr-s;
}

int main(void)
{
    char* test_case = "(){[]}";
    assert(isValid(test_case));

    return 0;
}
```

It turns out the **`SEGFAULT`** goes away when I changed the **pointer notation** to the **array subscripting notation** as shown below. Let's study further and close the gap in my knowledge of how these two method differ from the other.

```diff
int main(void)
{
--  char* test_case = "(){[]}";
++  char test_case[] = "(){[]}";
    assert(isValid(test_case));

    return 0;
}
```

## Back to Basics

According to the C11 Standard, **modifying** the content of `char* test_case` is an **undefined behavior.**

> **C11 § 6.7.9 32.) EXAMPLE 8 The declaration**
> ```c
> char s[] = "abc", t[3] = "abc";
> ```
>  defines "plain" `char` array objects `s` and `t` whose elements are initialized with character string literals. This declaration is identical to
> ```c
> char s[] = { 'a', 'b', 'c', '\0' }, t[] = { 'a', 'b', 'c' };
> ```
> The contents of the arrays are **modifiable**. On the other hand, the declaration
> ```c
> char *p = "abc";
> ```
> defines `p` with type "pointer to `char`" and initializes it to point to an object with type "array of `char`" with length 4 whose elements are initialized with a character string literal. If an attempt is made to use `p` to modify the contents of the array, **the behavior is undefined**.

With that being said, the C11 Standard only states it as an **undefined behavior**, so it is still an unresolved question why a **segmentation fault** is complained. Let's move on to the definition of the **segmentation fault** and see if we can find some evidence from that.

## `SEGFAULT`

`SEGFAULT` or **segmentation fault** is defined as[^1]

> In computing, a **segmentation fault** (often shortened to **segfault**) or **access violation** is a fault, or failure condition, **raised by hardware with memory protection**, **notifying an operating system (OS)** the software has **attempted to access a restricted area of memory (a memory access violation)**. On standard x86 computers, this is a form of general protection fault. The operating system kernel will, in response, usually perform some corrective action, generally passing the fault on to the offending process by sending the process a signal. Processes can in some cases install a custom signal handler, allowing them to recover on their own, but otherwise the OS default signal handler is used, generally causing abnormal termination of the process (a program crash), and sometimes a core dump.
>
> [TL;DR]
>
> #### Writing to read-only memory
>
> Writing to **read-only memory** raises a segmentation fault. At the level of code errors, this occurs when the program writes to part of its own code segment or the read-only portion of the data segment, as these are loaded by the OS into read-only memory.

From the description above, we know the `SEGFAULT` is a behavior conducted by an **operating system** when a program tries to access a restricted area of memory.

> 📝 **Make use of the SEGFAULT**
> We can make use of this feature to store data that we don't want to be mutated in a **read-only** segment. When someone tries to modify the content, it raises `SEGFAULT`.

Up to this point, we can safely suggest that and nearly conclude that `char* test_case` (pointer notation) place the `test_case` in a **read-only** area.

## Conclusion

At last, let's conclude by justifying where exactly in the memory are the two following variables located.

```c
char test_case_0[] = "()"; // pointer notation
char *test_case_00 = "()"; // array subscripting notation
```

```sh
gef➤  p &*test_case_0
$0 = 0x7fffffffda12 "()"

gef➤  p &*test_case_00
$1 = 0x555555556008 "()"
```

Use `vmmap` command to observe the **memory layout** and their **permission**.

```sh
gef➤  vmmap
Start              End                Offset             Perm Path
0x00555555556000 0x00555555557000 0x00000000002000 r-- [executable binary]
[...]
0x007ffffffdd000 0x007ffffffff000 0x00000000000000 rw- [stack]
```

From the above, we conclude

- the **pointer notation** put a variable statically in the `.rodata` section
- the **array subscripting notation** makes a variable being operated in the stack

[^1]: Wikipedia, "Segmentation fault", https://en.wikipedia.org/wiki/Segmentation_fault
