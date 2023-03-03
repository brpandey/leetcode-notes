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
