---
layout: post
title: "Paper reading: LLM4Testing"
date: 2025-04-18 19:56:00
categories: [notes]
tags: [research]
---

Notes on paper related to LLM4Testing. 

<!--more-->

# [1. Towards Understanding the Effectiveness of Large Language Models on Directed Test Input Generation](https://dl.acm.org/doi/pdf/10.1145/3691620.3695513)

## Problems with constraint-based fuzzing:

- path explosion: every conditional statement doubles the number of paths to explore

- external systems or channels: input for any valid Linux command is hard to be modeled (e.g. ls)

## Existing ways to utilize LLMs

Feed contexts in two ways:

- the context in natural language: documentation and error messages 

- the context in code: unreachable branches or executable paths

## Key idea of this paper

> It's similar with the key idea behind LegoFuzz, as LegoFuzz also tries to combine stregths from both LLMs and traditional methods.  

**Hybrid testing with LLMs: incorporate LLMs and traditional tools.**

Interesting observations here: 

1. For tasks that require specific background knowledge, traditional methods cannot construct a general model. But LLMs can. 

2. For tasks that require precise operations (e.g. caculations), LLMs cannot provide precise result, which is easy for traditional tools to deal with. 

For me, the difference lies in whether or not the task requires things that cannot be modeled. 
This paper provides an interesting example: command in string. It cannot be generalized in any way. That's when LLMs comes into help.
It's similar to the Creal's extractor problem (i.e. integer and struct). 


## Benchmark construction: from real-world database

> It is similar to Creal and LegoFuzz :)

Simply mask off the input from HumanEval and HumanEval-X.

## Evaluation and Findings

RQ2 is the most interesting one. This paper found a way to compare LLMs and traditional tools by setting them under different categories. 
But the number of examples is small, which makes it questionable on the findings in some categories such as integer_overflow and symbolic_jump.


### Takeaways from experiment design

- Table 5: It classifies input types into Str, Float, Aggr, No Aggr.

- Figure 9: It divides the pass rate to 0-1/3, 1/3-2/3, 2/3-1 for each code group. It's clear when we have multiple language settings. 

### Interesting findings

- Finding 5: LLMs can more effectively address challenges that involve implicit data flow (e.g., interactive with operation system) and out-of-code constraints (e.g., a valid IP address). However, their performance is less satisfactory when requiring precise solutions (e.g., an exact N-length string).

- Finding 6 (this one is trivial but somewhat intererting): LLMs and KLEE exhibit stark contrasts in handling aggregate inputs and outputs, with LLMs performing worse on the summation or selection elements from aggregate inputs while KLEE performing worse on aggregate type to aggregate type transformation such as array sorting.

- Finding 8: GPT-3.5 have better generalizability as it can often provide some valid inputs for a program, but its ability to generate comprehensive test cases is not as strong as KLEE. In contrast, KLEE is more code-sensitive. It can solve all paths on half of the code snippets in our dataset. However, there are also 28.13% samples that do not find any solution at all.

- Finding 9 (STRONGLY AGREE): It is feasible and promising to integrate LLMs with constraint-based tools and the evaluation results show that our novel solution can yield significant improvements. We believe this is a promising direction worthy of further exploration. 

