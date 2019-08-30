---
layout: post
title: Recursivity
date: 2019-08-15
categories: recursivity
tags: [bottom-up, top-down]
---

## 64. Minimum Path Sum

Given a `m x n` grid filled with non-negative numbers, find a path from top left to bottom right which minimizes the sum of all numbers along its path.

*Solution*

We create a recursive function where we return the minimum sum to the end at each element. 

The problem arises if we want to cache the results. What can we cache at each `(row, column)` ?

It should be the minimum distance to the end point. So for example for the following matrix:

    [1, 3, 1],
    [1, 5, 1],
    [4, 2, 1]

at each position we should cache the min distance to the end. The cache matrix should look like this:

    [7, 6, 3]
    [8, 7, 2]
    [7, 3, 1]

So at `grid(2,2)` the sum is `1`. At `grid(2,1)` the min sum is `3`.

This is the top-down approach:

```java 

    private int minPathSum(int[][] grid, int row, int column, Integer[][] cache) {


        // if we go out of bounds
        if (row == grid.length || column == grid[0].length) {
            return Integer.MAX_VALUE;
        }

        // if we already know the minimum path sum from this point to the end
        if (cache[row][column] != null) {
            return cache[row][column];
        }

        // we reached the target
        if (row == grid.length -1 && column == grid[0].length -1) {
            cache[row][column] = grid[row][column];
            return grid[row][column];
        } else {

            // we compute the minimum path at this point
            int min = Math.min(minPathSum(grid, row + 1, column, cache), minPathSum(grid, row, column + 1, cache)) + min + grid[row][column];

            cache[row][column] = min;

            return min;
        }

    }

```

If we execute the code we get the following cache hits:

    cache hit at [2,1]: 3
    cache hit at [2,2]: 1
    cache hit at [1,1]: 7
    cache hit at [1,2]: 2


*Solution 2 (bottom-up)* 

We start with the end and another matrix for computing the paths. 
We copy the bottom right position, and for each  position we have we use the min between the down path and the right path plus the value at that position.

```java

    public int minPathSum(int[][] grid) {
        int[][] cache = new int[grid.length][grid[0].length];

        for (int row = grid.length-1; row >= 0; row--) {

            for (int col = grid[0].length-1; col >= 0; col--) {

                int min;
                
                if (row == grid.length -1 && col == grid[0].length -1) {
                    min = 0;
                } else if( row == grid.length -1) {
                    min = cache[row][col+1];
                } else if (col == grid[0].length -1) {
                    min = cache[row+1][col];
                } else {
                    min = Math.min(cache[row][col+1], cache[row+1][col]);
                }

                cache[row][col] = min + grid[row][col];
            }
        }


        return cache[0][0];
    }

```

Note: I was tempted to use a bigger cache matrix so I don't have to handle the borders (`row + 1, col + 1`) but I got into troubles.

tags: #recursive. #bottom-up


## 279. Perfect Squares

Given a positive integer n, find the least number of perfect square numbers (for example, 1, 4, 9, 16, ...) which sum to n.

*Solution 1 (top down)*

We create a recursive function where we go to the end by removing a square from the target. If we reach zero we return 1. If we bypass it we return a invalid value, (Integer.MAX_VALUE) should be safe cause there is no way we would have so many tries. 

So for example we need to reach `10` which is `9` and `1`. 
We start with `3`, `2` and `1` ( no point in using `0`).

We notice an upper bound, the number we use for constructing the square should not be greater than `Math.sqrt(target)`.

```java
    public int numSquares(int n) {
        
        if ( n < 0) {
            return Integer.MAX_VALUE;
        } else if (n == 0) {
            return 0;
        }

        int min = Integer.MAX_VALUE;

        int step = (int) Math.sqrt(n);

        for (int i = step; i > 0; i--) {
            min = Math.min(count(n - i  * i), min);
        }

        return min + 1;

    }

For 3 would have the following recursive calls :

    10 -> 3         
        1 -> 1
           0  

So at that point we could cache `1` with `1` square and `10` with `2` squares.

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

```

What could we cache ? The minimum value at each n. So we know at n it would be 0. At `n-1` it would be `1` and so on.

*Solution 2 (bottom up)*

How can we build a bottom up solution from it ? We start from the end. 

