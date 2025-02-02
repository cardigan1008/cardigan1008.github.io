---
layout: post
title: "How to report bugs (to GCC/LLVM)?"
date: 2025-01-31 20:27:00
categories: [notes]
tags: [research, testing]
---

Recently, I've been reporting bugs to GCC and LLVM, and I’d like to share some of my thoughts on the process. 

<!--more-->

I've been working on a compiler testing project since August 2024.
Since our fuzzer generates thousands of lines of code, understanding the root cause of a bug can be challenging. 
Thus, **reduction** is an essential step. 

Initially, my instructor recommended [CReduce](https://github.com/csmith-project/creduce), an impressive tool that not only minimizes C code but also supports other languages like Rust. 

CReduce requires users to write a shell script that returns 0 when the reduced code still exhibits the expected behavior. 
In our case, the expected behavior is either a crash or miscompilation. 

# Reduction of Crash Bugs

I started by writing a simple script for crash detection, and it worked perfectly:

```sh
#!/bin/sh

COMPILER=/path/to/gcc
CASE=/path/to/case

$COMPILER -O3 $CASE -o O3.out > output.txt 2>&1
./O3.out

grep -q 'internal compiler error' output.txt

exit $?
```

This script is designed for reducing crash cases when compiled with GCC. It works by checking whether the compiler’s output contains a specific marker indicating an **internal compiler error (ICE)**.

## Miscompilation: More Complex

Detecting miscompilation is more complex. Miscompilation occurs when the compiler produces different outputs at different optimization levels. But can we simply check if the outputs at two optimization levels differ significantly? 

The short answer is no—this approach has several pitfalls: 

- **Limited Comparison Scope**: If we only compare two optimization levels, even a simple `printf("%d\n");` statement could yield different results, making the check unreliable.

- **Existence of UB**: If the original test case contains UB, different outputs across optimization levels may occur naturally, even if the compiler is functioning correctly.

To tackle these issues, we need a more sophisticated reduction script, such as the following: 

```sh
#!/bin/bash -e

ulimit -t 10

CLANG=/path/to/clang
CASE=/path/to/case

clang -O0 $CASE -fsanitize=address,undefined -fno-sanitize-recover=all -o san.out >/dev/null 2>&1
if [ $? -ne 0 ];then
    exit 1
fi
./san.out >/dev/null 2>&1
if [ $? -ne 0 ];then
    exit 1
fi
clang -O0 $CASE -fsanitize=memory -o san.out >/dev/null 2>&1
if [ $? -ne 0 ];then
    exit 1
fi
./san.out >/dev/null 2>&1
if [ $? -ne 0 ];then
    exit 1
fi

$CLANG -O0 $CASE -o O0.out >/dev/null 2>&1
if [ $? -ne 0 ];then
    exit 1
fi
./O0.out >O0.txt 2>&1
if [ $? -ne 0 ];then
    exit 1
fi
$CLANG -O1 $CASE -o O1.out >/dev/null 2>&1
if [ $? -ne 0 ];then
    exit 1
fi
./O1.out >O1.txt 2>&1
if [ $? -ne 0 ];then
    exit 1
fi

$CLANG -O3 $CASE -o O3.out >/dev/null 2>&1
if [ $? -ne 0 ];then
    exit 1
fi
./O3.out >O3.txt 2>&1
if [ $? -ne 0 ];then
    exit 1
fi
diff O0.txt O1.txt && ! diff O0.txt O3.txt > /dev/null 2>&1
```

This script is designed for detecting miscompilation at the `-O3` level in Clang.
We add an additional opt level for comparsion and use sanitizer to avoid UB. 

### Perfect? No!

You might have noticed that I enabled MemorySanitizer (MSAN) in the script. However, there's a catch—MSAN also detects memory leaks, which are not a form of UB. 

To prevent false positives, leak detection should be turned off. This can be done by setting the `detect_leaks=0` option in the `ASAN_OPTIONS` environment variable. [^1]

[^1]: [AddressSanitizer LeakSanitizer Documentation](https://github.com/google/sanitizers/wiki/AddressSanitizerLeakSanitizer)


```sh
#!/bin/bash -e

ulimit -t 10

CLANG=/path/to/clang
CASE=/path/to/case

export ASAN_OPTIONS=detect_leaks=0 # NEW LINE!

clang -O0 $CASE -fsanitize=address,undefined -fno-sanitize-recover=all -o san.out >/dev/null 2>&1

# ...
```

This ensures that only actual UB-related issues are detected, without unnecessary noise from memory leak reports.

### One More Takeaway

If the original test case includes external libraries, your reduced case might end up looking like this:

```c
#include <csmith.h>
#include <stdio.h>
// more includes
```

A simple way to remove unnecessary dependencies is to expand the code first using the preprocessor: 

```sh
gcc/clang -E case.c > simple.c
```

Then, in your reduction script, replace `$CASE` with `simple.c`. Done!

## Correctly Report Bugs

Before writing a bug report, it's important to ensure that your case is valid and not a duplicate. Here are a few key checks to perform beforehand:

- Check Compiler Warnings: Compile it with `-Wall -Wextra` or `-fno-strict-aliasing -fwrapv` to see if there's anything different. 

- Search for Duplicate Reports: Use relevant keywords to search existing bug reports. For example, searching crash info for crash cases.  

## Gracefully Report Bugs

Reporting bugs isn’t just about dumping everything onto maintainers—it’s about making their job easier by providing clear, concise, and actionable reports. A well-structured bug report increases the chances of a quick resolution and helps maintainers efficiently diagnose the issue.

Here are some of my tips for reporting bugs effectively:

- A Simple, Informative Title: For crash, I use the format: `Crash at [opt-level]: [info]`. For example, one of my Clang crash reports was titled: \`Crash at -O2: Assertion \`isAvailableAtLoopEntry(LHS, L) && "LHS is not available at Loop Entry"' failed.\`
For miscompilation bugs, I follow the format:  `Miscompilation at [opt-level]`. 

- A Reproducible Test Case: I always provide a minimal reduced test case that triggers the bug. Also, I attach a [Compiler Explorer](https://godbolt.org/) link. 

- Bisection Information: If possible, I include the bisected commit ID to pinpoint when the issue was introduced. Otherwise, I provide a version range where the bug is observed. 

- Essential Diagnostic Info: For crash, crash info and backtrace are required. 

---

I’ve made some silly mistakes when reporting bugs in the past—sorry for taking up so much of the maintainers' time. Hopefully, this guide will help you report bugs more smoothly to the GCC/LLVM community or any other compiler community. 🙂
