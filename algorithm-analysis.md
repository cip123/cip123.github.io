### Daily Temparatures

 Given a list of daily temperatures T, return a list such that, for each day in the input, tells you how many days you would have to wait until a warmer temperature. If there is no future day for which this is possible, put 0 instead.

For example, given the list of temperatures `T = [73, 74, 75, 71, 69, 72, 76, 73]`, your output should be `[1, 1, 4, 2, 1, 1, 0, 0]`.

Note: The length of temperatures will be in the range `[1, 30000]`. Each temperature will be an integer in the range `[30, 100]`.     

*Solution 1*

An intresting detail is that the low range for temperatures `[0, 30]`. That means that for seeing the next temperature, instead of looking into the entries of the array, we could just store the temperature and its index into a small array, and we would just iterate this array starting from the `current temperature + 1` to the end, and see it's index. There is problem though, how do we store the temperatures so that we always now the next temperature ?

The answer is start from the end. In our example, we mark `73` with the index `7`. Then we go to index `6`, we iterate through all temperatures from `77` to `100` and when we look for the smaller index. 

Tags: #constant-input

*Solution 2*

`Input:  [73, 74, 75, 71, 69, 72, 76, 73]`

`Output: [1, 1, 4, 2, 1, 1, 0, 0]`

The brute force solution would be to look for the first bigger temperature for each project. For example, for `73` it would be `74`, for `74` it would be `75` , for `75` it would be `76` and so on. N<sup>2</sup> complexity. 

On the other hand we always have to look at the next values to get a result, so we can try to navigate it backwards. While doing this, we notice that once we have a value bigger than the previous we don't need to keep the previous one since it would not be useful anymore. 

So we begin with `73`, this is the last one, so the result would be `0` since there is no recorded temperature after it. We go to `76`, the result would be still `0`, `73` won't help us here. We can forget about `73` and we keep track of `76`. We go to `72` the first one is `76` so we need to store it's index somehow. 

Question is, now that we have `72`, do we need to keep `76` around ? Yes. We will run into `75` later, and we need to know the `76` index in order to compute it. 

So we need to keep track of the index of the temperatures in a increasing order, starting from the end. At any point, we might need to remove from the head of the list. For this situations, the best suited tool would be a stack.

Tags: #stack

### Queue reconstruction by height

Suppose you have a random list of people standing in a queue. Each person is described by a pair of integers `(h, k)`, where `h` is the height of the person and `k` is the number of people in front of this person who have a height greater than or equal to `h`. Write an algorithm to reconstruct the queue.

Input 
```
[[7,0], [4,4], [7,1], [5,0], [6,1], [5,2]]
```
Output
```
[[5,0], [7,0], [5,2], [6,1], [4,4], [7,1]]
```

*Solution*

The question is, can you take an element and make sure it is in proper position. The answer is, yes. At the first glance we can look at the rank and take, `[5,0]` and then `[7,0]` and put it at the beginning. That already smells like greedy algorith. That is a good catch but you are stuck here. The next step could be any of `[5,2],[6.1]` or `[7,1]`.

Is there other possibility ? We read the problem and it says that only the taller or equal people count when counting rank, so if we take the maximum height we can arrange them in order of the rank `[7,0]` and `[7,1]`. 

Then we take the next tallest person, which is `6` and we can treat is at the taller ones since its rank will take into consideration the taller ones, but on the other hand the tallest won't be affected by it. So in this case it's `[6,1]` and we put it at position `1`. Then we take `5` and we put it at position `0` and `2` and so on. 

Note for this algorithm to work we need to use a list and always insert at it's position.

Tags: #greedy, #list


### Palindromic substrings

Given a string, your task is to count how many palindromic substrings in this string.

The substrings with different start indexes or end indexes are counted as different substrings even they consist of same characters.

Example 1:

    Input: "abc"
    Output: 3
    Explanation: Three palindromic strings: "a", "b", "c".