```
    cache[n]   = 0
    cache[n-1] = 1
    cache[n-2] = 2
    cache[n-3] = 3
    cache[n-4] = 1
    ...
    at each i, we iterate from Math.sqrt(i) to 1 and we compute the minimum
```

```java
     public int numSquares(int n) {

        int[] cache = new int[n + 1];
        Arrays.fill(cache, Integer.MAX_VALUE);

        cache[n] = 0;

        int sqrt = (int) Math.sqrt(n);
        
        for (int i = n-1; i >= 0; i--) {

            for (int k = 1; k <= sqrt; k++) {
                if (i + k * k > n) break;
                cache[i] = Math.min(cache[i], cache[i + k * k] + 1);
            }
        }

        return cache[0];
     }

```        


## 139. Word Break


Given a non-empty string s and a dictionary wordDict containing a list of non-empty words, determine if s can be segmented into a space-separated sequence of one or more dictionary words.

    Input: s = "applepenapple", wordDict = ["apple", "pen"]

*Solution*

This sounds like a good candidate for recursion. We try each combination of letters and we see if all the parts are matched by the dictionary.

```java


private boolean verify(String s, int start, int end, Set<String> set) {

    if (start > end) return false;

    if (set.contains(s.substring(start, end+1))) return true;

    for (int i = start; i  < end; i++) {
        if (verify(s, start, i, set) && verify(s, i + 1, end, set)) return true;
    }

    return false;

}

```
 There is one gotcha, in the loop iteration we need to make sure we get the bounds correct. So we use `i < end`. Otherwise if we have used `i <= end` we would have an infinite loop since that case would be equal to the parent method call `boolean verify(String s, int start, int end, Set<String>set)`

Can we do better (without caching) ?

We can notice that the left side will be valid if it is a word in a dictionary. No need to split it since once we get get the first word right, we move the start to the end of that word. So the code becomes much simpler, using just one recursive call.


```java
private boolean verify(String s, int start, Set<String> set) {
    
    if (start == s.length()) return true;

    for (int i = start + 1; i <= s.length(); i++) {
        if (set.contains(s.substring(start, i)) && verify(s, i, set)) return true;
    }
    return false;

}
```

### Can we cache ? 
Yes. We just need to store the start index and it's boolean result, true or false.

### Can we turn it around ? 


s = "catsanddog", wordDict = ["cats", "dog", "sand", "and", "cat"]

```java
    public boolean wordBreak(String s, List<String> wordDict) {

        Set<String> set = new HashSet<>(wordDict);

        boolean[] dp = new boolean[s.length()+1];
        dp[s.length()] = true;
        for (int start = s.length()-1; i >= 0; i--) {
            for (int i = start + 1; i <= s.length(); i++) {
                if (set.contains(s.substring(start, i)) && dp[i]) {
                    dp[start] = true;
                    break;
                }
            }
        }
        return dp[0];

    }
```    



## So how did we reverse it ?

 `if (start == s.length()) return true` became `dp[s.length()] = true`
 

And
```java
for (int i = start + 1; i <= s.length(); i++) {
    if (set.contains(s.substring(start, i)) && verify(s, i, set)) return true;
}
```    

became
```java
for (int i = start + 1; i <= s.length(); i++) {
    if (set.contains(s.substring(start, i)) && dp[i]) {
        dp[start] = true;
        break;
    }
}
```            

## 399. Evaluate Division

Equations are given in the format A / B = k, where A and B are variables represented as strings, and k is a real number (floating point number). Given some queries, return the answers. If the answer does not exist, return -1.0.

    Example:
    Given a / b = 2.0, b / c = 3.0.
    queries are: a / c = ?, b / a = ?, a / e = ?, a / a = ?, x / x = ? .
    return [6.0, 0.5, -1.0, 1.0, -1.0 ].

*Solution* 

We need to find a link between a and c. The link can be in form `a/b` or  b/a. So it would be easier to canonize it somehow, sorting them so the first letter is always smaller.

So we would have `a/b, b/c, c/d` If we would like to find `a\d` we navigate from `a` until we reach `d` and we keep the product. We could even cache, each product for example `a/c = 6` so we would have reuse it if needed. 

### How would we cache it ? 
We can't use a map with a `a/c` key because we would like to iterate all the elements that start with `a`.

