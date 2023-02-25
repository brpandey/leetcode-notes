Hash Map

* Note: Where n is the number of items and m is the size

| Operations | Time Complexity Best | Avg Complexity | Worse Case |
|------------|-----------------|----------------| -----------|
| Insert | O(1) | O(n/m) | O(log n) or even O(n)
| Remove | O(1) | O(n/m) | O(log n) or even O(n)
| Contains or Search / Worse Case | O(1) | O(n/m) | O(log n) or even O(n)

* If the hashing function is not good and doesn't spread keys uniformly, the hash map bucket entries could degrade into a single linked list, better to use a balanced bst so worse case is O(log n)
* HashMap is an unordered collection
* HashMap is superficially similiar to an Array as it can be indexed into, e.g. 

| Index | Value |
|-------|-------|
| 0     | 34    |
| 1     | 22    |
| 2     | 8     |

* However a HashMap calls its Index --> Key
* HashMap Key values are non-contiguous indices that are not bounded but can be any hashable value (e.g. chars or strings or anything that implements a hashable trait)

| Key | Value |
|-------|-------|
| 980   | 34    |
| 37    | 22    |
| 10    | 8     |

or 

| Key | Value |
|-------|-------|
| "apple"   | 34  |
| "banana"  | 22  |
| "cherry"  | 8   |
| "donut"   | 100 |

* Essentially a hashmap combines array-like table (e.g. size 4) indexing, with linked list like chains

| Hashed Value | Head | -> Next Value | -> Next Value |
|----|----|-------| ---- |
| 0  | 34  | -> 47 | -> 99 |
| 1  | 22  | 
| 2  | 8   |
| 3  | 100 |


or

```mermaid
flowchart TD
    subgraph HashMap size m or 4
        id1[[0: see chain]] -.- id2
        id2[[1: 22]] -.- id3
        id3[[2: 8]] -.- id4
        id4[[3: 100]]
    end
 ```
 
 * Chain: size is 3, total entries is n or 3 + 3 = 6, so average complexity is O(6/4) or O(1)
 * assume keys: apple, tortoise22, and karo98, all collide to the same offset 0 bucket after hashing
 ```mermaid
flowchart LR
      id0(chain head) -.-> id4
      id4(apple, 34) -.-> id5
      id5(tortoise22, 47) -.-> id6
      id6(karo98, 99) -.-> id7
      id7(Null)
 ```

[Example](https://github.com/brpandey/leetcode/blob/9e0307e896995d7d2674a11465d265c02fb09204/rust/src/p0001_two_sum.rs)
```rust
//  assert_eq!(vec![0, 1], Solution::two_sum(&vec![2, 7, 11, 15], 9));

    pub fn two_sum(numbers: &[i32], target: i32) -> Vec<i32> {
        // HashMap<i32, i32> follows copy semantics,
        // keys are values in numbers list, values are indices in numbers list
        let mut map = HashMap::new(); 
        let mut result: Vec<i32> = Vec::new();
        let mut complement: i32;

        for (i, num) in numbers.iter().enumerate() {
            complement = target - num;

            // found a complement
            if let Some(&c_i) = map.get(&complement) {
                result.push(c_i as i32); // complement index
                result.push(i as i32); // original index
                break;
            }

            map.insert(num, i);
        }

        result
    }
 ```
