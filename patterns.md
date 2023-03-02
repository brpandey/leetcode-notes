## Patterns

### A) Arrays & Hashing

### B) Two Pointers

### C) Stack

### D) Binary Search 
> [Playlist](https://www.youtube.com/playlist?list=PLot-Xpze53leNZQd0iINpD-MAhMOMzWvO)
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

### E) Sliding Window 
> [Playlist](https://www.youtube.com/watch?v=1pkOgXD63yU&list=PLot-Xpze53leOBgcVsJBEGrHPd_7x_koV)
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

### F) Linked List

### G) Trees (DFS & BFS) 
> [Playlist](https://www.youtube.com/watch?v=OnSn2XEQ4MY&list=PLot-Xpze53ldg4pN6PfzoJY7KsKcxF1jg&index=2&t=0s)

### H) Tries 

### I) Heap / Priority Queue 

### J) **Backtracking** 
> [Playlist](https://www.youtube.com/watch?v=pfiQ_PS1g8E&list=PLot-Xpze53lf5C3HSjCnyFghlW0G1HHXo)


### K) Graphs 
> [Playlist](https://www.youtube.com/watch?v=EgI5nU9etnU&list=PLot-Xpze53ldBT_7QA8NVot219jFNr_GI&index=1&t=0s)
> DFS, BFS, Union Find - UF, Topological Sort - TS, Minimum Spanning Tree (prim / kruskal) - MST, 
> Shortest Path (source to all vertices - dijkstra) SP1, Shortest Path (from every vertex to every other vertex - floyd warshall) SP2


### L) Dynamic Programming 1-D 
> [Playlist](https://www.youtube.com/watch?v=g0npyaQtAQM&list=PLot-Xpze53lcvx_tjrr_m2lgD2NsRHlNO)
> Fibonacci Numbers (FIB), 0/1 Knapsack (01K), Unbounded Knapsack (UNK), Palindrome (PAL), Longest common subsequence (LCS)
> [Fibonacci Number](https://github.com/brpandey/leetcode/blob/master/rust/src/p0509_fibonacci_number.rs) FIB 509 E


### M) Intervals 
### N) Greedy 
### P) Dynamic Programming 2-D 

> Fibonacci Numbers (FIB), 0/1 Knapsack (01K), Unbounded Knapsack (UNK), Palindrome (PAL), Longest common subsequence (LCS)

### Q) Bit Manipulation 

### R) Math & Geometry 
