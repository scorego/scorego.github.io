---
title: Binary Search Pattern in Coding Interviews
date: 2026-01-30 17:00:00 +0800
categories: [Algorithms]
tags: [binary search, coding interview]
math: true
layout: post
---

> "Although the basic idea of binary search is comparatively straightforward, the details can be surprisingly tricky."
> — **Donald E. Knuth**, *The Art of Computer Programming*

If you've ever prepared for a coding interview, you’ve definitely encountered Binary Search (also known as half-interval search, logarithmic search, or binary chop). We all know the classic version: use two pointers at the start and end of a sorted array, repeatedly halve the range, and find the target in $O(\log n)$ time. However, many problems that use binary search don’t look like classic “search in array” questions at all. What interviewers actually want to test is whether you understand binary search as a way to find the **boundary** of **a monotonic condition**.
**Boundary** and **monotonic condition** are the two keys to understanding the binary search pattern and writing bug-free code. Let's break them down!

## What is Binary Search?

What would you say if the interviewer asked you to explain the binary search algorithm in one or two sentences? [Wikipedia](https://en.wikipedia.org/wiki/Binary_search) explains it as follows:
> ... is a search algorithm that finds the position of a target value within a sorted array. Binary search compares the target value to the middle element of the array. If they are not equal, the half in which the target cannot lie is eliminated and the search continues on the remaining half, again taking the middle element to compare to the target value, and repeating this until the target value is found. If the search ends with the remaining half being empty, the target is not in the array.

This is indeed how most people understand binary search. Most of us first learn binary search as "finding a target value in a sorted array". However binary search is not just about arrays or **indices** -- it is about **structure**. The search space could be:
- indices in an array
- possible answers (speed, capacity, time, size)
- a conceptual range that never explicitly appears in the input

Here is a golden rule:
> Binary search finds the boundary of a monotonic predicate by repeatedly halving the search space.

This single sentence helps us to communicate clearly with interviewers and write bug-free code more reliably. Understand that binary serach is for boundaries, not for targets. Pay attention to the key words: **boundary**, **monotonic predicate** and **search space**.

## Binary Search Template (Lower Bound)

Most interview problems use the lower-bound variant of binary search.

### Problem

> Lower bound (LeetCode 35): Given a sorted array of distinct integers and a target value, return the index if the target is found. If not, return the index where it would be if it were inserted in order.
> You must write an algorithm with `O(log n)` runtime complexity.
>
> Example 1:
> Input: nums = [1,3,5,6], target = 5
> Output: 2
>
> Example 2:
> Input: nums = [1,3,5,6], target = 2
> Output: 1

To solve this problem, we use two pointers, `left` and `right`, to maintain an interval to search (the search space). We compare the target value with the value at the middle of the interval, and remove half of the search space by moving `left` or `right` pointer at each step.
There are three common patterns for implementing this solution,

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

All three patterns work correctly, and you can choose which you prefer. I personally prefer *Pattern C* because its invariant is explicit and easy to reason about.

### Answer Space

> Answer space is the conceptual domain of possible answers.
> Search space is the current interval where the answer must lie.

The answer space is all the values that an answer could be, it is also the space we are searching on. Many people instinctively think that we are searching the index range of array, which is `[0, 1, 2, ..., n - 1]`. This intuition is actually incomplete,
1. if `target <= max(nums)`, the answer is an index in `[0, n - 1]`,
2. if `target > max(nums)`, the correct insertion position may be `n`.
So the answer space is all possible positions where the target could be placed, which is `[0, 1, 2, ..., n]`. The key takeaway is that binary search operates on the space of possible answers, not necessarily on the input array itself.

### Monotonic Predicate

The most important question to ask yourself is:
“If I increase (or decrease) my candidate answer, does the condition change in only one direction?”

Formally, we define a predicate `P(x) = whether x is a valid answer`, which in lower bound is:
```python
def P(x):
    return nums[x] >= target
```

For binary search to work, `P(x)` must be monotonic: when we move the candidate answer in one direction on the answer space, once it becomes true, it stays true (or vice versa). For lower bound, this means that once `P(x)` becomes true as we move right, it stays true thereafter.

This is obvious because the array is sorted,
- If `nums[mid] >= target`, then all elements on the right side of `mid` are greater or equal to the target.
- If `nums[mid] < target`, then all elements on the left side of `mid` are less than the target.

```
# Input: nums = [-1,0,3,5,9,12], target = 9

index:               (-1)                      [  0   1   2   3   4   5  ]                    (n)
nums[i]:  (assumption of negative infinity)      -1   0   3   5   9   12             (assumption of infinity)
P(i):                 (F)                         F   F   F   F   T   T                      (T)
```

So we can safely halve the search space in each step. This is why we can shrink the space in every move.

> This is why many interview problems say things like:
> - minimum speed
> - maximum capacity
> - earliest day
> - smallest value that satisfies a condition
> These phrases are strong hints that binary search may apply.

### The invariant

What condition must always hold while we shrink the search space?

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

In Pattern C, we maintain the following invariant throughout the search:
> At any moment:
>    - all indices at or left of `left` are guaranteed to be `False`
>    - all indices at or right of `right` are guaranteed to be `True`

This invariant is what prevents infinite loops and off-by-one errors—every update strictly shrinks the search space.

This also reveals our effective search space. We are searching in the range of `(left, right]` (or `[left + 1, right]`).

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

At the beginning, we know nothing about any index. But we make the following assumptions:
- we conceptually assume the predicate holds `False` at index `-1`
- we conceptually assume the predicate holds `True` at index `len(nums)`

When we test `mid`,
- if `P(mid) is True`, then `mid` belongs to the `True` region
- if `P(mid) is False`, then `mid` belongs to the `False` region.

Because this invariant always holds, the search space strictly shrinks and the loop must terminate.

Eventually, the unknown region disappears at `left + 1 == right`, and `right` is the final answer.

### Common Failure Modes

- Infinite loop
    - Search space does not strictly shrink
    - Invariant is broken
- Wrong boundary updates
    - Mixing patterns without preserving invariants

### Summary

At this point, we have covered all the core concepts of binary search:
- Monotonic
- Search space
- Predicate
- Invariant


|Pattern| Search Space | Invariant | End of the loop |
|:-|:-|:-|:-|
|A| `[left, right + 1]`   |  `(not P(left - 1)) AND P(right + 1)` |`left - 1 == right` |
|B|  `[left, right]`      |  `(not P(left - 1)) AND P(right)` | `left == right`|
|C| `(left, right]` | `(not P(left)) AND P(right)` | `left + 1 == right`|
* Predicate: `P(i) = nums[i] >= target`
* We conceptually assume sentinel values beyond array bounds

### Extended use case

Lower bound is the fundamental problem and template solution for binary search. Actually lower bound can be used to find more index other than insertion index.

|Goal | Usage | If target index not exists|
|:-|:-|:-|
|The first index that >= x | `lowerBound(nums, x)` | return `len(nums)` |
|The first index that > x | `lowerBound(nums, x + 1)` | return `len(nums)` |
|The first index that < x | `lowerBound(nums, x) - 1` | return `-1` |
|The first index that <= x | `lowerBound(nums, x + 1) - 1` | return `-1` |

## Example Problems

### Koko Eating Bananas (LeetCode 875）

> https://LeetCode.com/problems/koko-eating-bananas

#### Answer Space

The answer space is `[1, max(piles)]`
- 0 is surely not a valid answer
- Since the problem guarantees `piles.length <= h`, `max(piles)` is a valid answer

#### Predicate

Eating speed k is valid only if Koko can eat all bananas under it.

```python
def check(s: int) -> bool:
    return sum([math.ceil(p / s) for p in piles]) <= h
```

#### Invariant

- Any speed `k <= left` is not a valid answer
- Any speed `k >= right` is a valid answer

#### Code

```python
class Solution:
    def minEatingSpeed(self, piles: List[int], h: int) -> int:
        def check(s: int) -> bool:
            return sum([math.ceil(p / s) for p in piles]) <= h

        left, right = 0, max(piles)
        while left + 1 < right:
            mid = (left + right) // 2
            if check(mid):
                right = mid
            else:
                left = mid
        return right
```

### Find Minimum in Rotated Sorted Array (LeetCode 153)

- https://LeetCode.com/problems/find-minimum-in-rotated-sorted-array/

This is a classic problem where the sorted structure is hidden, and many candidates fail because they try to “compare neighboring elements” instead of defining a predicate.

The array was originally sorted in ascending order, then rotated at some pivot. This means:
- There exists a minimum element
- To the left of the minimum, elements are larger
- From the minimum onward, the array is sorted again
So the array consists of two parts:
```
[ large ... large | small ... large ]
                  ↑
               minimum
```

#### Answer Space

We are searching for the index of the minimum element.
- All valid indices `[0, n - 1]` are possible answers
- Using Pattern C, our search space is `(left, right]`

So we initialize:
```
left, right = -1, len(nums) - 1
```
- `left` represents indices we are sure are not the minimum
- `right` represents indices that could be the minimum


#### Predicate

The key is to compare each element with the last element of the array. Why? Because:

- The rightmost element is always in the second (sorted) part
- All elements in the second part are `<= nums[-1]`
- All elements in the first part are `> nums[-1]`

So we define the predicate:
```
P(i): nums[i] <= nums[-1]

Index:                       -1                             [ 0   1   2   3   4   5 ]
nums:        (some number between nums[-1] and nums[0])       4   5   6   7   0   1
nums[-1]=1

P(i):                         F                                F   F   F   F   T   T

```
The first `True` corresponds exactly to the minimum element.


#### Invariant

At any moment:
- all indices `<= left` satisfy `P(i) == False` (`left` is always in the “larger values” region)
- all indices `>= right` satisfy `P(i) == True` (`right` is always in the “candidate minimum” region)

#### Code

```python
class Solution:
    def findMin(self, nums: List[int]) -> int:
        left, right = -1, len(nums) - 1
        while left + 1 < right:
            mid = (left + right) // 2
            # it is also work if we use: if nums[mid] < nums[right]:
            if nums[mid] < nums[-1]:
                right = mid
            else:
                left = mid
        return nums[right]
```

#### Interview Script (What to Say)

You can literally say this during the interview:
> “Although the array looks unsorted, it actually contains a hidden monotonic structure.
If I compare elements with the last element, I can define a predicate that is false before the minimum and true from the minimum onward.
Then I apply binary search to find the first true position.”

If they ask why not compare with the first element:
> “Comparing with the last element gives a cleaner monotonic predicate that directly maps to finding the minimum.”

## Appendix

### Interview-Ready Binary Search Sentences

You can memorize these sentences verbatim for interviews.

#### Identifying Binary Search

“This problem has a monotonic condition over the answer space.”

“Binary search here is applied to possible answers, not array indices.”

#### Defining the Predicate

“I define a predicate P(x) that checks whether x is a valid answer.”

“Once P(x) becomes true, it remains true for all larger values.”

#### Invariant & Boundaries

“Everything to the left of left is invalid, and everything at or to the right of right is valid.”

“If P(mid) is true, mid could still be the answer, so I move right.”

#### Ending the Loop

“At termination, right points to the smallest value that satisfies the predicate.”

#### More tips

When you feel stuck writing: write the invariant first, not the algorithm. If you can state:
- the predicate
- the answer space
- the invariant
The code almost writes itself—and so does the explanation.

### Proof of Time complexity (Optional)

The [Master Theorem](https://en.wikipedia.org/wiki/Master_theorem_(analysis_of_algorithms)) provides a direct way to determine the asymptotic runtime of divide-and-conquer algorithms.

For binary search, the upper bound of running time `T(n)` for an input size `n` is
```
T(n) = T(n/2) + O(1)
     = T(n/2) + O(n^0)
```

Therefore `T(n)` is $O(\log n)$.

