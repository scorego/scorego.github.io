---
title: Understanding Binary Search - An interviewee's perpestive
date: 2026-01-30 17:00:00 +0800
categories: [Algorithms]
tags: [binary search, coding interview]
math: true
---

> "Although the basic idea of binary search is comparatively straightforward, the details can be surprisingly tricky."
> — **Donald E. Knuth**, *The Art of Computer Programming*

If you've ever prepped for a coding interview, you’ve definitely met Binary Search (also known as half-interval search, logarithmic search, or binary chop). On paper, it’s one of the simplest things in the world. We all know the "magic" time complexity is $O(\log n)$, and we all think we can code it in our sleep. However, many (including me) struggle to implement it correctly under pressure, especially during an interview. In my experience, the difficulty lies in two areas:
1. **Identification:** How do we recognize a problem as a Binary Search problem and convert it into one?
2. **Implementation:** How do we write bug-free code that handles edge cases like infinite loops?

Let's conquer it in this post!

## What is Binary Search?

Imagine this, what would you say if the interviewer want you to summarize the binary search algorithm in one sentense? In [wikipedia](https://en.wikipedia.org/wiki/Binary_search), it says,
> ... is a search algorithm that finds the position of a target value within a sorted array. Binary search compares the target value to the middle element of the array. If they are not equal, the half in which the target cannot lie is eliminated and the search continues on the remaining half, again taking the middle element to compare to the target value, and repeating this until the target value is found. If the search ends with the remaining half being empty, the target is not in the array.

This indeed is what most people think in their mind. We probably starts from find a target value in a sorted array when learning BS algorithm in school. But in an interview, BS is not only about arrays -- it is about **structure**. We can use BS in array, as well as in,
- Time.
- Distance
- Capacity
- any other structures that can be abstract into a sorted array

Actually, in an interview there's probably no array in the problem itself, the problem hide the sorted structure. You need to abstract and extract a vitual array in your mind. So back to the question, if we need to explain to the interviewer what is binary search, this is what I think,
- Binary Search is used to search a monotonic space by halving it.

Pay special attention to the key words: monitonic, space. Let's dive deep one by one.

## Framework we use to understand Binary Search

To master Binary Search, we must move beyond memorizing solutions for specific problems. We need to identify patterns, understand **invariants**, use anchor problems to guide our logic, and learn how to communicate our thought clearly to interviews.

### Step 1 Pattern Contract

First we will identify the common pattern among the coding problems that can be resolved with BS algorithm. And once we decide to use BS, what key information should we abstract and exact from the problem.

### Step 2 The invariant

After deciding to use BS and get all the information, what is the invariant that our code need to follow. Once we figure out it, it's close to write a bug-free code.

### Step 3 The code template

There are more than one approach for binary search. We discuss the invariant and difference between them, and finally pick up a favorite one.

### Step 4 The anchor problems

We use example problems to understand the concept above, and mock the process how we think, how we communication with interviewers, and how to code correctly.

### Step 5 Failure‑Mode Notes

What is the common fault we made for a BS problem?



## Fill the framework

Before writing any code, we need to answer a more fundamental question:
> Is this problem suitable for Binary Search?

There are many "interview tricks" for identifying algorithms. For example, people often say "If the required time complexity is $O(\log n)$ , then it must be Binary Search". It sometime works -- but it is neither reliable nor complete.
Instead of relying on hints, we need a deep understanding of binary search.
Let's go back to the essence of binary search.

In this chapter, we use the most well-known bs problem, which is lower bound, for better understanding.
> Lower bound: Given a sorted array, find the first index of target value. If not found, return the inserted index.

### The Binary Search Contract


Let'd understand the defination. BS applies when the problem can be abstracted into the following contract:
1. There exists a search space
  - The search space can be an array, an index range, a number range, or even the space of possible answers.
2. There exists a predicate (a yes/no question)
  - For any candidate value `x` in the search space, we can answer a question like: `Is x valid?`
3. The predicate is monotonic.
  - once the predicate becomes `true` / `false`, it NEVER flips back to `false` / `true`.

If any one of these conditions is missing, BS is not the right tool. And once we recognize all the conditions, we are close to write a bug-free code.

### 


