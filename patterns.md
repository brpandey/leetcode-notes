## Heap


## Sliding Window
[Playlist](https://www.youtube.com/watch?v=1pkOgXD63yU&list=PLot-Xpze53leOBgcVsJBEGrHPd_7x_koV)

Topics
- Choose two pointers or two values

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

- [Best Time to Buy and Sell a Stock](https://github.com/brpandey/leetcode/blob/master/rust/src/p0121_best_time_to_buy_sell.rs)
- Longest Substring Without Repeating Characters - Leetcode 3
- Minimum Window Substring - Airbnb Interview Question - Leetcode 76
- Longest Repeating Character Replacement
- Find All Anagrams in a String - Leetcode 438
- Sliding Window Maximum - Monotonic Queue - Leetcode 239
- Permutation in String - Leetcode 567 -
- Minimum Size Subarray Sum - Leetcode 209
- Repeated DNA Sequences - Leetcode 187

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

## Trees
- [Tree playlist](https://www.youtube.com/watch?v=OnSn2XEQ4MY&list=PLot-Xpze53ldg4pN6PfzoJY7KsKcxF1jg&index=2&t=0s)

Topics
- Breadth First Search (BFS)
- Depth First Search (DFS)

Leetcode
- Invert Binary Tree - Depth First Search - 226 [Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0226_invert_binary_tree.rs)
- Merge Two Binary Trees - Leetcode 617
- Convert Sorted Array to Binary Search Tree - 108 - [Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0108_convert_sorted_array_to_bst.rs)
- Validate Binary Search Tree - DFS - Leetcode 98 [Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0098_validate_binary_search_tree.rs)
- Triangle - Dynamic Programming made Easy - Leetcode 120
- Sum Root to Leaf Numbers - Coding Interview Question - Leetcode 129
- Unique Binary Search Trees - Leetcode 96 - Dynamic Programming
- Binary Tree Right Side View - Breadth First Search - Leetcode 199
- Kth Smallest Element in a BST - 230 - [Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0230_kth_smallest_element_in_bst.rs)
- Diameter of a Binary Tree - Leetcode 543
- House Robber III - Tree - Leetcode 337
- Binary Tree Level Order Traversal - BFS - Leetcode 102 [Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0102_binary_tree_level_order_traversal.rs)
- Construct Binary Tree from Inorder and Preorder Traversal - Leetcode 105 [Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0105_construct_binary_tree_from_preorder_and_inorder_traversal.rs)
- Implement Trie (Prefix Tree) - Leetcode 208 [Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0208_implement_trie.rs)
- Same Tree - Leetcode 100 - [Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0100_same_tree.rs)
- Serialize and Deserialize Binary Tree - Preorder Traversal - Leetcode 297 [Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0297_serialize_and_deserialize_binary_tree.rs)
- Lowest Common Ancestor of a Binary Search Tree - Leetcode 235 [Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0235_lowest_common_ancestor_of_a_binary_tree.rs)
- Design Add and Search Words Data Structure - Leetcode 211 [Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0211_add_and_search_word.rs)
- Binary Tree Maximum Path Sum - DFS - Leetcode 124 [Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0124_binary_tree_max_path_sum.rs)
- Maximum Depth of Binary Tree - 3 Solutions - Leetcode 104 [Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0104_maximum_depth_of_binary_tree.rs)
- Flip Equivalent Binary Trees - Leetcode 951
- Flatten Binary Tree to Linked List - Leetcode 114 
- All Possible Full Binary Trees - Memoization - Leetcode 894
- Subtree of Another Tree - Leetcode 572 - [Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0572_subtree_of_another_tree.rs)
- Find Bottom Left Tree Value - Leetcode 513
- Trim a Binary Search Tree - Leetcode 669
- Path Sum - Leetcode 112
- Word Search II - Backtracking Trie - Leetcode 212 - [Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0212_word_search_ii.rs)
- Iterative & Recursive - Binary Tree Inorder Traversal - Leetcode 94
- Binary Search Tree Iterator - Leetcode 173
- Populating Next Right Pointers in Each Node - Leetcode 116
- Convert BST to Greater Tree - Leetcode 538
- Construct String from Binary Tree - Leetcode 606

## Graph
- [Graph playlist](https://www.youtube.com/watch?v=EgI5nU9etnU&list=PLot-Xpze53ldBT_7QA8NVot219jFNr_GI&index=1&t=0s)

Topics
- Shortest Path from source to all vertices **Dijkstra**
- Shortest Path from every vertex to every other vertex **Floyd Warshall**
- To detect cycle in a Graph **Union Find**
- Minimum Spanning tree **Prim**
- Minimum Spanning tree **Kruskal**
- Topological Sort
- Boggle (Find all possible words in a board of characters)
- Bridges in a Graph

Leetcode
- Course Schedule - Graph Adjacency List - Leetcode 207 [Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0207_course_schedule.rs)
- Course Schedule II - Topological Sort - Leetcode 210 [Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0210_course_schedule_ii.rs)
- NUMBER OF ISLANDS - Leetcode 200 - [Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0200_number_of_islands.rs)
- Clone Graph - Depth First Search - Leetcode 133 - [Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0133_clone_graph.rs)
- Word Ladder - Breadth First Search - Leetcode 127 - [Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0127_word_ladder.rs)
- Pacific Atlantic Water Flow - Leetcode 417 - [Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0417_pacific_atlantic_water_flow.rs)
- Network Delay Time - Dijkstra's algorithm - Leetcode 743 [Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0743_network_delay_time.rs)
- Word Search - Backtracking - Leetcode 79 [Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0079_word_search.rs)
- Island Perimeter - Graph - Leetcode 463 
- Redundant Connection - Union Find - Leetcode 684 - Python
- Graph Valid Tree - Leetcode 261 - [Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0261_graph_valid_tree.rs)
- Word Search II - Backtracking Trie - Leetcode 212 - [Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0212_word_search_ii.rs)
- Number of Connected Components in an Undirected Graph - Union Find - Leetcode 323 - [Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0323_number_of_connected_components_in_undirected_graph.rs)
- Alien Dictionary - Topological Sort - Leetcode 269 - [Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0269_alien_dictionary.rs)
- Prim's Algorithm - Minimum Spanning Tree - Min Cost to Connect all Points - Leetcode 1584
- Count Sub Islands - DFS - Leetcode 1905 - Python
- Swim in Rising Water - Dijkstra's Algorithm - Leetcode 778
- Walls and Gates - Multi-Source BFS - Leetcode 286 
- Surrounded Regions - Graph - Leetcode 130
- Bellman-Ford - Cheapest Flights within K Stops - Leetcode 787 - [Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0787_cheapest_flights_within_k_stops.rs)
- Max Area of Island - Leetcode 695 
- Reconstruct Itinerary - Leetcode 332 - 
- Rotting Oranges - Leetcode 994 - Python
- Longest Increasing Path in a Matrix - Leetcode 329
- Snakes and Ladders - Leetcode 909 - Python

## Recursion
[Backtracking playlist](https://www.youtube.com/watch?v=pfiQ_PS1g8E&list=PLot-Xpze53lf5C3HSjCnyFghlW0G1HHXo)

- Word Search - Backtracking - Leetcode 79 - [Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0079_word_search.rs)
- Permutations - Leetcode 46 - [Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0046_permutations.rs)
- Permutations II - Backtracking - Leetcode 47
- Subsets - Backtracking - Leetcode 78 - [Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0078_subsets.rs)
- Palindrome Partitioning - Backtracking - Leetcode 131 
- Letter Combinations of a Phone Number - Backtracking - Leetcode 17 - [Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0017_letter_comb_phone.rs)
- Combination Sum - Backtracking - Leetcode 39 - [Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0039_comb_sum_i.rs)
- Word Search II - Backtracking Trie - Leetcode 212 - [Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0212_word_search_ii.rs)
- N-Queens - Backtracking - Leetcode 51 - [Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0051_n_queens.rs)
- Combination Sum II - Backtracking - Leetcode 40 - [Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0040_combination_sum_ii.rs)
- Combinations - Leetcode 77
- Partition to K Equal Sum Subsets


## Dynamic Programming
[DP playlist](https://www.youtube.com/watch?v=g0npyaQtAQM&list=PLot-Xpze53lcvx_tjrr_m2lgD2NsRHlNO)

1. Fibonacci Numbers
- Climbing Stairs [Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0070_climbing_stairs.rs)
- House Robber	[Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0198_house_robber.rs)
- Fibonacci Number [Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0509_fibonacci_number.rs)
- Maximum Alternating Subsequence Sum
		
2. Zero / One Knapsack		
- Partition Equal Subset Sum [Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0416_partition_equal_subset_sum.rs)
- Target Sum	[Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0494_target_sum.rs)
		
3. Unbounded Knapsack		
- Coin Change	[Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0322_coin_change.rs)
- Coin Change II	[Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0518_coin_change_ii.rs)
- Minimum Cost for Tickets
		
4. Longest Common Subsequence		
- Longest Common Subsequence	[Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p1143_longest_common_subsequence.rs)
- Longest Increasing Subsequence	[Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0300_longest_increasing_subsequence.rs)
- Edit Distance	
- Distinct Subsequences	[Video](https://youtu.be/-RDzMJ33nx8)	[Problem](https://leetcode.com/problems/distinct-subsequences/)
		
5. Palindromes		
- Longest Palindromic Substring	[Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0005_longest_palindrome_substring.rs)
- Palindromic Substrings	[Code](https://github.com/brpandey/leetcode/blob/master/rust/src/p0647_palindromic_substrings.rs)
- Longest Palindromic Subsequence		
