## Heap


## Sliding Window
[Playlist](https://www.youtube.com/watch?v=1pkOgXD63yU&list=PLot-Xpze53leOBgcVsJBEGrHPd_7x_koV)

```rust
pub fn best_time_to_buy_sell(nums: &[i32]) -> i32 {
        if nums.len() < 1 { return 0 }
        let (mut sell, mut min_buy);
        let mut profit_max = 0;

        min_buy = nums[0];

        // 7,1,5,3,6,4, first iter sell = 1, min_buy 7, second iter sell = 5, min_buy 1
        for i in 1..nums.len() {
            sell = nums[i]; // enumerate through the list with sell being current element
            profit_max = cmp::max(sell - min_buy, profit_max); // keep highest
            min_buy = cmp::min(sell, min_buy) // keep lowest buy stock
        }

        profit_max
    }
```

- Strategy: Choose two pointers or two values 
- Best Time to Buy and Sell a Stock [Video](https://www.youtube.com/watch?v=1pkOgXD63yU) [Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0121_best_time_to_buy_sell.rs)

- Sliding Window: Best Time to Buy and Sell Stock - Leetcode 121
- Longest Substring Without Repeating Characters - Leetcode 3
- Minimum Window Substring - Airbnb Interview Question - Leetcode 76
- Longest Repeating Character Replacement
- Find All Anagrams in a String - Leetcode 438
- Sliding Window Maximum - Monotonic Queue - Leetcode 239
- Permutation in String - Leetcode 567 -
- Minimum Size Subarray Sum - Leetcode 209 - Pyt
- Repeated DNA Sequences - Leetcode 187 - Python

## Binary Search
[Playlist](https://www.youtube.com/playlist?list=PLot-Xpze53leNZQd0iINpD-MAhMOMzWvO)

```rust

    pub fn search(nums: Vec<i32>, target: i32) -> i32 {
        let mut pivot;

        let mut lo = 0;
        let mut hi = nums.len() - 1;  // 5

        while lo < hi {

            pivot = lo + (hi - lo) / 2; // 2.5 or 2

            if nums[pivot] == target {
                return pivot as i32
            }

            if target < nums[pivot] {
                hi = pivot - 1
            } else {
                lo = pivot + 1
            }
        }

        return -1
    }
 ```

- [Search in rotated array](https://github.com/brpandey/leetcode/blob/master/rust/src/p0033_search_in_rotated_sorted_array.rs)
- [Binary search](https://github.com/brpandey/leetcode/blob/master/rust/src/p0704_binary_search.rs)
- [Time based key value store](https://github.com/brpandey/leetcode/blob/master/rust/src/p0981_time_based_key_value_store.rs)
- [Koko eating bananas](https://github.com/brpandey/leetcode/blob/master/rust/src/p0875_koko_eating_bananas.rs)
- [Median two sorted arrays](https://github.com/brpandey/leetcode/blob/master/rust/src/p0004_median_two_sorted_arrays.rs)

## DFS & BFS
- [Tree playlist](https://www.youtube.com/watch?v=OnSn2XEQ4MY&list=PLot-Xpze53ldg4pN6PfzoJY7KsKcxF1jg&index=2&t=0s)

## Graph
- [Graph playlist](https://www.youtube.com/watch?v=EgI5nU9etnU&list=PLot-Xpze53ldBT_7QA8NVot219jFNr_GI&index=1&t=0s)

1. Union-Find
2. Topological Sort
3. Dijkstra's Algo

## Recursion
[Backtracking playlist](https://www.youtube.com/watch?v=pfiQ_PS1g8E&list=PLot-Xpze53lf5C3HSjCnyFghlW0G1HHXo)

- Word Search - Backtracking - Leetcode 79
- Backtracking: Permutations - Leetcode 46
- Permutations II - Backtracking - Leetcode 47
- Subsets - Backtracking - Leetcode 78
- Palindrome Partitioning - Backtracking - Leetcode 131
- Letter Combinations of a Phone Number - Backtracking - Leetcode 17
- Combination Sum - Backtracking - Leetcode 39
- Word Search II - Backtracking Trie - Leetcode 212
- N-Queens - Backtracking - Leetcode 51
- Combination Sum II - Backtracking - Leetcode 40
- Combinations - Leetcode 77
- Partition to K Equal Sum Subsets

## HashMap



## Dynamic Programming
[DP playlist](https://www.youtube.com/watch?v=g0npyaQtAQM&list=PLot-Xpze53lcvx_tjrr_m2lgD2NsRHlNO)

1. Fibonacci Numbers	Solution Link	Problem Link
- Climbing Stairs	[Video](https://youtu.be/Y0lT9Fck7qI)	[Problem](https://leetcode.com/problems/climbing-stairs/) [Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0070_climbing_stairs.rs)
- House Robber	[Video](https://youtu.be/73r3KWiEvyk)	[Problem](https://leetcode.com/problems/house-robber/)
- Fibonacci Number		[Problem](https://leetcode.com/problems/fibonacci-number/)
- Maximum Alternating Subsequence Sum	[Video](https://youtu.be/4v42XOuU1XA)	[Problem](https://leetcode.com/problems/maximum-alternating-subsequence-sum/)
		
2. Zero / One Knapsack		
- Partition Equal Subset Sum	[Video](https://youtu.be/IsvocB5BJhw)	[Problem](https://leetcode.com/problems/partition-equal-subset-sum/)
- Target Sum	[Video](https://www.youtube.com/watch?v=g0npyaQtAQM)	[Problem](https://leetcode.com/problems/target-sum/)
		
3. Unbounded Knapsack		
- Coin Change	[Video](https://youtu.be/H9bfqozjoqs)	[Problem](https://leetcode.com/problems/coin-change/)
- Coin Change II	[Video](https://www.youtube.com/watch?v=Mjy4hd2xgrs)	[Problem](https://leetcode.com/problems/coin-change-2/)
- Minimum Cost for Tickets	[Video](https://www.youtube.com/watch?v=4pY1bsBpIY4)	[Problem](https://leetcode.com/problems/minimum-cost-for-tickets/)
		
4. Longest Common Subsequence		
- Longest Common Subsequence	[Video](https://youtu.be/Ua0GhsJSlWM)	[Problem](https://leetcode.com/problems/longest-common-subsequence/)
- Longest Increasing Subsequence	[Video](https://youtu.be/cjWnW0hdF1Y)	[Problem](https://leetcode.com/problems/longest-increasing-subsequence/)
- Edit Distance	[Video](https://youtu.be/XYi2-LPrwm4)	[Problem](https://leetcode.com/problems/edit-distance/)
- Distinct Subsequences	[Video](https://youtu.be/-RDzMJ33nx8)	[Problem](https://leetcode.com/problems/distinct-subsequences/)
		
5. Palindromes		
- Longest Palindromic Substring	[Video](https://youtu.be/XYQecbcd6_c)	[Problem](https://leetcode.com/problems/longest-palindromic-substring)
- Palindromic Substrings	[Video](https://youtu.be/4RACzI5-du8)	[Problem](https://leetcode.com/problems/palindromic-substrings/)
- Longest Palindromic Subsequence		[Problem](https://leetcode.com/problems/longest-palindromic-subsequence/)

