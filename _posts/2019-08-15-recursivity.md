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


## 322. Coin Change

You are given coins of different denominations and a total amount of money amount. Write a function to compute the fewest number of coins that you need to make up that amount. If that amount of money cannot be made up by any combination of the coins, return -1.

Example 1:

    Input: coins = [1, 2, 5], amount = 11
    Output: 3 
    Explanation: 11 = 5 + 5 + 1

Example 2:

    Input: coins = [2], amount = 3
    Output: -1


*Solution 1 (recursive: top down)*

At each coin index we have the option of using that coin or not use it.

```java

    private int minCoins(int[] coins, int i, int amount, Integer[][] cache) {

        if (amount == 0) return 0;
        
        if (amount < 0 || i == coins.length) {
            return -1;
        }

        if (cache[i][amount] != null) {
            return cache[i][amount];
        }

        int with = minCoins(coins, i, amount-coins[i], cache);

        if (with  >= 0) {
            with++;
        }

        int without = minCoins(coins, i +1, amount, cache);

        int result;

        if (with > 0 && without > 0) {
            result =  Math.min(with, without);
        } else if (with < 0) {
            result = without;
        } else {
            result = with;
        }

        cache[i][amount] = result;

        return result;
    }

```

This takes 35 ms on leet code, probably because of the strack depth and we use a bidimensional Integer array. There is one trick, we should always consider -1 as an invalid value, so we should not take into cosideration when adding 1.

*Solution 2 (recursive: top-down)*

Another aproach is to avoid passing the coin index at every strack trace, so for each amount we compute all the coin combinations. The algoritm would look like this:

```java

 private int minCoins(int[] coins, int amount, Integer[] cache) {

        if (amount < 0) {
            return -1;
        } else if (amount == 0) {
            return 0;
        }

        if (cache[amount] != null) return cache[amount];

        int min = Integer.MAX_VALUE;

        for (int coin : coins) {
            int current = minCoins(coins, amount -coin, cache);
            if (current != -1) {
                min = Math.min(min, current+1);
            }
        }

        int result = min == Integer.MAX_VALUE ? -1 : min;
        cache[amount] = result;


        return result;

    }


```

*Solution 3 (bottom-up for solution 1)*

For every coin index we iterate the amount, computing the minimum coins up to that index that we can use to make that amount.

```java
    public int coinChange(int[] coins, int amount) {
        int[] cache = new int[amount+1];
        Arrays.fill(cache, -1);
        cache[0] = 0;

        for (int i = 0; i < coins.length; i++) {
            for (int j = 1; j <= amount; j++) {

                int min = Integer.MAX_VALUE;

                int without = cache[j];

                if (without != -1) {
                    min = Math.min(without, min);
                }

                if (coins[i] <= j) {

                    int with = cache[j - coins[i]];

                    if (with != -1) {

                        min = Math.min(with + 1, min) ;
                    }
                }

                cache[j] = min == Integer.MAX_VALUE ? -1 :min;
            }

        }
        return cache[amount];
    }

```

*Solution 4 (bottom-up for solution 2)*

The array cache length is the same. For each amount we compute the minimum we can make using all the coins.

```java
    public int coinChange(int[] coins, int amount) {
        int[] cache = new int[amount +1];
        Arrays.fill(cache, -1);
        cache[0] = 0;

        for (int i = 1; i<= amount; i++) {

            int min = Integer.MAX_VALUE;

            for (int j = 0; j < coins.length; j++) {
                if (coins[j] <= i) {
                    if (cache[i-coins[j]] != -1) {
                        min = Math.min(cache[i - coins[j]] + 1, min);
                    }
                }
            }

            if (min == Integer.MAX_VALUE) {
                min = -1;
            }

            if (cache[i] == -1) {
                cache[i] = min;
            } else {
                cache[i] = Math.min(cache[i], min);
            }

        }

        return cache[amount];    
    }
```

Another aproach would be instead of using -1 an an invalid value, to use a large value that could not be obtained (eg. amount + 1). The benefit is that we won't need to check for -1 every time.

```java
    public int coinChange(int[] coins, int amount) {
        int[] cache = new int[amount +1];
        Arrays.fill(cache, amount + 1);
        cache[0] = 0;

        for (int i = 1; i<= amount; i++) {

            for (int j = 0; j < coins.length; j++) {

                if (coins[j] <= i) {
                    cache[i] = Math.min(cache[i - coins[j]] + 1, cache[i]);
                }
            }
        }

        return cache[amount] > amount ? -1 : cache[amount];    
    }
```