So it's better to use a two dimensional array.

ON the other hand there is a hint in the problem's body that suggests constructin the cache first. In fact this seems very intuitive. 



### 494. Target Sum

 You are given a list of non-negative integers, a1, a2, ..., an, and a target, S. Now you have 2 symbols + and -. For each integer, you should choose one from + and - as its new symbol.

Find out how many ways to assign symbols to make sum of integers equal to target S. 

    Input: nums is [1, 1, 1, 1, 1], S is 3. 
    Output: 5
    Explanation: 

    -1+1+1+1+1 = 3
    +1-1+1+1+1 = 3
    +1+1-1+1+1 = 3
    +1+1+1-1+1 = 3
    +1+1+1+1-1 = 3

*Solution 1*

recursive with memoizatoin, not much to do here, at each element we choose to add + or -, meaning that we add or substract the current element to the target sum. We return the sum of those two ways.

*Solution 2: Bottom-up*

This is more interesting, we can't go from one end to the other cause at some point we need to know the current value + x and the current value - x. That means we should start at 0. We could use a hashmap for indices or an array from 0 to 2 * sum, where sum doesn't exceeed 1000 and we consider sum to be the middle of the array. 

For clarity purposes let's say the array is between 0 and 2000 so we put 1 and the middle  (1000) and we start from there. If the last element is 1, we will have a way to do the -1 from 999 and a way to do 1 from 1001.

We keep a bidimensional array cause the results from the previous index are not valid at the current index.


*Solution 3: Subsets*

We can think about the problem, differently. Let's start with an example

    [4, 1, 3, 2]  4

The sum is 6, so we need to find 2 subsets such as the difference between them is 4.

    a + b = 10
    a - b = 4
    -----------
    2 * a = 14, a = 7, b = 3
    
So we need to find the numbers of way we can make 5.

This can be reduced at a smaller problem. We can do it recursively, at each point we have the option to use a number or not to use it.

So there will be the following calls

    ways(nums, 0, 7)  
        ways(nums, 1,  3)  
            ways(num, 2, 2) 
                ways(nums, 3, -1)
                    ways(nums, 4, -3)
                    ways(nums, 4, -1)
                ways(nums, 3, 2)  
                    ways(nums, 4, 0) // used 4, 1, 2
                    ways(nums, 4, 2)
            ways(nums, 2, 3)
                ways(nums, 3, 0) 
                    ways(nums, 4, -2) 
                    ways(nums, 4, 0) // used 4, 3
                ways(nums, 3, 3) 
                    ways(nums, 4, 1) 
                    ways(nums, 4, 3) 

        ways(nums, 1,  7)
            ways(nums, 2, 6)  
                ways(nums, 3, 3)
                    ways(nums, 4, 1)
                    ways(nums, 4, 3)
                ways(nums, 3, 6) 
                    ways(nums, 4, 4)
                    ways(nums, 4, 6)
            ways(nums, 2,  7)
                ways(nums, 3, 4)
                    ways(nums, 4, 2) 
                    ways(nums, 4, 4) 
                ways(nums, 3, 7) 
                    ways(nums, 4, 5) 
                    ways(nums, 4, 7) 

 Looks like there are 2 ways. Rows is equal to the nums indices and columns are the target. We start with `(0,0) = 1`. Then there is a possibility to make target `1` with the first number, 2 posibilities to make `3` with numbers 1 to 3,

    [4, 1, 3, 2]

      0 1 2 3 4 5 6 7
    0 1 0 0 0 0 0 0 0
    4 1 0 0 0 1 0 0 0
    1 1 1 0 0 1 1 0 0
    3 1 1 0 1 2 1 0 1 
    2 1 1 1 2 2 2 2 2

This uses a 2D matrix. 

With a 4 we can make the target 4.
With a 4 and a 1, we can make the target 1, 4 and 5 
With 4,1,3, we can make 1, 3, 4 (x2) ,5 and 7
With 4,1,3,2 we can make 1,2, (3x2), (4x2), (5x2), (6x2), (7x2)


We can change it to use a single one, going from the back to front and updating the index at the right with the index to the left which was not processed yet

      0 1 2 3 4 5 6 7
    0 0 0 0 0 0 0 0 1
    4 0 0 0 1 0 0 0 1
    1 0 0 1 1 0 0 1 1
    3 1 0 1 2 1 0 1 1 
    2 2 2 2 2 2 1 1 1


