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

Initially, my instructor encouraged me to explore code reduction on my own. He recommended `CReduce`, an impressive tool that not only minimizes C code but also supports other languages like Rust. 

CReduce requires users to write a shell script that returns 0 when the reduced code still exhibits the expected behavior. 
In our case, the expected behavior is either a crash or miscompilation. I started by writing a simple script for crash detection, and it worked perfectly:

```sh
COMPILER=/path/to/compiler
CASE=/path/to/case
```
