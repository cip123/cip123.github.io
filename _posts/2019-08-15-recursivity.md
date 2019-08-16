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
