### Stack

also

### Dynamic Array / Vec

A stack is a collection of elements, a *push* adds to the collection, a *pop* removes the most recently added element
which resides on the stack top. In Rust, a stack is implemented using a dynamic array - Vec.

[Vec](https://doc.rust-lang.org/std/vec/struct.Vec.html#) (A contiguous growable array type) 
* Methods:
  * first, first_mut, Vec::from, insert, is_empty, get, get_mut, last, last_mut, len, Vec::new, pop, push, remove, sort, sort_by, Vec::with_capacity

```rust
let mut vec = vec![1, 2];
vec.push(3);
assert_eq!(vec.pop(), Some(3));
assert_eq!(vec, [1, 2]);
```

1) Initial Stack using a Vec

| index 1 | 2 | <-- Back |
|---|---| --- |
| index 0 | 1 | <-- Front |

2) Push on value 3

| 3 |
|---|
| 2 |
| 1 |

3) Pop ()

| 2 |
|---|
| 1 |

Sorting
```rust
let mut v = [-5, 4, 1, -3, 2];

v.sort();
assert!(v == [-5, -3, 1, 2, 4]);

let mut v = [5, 4, 1, 3, 2];
v.sort_by(|a, b| a.cmp(b));
assert!(v == [1, 2, 3, 4, 5]);

// reverse sorting
v.sort_by(|a, b| b.cmp(a));
assert!(v == [5, 4, 3, 2, 1]);
```

Stack

| Operations | Time Complexity | Notes | 
|------------|-----------------| ------|
| Top (right) | O(1) | // v.last() or v.get(v.len-1) |
| Push (right) | O(1) | // adds to back |
| Pop (right) | O(1) |  // removes last element |
| Access with index | O(1) | [index operator complexity](https://doc.rust-lang.org/std/vec/)|
| Remove with index | O(n) | |
| Search | O(n) | |

```rust
let mut vec = Vec::new();
vec.push(1);
vec.push(2);

assert_eq!(vec.len(), 2);
assert_eq!(vec[0], 1);

assert_eq!(vec.pop(), Some(2));
assert_eq!(vec.len(), 1);

vec[0] = 7;
assert_eq!(vec[0], 7);

vec.extend([1, 2, 3]);

for x in &vec {
    println!("{x}");
}
assert_eq!(vec, [7, 1, 2, 3]);

// The vec! macro is provided for convenient initialization:

let mut vec1 = vec![1, 2, 3];
vec1.push(4);

let vec2 = Vec::from([1, 2, 3, 4]);
assert_eq!(vec1, vec2);
```


Stack Examples

[p0509 fibonacci numbers](https://github.com/brpandey/leetcode/blob/5df8127017ff45b250136ab770844fa6b7fac867/rust/src/p0509_fibonacci_number.rs#LL37-L54C6)
```rust
    // Solution 1) Iterative
    pub fn fib(mut n: u32) -> u32 {
        let mut stack = Vec::with_capacity(n as usize + 1);
        let (mut a, mut b) : (u32, u32);
        stack.push(0);
        stack.push(1);

        //  0, 1, 2, 3, 4, 5, 6,  7,  8,  9, 10, 11,  12,  13,  14,  15,  16, ...
        //  0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233, 377, 610, 987, ... 
        while n > 1 {
            b = stack.pop().unwrap(); // 1 base case
            a = stack.pop().unwrap(); // 0 base case
            stack.push(b);
            stack.push(b + a);
            n -= 1;
        }
        stack.pop().unwrap()
    }
```

[p0227 basic calculator ii](https://github.com/brpandey/leetcode/blob/5df8127017ff45b250136ab770844fa6b7fac867/rust/src/p0227_basic_calculator_ii.rs#L52)
```rust
 pub fn calculate(s: String) -> i32 {
        let mut last: i32 = 0;
        let mut stack: Vec<i32> = vec![];
        let mut operator: char = '!';
        let mut digit;

        let list: Vec<char> = s.chars().collect();
        let len = list.len();

        for (i, c) in list.iter().enumerate() {
            digit = *c >= '0' && *c <= '9';

            // only skip for whitespace if not last char
            if *c == ' ' && i + 1 != len {
                continue
            }

            // accumulate number if multi-digit
            if digit {
                last = last * 10 + (*c as i32 - '0' as i32);
            } 

            // push and pop from stack
            // for mult and div resolve immediately since higher precedence
            if !digit || i + 1 == len {
                match operator {
                    '+' | '!' => {
                        stack.push(last);
                    },
                    '-' => {
                        stack.push(last * -1);
                    },
                    '*' => {
                        let temp = stack.pop().unwrap();
                        stack.push(temp*last);
                    },
                    '/' => {
                        let temp = stack.pop().unwrap();
                        stack.push(temp/last);
                    },
                    _ => (),
                }

                // trick is to have a local variable 'stack' operator
                // to capture what the operation is, used once we parse the next
                // number
                operator = *c;
                // if we've seen an operator that means the last number has already
                // been pushed to stack and we reset it to create space for new one
                last = 0;
            }
        }

        let mut sum = 0;

        // commutatively sum reduce the remaining of the stack
        while !stack.is_empty() {
            sum += stack.pop().unwrap();
        }

        sum
    }
```

[739 daily temperatures](https://github.com/brpandey/leetcode/blob/5df8127017ff45b250136ab770844fa6b7fac867/rust/src/p0739_daily_temperatures.rs#L57)
```rust
    //  assert_eq!(vec![1,1,4,2,1,1,0,0], Solution::daily_temperatures(vec![73,74,75,71,69,72,76,73]));
    
    pub fn daily_temperatures(temperatures: Vec<i32>) -> Vec<i32> {
        let mut stack: Vec<(i32, usize)> = vec![];
        let mut result: Vec<i32> = vec![0; temperatures.len()];

        // Scan through temperature values, while value is greater than
        // stack top, pop stack, and update result vector accordingly with # days
        for i in 0..temperatures.len() {
            let t = temperatures[i];
            while !stack.is_empty() && stack[stack.len() - 1].0 < t {
                let (_v, v_index) = stack.pop().unwrap();
                result[v_index] = (i - v_index) as i32;
            }

            stack.push((t,i));
        }

        result
    }
```
