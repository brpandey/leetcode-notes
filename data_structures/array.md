Linked List

| Operations | Time Complexity |
|------------|-----------------|
| Insert End | O(1) |
| Remove End | O(1) |
| Insert Mid | O(n) |
| Remove Mid | O(n) |

Array

| 0 | 1 | 2 | 3 | 4 | 5 |
|---|---|---|---|---|---|
| 1 | 3 | 4 | 7 | 8 |   |

* Array values are contiguous in RAM, so each value is next to each other
* Given index, can directly index into value hence O(1) value access
* To remove elements, except for end, must shift the other elements (up to n elements) O(n)! 
* When insert have to shift others right! When remove have to shift others left -- keeps everything contiguous still!

Array Insert - move elements over to create space - O(n)

| 0 | 1 | 2 | 3 | 4 | 5 |
|---|---|---|---|---|---|
| 1 | 3 |   | 4 | 7 | 8 |

Array Insert - add new element value

| 0 | 1 | 2 | 3 | 4 | 5 |
|---|---|---|---|---|---|
| 1 | 3 | 5 | 4 | 7 | 8 |


```mermaid
flowchart LR
    subgraph RAM
        id1[[0x658: 1]] -.- id2
        id2[[0x659: 3]] -.- id3
        id3[[0x660: 4]] -.- id4
        id4[[0x661: 7]] -.- id5
        id5[[0x662: 8]]
    end
 ```
 
[Example](https://github.com/brpandey/leetcode/blob/master/rust/src/p0004_median_two_sorted_arrays.rs)

```rust
    pub fn search(nums: [i32, 6], target: i32) -> i32 {
        let mut pivot;

        let mut lo = 0;
        let mut hi = nums.len() - 1;  // 5

        while lo < hi {

            // 0 1 2 3 4 : indexes
            // ---------
            // 1 3 4 7 8 : values
            
            // first run pivot = 0 + (4-0)/2 = 2 or value 4

            // Note: pivot = (lo + hi) / 2, could result in an overflow exception, hence below is used
            pivot = lo + (hi - lo) / 2;

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
