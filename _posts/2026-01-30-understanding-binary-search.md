---
title: Understanding Binary Search - An interviewee's perpestive
date: 2026-01-30 17:00:00 +0800
categories: [Algorithms]
tags: [binary search, coding interview]
math: true
---

> "Although the basic idea of binary search is comparatively straightforward, the details can be surprisingly tricky."
> — **Donald E. Knuth**, *The Art of Computer Programming*

If you've ever prepared for a coding interview, you’ve definitely met Binary Search (also known as half-interval search, logarithmic search, or binary chop). On paper, it’s one of the simplest things in the world. We all know the "magic" time complexity is $O(\log n)$, and we all think we can code it in our sleep. However, many (including me) struggle to implement it correctly, especially under pressure during an interview. In my experience, the difficulty lies in two areas:
1. **Identification:** How do we recognize a problem as a Binary Search problem and convert it into one?
2. **Implementation:** How do we write bug-free code that handles edge cases like infinite loops?

Let's conquer it in this post!

## What is Binary Search?

What would you say if the interviewer want you to introduce the binary search algorithm in one sentense? In [wikipedia](https://en.wikipedia.org/wiki/Binary_search), it says,
> ... is a search algorithm that finds the position of a target value within a sorted array. Binary search compares the target value to the middle element of the array. If they are not equal, the half in which the target cannot lie is eliminated and the search continues on the remaining half, again taking the middle element to compare to the target value, and repeating this until the target value is found. If the search ends with the remaining half being empty, the target is not in the array.

This indeed is what most people think in their mind. We probably starts from find a target value in a sorted array when learning BS algorithm at school. But in an interview, BS is not only about arrays -- it is about **structure**. We can use BS in array, as well as in,
- Time
- Distance
- Capacity
- any other structures that can be abstract into a sorted array

Actually, in an interview there's probably no array in the problem itself, the problem hide the sorted structure. You need to abstract and extract a vitual array in your mind. So back to the question, if we need to explain to the interviewer what is binary search, this is what I think,
```
Binary Search is used to search a monotonic space by halving it.
```

I am not forcefully fabricating new concepts. The one sentense can help us to clearly communicate with interviews and write bug-free easily. Pay special attention to the key words: monitonic, space. Let's explain this with a fundamental BS problem, lower bound.

## Anchor Problem: Lower Bound

Before writing any code, we need to answer a more fundamental question:
> Is this problem suitable for Binary Search?

There are many "interview tricks" for identifying algorithms. For example, people often say "If the required time complexity is $O(\log n)$ , then it must be Binary Search". It sometime works -- but it is neither reliable nor complete.
Instead of relying on hints, we need a deep understanding of binary search. Let's take the most well-known BS problem, [lower bound](https://leetcode.com/problems/binary-search/description/), as an example.

> Lower bound: Given a sorted array, find the first index of target value. If not found, return the inserted index.
>
> Example 1:
>
> Input: nums = [-1,0,3,5,9,12], target = 9
> Output: 4
> Explanation: 9 exists in nums and its index is 4
>
> Example 2:
>
> Input: nums = [-1,0,4,5,9,12], target = 3
> Output: 2
> Explanation: 3 does not exist in nums, and the array keeps order if insert it at index 2: [-1,0,3,4,5,9,12]


To solve this problem, we use two pointers, `left` and `right`, to shape our search space. We compare target with the value at middle index, and halve the search space by moving left or right at each pace.
 There're 3 approaches,

```python
def lower_bound(nums: List[int], target: int) -> int:
    '''
    Pattern A
    '''
    left, right = 0, len(nums) - 1
    while left <= right:
        mid = (left + right) // 2
        if nums[mid] >= target:
            right = mid - 1
        else:
            left = mid + 1
    return left

def lower_bound(nums: List[int], target: int) -> int:
    '''
    Pattern B
    '''
    left, right = 0, len(nums)
    while left < right:
        mid = (left + right) // 2
        if nums[mid] >= target:
            right = mid
        else:
            left = mid + 1
    return left

def lower_bound(nums: List[int], target: int) -> int:
    '''
    Pattern C: Recommended
    '''
    left, right = -1, len(nums)
    while left + 1 < right:
        mid = (left + right) // 2
        if nums[mid] >= target:
            right = mid
        else:
            left = mid
    return right
```

Lower bound is really the fundamental problem and template solution to binary search problems. We can use this functionality to almost any BS problems. Here is a cheat sheet,

|Goal | Usage | If target index not exists|
|:-|:-|:-|
|The first index that >= x | `lowerBound(nums, x)` | return `len(nums)` |
|The first index that > x | `lowerBound(nums, x + 1)` | return `len(nums)` |
|The first index that < x | `lowerBound(nums, x) - 1` | return `-1` |
|The first index that <= x | `lowerBound(nums, x + 1) - 1` | return `-1` |

## Framework we use to understand Binary Search

To master Binary Search, we must move beyond memorizing solutions for specific problems. We need to identify patterns, understand **invariants**, use anchor problems to guide our logic, and learn how to communicate our thought clearly to interviews.

### Step 1 Pattern Contract

As I have said earlier, the pattern of BS is,
> search a monotonic space by halving it

For lower bound,

### Step 2 The invariant

After deciding to use BS and get all the information, what is the invariant that our code need to follow. Once we figure out it, it's close to write a bug-free code.

### Step 3 The code template

There are more than one approach for binary search. We discuss the invariant and difference between them, and finally pick up a favorite one.

### Step 4 The anchor problems

We use example problems to understand the concept above, and mock the process how we think, how we communication with interviewers, and how to code correctly.

### Step 5 Failure‑Mode Notes

What is the common fault we made for a BS problem?

### The Binary Search Contract


Let'd understand the defination. "Binary Search is used to search a monotonic space by halving it."

What is monotonic?
Monotonic means 




BS applies when the problem can be abstracted into the following contract:
1. There exists a search space
  - The search space can be an array, an index range, a number range, or even the space of possible answers.
2. There exists a predicate (a yes/no question)
  - For any candidate value `x` in the search space, we can answer a question like: `Is x valid?`
3. The predicate is monotonic.
  - once the predicate becomes `true` / `false`, it NEVER flips back to `false` / `true`.

If any one of these conditions is missing, BS is not the right tool. And once we recognize all the conditions, we are close to write a bug-free code.

## Appendix

### Prove of Time complexity (Optional)

The [Master Theorem](https://en.wikipedia.org/wiki/Master_theorem_(analysis_of_algorithms)) provides a direct way to determine the asymptotic runtime of divide-and-conquer algorithms.

For binary search, the upper bound of running time `T(n)` for an input size `n` is
```
T(n) = T(n/2) + O(1)
     = T(n/2) + O(n^0)
```

So T(n) is O(logn).