Example 2:

    Input: "aaa"
    Output: 6
    Explanation: Six palindromic strings: "a", "a", "a", "aa", "aa", "aaa".

*Solution 1*

We consider each middle of the palindrome, which could be a letter or an odd palindrom or a place between two letters for even palindromes. We grow from that point left and right until we still have a palindrome incrementing the count.

*Solution 2*

involves Manacher algorithm. That algorithm creates a second array and at each index it stores the maximum size of the palindrome created at that index. The algorithm it's too difficult though to consider it for testing, 

### Inorder Traversal iterative

Given a binary tree, return the inorder traversal of its nodes' values.

__Follow up__: Recursive solution is trivial, could you do it iteratively?

*Solution*

For mimicking a recursive solution iteratively, we use as stack. The only trick is that once you put the next elements in the stack you have to remove the links otherwise they will be processed again. For example :

Input: `[1,2,3]`
```
     1     
   /   \
  2     3
```    

For example we put `1` into the stack, and then we put `2` and then we put `3`. Then we pop `2`, we add it to the list we get back at `1` and then we would add `2` again on the stack. So it will never stop. Instead once we put `2` on the stack we cut the link between `1` and `2` by setting it to `null`.

So as long as we have left we put it on the stack and then `continue`. Otherwise we pop from the stack and push the right node, if present.

Tags: #binary-tree, #iterative

### Permutations

Given a collection of distinct integers, return all possible permutations.

Input: `[1, 2, 3]`

*Solution*

The simplest way to handle this would be a recursive solution. We take the first element and we put it in each slot. Then we take the second element and we put in in the remaining spots.

tags: #recursive

### Top K Frequent Elements
Given a non-empty array of integers, return the k most frequent elements.

Input: `nums = [1,1,1,2,2,3]`,  ` k = 2`

We could count the frequencies, store them in a heap and then pull only the first k ones. To construct the heap it would take `n` so it it won't work. 

* our algorithm's time complexity must be better than O(n log n), where n is the array's size.

### Subsets

Given a set of distinct integers, nums, return all possible subsets (the power set).

Note: The solution set must not contain duplicate subsets.

[1,2,3]

*Solution 1*

we construct the set using the previous solutions. So we start with the empty `[]` and so for each previous set we add `1`. So then we have `[],[1]`. Next one is `2` so that we have `[],[1],[2],[1,2]`. Next one is three so the final result would be `[],[1],[2],[1,2],[3],[1,3],[2,3],[1,2,3]`. 

We will return a list from each recursive call.

tags: #recursive

Note: I was tempted to start with all the numbers `[1, 2, 3]` and then add each time all the numbers again. That was clearly a stupid approach.

*Solution 2*

Basically for each number we have the option to put it in the set or not.

So for example, if we would have two elements, `[1,2]` we could have `[0,0], [0,1], [1, 0], [1, 1]`. In other words the number from `0` to `3` ( <code>2<sup>2</sup>-1</code>).

tags: #binary

* ### Letter Combinations of a Phone Number

Given a string containing digits from 2-9 inclusive, return all possible letter combinations that the number could represent.

A mapping of digit to letters (just like on the telephone buttons) is given below. Note that 1 does not map to any letters.

*Solution*
For each digit we recurse using each letter combination.

* ### Longest Substring Without Repeating Characters

Given a string, find the length of the longest substring without repeating characters.

Input: "abcabcbb"

*Solution 1* 

Brute force, starting at each digit we compute the longest string withough repeating characters, keeping a set which we reset.

*Solution 2*
 
We keep a sliding window, with 2 pointers, start and end. If we didn't see the end character before, we advance the end. Otherwise we advance the start and clear the seen status.

*Solution 3*

We are still using two pointers, start and end but we keep the index so we can advance the start pointer faster. Whenever we see that the end character was seen before, we advance the start character to it's index + 1. There is one gotcha, the index can't go back so we need to make sure it is `max(start, indices[s.charAt(end)])+1`.


* ### 3 Sum

