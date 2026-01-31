---
title: Understanding Binary Search - An interviewee's perpestive
date: 2026-01-30 17:00:00 +0800
categories: [Algorithms]
tags: [binary search, coding interview]
math: true
---

> "Although the basic idea of binary search is comparatively straightforward, the details can be surprisingly tricky."
> — **Donald E. Knuth**, *The Art of Computer Programming*

If you've ever prepared for a coding interview, you’ve definitely met Binary Search (also known as half-interval search, logarithmic search, or binary chop). On paper, it’s one of the simplest problems in Leetcode. We all know that we use 2 pointers point at the start and end of target array, and halve the array to search a target $O(\log n)$ times. However, many (including me) struggle to implement it correctly, especially under pressure during an interview. In my experience, the difficulty lies in two areas:
1. **Identification:** How do we recognize a problem as a Binary Search problem and convert it into one?
2. **Implementation:** How do we write bug-free code that handles edge cases like infinite loops?

Let's discuss them!

## What is Binary Search?

What would you say if the interviewer want you to introduce binary search algorithm in one or two sentences? In [Wikipedia](https://en.wikipedia.org/wiki/Binary_search), it says,
> ... is a search algorithm that finds the position of a target value within a sorted array. Binary search compares the target value to the middle element of the array. If they are not equal, the half in which the target cannot lie is eliminated and the search continues on the remaining half, again taking the middle element to compare to the target value, and repeating this until the target value is found. If the search ends with the remaining half being empty, the target is not in the array.

This indeed is what most people think in their mind. We probably starts from finding a target value in a sorted array when learning BS at school. But in an interview, BS is not only about arrays -- it is about **structure**. We can use BS to search an array, as well as search the service capacity, the distance, or any other structures that can be abstract into a sorted sequence. When I say sequence, I mean the searched structure can be infinite.

Actually, in an interview there's probably no array in the problem itself, the problem hide the sorted structure. You need to abstract and extract a vitual array in your mind. So back to the question, if we need to explain to the interviewer what is binary search, here is a golden rule worth to memorize,
```
Binary Search is used to search a monotonic space by halving it.
```

I am not forcefully fabricating new concepts. This one sentense can help us to clearly communicate with interviews and write bug-free code easily. Note the key words: monitonic, space. They will helps us a lot. Let's explain this with a fundamental BS problem, lower bound.

## Anchor Problem: Lower Bound

Let's take the most well-known BS problem, [lower bound](https://leetcode.com/problems/binary-search/description/), as an example.

### Problem

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

To solve this problem, we use two pointers, `left` and `right`, to maintain an interval to search (the search space). We compare target value with the value at the middle if interval, and remove half of the search space by moving left or right at each pace.
Usually there are 3 common patterns to code this solution,
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

Both the 3 approach works well and feel free to choose one as your preference. For me, I use to write pattern C as I feel it's more easy to write bug-free code with this pattern.

### Discussion

#### Sorted

We all know that lower bound can be resolved with BS. But let's forget this for a while and think how to answer the question:
- Is this problem suitable for Binary Search?

> There are many "interview tricks" for identifying algorithms. For example, people often say "If the required time complexity is $O(\log n)$ , then it must be Binary Search". It sometime works -- but it is neither reliable nor complete.
Instead of relying on hints, we need a deep understanding of binary search.

The reason that we can use BS is that the array is sorted.
- If `nums[mid] > target`, then all the element is the right side of mid is greater than target.
- If `nums[mid] < target`, then all the element is the left side of mid is less than target.
So we can safely halve the search space in each step. This is why we can shrink the space in every move.

#### Search Space

This is a very common source of confusion. When solving the lower bound problem, many people instinctively think that we are searching the index range of array, which is `[0, 1, 2, ..., n - 1]`. Actually this is incorrect,
1. if `target <= max(nums)`, the answer is an index in `[0, n - 1]`,
2. if `target > max(nums)`, the correct insertion position may be `n`.
So the answer space is all possible position where the target could be placed. Remember this, it's important in the question we are going to discuss next.

#### Predicate

In terms of indices, the problem becomes:
- Find the first index `i` that `nums[i] >= target` (if no such i, return `len(nums)`).

This immediatelt reveals the structure we need. We defini a predicate that,
```python
def check(i):
    return num[i] >= target
```

Now observe what happens as i increases:
```
# Input: nums = [-1,0,3,5,9,12], target = 9

index:               (-1)                 [ 0   1   2   3   4   5  ]                    (n)
nums[i]:  (assumed to negetive infinite)   -1   0   3   5   9  12             (assumed to infinite)
P(i):                 (F)                   F    F   F   F   T   T                      (T)
```

The predicate changes exactly once—from `False` to `True`. This is a monotonic sequence.

#### The invariant

```python
# Pattern C
def lower_bound(nums: List[int], target: int) -> int:
    left, right = -1, len(nums)
    while left + 1 < right:
        mid = (left + right) // 2
        if nums[mid] >= target:
            right = mid
        else:
            left = mid
    return right
```

We convert the problem from original description to a perpestive of indices.
Now we identify the search space, the next question is,
- What must always be true while we are shrinking this space?

In Pattern C, we maintain the following invariant throughout the search:
> At any moment:
>    - all indices at or left of `left` are guaranteed to be `False`
>    - all indices at or right of `right` are guaranteed to be `True`

```
P(i) = nums[i] >= target

Init state:
(-1) [ Unknown  Unknown Unknown Unknown Unknown] (True)
 ↑                                                  ↑
left                                              right

During the loop:
(False) [ False ... False | Unknown ... Unknown | True ... True ] (True)
                ↑                   ↑                   ↑
               left                mid                right

End of the loop:
(False) [ False ... False  False   True   True   True ... True ] (True)
                             ↑       ↑
                             left   right

```

At the beginning, we know nothing about index. But we know that we made an assumaption that,
- the predication holds `False` at index `-1`
- the predication holds `True` at index `len(nums)`

When we test `mid`, 
- if `P(mid) is True`, then `mid` belongs to the `True` region
- if `P(mid) is Fasle`, then `mid` belongs to the `False` region.

In both cases, the boundary moves inward and unknown region strictly shrinks.

Eventually, the unknown region disappears at `left + 1 == right`, and `right` is the final answer.

We are looking for the smallest index that keeps predication `True` (the most left element that is greater or equals to target), and we always keep the invariant, so `left` is never an answer candidate and `right` is always one. That means,
- We are searching in the range of `(left, right]`.

### Failure



### Summary

For now, we figure out all the concepts about binary search,
- Monotonic
- Search space
- Predication
- Invariant


|Pattern| Search Space | Predication | Invariant| End of the loop |
|:-|:-|:-|:-|
|A| `[left, right + 1]`   | `P(i) = nums[i] >= target`| `(not P(left - 1)) AND P(right + 1)` |`left - 1 == right` |
|B|  `[left, right]`      | `P(i) = nums[i] >= target`| `(not P(left - 1)) AND P(right)` | `left == right`|
|C| `(left, right]` | `P(i) = nums[i] >= target` | `(not P(left)) AND P(right)` | `left + 1 == right`|

### Extented use case

Lower bound is really the fundamental problem and template solution to binary search problems. Actually lower bound can be used to find more index other that insertion index.

|Goal | Usage | If target index not exists|
|:-|:-|:-|
|The first index that >= x | `lowerBound(nums, x)` | return `len(nums)` |
|The first index that > x | `lowerBound(nums, x + 1)` | return `len(nums)` |
|The first index that < x | `lowerBound(nums, x) - 1` | return `-1` |
|The first index that <= x | `lowerBound(nums, x + 1) - 1` | return `-1` |

## Dive deep Binary Search

To master Binary Search, we must move beyond memorizing solutions for specific problems. We need to identify patterns, understand **invariants**, use anchor problems to guide our logic, and learn how to communicate our thought clearly to interviews.

### Step 1 Pattern Contract

As I have said earlier, the pattern of BS is,
> search a monotonic space by halving it

What is monotonic?

Monotonic means that once a condition on the value `x` becomes `true` (or `false`), it never flip back again. That is, for a sequence, if we use a condition to validate its elements one by one, it looks like,
```
[False, False, False, True, True, True, ...]
[True, True, True, False, False, False, ...]
```

This is the key propertity of searched sequence.


What is search space?

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

