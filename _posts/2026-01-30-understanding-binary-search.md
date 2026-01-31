---
title: Understanding Binary Search: An Interviewee's Perspective
date: 2026-01-30 17:00:00 +0800
categories: [Algorithms]
tags: [binary search, coding interview]
math: true
---

> "Although the basic idea of binary search is comparatively straightforward, the details can be surprisingly tricky."
> — **Donald E. Knuth**, *The Art of Computer Programming*

If you've ever prepared for a coding interview, you’ve definitely encountered Binary Search (also known as half-interval search, logarithmic search, or binary chop). On paper, it’s one of the simplest problems on LeetCode. We all know the classic version: use two pointers at the start and end of a sorted array, repeatedly halve the range, and find the target in $O(\log n)$ time. However, many people (including me) struggle to implement it correctly  - especially under interview pressure. In my experience, the difficulty lies in two areas:
1. **Identification:** How do we recognize a problem as a Binary Search problem and convert it into one?
2. **Implementation:** How do we write bug-free code that handles edge cases like infinite loops?

Let's break them down!

## What is Binary Search?

What would you say if the interviewer asked you to explain the binary search algorithm in one or two sentences? [Wikipedia](https://en.wikipedia.org/wiki/Binary_search) exlpains it as follows:
> ... is a search algorithm that finds the position of a target value within a sorted array. Binary search compares the target value to the middle element of the array. If they are not equal, the half in which the target cannot lie is eliminated and the search continues on the remaining half, again taking the middle element to compare to the target value, and repeating this until the target value is found. If the search ends with the remaining half being empty, the target is not in the array.

This is indeed how most people understand binary search. Most of us first learn binary search as "finding a target value in a sorted array". But in interviews, binary search is not just about arrays -- it is about **structure**. We can use binary search not only on arrays, but also on service capacity, distances, or any other structures that can be abstracted into a sorted sequence. When I say sequence, I even include cases where the search space is conceptually infinite.

In fact, in many interview problems, there is no explicit array at all -- the sorted structure is hidden. You must abstract and extract a *virtual* array in your mind. So back to the question, if you need to explain binary search to an interviewer, here is a golden rule worth memorizing,
```
Binary Search finds the boundary of a monotonic predicate by repeatedly halving the search space.
```

This single sentence helps us to communicate clearly with interviewers and write bug-free code more reliably. Pay attention to the key words: **monotonic predicate** and **search space**. They will help us a lot.

## Anchor Problem: Lower Bound

Let's take the most well-known BS problem, lower bound (LeetCode 35), as an example.

### Problem

> Lower bound: Given a sorted array of distinct integers and a target value, return the index if the target is found. If not, return the index where it would be if it were inserted in order.
> You must write an algorithm with O(log n) runtime complexity.
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

### Discussion

#### Sorted

We all know that lower bound can be resolved with BS. But let's set that aside for a moment and ask a more fundamental question:
- Is this problem suitable for Binary Search?

> There are many "interview tricks" for identifying algorithms. For example, people often say "If the required time complexity is $O(\log n)$ , then it must be Binary Search". This sometimes works -- but it is neither reliable nor complete.
> Instead of relying on surface hints, we need a deeper understanding of binary search.

The reason that we can use BS is that the array is sorted.
- If `nums[mid] > target`, then all elements on the right side of `mid` are greater than the target.
- If `nums[mid] < target`, then all elements on the left side of `mid` are less than the target.
So we can safely halve the search space in each step. This is why we can shrink the space in every move.

#### Search Space

This is a very common source of confusion. When solving the lower bound problem, many people instinctively think that we are searching the index range of array, which is `[0, 1, 2, ..., n - 1]`. This intuition is acutally incorrect,
1. if `target <= max(nums)`, the answer is an index in `[0, n - 1]`,
2. if `target > max(nums)`, the correct insertion position may be `n`.
So the answer space is all possible positions where the target could be placed. Remember this point, it will be crucial in the next section.
The key takeaway is that binary search operates on the space of possible answers, not necessarily on the input array itself.

#### Predicate

In terms of indices, the problem becomes:
- Find the first index `i` that `nums[i] >= target` (if no such i, return `len(nums)`).

This immediately reveals the structure we need. We define a predicate that,
```python
def check(i):
    return nums[i] >= target
```

Now observe what happens as `i` increases:
```
# Input: nums = [-1,0,3,5,9,12], target = 9

index:               (-1)                      [  0   1   2   3   4   5  ]                    (n)
nums[i]:  (assumption of negative infinite)     -1   0   3   5   9   12             (assumption of infinite)
P(i):                 (F)                         F   F   F   F   T   T                      (T)
```

The predicate changes exactly once — from `False` to `True`. This is a monotonic predicate.

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

What condition must always hold while we shrink the search space?

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

At the beginning, we know nothing about any index. But we make the following assumptions:
- the predicate holds `False` at index `-1`
- the predicate holds `True` at index `len(nums)`

When we test `mid`, 
- if `P(mid) is True`, then `mid` belongs to the `True` region
- if `P(mid) is Fasle`, then `mid` belongs to the `False` region.

In both cases, the boundary moves inward and unknown region strictly shrinks.

Eventually, the unknown region disappears at `left + 1 == right`, and `right` is the final answer.

We are looking for the smallest index for which the predicate is `True` (the leftmost element that is greater or equals to target), and we always keep the invariant, so `left` is never an answer candidate and `right` is always one. That means,
- We are searching in the range of `(left, right]`.

### Common Failure Modes

- Infinite loop
    - usually caused by failing to strictly shrink the search space (e.g. left = mid instead of mid - 1).
    - or incorrectly initialization of `left` and `right`
- Wrong boundary updates
    - mixing invariants from different patterns leads to off-by-one errors.

### Summary

At this point, we have covered all the core concepts of binary search:
- Monotonic
- Search space
- Predicate
- Invariant


|Pattern| Search Space | Invariant| End of the loop |
|:-|:-|:-|:-|
|A| `[left, right + 1]`   |  `(not P(left - 1)) AND P(right + 1)` |`left - 1 == right` |
|B|  `[left, right]`      |  `(not P(left - 1)) AND P(right)` | `left == right`|
|C| `(left, right]` | `(not P(left)) AND P(right)` | `left + 1 == right`|
* predicate: `P(i) = nums[i] >= target`
* We conceptually assume sentinel values beyond array bounds

### Extended use case

Lower bound is the fundamental problem and template solution for binary search. Actually lower bound can be used to find more index other that insertion index.

|Goal | Usage | If target index not exists|
|:-|:-|:-|
|The first index that >= x | `lowerBound(nums, x)` | return `len(nums)` |
|The first index that > x | `lowerBound(nums, x + 1)` | return `len(nums)` |
|The first index that < x | `lowerBound(nums, x) - 1` | return `-1` |
|The first index that <= x | `lowerBound(nums, x + 1) - 1` | return `-1` |

## More Anchor Problems

### Koko Eating Bananas (LeetCode 875）

> https://LeetCode.com/problems/koko-eating-bananas

#### Search Space

0 is surely not a valid answer, we can initial `left` to 0.
Since the problem guarantees `piles.length <= h`, `max(piles)` is a valid answer. we can initial `right` to this.

#### predicate

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

#### Search Space

We are searching for the index of the minimum element.
- All valid indices `[0, n - 1]` are possible answers
- Using Pattern C, our search space is `(left, right]`

So we initialize:
```
left, right = -1, len(nums) - 1
```
- `left` represents indices we are sure are not the minimum
- `right` represents indices that could be the minimum


#### predicate

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

### Quick Note

#### Pattern Contract

##### What is monotonic?

Monotonic means that once a condition on the value `x` becomes `true` (or `false`), it never flip back again. That is, for a sequence, if we use a condition to validate its elements one by one, it looks like,
```
[False, False, False, True, True, True, ...]
[True, True, True, False, False, False, ...]
```

##### What is search space?

The space that answer always lies in during our loop. For pattern C, it's `(left, right]` (or `[left + 1, right]`).


#### The invariant

At any moment, Pattern C:
    - all indices at or left of `left` are guaranteed to be `False`
    - all indices at or right of `right` are guaranteed to be `True`

### Proof of Time complexity (Optional)

The [Master Theorem](https://en.wikipedia.org/wiki/Master_theorem_(analysis_of_algorithms)) provides a direct way to determine the asymptotic runtime of divide-and-conquer algorithms.

For binary search, the upper bound of running time `T(n)` for an input size `n` is
```
T(n) = T(n/2) + O(1)
     = T(n/2) + O(n^0)
```

Therefore T(n) is O(log n).

### Interview-Ready Binary Search Sentences

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