## 31. Palindrome Partitioning

Given a string s, partition s such that every substring of the partition is a palindrome.

Return all possible palindrome partitioning of s.

Example:

    Input: "aab"
    Output:
    [
        ["aa","b"],
        ["a","a","b"]
    ]


*Solution *

There are two things that I have learned from this algorithm. 

* When we are required to pass along a list of results, we don't need to clone it in order to pass it around for next step. We just add it before the recursive call and we remove it right afterwards. First recursive call goes until the end, then the second one and so one. They don't go in parallel :D

* We can use the bottom up algorithm to compute the palindrome subsequences of a string. Let's take the following string as an example:
 `a b b a c d`


We create a bidimensional matrix, rows meaning the start and columns meaning the end (start can be less than the end).



      a b b a c d
    a 1 0 0 1 0 0
    b   1 1 0 0 0
    b     1 0 0 0
    a       1 0 0
    c         1 0
    d           1



For each element and each length we compute if it's a palindromic sequence.


* if `l == 1`; `start == end` and the value will aways be a palindrome
* if `l == 2`, `end = start +1`  and it is a palindrome if `start == end`.
* if `l > 2` in addition to start being equal to end we need to see that `start+1, end-1` is also a palindrome.



## 312. Burst Balloons

Given n balloons, indexed from 0 to n-1. Each balloon is painted with a number on it represented by array nums. You are asked to burst all the balloons. If the you burst balloon i you will get nums[left] * nums[i] * nums[right] coins. Here left and right are adjacent indices of i. After the burst, the left and right then becomes adjacent.

Find the maximum coins you can collect by bursting the balloons wisely.

Note:

    You may imagine nums[-1] = nums[n] = 1. They are not real therefore you can not burst them.
    0 ≤ n ≤ 500, 0 ≤ nums[i] ≤ 100

Example:

    Input: [3,1,5,8]
    Output: 167 

Explanation: 
    
    nums = [3,1,5,8] --> [3,5,8] -->   [3,8]   -->  [8]  --> []
             coins =  3*1*5      +  3*5*8    +  1*3*8      + 1*8*1   = 167


*Solution 1 (first try)*

I tried the recurive variant where I used an array list and as the problem explained I removed an element from it and pass the array later on. I got a time limit exceeded.

The problem with this approach is that I could not cache anything. 
Looking at the example `[3, 1, 5, 8]`  we realize that we could cache the maximum profit we can make before two indices, `left` and `right`.  So we can start with the smallest difference between indices, which is 2 and build from there. 


I tried to use a approach where once i check an index I set it to -1 in the array and then iterate through the rest to find a valid pair but this is not the right approach. It is better to use divide and conquer.


Here is an example for the following baloons:

    0 1 2 3 4 5
    -----------
    1 3 1 5 8 1


We start with `left=0, right = 5`. We add for example `i=4`. So the problem becomes `left=0, right=4` + `left=4, right=5 + baloons[left] + baloons[right] + baloons[i]`.

We can buld a top-down and bottom up approch, just be careful at the bottom up approach we have to start with the minimum distance between left and right, which is 2 ( and start from there).




## 140. Word Break II

Given a non-empty string s and a dictionary wordDict containing a list of non-empty words, add spaces in s to construct a sentence where each word is a valid dictionary word. Return all such possible sentences.

Note:

    The same word in the dictionary may be reused multiple times in the segmentation.
    You may assume the dictionary does not contain duplicate words.

Example 1:

Input: 
    
    s = "catsanddog"

    wordDict = ["cat", "cats", "and", "sand", "dog"]

Output:
    [
    "cats and dog",
    "cat sand dog"
    ]

*Solution 1*

I have tried a trie approach where I iterate one by one and check if at the current index we have a valid word and if so I reset the trie.
 
 *Solution 2*


This is unnecessary though, because a trie is used when we need to search a character in multiple words. For this case we can just iterate from start to end and see if the current word is in the dictionary. If it is we do the recursive call with the end index at the start index.


## 44. Wildcard Matching

Given an input string (s) and a pattern (p), implement wildcard pattern matching with support for '?' and '*'.

'?' Matches any single character.
'*' Matches any sequence of characters (including the empty sequence).

*Solution*

The tricky thing here is that when the string is empty the pattern can only consist of `*` characters.

Other than that the problem can be solved recursively using top down or bottom up.


For a regular digit or the '?' sign the problem is straight forward. For the `*` sign though, 
we have to look at several previous solutions: we have the option of not using it and then we just advance the pattern index, or the option of using it, advancing the string index and the pattern index.