Given an array nums of n integers, are there elements a, b, c in nums such that a + b + c = 0? Find all unique triplets in the array which gives the sum of zero.


*Solution* 

We sort the array, and we iterate each element as long as it is:
1. different than the previous (we want unique indices) 
2. greater than `0` (otherwise the list being ordered, we can't total to `0`).

 For the other two elements we do a caterpillar navigation starting from each end. Whenever we found the sum to be equal to the target we add it to the list. We also `continue` as long as we see a value similar to the previous one.

* ### Given an array of integers nums sorted in ascending order, find the starting and ending position of a given target value.

Your algorithm's runtime complexity must be in the order of O(log n).

If the target is not found in the array, return `[-1, -1]`.

*Solution*
The solution would be 2 binary modified binary searches where we don't stop when we see a candidate but we go left or right instead.

tags: #binary-search


* Combination Sum

Given a set of candidate numbers (candidates) (without duplicates) and a target number (target), find all unique combinations in candidates where the candidate numbers sums to target.

    Input: candidates = [2,3,6,7], target = 7,

*Solution*
The problem seems like a good candidate for recursion. For each entry we have the choice of using the number (multiple times, or not using it). We can stop whenever the existing number exceeds the target.

Tags: #recursion

* ### Rotate Image

You are given an n x n 2D matrix representing an image.

Rotate the image by 90 degrees (clockwise).

Note:

You have to rotate the image in-place, which means you have to modify the input 2D matrix directly. DO NOT allocate another 2D matrix and do the rotation.

*Solution* 
This is not a hard problem, but you have to be careful. You can use layers, and for each layer you do the proper rotation. It is useful to keep good naming variables, for example offset is layer + index

* ### Unique Paths
A robot is located at the top-left corner of a m x n grid (marked 'Start' in the diagram below).

The robot can only move either down or right at any point in time. The robot is trying to reach the bottom-right corner of the grid (marked 'Finish' in the diagram below).

How many possible unique paths are there?

*Solution*
We can do the problem recursive, at each position he has the option to go right or down. We can stop whenever bottom and right or out of bounds and we can also cache the results. We have to make sure we get the index right if the grid is `m x n` we have to stop at `m-1, n-1`.


* ### Sort Colors

Given an array with n objects colored red, white or blue, sort them in-place so that objects of the same color are adjacent, with the colors in the order red, white and blue.

Here, we will use the integers 0, 1, and 2 to represent the color red, white, and blue respectively.

Note: You are not suppose to use the library's sort function for this problem.

*Solution 1*

This is a case of bucket sort. We count the frequencies and then layout the result.

*Solution 2*

Flag sort. We keep a left pointer for zeroes and a right pointer for twos.
Whenever we have a zero we swap it with p0 and we increment p0. When we have a two we swap it with p2 and we decrement p2. That way all the elements at the left of p0 are 0 and to the right of p2 are 2. 

There are two tricks:
1. when do we go to the next element ?
if the number is `0` we know we can advance because we put it to the left and we are done with it. If the number is 1, it is in the right place so we can advance as well. If the number is 2, we don't know what will come from the swap. A zero, a one or a two. So we must wait.
2. When do we stop.We should stop when index greater than p2 since all the elements there should be already ordered. 


 * ### Unique Binary Search Trees

 Given n, how many structurally unique BST's (binary search trees) that store values 1 ... n?

 Given n = 3, there are a total of 5 unique BST's:

    1          3     3      2      1
      \       /     /      / \      \
       3     2     1      1   3      2
      /     /       \                 \
     2     1         2                 3


We compute a solution based on the previous one. At this point it seems easier to start bottom up. So when we have n = 1 we have a tree.
When we have `n = 2` we have `2*1`.

We skip to three. We have two simetric trees, when we put `1` as root or when we put `3` as root. And we have another one where we have `2` as root. 
So `calc(3) = 2 * calc(2) + 1`

Let's try to compute it for `4`. We have 2 simmetric trees using `1` and `4` as the root. We have another `2` simmetric trees when we use `2` and `3` as the root. 

For `1` and `4` the values are the same as `calc(3)`.
When we use `2` or `3` as the root, we have a tree with a child on one side and 2 children on the other side. So the number of children would be `calc(2)`. The total formula would be `2 * calc(3) + 2 * calc(2)`.

For `5` we would have `2 * calc(4)` and `2 * calc(3)`. Then we would have a new case where we would have `2` on one side and `2` on the other side.
Total: `2 * calc(4) + 2* calc(3) + 1 * (2* 2)`

For `6` we would have, `2 * calc(5) + 2 * calc(4)`. Now we would have a new case as well. 2 tree with 2 children on one side and 3 children on another side.
Total:  `2 * calc(5) + 2 * calc(4) + 2 * calc(3) * calc(2)`

I think we can extrac the formula. We can observe a special case which would help us built the formula faster. Whetever we have one  child or 0 children on a branch the modifier is 1. So we can start our bottom up approach with:

    dp[0] = 0;
    dp[1] = 1;
    dp[2] = 2 * dp[1]* dp[0];
    dp[3] = 2 * dp[2] + dp[1] * dp[1]
    dp[4] = 2 * dp[3] * dp[0] + 2 * dp[2] * dp[1]
    dp[5] = 2 * dp[4] * dp[0] + 2 * dp[3]*dp[1] + dp[2] * dp[2]
    dp[6] = 2 * dp[5]dp[0] + 2 * dp[4]* dp[1] +  + 2 * dp[2] * dp[3]

tags: #bottom-up

* ### Validate Binary Search Tree

Given a binary tree, determine if it is a valid binary search tree (BST).

The problems seems simple enough, just need to validate for each node if the left child is smaller than the parent and the right child is bigger than the parent. There is one gotcha though, the outermost right child of the left tree should be smaller than the root and the outermost left child of the right tree should be bigger than the root. So whenver we do the validation we should pass in the correct interval.

The root can be between any bounds so we can pass `Integer.MIN_VALUE` and `Integer.MAX_VALUE`. When we go to the left the interval becomes, `[Integer.MIN_VALUE..root.val]`. To the right it becomes, `[root.val .. Integer.MAX_VALUE]`. The limits are exclusive. The problem is that one tricky test case could be on of the `Integer.MIN_VALUE, Integer.MAX_VALUE` so we would better use `null` instead.

* ### Binary Tree Level Order Traversal

We use a list for each level. We do it recursively (depth first search), and we pass down the level. If we are using an in order traversal we should have the list propery alighned.

Note: I always get the tendency to do it BFS, since we need to operate at each level but it doesn't seem to be needed, in fact it looks much harder that way. I didn't yet find a problem where a BFS solution would be more appropriate.


* ### Construct binary tree from preorder and inorder traversal

Given preorder and inorder traversal of a tree, construct the binary tree.

    preorder = [3,9,20,15,7]

    inorder = [9,3,15,20,7]


*Solution* 

We go for the root. As we have a preorder we know it is the first element : `3`. We also know that it is has rank `1`, meaning that there is one element to the left and three elements to the right. 

We keep three indices, one index for the element that we are processing from perorder: `index`. We keep a left and right index for the `inorder` array as well. That way we can compute the rank of the element in constant time.


So we got to `index = 0, left = 0, right = 4`.

We notice that rank of `3` is `1`. So we build left subtree with `[0] = 9` and the right subtree with `[2, 4] = (15, 20, 7)`.

One tricky thing is how to pass the index around without using a global variable. For the left node is easy, it is `index + 1`. For the right node is `index + 1` + number of the elements which will go to the left. That is `rank-left`. 



279. ###Perfect Squares

Given a positive integer n, find the least number of perfect square numbers (for example, 1, 4, 9, 16, ...) which sum to n.

*Solution*

Actually the solution is more complicated then it seems. We can cache the minimum entry it took to reach each entry.

So for example we need to reach `10` which is `9` and `1`. 
We start with `3`, `2` and `1` ( no point in using `0`).

For 3 would have the following recursive calls :

    10 -> 3         
        1 -> 1
           0  

So at that point we could cache `1` with `1` move and `10` with `2` moves

    10 -> 2             // cache 10 with 4 squares
        6 -> 2          // cache 6 with 3 squares    
            2 -> 2  
                -2 -> 2
            2 -> 1      // cache 2 with 2 squares
                1 -> 1  // cache 1 with 1
                    0

    10 -> 1              //10 -> 7 squares
        9 -> 1           // 9 -> 6 squares
            8 -> 1       // 8 -> 5 squares    
                7 -> 1   // 7 in 4 squares
                    6 -> // we have it in cache. We return 3 



## 378. Kth Smallest Element in a Sorted Matrix

Given a n x n matrix where each of the rows and columns are sorted in ascending order, find the kth smallest element in the matrix.

Note that it is the kth smallest element in the sorted order, not the kth distinct element.

Example:

    [ 1,  5,  9],
    [10, 11, 13],
    [12, 13, 15]

    k = 8,

    return 13.


*Solution 1 (merge arrays)*

We can look at this problem as a set of sorted arrays. We can simulate a merge , (using a priority queue?), taking the smallest element each time and counting them. Once we reach K, we return.

*Solution 2 (binary search + caterpillar)*

We know how to search a element in O(N), we just don't know what the element is. So we can try to guess it using binary search (it might be or not in the matrix) and then count how many elements are smaller or equal with it. If the count is too small we use binary search to find the next number.

There is one tricky part, when doing binary search if we get the exact count this doesn't necessarily mean that we found the result. It could be a number that we made up which we know that is a number bigger than n elements which fits our requirement. But it could be other, so we mark it as a candidate and we try a smaller one.


341. Flatten Nested List Iterator

Given a nested list of integers, implement an iterator to flatten it.

Each element is either an integer, or a list -- whose elements may also be integers or other lists.

Example 1:

    Input: [[1,1],2,[1,1]]
    Output: [1,1,2,1,1]
    Explanation: By calling next repeatedly until hasNext returns false, the order of elements returned by next should be: [1,1,2,1,1].

Example 2:

    Input: [1,[4,[6]]]
    Output: [1,4,6]
    Explanation: By calling next repeatedly until hasNext returns false, the order of elements returned by next should be: [1,4,6].

*Solution 1: preprocess and using a queue*

We ignore the fact that this is an iterator and we just compute all the values upfront using a DFS and a queue. If it's an iteger we add it to the stack, else we add all it's field to the stack recursively.

```java
    private void addToQueue(NestedInteger nestedInteger, Queue queue) {
        
        
        if (nestedInteger.isInteger()) {
            queue.add(nestedInteger.getInteger());
        } else {
            
            for (NestedInteger child : nestedInteger.getList()) {
                addToQueue(child, queue);
            }
            
        }
        
    }
```

*Solution 2: (real iterator)*

In order to implement a real iterator, we wouldn't need to copy the structure but to keep a iterator to the nested list.

This is the covariants.

* If the current element is of type integer, it is straight forward. The child iterator is null and we have a next.
* If the current element is of type list, we have a child iterator and we have a next if the list is not empty or there are other elements in the previous list. 

Ideally we would like all this covariants setup at every time so whenever we call hasNext and next everything is ready so we can return in O(1).
The problem appears when the current element is of type list and it is empty and it is not the last element in the array. Cause the hasNext will return true (because there are next elements) and next will throw a error, cause we try to get index 0 of an empty list.

So the best thing to do is to ignore a list which is empty.
Here is the logic which finds a valid position:

```java
    private void moveToValidPosition() {
        
        while (index < list.size()) {                
            NestedInteger current = list.get(index);    
            
            if (current.isInteger()) break;
            
            childIterator = new NestedIterator(current.getList());
            if (!childIterator.hasNext()) {
                index++;
            } else {
                break;
            }

        }
        
    }
```    

## 334. Increasing Triplet Subsequence


Given an unsorted array return whether an increasing subsequence of length 3 exists or not in the array.

Formally the function should:

Return true if there exists `i, j, k` such that `arr[i] < arr[j] < arr[k]` given `0 ≤ i < j < k ≤ n-1` else return false.

Note: Your algorithm should run in `O(n)` time complexity and O(1) space complexity.

*Solution*


At each iteration we will keep track of the minimum 2 values seen so far. The mid variable denotes the end of the smallest subsequence seen so far.

For example.
[5,6,5,1,2,4]


* i = 0, low = 5, mid = Integer.MAX_VALUE;
* i = 1, low = 5, mid = 6;
* i = 2, low = 5, mid = 6;
* i = 3, low = 1, mid = 6;

Now this is the bit where it seems counterintuitive. The low and mid doesn't seem to be consistent since the mid is before the low in the original array. But the fact that mid is 6 means that if we see something bigger than 6 we return. 
* i = 4, low = 1, mid = 2;
* i = 5, low = 1, mid = 2; high == 4, return


## 395. Longest Substring with At Least K Repeating Characters

Find the length of the longest substring T of a given string (consists of lowercase letters only) such that every character in T appears no less than k times.

Example 1:

    Input:
    s = "aaabb", k = 3

    Output:
    3

The longest substring is "aaa", as 'a' is repeated 3 times.

Example 2:

    Input:
    s = "ababbc", k = 2

    Output:
    5

The longest substring is "ababb", as 'a' is repeated 2 times and 'b' is repeated 3 times.


*Solution 1*

I tried the brute force solution, for each index, `i` and `j`, computed the frequencies and checked if the substring is valid. 
Didn't pass.


*Solution 2*

    "aaacbb"

We can check each character assuming that the string is ending at each char.

* 0, a = 1, can be a solution so we go forward
* 1, a = 2, can be a solution so we go forward
* 2, a = 3, we have a result
* 3, a = 3, b = 1, b is not a candidate so we reset the start to end + 1.

Case 2:
    
    "ababbc"
    freqs = {a=2,b=3,c=1}, min = 2


* 0, a, a = 1, can be a solution so we go forward, min = 1
* 1, b = 1, can be a solution so we go forward
* 2, a = 2, b = 1 can be a solution so we go forwarrd
* 3, a = 2, b = 2, it is a solution, we go forward
* 4, a = 2, b = 3, it is a solution, 
* 5, c is not a solution

Case 3:

    "bbaaacbd"
    freqs = {a=3,b=3,c=1, d=1}, min = 3

* 0, b, b=1 can be a solution
* 1, b, b=2 can be a solution
* 2, a, b=2, a=1 can be a solution
* 3, a, b=2, a=2 can be a solution
* 4, a, b=2, a=3, it is a solution, we just need to get rid of extra `b` from the beginning. We remove from the start window until the string is valid. 

We use two helper functions: 

```java
    boolean hasValidResults(int[] freqs, int k) {
        
        for (int n : freqs) {
            if (n >= k) {
                return true;
            }
        }
        
        return false;
    }
    
    boolean isValid(int[] freqs, int k) {
        
        int min = Integer.MAX_VALUE;;
        
        for (int n : freqs) {
            if (n != 0) {
                if (n < k) return false;
            }
        }
        
        return true;
        
    }
```

That can pass but it takes 2ms when it is a little optimized.
We lose a lot of time when we are shrinking the window, it would be nice to see in one bit if shrinking the window fixed the validation.

We can track the number of characters that have k count and the number of unique characters. kCount and uCount. if Kcount == uQcount we have a valid result else is valid.

Update, it take the same.


TODO: recursive approach

## 134. Gas Station

There are N gas stations along a circular route, where the amount of gas at station i is gas[i].

You have a car with an unlimited gas tank and it costs cost[i] of gas to travel from station i to its next station (i+1). You begin the journey with an empty tank at one of the gas stations.

Return the starting gas station's index if you can travel around the circuit once in the clockwise direction, otherwise return -1.

**Note:**

If there exists a solution, it is guaranteed to be unique.
Both input arrays are non-empty and have the same length.
Each element in the input arrays is a non-negative integer.

Example 1:

Input: 

    gas  = [1,2,3,4,5]
    cost = [3,4,5,1,2]

Output: 3

*Solution 1*

First one is the obvious, we travel through each stations and we compute the tank and if it's possible to get to the next one until we do a full trip. Complexity is O(N^2).

*Solution 2*

This seems to be the same pattern with the celebrity one. to do in We try to do it in linear time and we aim not to find the solution but to eliminate all the other candidates.


How can we do that ? We assume that the starting gas stations is zero and at each round we check if the tank is smaller than 0. If it is the startingIndex is not a solution so we set the starting index to be the next gas station. 

* What if we missed a candidate on the road ? For example we started at zero and we noticed that tank is negative at 2, could 1 be a solution ? No, 1 starts with leftover tank from 0 so it has at least as much gas as it would have it starting from 1.

So know we know that all the other elements before startingIndex are not candidates. How do we check that startingIndex is the solution. We know that we can reach from startingIndex to the end, how do we know that we can reach from 0 to the starting index ?

We check the total gas, if total gas is positive, does it mean that we can get from a round trip from a point. What does it mean to not be able to go to a roundtrip from any point. It means that there is a point on that road where totalGas is smaller than the totalCost. On the other hand if the end totalGas is greater or equal to totalCost, there should also be a gas station which compensates for all negative numbers, and then if we start at that gas station we are set. 

What if there are 3 negative and 2 positive, -3, -4, -5 and 1, 11. 

We can arrange them in anyway, but we will find a way to traverse them:

    -3, -4, -5, 1, 11. 
    -3, 1, -4, -5 11.


## 324. Wiggle Sort II


Given an unsorted array nums, reorder it such that nums[0] < nums[1] > nums[2] < nums[3]....

Example 1:

    Input: nums = [1, 5, 1, 1, 6, 4]
    Output: One possible answer is [1, 4, 1, 5, 1, 6].

Example 2:

    Input: nums = [1, 3, 2, 2, 3, 1]
    Output: One possible answer is [2, 3, 1, 3, 1, 2].


*Solution 1*

We try a greed algorithm at each position we calculate the best outcome for that position using the neighbours.

Unfortunately this does not work with long string of equal values. Eg

    1 1 1 1 2 3 4 5

So we can try another approach, sort the array and then modify it using one of those approaches.

* take a element from the start and then an element from the end. This will fail for `4,5,5,6`, it will result in `4,6,5,5` which does not work.
* take an element from the start and a element from the mid. It still will not work because `4, 5, 5, 6` will result in `4, 5, 5, 6` which is still not valid.


The problem with both the above approaches is that they either end in the same place, (first approach) or as the string is small the firt one ends just after the the second one starts. 

A valid response would be `5, 6, 4, 5`, so we start from the mid and end and that we make sure we have a smaller item between them

*Note* this is diferrent then wiggle sort where the elements could have been equal where we could greedly correct the location in at each point.

*Solution 2 O(n)* 

We could try to use selection rank algorithm until we find a mid such as all the elements before mid are smaller than him and all the elements after mid are greater or equal and then we intertwine them. Actually the solutions is pretty complicated, a guy wrote an entire essay about wiggle sort which can be found here.

https://leetcode.com/problems/wiggle-sort-ii/discuss/77684/Summary-of-the-various-solutions-to-Wiggle-Sort-for-your-reference

The key thing here is that we have to use map indexing or virtual indexing for this and this it out of scope for this preparation.





## 239. Sliding Window Maximum

Given an array nums, there is a sliding window of size k which is moving from the very left of the array to the very right. You can only see the k numbers in the window. Each time the sliding window moves right by one position. Return the max sliding window.

Example:

    Input: nums = [1,3,-1,-3,5,3,6,7], and k = 3
    Output: [3,3,5,5,6,7] 


*Solution 1*

We can start with the quadratic solution, it means that for every index, we go back `k` elements and find the maximum.

How can we optimize this ? 


At each iteration we could remove the elements which are smaller then the last element added. So that way the first element will always be the maximum.


There are two ways of doing it, start from the head or start from the tail. If we start from the head, we might have elements unsorted elements to the end of the queue, because we are adding elements from the right and prune them from the head. We should return elements from the same side that we added them.

This reduces to a stack problem where we pop elements from the end if the current element is bigger then them. 

At the same time we need to access the head for removing it if it is out of the sliding window, or returning when we need to find the maximum in real time.



Example:

    [1,3,1,2,0,5]
    3


* `0`, deque = [1]
* `1`, deque = [3,1]
* `2`, deque = [3,2]
* `3`, deque = [5]


One additional thing is that, because we have to remove indices from the start of the queue sometimes, we better keep indices in the queue instead of numbers, so we can see whenit's time for an index to go away.


*Solution 2*

We use two precalculated array left and right, where for each position i we know the maximum from the start of the block to i, and the max from the end of the block to i. So for each interval `i, j` we will know the exact maximum, if it is in the same block or in different blocks.


## 308. Range Sum Query 2D - Mutable
Hard

Given a 2D matrix matrix, find the sum of the elements inside the rectangle defined by its upper left corner (row1, col1) and lower right corner (row2, col2).


Example:


    [3, 0, 1, 4, 2],
    [5, 6, 3, 2, 1],
    [1, 2, 0, 1, 5],
    [4, 1, 0, 1, 7],
    [1, 0, 3, 0, 5]

    sumRegion(2, 1, 4, 3) -> 8
    update(3, 2, 2)
    sumRegion(2, 1, 4, 3) -> 10

*Solution*

We can compute a prefix sum in order to get the response faster. There are two ways to do it:

* per matrix, meaning that for each submatrix we compute it's prefix sum and the query is O(1);. The down side is that the update cost O(n<super>2</super>)
* per row, we compute the row's sum and the query is (O(n)). The update is also O(n).

Since the first method involves more complex calculations, and the updates happen as frequently as queries it is better to go with the second alternative.

**Note** another alternative is to use Binary Index Trees that were created with this aspect in mind.


## 84. Largest Rectangle in Histogram

Given n non-negative integers representing the histogram's bar height where the width of each bar is 1, find the area of largest rectangle in the histogram.

 
*Solution 1*

a N^2 solution where I just iterate through each interval and for each start I find the minimum at each end and I calculcate the distance to each end.

*Solution 2* 

We can use a stack, we go from right to left and we only keep the minimums into the stack. The logic is that any elements after the minimum cannot be used. We use the indices in order to compute the area.

We navigate backwards and whenever we get an element greater than the current element, we pop it. At each index we have in the stack the indices to the elements smaller than it. We can easily calculate the distance to the next element bigger than it, by computing the next missing index from the stack but if we want to calculacte the distance to all remaining indices we can do it in N^2.

We can try another approach, we go right keeping elements in increasing order, the max being at the top of the stack. Whenever we see a decreasing element we pop all the elements smaller then him, computing the area till the beginning. 

How do we compute the area ? 

    [6, 7, 5, 2, 4, 5, 9, 3]

* we add `0(6)` to the stack
* we add `1(7)` to the stack

 * `2(5)` we pop `0(7)` and we compute the area `7 (7 * (2 - 1 - 0))` and we pop `1(6)` we have `12 (5 * (2 - 1 - -1))`. We add `2(5)` to the stack. 
 * `3(2)`, we pop `2(5)` and we add the area formed with it. We just need to find the next element smaller than it, which is none `-1`, so the area is `15 (5 * (2 - -1))`
















