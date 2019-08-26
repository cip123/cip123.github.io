---
layout: post
title: Linked Lists
date: 2019-08-19
categories: algorithms
tags: [linked-lists, fast-runner]
---


### 142. Linked List Cycle II

Given a linked list, return the node where the cycle begins. If there is no cycle, return null.

To represent a cycle in the given linked list, we use an integer pos which represents the position (0-indexed) in the linked list where tail connects to. If pos is -1, then there is no cycle in the linked list.

Note: Do not modify the linked list.


*Solution 1* 

use a set to store the values. If we have have seen it before we return it.

*Solution 2: Floyd's Tortoise and Hare* 


We use a normal pointer and fast pointer. We run until fast runner is null
or they intercept. 

If they intercept we know that they are in a cycle. 

Now it comes the most important part: 

the distance between intersection point and the beginning of the cycle is the same with the distance between the start of the list and the beginning of the cycle. 

*How do we know that ?*

                  4 -> 5
    1 → 2 → 3   /      |
                \ 7 <- 6 

So 

    H -> 3
    T -> 2

    H -> 5
    T -> 3

    H -> 7
    T -> 4

    H -> 4
    T -> 5

    H -> 6
    T -> 6

So when the tortoise entered the cycle after `k` nodes (at node `3`), the hare did `2 * k`, meaning it was `k` nodes inside the loop.  We note `k` is the distance from the start of the loop.

So T = 3, H = 5. k = 2.

They meet at node `6` which is also `k` nodes from the start of the loop. How is that ?

The cycle length is `5`, and they would have met under `5` nodes if they started at the same place. H had an advance of `k % 5` and so they met after `5 - k = 3` nodes from the beginning of the list. So H is  `k (2)` nodes from completing the cycle


### 287. Find the Duplicate Number

Given an array nums containing n + 1 integers where each integer is between 1 and n (inclusive), prove that at least one duplicate number must exist. Assume that there is only one duplicate number, find the duplicate one.


Example 1:

    Input: [1,3,4,2,2]
    Output: 2

*Solution*
     0 1 2 3 4
    [1,3,4,2,2]

We can look at this as a linked list:
    
    1 -> 3 -> 2 -> 4 -> 2

So we can do the same Floyd's tortoise and hare scheme.

## 148. Sort List

Sort a linked list in O(n log n) time using constant space complexity.

*Solution*

The best fit for this kind of sort is merge sort. We only have to be careful about how we are treating the list segments when we split the list in tow. The most elegant way to do it is to break the connection betweeen nodes. The alternative would be to keep to pass along the left and the right length to the merge function.