### 309. Best Time to Buy and Sell Stock with Cooldown

Say you have an array for which the ith element is the price of a given stock on day i.

Design an algorithm to find the maximum profit. You may complete as many transactions as you like (ie, buy one and sell one share of the stock multiple times) with the following restrictions:

* You may not engage in multiple transactions at the same time (ie, you must sell the stock before you buy again).
 * After you sell your stock, you cannot buy stock on next day. (ie, cooldown 1 day)

Example:

    Input: [1,2,3,0,2]
    Output: 3 
    Explanation: transactions = [buy, sell, cooldown, buy, sell]

*Solution 1: Top Down*

At each point we have three options, buy, sell or wait.

The restrictions are: we can't sell without a buy, we can't buy after a sell. So we can pass 2 parameters hasSold and hasBought.

If we have just sold we must wait
If we bought, we can sell or wait.
If we neither just sold nor bought, we can wait or we can buy.


If we sell at a point, we return all the profit made so far + nums[i] 

If we buy at a point we return the profit made so far - nums[i]. 

If we wait, we don't do nothing. 

If we want to cache, we can't just use the index, we must use index + hasBought + hasJustSold.


Alternative: 
Another approach would be to send the information from the bottom up and construct the result. Although this has better performance, it complicates the problem a little bit cause we have to keep track of an additional variable:

```java

    static class Result {
        int sold;
        int buy;
        int waitWithStock;
        int waitWithoutStock;
    }

```

The logic itself is more complicated

```java

    private Result profit(int[] prices, int i) {

        if (i == prices.length) {
            return new Result();
        }

        Result next = profit(prices, i + 1);

        Result result = new Result();

        result.sold = Math.max(next.waitWithoutStock + prices[i], next.sold);
        result.buy = Math.max(next.waitWithoutStock, Math.max(next.buy, next.sold)) - prices[i];
        result.waitWithStock = Math.max(next.waitWithStock, next.sold);
        result.waitWithoutStock = Math.max(next.waitWithoutStock, next.buy);

        return result;
    }
```


* Solution 3*

Let's try a bottom up approach using the first alternative.

We initialize a two dimensional array, for buy and sell. This array represents the biggest profit in any of the 2 situations, with and without stock. Bought holds the money when we have stock, sold money without stock. We compute each one using another, and at the end we will look at length of sell.

    [1,2,3,0,2]

At the first position. We can only buy. Second position we can either buy, sell or wait. If we wait things don't change. 

 `bought[i] = Math.max(bought[i-1], sold[i-2] - price[i])`

 `sold[i] = Math.max(sold[i-1], bought[i-1] + price[i])`

             1  2  3  0  2
    bought: -1 -1 -1  1  1
    sold:    0  1  2  2  3



### 300. Longest Increasing Subsequence
Given an unsorted array of integers, find the length of longest increasing subsequence.

    Input: [10,9,2,5,3,7,101,18]
    Output: 4 
Explanation: The longest increasing subsequence is [2,3,7,101], therefore the length is 4. 

*Note* I fell twice in trap of thinking I can do it in O(n), using a greedy algorithm, taking the max element so far and finding the minimum ones.

*Solution 1 Recursivity, top-down* 

We can try a recursive solution, for each element we have the option of using it (provided it is bigger than the previous) or not. The problem is that we don't use it we need to pass forward the previous element that we used, kind of cumbersome. When we cache we cache the element and the previous. This is more difficult to do if you want to reverse it in a bottom-up approach.

*Solution 2 Recursivity, top-down* 

Usually in this kind of problems, increasing sequences we should try starting from the end, that way we don't need to pass the extra parameter.

So at the last element we iterate through all the previous elements. If the previous element is smaller then the current element we add 1 to its maximum, otherwise we just copy it.

*Solution 3 - Bottom up* 

The previous solution is easier to turn around in a bottom-up approach. We just keep a unidimensional array and we compute the maximums of the previous elements + 1  if appropriate.

    [ 10,  9,  2,  5,  3,  7,101, 18]
    [  1,  1,  1,  2,  2,  3,  4,  4]


