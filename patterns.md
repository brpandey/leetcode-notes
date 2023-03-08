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

```rust

/*
   ------------
   | Option   |  [as_ref(), unwrap()]
   ------------
   | Rc       |  Rc::clone()
   ------------
   | RefCell  |  [borrow(), borrow_mut()]
   ------------
   | ListNode |  (.data, next)
   ------------
   
 */

pub type NodeRef<T> = Rc<RefCell<ListNode<T>>>;

#[derive(PartialEq, Eq, Debug)]
pub struct ListNode<T> {
    pub data: T,
    pub next: Option<NodeRef<T>>
}

impl<T> ListNode<T> {
    pub fn new(data: T) -> NodeRef<T> {
        Rc::new(
            RefCell::new(
                ListNode {
                    next: None,
                    data
                }
            )
        )
    }

    pub fn clone(node: &Option<NodeRef<T>>) -> Option<NodeRef<T>>{
        node.as_ref().map(Rc::clone)
    }

    pub fn next(current: &Option<NodeRef<T>>) -> Option<NodeRef<T>>{
        if current.is_none() { return None }
        current.as_ref().unwrap().borrow().next.as_ref().map(Rc::clone)
    }
}
```

### G) Trees (DFS & BFS) 
> [Playlist](https://www.youtube.com/watch?v=OnSn2XEQ4MY&list=PLot-Xpze53ldg4pN6PfzoJY7KsKcxF1jg&index=2&t=0s)

```rust
use std::rc::Rc;
use std::cell::RefCell;

pub type TreeNodeRef = Rc<RefCell<TreeNode>>;

/*
   ------------
   | Option   |  [as_ref(), unwrap()]
   ------------
   | Rc       |  Rc::clone()
   ------------
   | RefCell  |  [borrow(), borrow_mut()]
   ------------
   | TreeNode |  (.val, .left, .right)
   ------------

    hence if, node = Option<TreeNodeRef>
    
    node.as_ref().unwrap().borrow().val 
    
    or
    
    node.as_ref().unwrap().borrow().left.as_ref().map(Rc::clone)
                                      ^
                                      |------ new Option<TreeNodeRef> so start again, as_ref...
*/

// Definition for a binary tree node.
#[derive(Debug, PartialEq, Eq)]
pub struct TreeNode {
  pub val: i32,
  pub left: Option<TreeNodeRef>,
  pub right: Option<TreeNodeRef>,
}

impl TreeNode {
    #[inline]
    pub fn new(val: i32) -> TreeNodeRef {
        Rc::new(
            RefCell::new(
                TreeNode {
                    val,
                    left: None,
                    right: None
                }
            )
        )
    }

    pub fn value(node: &Option<TreeNodeRef>) -> i32 {
        node.as_ref().unwrap().borrow().val
    }

    pub fn left(node: &Option<TreeNodeRef>) -> Option<TreeNodeRef> {
        node.as_ref().unwrap().borrow().left.as_ref().map(Rc::clone)
    }

    pub fn right(node: &Option<TreeNodeRef>) -> Option<TreeNodeRef> {
        node.as_ref().unwrap().borrow().right.as_ref().map(Rc::clone)
    }
    
    pub fn from_list(a: &[T]) -> Option<NodeRef<T>> 
    where T: Copy {
        let mut head: Option<NodeRef<T>> = None;
        let mut n: NodeRef<T>;

        // Reverse the array list so 4 then points to 5 etc..
        for v in a.iter().rev() {
            n = ListNode::new(*v);
            if head.is_none() {
                n.borrow_mut().next = head;
            } else {
                n.borrow_mut().next = Some(Rc::clone(&head.unwrap()));
            }
            head = Some(n);
        }
        head
    }
}
```

[572 subtree of another tree](https://github.com/brpandey/leetcode/blob/master/rust/src/p0572_subtree_of_another_tree.rs)
```rust
    pub fn is_subtree(root: Option<TreeNodeRef>, sub_root: Option<TreeNodeRef>) -> bool {
        if sub_root == None { return true }
        if root == None { return false }

        // create the extra ref ptrs to the root and subroot as we need
        let r1 = root.as_ref().map(Rc::clone);
        let r2 = root.as_ref().map(Rc::clone);

        let s1 = sub_root.as_ref().map(Rc::clone);
        let s2 = sub_root.as_ref().map(Rc::clone);
        let s3 = sub_root.as_ref().map(Rc::clone);

        if Self::is_same(r1, s1) { return true }

        // adjust the tree by taking the sub_root and matching against its left or right subtrees
        Self::is_subtree(TreeNode::left(&r2), s2) || Self::is_subtree(TreeNode::right(&r2), s3)
    }

    pub fn is_same(node1: Option<TreeNodeRef>, node2: Option<TreeNodeRef>) -> bool {
        if node1 == None && node2 == None {
            return true
        }

        if node1 == None || node2 == None {
            return false
        }

        // If the current node value matches, check the that the l & r subtrees both match, recursively
        if TreeNode::value(&node1) == TreeNode::value(&node2) {
            return Self::is_same(TreeNode::left(&node1), TreeNode::left(&node2)) &&
                Self::is_same(TreeNode::right(&node1), TreeNode::right(&node2))
        }

        // if no match up to this point the nodes/trees aren't the same
        false
    }
```

[104 max depth](https://github.com/brpandey/leetcode/blob/master/rust/src/p0104_maximum_depth_of_binary_tree.rs)
```rust
    pub fn max_depth(root: Option<TreeNodeRef>) -> i32 {
        if root.is_none() { return 0 }

        let l = TreeNode::left(&root);
        // root.as_ref().unwrap().borrow().left.as_ref().map(Rc::clone);

        let r = TreeNode::right(&root);
        // root.as_ref().unwrap().borrow().right.as_ref().map(Rc::clone);

        return 1 + std::cmp::max(Solution::max_depth(l), Solution::max_depth(r));
    }
```

[226 invert_binary tree](https://github.com/brpandey/leetcode/blob/master/rust/src/p0226_invert_binary_tree.rs)
```rust
    pub fn invert_tree(root: Option<Rc<RefCell<TreeNode>>>) -> Option<Rc<RefCell<TreeNode>>>{
        if root == None { return None }

        let left = root.as_ref().unwrap().borrow_mut().left.take();
        let right = root.as_ref().unwrap().borrow_mut().right.take();

        root.as_ref().unwrap().borrow_mut().left = Solution::invert_tree(right);
        root.as_ref().unwrap().borrow_mut().right = Solution::invert_tree(left);

        root
    }
 ```
 
 [105 build_tree](https://github.com/brpandey/leetcode/blob/master/rust/src/p0105_construct_binary_tree_from_preorder_and_inorder_traversal.rs)
 ```rust
     pub fn build_tree(preorder: &[i32], inorder: &[i32]) -> Option<Rc<RefCell<TreeNode>>> {
        if preorder.len() != inorder.len() || preorder.is_empty() || inorder.is_empty() {
            return None
        }

        // Find root node, in preorder it's the first element e.g. preorder = NLR
        let root = preorder[0];
        let root_index = inorder.iter().position(|&v| v == root).unwrap();

        // Given inorder = LNR, we know that 0..root_index is left subtree, and root_index+1.. is right subtree (for inorder vec)
        // For preorder, since it is NLR, we know that 0 is root and  1..root_index+1 is the left subtree, we know this because
        // looking at the inorder vector 3 is the root and is at index 1, which says that only 1 element which is 9 (at index 0) is in the left subtree

        // build tree recursively by building subtrees recursively
        let r = TreeNode::new(root);

        // select left subtree portions of preorder and inorder lists, do same for right
        r.as_ref().borrow_mut().left = Self::build_tree(&preorder[1..root_index+1], &inorder[0..root_index]);
        r.as_ref().borrow_mut().right = Self::build_tree(&preorder[root_index+1..], &inorder[root_index+1..]);

        // subtree value is moved out
        Some(r)
    }
 ```

### H) Tries 

### I) Heap / Priority Queue 

[295 find median from data stream](https://github.com/brpandey/leetcode/blob/master/rust/src/p0295_find_median_from_data_stream.rs#L66)
[621 task scheduler](https://github.com/brpandey/leetcode/blob/master/rust/src/p0621_task_scheduler.rs)
```rust
pub fn task_scheduler(tasks: Vec<&str>, n: i32) -> i32 {
        let mut time: i32 = 0;

        // collect tasks into HashMap, incrementing count for each key
        let task_counts: HashMap<&str, u16> =
            tasks.into_iter().fold(HashMap::new(), |mut acc, t| {
                acc.entry(t).and_modify(|c| *c += 1).or_insert(1);
                acc
            });

        // flip the pair so that the binary heap tuple has the count first in the tuple
        // tuple ord comparison looks at the first element of the tuple
        let mut max_heap: BinaryHeap<(u16, &str)> =
            BinaryHeap::from_iter(task_counts.into_iter().map(|(ta, c)| (c, ta)));

        // (task name, count, wait-time)
        let mut waitq: VecDeque<(&str, u16, i32)> = VecDeque::new();

        // if we've just started waitq is empty and max_heap is not empty
        // if everything is waiting and no tasks are active, then max heap is empty and wait queue is not empty
        while !max_heap.is_empty() || !waitq.is_empty() {
            // if there's a matching task_tup on the wait queue
            // transfer it over to the binary heap as waiting is finished
            // and task can be actively considered
            if let Some((_, _, ti)) = waitq.front() {
                if *ti == time {
                    let (ta, c, _) = waitq.pop_front().unwrap();
                    max_heap.push((c, ta));
                }
            }

            time += 1;

            // grab next available task
            if let Some((mut c, ta)) = max_heap.pop() {
                // consume a task  print!("{} -> ", &ta);
                // now that we've "consumed" a task, decrement the number of its occurrences
                c -= 1;
                if c > 0 { // push only tasks that are remaining onto the wait queue
                    waitq.push_back((ta, c, time + n))
                }
            } // else consume idle, print!("idle -> ");
        }

        time
    }
```

### J) **Backtracking** 
> [Playlist](https://www.youtube.com/watch?v=pfiQ_PS1g8E&list=PLot-Xpze53lf5C3HSjCnyFghlW0G1HHXo)

[39 comb sum i](https://github.com/brpandey/leetcode/blob/master/rust/src/p0039_comb_sum_i.rs)
```rust
    pub fn recurse(index: usize, target: i32, cand: &Vec<i32>, sequence: &mut Vec<i32>, result: &mut Vec<Vec<i32>>) {
        // print A - potential comb

        if target == 0 {
            // print B - winning
            result.push(sequence.clone());
            return
        }

        for i in index..cand.len() {
            if target < cand[i] {
                // print C - won't work
                return // no valid path possible return early -- backtrack
            }

            let value = cand[i];
 ----->     sequence.push(value);
            Solution::recurse(i, target - value, cand, sequence, result);
 ----->     sequence.pop();
        }
    }
```

[46 permutations](https://github.com/brpandey/leetcode/blob/master/rust/src/p0046_permutations.rs)
```rust
pub fn dfs(nums: &mut Vec<i32>, start: usize, res: &mut Vec<Vec<i32>>){
        if (start+1) as usize == nums.len(){
            res.push(nums.clone());
            return;
        }

        for i in start..nums.len(){
            Solution::swap(nums, i, start);
            Solution::dfs(nums, start+1, res);
            Solution::swap(nums, i, start); // undo previous swap
        }
    }
```

[78 subsets](https://github.com/brpandey/leetcode/blob/master/rust/src/p0078_subsets.rs)
```rust
    pub fn generate(index: usize, nums: &[i32], subset: &mut Vec<i32>, result: &mut Vec<Vec<i32>>) {
        // Print - Add
        result.push(subset.clone());

        for i in index..nums.len() {
            subset.push(nums[i]);
            // Print - Push
            Solution::generate(i + 1, nums, subset, result);
            subset.pop();
            // Print - Pop
        }
    }
```

### K) Graphs 
> [Playlist](https://www.youtube.com/watch?v=EgI5nU9etnU&list=PLot-Xpze53ldBT_7QA8NVot219jFNr_GI&index=1&t=0s)
> DFS, BFS, Union Find - UF, Topological Sort - TS, Minimum Spanning Tree (prim / kruskal) - MST, 
> Shortest Path (source to all vertices - dijkstra) SP1, Shortest Path (from every vertex to every other vertex - floyd warshall) SP2

[210 course schedule ii](https://github.com/brpandey/leetcode/blob/master/rust/src/p0210_course_schedule_ii.rs)
```rust
 // n is the number of courses or vertices
    // prerequisites are a well-formed list of edges
    pub fn course_schedule_ii(n: u16, edges: &Vec<[u16; 2]>) -> Vec<u16> {
        let mut result = Vec::new();
        let mut count: u16 = 0;
        let mut in_degrees = vec![0; n as usize];

        // Set the in degrees of the vertices
        // NOTE: If prerequisites is [[1,0]], the graph looks like: 0 -> 1 or SRC -> DEST or [[DEST, SRC]]
        // Essentially the second or dst element of the edge is what the edge is pointing to so increment
        // that vertex's indegree count
        for e in edges {
            in_degrees[e[DST] as usize] += 1;
        }

        let mut queue: VecDeque<u16> = VecDeque::new();

        // Find all the vertices which have an in degree of 0 (meaning no dependencies or back arrows) and seed the queue
        for i in 0..in_degrees.len() {
            if in_degrees[i] == 0 {
                queue.push_back(i as u16);
            }
        }

        // process the in degrees of the vertices
        while !queue.is_empty() {
            let current = queue.pop_front().unwrap(); // remove from the queue and update indegree count(s)
            result.push(current); // every time we dequeue, add it to our results list (as it now has no prereqs or dependencies anymore)
            count += 1;

            // loop through the "graph" again search for if the vertex with nothing pointing at it has directed edges towards
            // destination vertices (atleast 1 must exist)
            for e in edges {
                // if current is equal to the edge src vertex, and since we've removed current from the queue already,
                // decrement the edge dest's vertex indegree, since we no longer have a vertex pointing towards it
                if current == e[SRC] {
                    in_degrees[e[DST] as usize] -= 1;
                    // if the edge destination vertex has no vertices pointing to it, add it to the queue
                    if in_degrees[e[DST] as usize] == 0 {
                        queue.push_back(e[DST]);
                    }
                }
            }
        }

        if n == count { return result } else { return vec![0] };
    }
```

[200 number of islands](https://github.com/brpandey/leetcode/blob/master/rust/src/p0200_number_of_islands.rs)
```rust
// Identify all the connected components using breadth-first-search to this current start location
    // (we have already tagged this as a single island)
    pub fn bfs(matrix: &Vec<Vec<char>>, visited: &mut Vec<Vec<bool>>, sequence: &Vec<(i32, i32)>, start: (i32, i32), size: (i32, i32)) {

        // Essentially, find the connected components related to the current location and add them
        // to the backlog queue for processing until there are no more connected components

        let mut backlog: VecDeque<(i32, i32)> = VecDeque::new();
        backlog.push_back(start); // seed our queue

        // Closure captures matrix and size
        // Safe to visit?
        // Check if positions are valid on the board
        // Check if we're on land '1' or water '0'
        let visitable = |(r, c)| {
            if r >= 0 && r < size.0 && c >= 0 && c < size.1 &&
                matrix[r as usize][c as usize] == '1' {
                    return true
                } else {
                    return false
                }
        };

        let mut peer;

        while !backlog.is_empty() {
            let current = backlog.pop_front().unwrap();

            // check peers (see if they're on the same island as us! may day! may day!)
            for offset in sequence {
                // Add offset into current to get peer locations (connected components)
                peer = (current.0 + offset.0, current.1 + offset.1);

                // If not already visited and safe to visit,
                // Enqueue to backlog and mark as visited
                if visitable(peer) && !visited[peer.0 as usize][peer.1 as usize] {
                    visited[peer.0 as usize][peer.1 as usize] = true;
                    backlog.push_back(peer);
                }
            }
        }
    }
```

[417 pacific atlantic water flow](https://github.com/brpandey/leetcode/blob/master/rust/src/p0417_pacific_atlantic_water_flow.rs)
```rust
 // DFS explores the heights grid space, marking if reachable from given ocean
    pub fn dfs(row: i32, col: i32, prev: i32, reachable: &mut HashSet<(i32, i32)>, heights: &Vec<Vec<i32>>, max: (i32, i32))  {

        let (max_rows, max_cols) = max;
        let (r, c) = (row as usize, col as usize);

        // Make sure hasn't been processed before
        // Ensure current height >= prev property is true otherwise return (climbing up the hill )
        // Sanity check the row/col values
        if reachable.contains(&(row,col)) || row < 0 || row >= max_rows || col < 0 || col >= max_cols || heights[r][c] < prev {
            return
        }

        // Reaching this point indicates that the current cell is reachable from an ocean
        // Since heights[row][col] >= prev

        // reachable is either atlantic or pacific
        reachable.insert((row, col));

        // search in the four directions around cells N, S, W, E (see if next ocean is reachable)
        Solution::dfs(row-1, col, heights[r][c], reachable, heights, max);
        Solution::dfs(row+1, col, heights[r][c], reachable, heights, max);
        Solution::dfs(row, col-1, heights[r][c], reachable, heights, max);
        Solution::dfs(row, col+1, heights[r][c], reachable, heights, max);
    }
```


### L) Dynamic Programming 1-D 
> [Playlist](https://www.youtube.com/watch?v=g0npyaQtAQM&list=PLot-Xpze53lcvx_tjrr_m2lgD2NsRHlNO)
> Fibonacci Numbers (FIB), 0/1 Knapsack (01K), Unbounded Knapsack (UNK), Palindrome (PAL), Longest common subsequence (LCS)
> [Fibonacci Number](https://github.com/brpandey/leetcode/blob/master/rust/src/p0509_fibonacci_number.rs) FIB 509 E

[139 word break](https://github.com/brpandey/leetcode/blob/master/rust/src/p0139_word_break.rs)
```rust
    pub fn word_break(s: String, word_dict: Vec<String>) -> bool {
        let slen = s.len(); // e.g. 7
        let mut dp = vec![false; slen+1]; // vec size 8, extra space for base case
        dp[slen] = true;  // this is base case, always set to true (e.g. s[7])

        for i in (0..slen).rev() { // start from string end
            for w in word_dict.iter() { // loop through dict
                let wlen = w.len();

                if i + wlen <= slen && // don't overshoot str len
                    &s[i..(i+wlen)] == w { // ensure s substring is word w
                        dp[i] = dp[i + wlen]; // chain together values, only works in the end if these are true
                    }

                if dp[i] {
                    break
                }
            }
        }

        return dp[0]
    }
```

### M) Intervals 

[252 meeting rooms i](https://github.com/brpandey/leetcode/blob/master/rust/src/p0252_meeting_rooms_i.rs)
```rust
pub fn meeting_rooms_i(input: &mut Vec<[u8; 2]>) -> bool {
        let n = input.len();

        // Sort vector time slots by start time so we can efficiently compare them
        input.sort_by(|i1, i2| i1[0].cmp(&i2[0]));

        // loop through vector grabbing the next two items at a time
        for i in 0..n-1 {
            let first = input[i];
            let second = input[i+1];

            // test for any overlaps, if so, we can't make all meetings if there is an overlap in meeting time
            // end1 > start2
            if first[1] > second[0] { return false };
        }

        return true;
    }
```

### N) Greedy

[53 max subarray](https://github.com/brpandey/leetcode/blob/master/rust/src/p0053_max_subarray.rs)
```rust
   pub fn max_subarray(nums: &[i32]) -> i32 {
        // Kadane's algorithm -> https://medium.com/@rsinghal757/kadanes-algorithm-dynamic-programming-how-and-why-does-it-work-3fd8849ed73d
        let acc = nums
            .iter()
            .fold((i32::min_value(), 0), |(mut global_max, mut local_max), &x| {
                // the local_maximum at index i is the maximum of nums[i] or x
                // and the sum of nums[i] or x and local_maximum at index i-1.
                local_max = cmp::max(x, x + local_max);
                global_max = cmp::max(local_max, global_max);

                (global_max, local_max)
            });

        acc.0
    }
```

[763 partition labels](https://github.com/brpandey/leetcode/blob/master/rust/src/p0763_partition_labels.rs)
```rust
pub fn partition_labels(s: String) -> Vec<i32> {
        let mut map = HashMap::with_capacity(26);
        let mut result = vec![];

        for (i, b) in s.bytes().enumerate().rev() {
            map.entry(b).or_insert(i);
        }

        let mut window_size = 0;
        let mut window_end = 0;

        for (i, b) in s.bytes().enumerate() {
            // grab character last index
            let last = *map.get(&b).unwrap();

            // if last index is past window end, update
            // window end (not done with current window yet)
            if last > window_end {
                window_end = last
            }

            window_size += 1;
            
            if window_end == i {
                result.push(window_size);
                window_size = 0;
                window_end = 0;
            }


        }

        result
    }
```

### O) Advanced Graphs

[787 find cheapest flights within k stops](https://github.com/brpandey/leetcode/blob/master/rust/src/p0787_cheapest_flights_within_k_stops.rs)
```rust
pub fn find_cheapest_price(n: i32, flights: Vec<Vec<i32>>, src: i32, dst: i32, k: i32) -> i32 {
        let mut prices = vec![INFINITY; n as usize];
        prices[src as usize] = 0;

        let mut new_prices: Vec<i32>;

        // use bellman ford algorithm
        // update destination prices to new_prices list given values from original prices
        // upon each k iteration set new_prices to prices

        for _i in 0..k+1 {

            new_prices = prices.clone();

            // iterate through all edges in flights directed graph (vec of vec)
            for edge in &flights {
                let (s, d, p) = (edge[0] as usize, edge[1] as usize, edge[2]);

                // lookup the source in the prices vec, if it is "infinity" ignore
                // because we aren't able to optimize for its destination if source is infinity still
                if prices[s] == INFINITY { continue }

                if prices[d] > prices[s] + p {
                    new_prices[d] = prices[s] + p;
                }
            }

            prices = new_prices;
        }

        if prices[dst as usize] == INFINITY { // meaning there's no path to dest, return -1
            -1
        } else {
            prices[dst as usize]
        }
    }
```

### P) Dynamic Programming 2-D 

> Fibonacci Numbers (FIB), 0/1 Knapsack (01K), Unbounded Knapsack (UNK), Palindrome (PAL), Longest common subsequence (LCS)

[Explanation - 518 coin change 2](https://github.com/brpandey/markdown-notes/blob/main/leetcode/p0518_coin_change_2.md)

[62 unique paths](https://github.com/brpandey/leetcode/blob/master/rust/src/p0062_unique_paths.rs)
```rust
    pub fn unique_paths(m: usize, n: usize) -> i32 {
        let row: Vec<i32> = vec![0 as i32; n];
        let mut dp: Vec<Vec<i32>> = vec![row; m]; // m rows of vecs size n (col)

        dp[0][0] = 1;

        // first row, initialize all col values to 1
        for col in 0..n {
            dp[0][col] = 1
        }

        // first col, initialize all row values to 1
        for row in 0..m {
            dp[row][0] = 1
        }

        for r in 1..m {
            for c in 1..n {
                dp[r][c] = dp[r-1][c] + dp[r][c-1]
            }
        }

        dp[m-1][n-1]
    }
```

[494 target sum](https://github.com/brpandey/leetcode/blob/master/rust/src/p0494_target_sum.rs)
```rust
pub fn find_target_sum_ways(nums: Vec<i32>, target: i32) -> i32 {
        let mut cache: HashMap<(usize, i32), i32> = HashMap::new();
        Self::dfs(0, 0, &nums, target, &mut cache)
    }

    pub fn dfs(index: usize, sum: i32, nums: &[i32], target: i32, 
               cache: &mut HashMap<(usize, i32), i32>) -> i32 {

        // Has this key already been precomputed? if so, return precomputed result
        if cache.contains_key(&(index, sum)) {
            return *cache.get(&(index, sum)).unwrap()
        }

        // If we've exhausted the path
        if index == nums.len() {
            // if matching, a single solution has been found
            if sum == target { return 1 } else { return 0 }
        }

        // If the path hasn't been exhausted yet,
        // Build up the cache for key (index, total) by exploring left and right paths
        let value = 
            Self::dfs(index + 1, sum + nums[index], nums, target, cache) +
            Self::dfs(index + 1, sum - nums[index], nums, target, cache);
    
        cache.insert((index, sum), value);

        return cache[&(index, sum)]
    }
```

### Q) Bit Manipulation 

### R) Math & Geometry 

[48 rotate image](https://github.com/brpandey/leetcode/blob/master/rust/src/p0048_rotate_image.rs)
```rust
pub fn run(matrix: &mut Vec<Vec<i16>>) {
        let size = matrix.len();

        if matrix[0].len() != size {
            panic!("matrix is not square");
        }
        // See Notes

        // Note: column is dependent on row so we have separate for loops
        for r in 0..size/2 { // Essentially how many squares (nested concentric squares including outer square)
            for c in r..(size-1-r) { // Start 1 col in extra on every depth essentially shaving off two from the sides each depth

                // Basically these are the transitions for the first cyle for input 1)
                /*
                1 ----> 3
                ^       |
                |       V
                7 <---- 9
                 */

                // 1) Create a hole at the top left corner where 1 originally is
                let mut hole = (r, c);
                let temp = matrix[hole.0][hole.1];

                // 2) Since we are shifting image clockwise by 90, move the closest thing into the hole
                // and then replace that with its antecedent and so on, so basically fill 1's place with 7,
                // fill 7's place with 9, fill 9's place with 3, and fill 3's place with temp
                // destination = f(source)

                // Closure for generating shift sequences
                // size is captured in the closure
                let shift = |(r,c): (usize, usize)| -> (usize, usize) {
                    (c as usize, size-1-r as usize)
                };

                // Shift values between 2d Vector locations
                let apply_shift = |m: &mut Vec<Vec<i16>>, src: (usize, usize), dest: (usize, usize)| {
                    m[dest.0][dest.1] = m[src.0][src.1];
                };

                // Create an iterator of shifts, the first three (given 4 sides of a square)
                // Seq           1    2    3    4
                // Cycle 1: 1 -> 3 -> 9 -> 7 -> 1

                // Cycle 2: 2 -> 6 -> 8 -> 4 -> 2
                // Assume 0 is the location of 1 or (r,c)

                let cycle = (1..4).scan(hole, |state, _x| {
                    *state = shift(*state);
                    Some(*state)
                }).collect::<Vec<_>>(); // We collect it so that we have a finite collection one that we can reverse

                // We are reversing cycle because to fill the hole
                // we shift in a counter-clockwise fashion (since 1's spot is empty, move 7 in, since 7 is empty move 9 in, etc..)
                // This is opposite of the clockwise rotation

                // Reverse beomes: 3    2    1
                //                 7 -> 9 -> 3

                // The source data now shifted becomes a new hole for the next source
                // 1st iteration: 1 is a hole, it now has 7's contents,
                // 2nd: 7 is now a hole, it now has 9's contents,
                // 3rd: 9 is now a hole, it now has 3's contents

                for &source in cycle.iter().rev() {
                    apply_shift(matrix, source, hole);
                    hole = source;
                }

                matrix[hole.0][hole.1] = temp; // 3 is a hole and it now has 1's original contents
            }
```
