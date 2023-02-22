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

1. Breadth First Search (BFS)
2. Depth First Search (DFS)
3. Shortest Path from source to all vertices **Dijkstra**
4. Shortest Path from every vertex to every other vertex **Floyd Warshall**
5. To detect cycle in a Graph **Union Find**
6. Minimum Spanning tree **Prim**
7. Minimum Spanning tree **Kruskal**
8. Topological Sort
9. Boggle (Find all possible words in a board of characters)
10. Bridges in a Graph

Balanced Binary Tree - Leetcode 110 - Python
Microsoft's Most Asked Question 2021 - Count Good Nodes in a Binary Tree - Leetcode 1448 - Python
Invert Binary Tree - Depth First Search - Leetcode 226
Merge Two Binary Trees - Leetcode 617
Convert Sorted Array to Binary Search Tree - Leetcode 108 - Python
Validate Binary Search Tree - Depth First Search - Leetcode 98
Triangle - Dynamic Programming made Easy - Leetcode 120
Sum Root to Leaf Numbers - Coding Interview Question - Leetcode 129
Unique Binary Search Trees - Leetcode 96 - Python Dynamic Programming
Binary Tree Right Side View - Breadth First Search - Leetcode 199
Kth Smallest Element in a BST
Diameter of a Binary Tree - Leetcode 543 - Python
House Robber III - Tree - Leetcode 337
Binary Tree Level Order Traversal - BFS - Leetcode 102
Construct Binary Tree from Inorder and Preorder Traversal - Leetcode 105 - Python
Implement Trie (Prefix Tree) - Leetcode 208
Same Tree - Leetcode 100 - Python
Serialize and Deserialize Binary Tree - Preorder Traversal - Leetcode 297 - Python
Lowest Common Ancestor of a Binary Search Tree - Leetcode 235 - Python
Design Add and Search Words Data Structure - Leetcode 211 - Python
Binary Tree Maximum Path Sum - DFS - Leetcode 124 - Python
Maximum Depth of Binary Tree - 3 Solutions - Leetcode 104 - Python
Flip Equivalent Binary Trees - Leetcode 951 - Python
Operations on Tree - Leetcode Biweekly Contest - 1993 - Python
Flatten Binary Tree to Linked List - Leetcode 114 - Python
All Possible Full Binary Trees - Memoization - Leetcode 894 - Python
Subtree of Another Tree - Leetcode 572 - Python
Find Bottom Left Tree Value - Leetcode 513 - Python
Trim a Binary Search Tree - Leetcode 669 - Python
Path Sum - Leetcode 112 - Python
Word Search II - Backtracking Trie - Leetcode 212 - Python
Iterative & Recursive - Binary Tree Inorder Traversal - Leetcode 94 - Python
Binary Search Tree Iterator - Leetcode 173 - Python
Populating Next Right Pointers in Each Node - Leetcode 116 - Python
Convert BST to Greater Tree - Leetcode 538 - Python
Construct String from Binary Tree - Leetcode 606 - Python

## Graph
- [Graph playlist](https://www.youtube.com/watch?v=EgI5nU9etnU&list=PLot-Xpze53ldBT_7QA8NVot219jFNr_GI&index=1&t=0s)

1. Union-Find
2. Topological Sort
3. Dijkstra's Algo

Course Schedule - Graph Adjacency List - Leetcode 207
Course Schedule II - Topological Sort - Leetcode 210
NUMBER OF ISLANDS - Leetcode 200 - Python
Clone Graph - Depth First Search - Leetcode 133
Word Ladder - Breadth First Search - Leetcode 127 - Python
Pacific Atlantic Water Flow - Leetcode 417 - Python
Network Delay Time - Dijkstra's algorithm - Leetcode 743
Word Search - Backtracking - Leetcode 79 - Python
Island Perimeter - Graph - Leetcode 463
Leetcode 1466 - REORDER ROUTES TO MAKE ALL PATHS LEAD TO THE CITY ZERO
Redundant Connection - Union Find - Leetcode 684 - Python
Graph Valid Tree - Leetcode 261 - Python
Word Search II - Backtracking Trie - Leetcode 212 - Python
Number of Connected Components in an Undirected Graph - Union Find - Leetcode 323
Alien Dictionary - Topological Sort - Leetcode 269 - Python
Prim's Algorithm - Minimum Spanning Tree - Min Cost to Connect all Points - Leetcode 1584
Count Sub Islands - DFS - Leetcode 1905 - Python
Swim in Rising Water - Dijkstra's Algorithm - Leetcode 778
Walls and Gates - Multi-Source BFS - Leetcode 286 
Surrounded Regions - Graph - Leetcode 130 - Python
Bellman-Ford - Cheapest Flights within K Stops - Leetcode 787
Max Area of Island - Leetcode 695
Reconstruct Itinerary - Leetcode 332 - 
Rotting Oranges - Leetcode 994 - Python
Longest Increasing Path in a Matrix - Leetcode 329
Snakes and Ladders - Leetcode 909 - Python

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

