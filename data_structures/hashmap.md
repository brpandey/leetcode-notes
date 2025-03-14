Hash Map

* Also: [HashSet](https://doc.rust-lang.org/std/collections/struct.HashSet.html#)
  * A hash set is implemented as a HashMap where the value is (). 
  * Methods: **contains**, difference, get, insert, intersection, is_empty, iter, len, remove, retain, union
  * Set aggregation methods need collect: let intersection: HashSet<_> = a.intersection(&b).collect();
* Note: Where n is the number of items and m is the size

| Operations | Time Complexity Best | Avg Complexity | Worse Case |
|------------|-----------------|----------------| -----------|
| Access | O(1) | O(n/m) | O(log n) or even O(n)
| Insert | O(1) | O(n/m) | O(log n) or even O(n)
| Remove | O(1) | O(n/m) | O(log n) or even O(n)
| Contains Key | O(1) | O(n/m) | O(log n) or even O(n)


[HashMap](https://doc.rust-lang.org/std/collections/struct.HashMap.html), [Entry](https://doc.rust-lang.org/std/collections/hash_map/enum.Entry.html)
* Important methods: 
   * capacity, clear, contains_key, entry, get, get_mut, insert, into_keys, into_values, is_empty, iter, iter_mut, keys, len, new, remove, reserve, retain, values, values_mut, with_capacity, 
```rust
use std::collections::HashMap;

// Type inference lets us omit an explicit type signature (which
// would be `HashMap<String, String>` in this example corresponding to book title and review).
let mut book_reviews = HashMap::new();

// Review some books. insert into hashmap key value pairs e.g. title, review
book_reviews.insert(
    "Adventures of Huckleberry Finn".to_string(),
    "My favorite book.".to_string(),
);

book_reviews.insert(
    "Pride and Prejudice".to_string(),
    "Very enjoyable.".to_string(),
);
book_reviews.insert(
    "The Adventures of Sherlock Holmes".to_string(),
    "Eye lyked it alot.".to_string(),
);

// Check for a specific one.
// When collections store owned values (String), they can still be
// queried using references (&str).
if !book_reviews.contains_key("Les Misérables") {
    println!("We've got {} reviews, but Les Misérables ain't one.",
             book_reviews.len());
}

// oops, this review has a lot of spelling mistakes, let's delete it.
book_reviews.remove("The Adventures of Sherlock Holmes");

// Look up the values associated with some keys.
let to_find = ["Pride and Prejudice", "Alice's Adventure in Wonderland"];
for &book in &to_find {
    match book_reviews.get(book) {
        Some(review) => println!("{book}: {review}"),
        None => println!("{book} is unreviewed.")
    }
}

// Look up the value for a key (will panic if the key is not found).
println!("Review for Jane: {}", book_reviews["Pride and Prejudice"]);

// Iterate over everything.
for (book, review) in &book_reviews {
    println!("{book}: \"{review}\"");
}

// A HashMap with a known list of items can be initialized from an array:

let solar_distance = HashMap::from([
    ("Mercury", 0.4),
    ("Venus", 0.7),
    ("Earth", 1.0),
    ("Mars", 1.5),
]);

//HashMap implements an Entry API, which allows for complex methods of getting, setting, updating and removing keys and their values:

// type inference lets us omit an explicit type signature (which
// would be `HashMap<&str, u8>` in this example).

let mut player_stats = HashMap::new();

fn fixed_value() -> u8 { 42 }

// insert a key only if it doesn't already exist
player_stats.entry("health").or_insert(100);

// insert a key using a function that provides a new value only if it
// doesn't already exist
player_stats.entry("defence").or_insert_with(fixed_value);

// UPDATE
// update a key, guarding against the key possibly not being set
let stat = player_stats.entry("attack").or_insert(100);
*stat += fixed_value();

// UPDATE
// modify an entry before an insert with in-place mutation
player_stats.entry("mana").and_modify(|v| *v += 200).or_insert(100);
```


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
 
 [Example](https://github.com/brpandey/leetcode/blob/master/rust/src/p0242_valid_anagram.rs)

 ```rust
     // assert_eq!(true, Solution::is_anagram("anagram".to_string(), "nagaram".to_string()));

     pub fn is_anagram(s: String, t: String) -> bool {

        // Reduce string to source map
        let mut source = s.as_bytes().iter().fold(HashMap::new(), |mut acc, b| {
            *acc.entry(b).or_insert(0) += 1;
            acc
        });

        // Update source map, decrementing counts based on bytes seen in t
        for b in t.as_bytes() {
            if !source.contains_key(b) {
                return false
            }

            source.entry(b).and_modify(|e| *e -= 1); 
        }

        let result = source.into_values().sum();
        //let result = source.into_iter().map(|(_,v)| v).sum();

        if 0 == result { true } else { false }
    }
 ```
   
Using the HashMap entry api to insert a default value or update an existing value using *.and_modify().or_insert()*   

[entry](https://doc.rust-lang.org/stable/std/collections/struct.HashMap.html#method.entry)

```rust
pub fn entry(&mut self, key: K) -> Entry<'_, K, V>
Gets the given key’s corresponding entry in the map for in-place manipulation.

Examples
use std::collections::HashMap;

let mut letters = HashMap::new();

for ch in "a short treatise on fungi".chars() {
    letters.entry(ch).and_modify(|counter| *counter += 1).or_insert(1);
}

assert_eq!(letters[&'s'], 2);
assert_eq!(letters[&'t'], 3);
assert_eq!(letters[&'u'], 1);
assert_eq!(letters.get(&'y'), None);
```

[and_modify](https://doc.rust-lang.org/stable/std/collections/hash_map/enum.Entry.html#method.and_modify)

 ```rust
pub fn and_modify<F>(self, f: F) -> Self
where
    F: FnOnce(&mut V),
Provides in-place mutable access to an occupied entry before any potential inserts into the map.

Examples
use std::collections::HashMap;

let mut map: HashMap<&str, u32> = HashMap::new();

map.entry("poneyland")
   .and_modify(|e| { *e += 1 })
   .or_insert(42);
assert_eq!(map["poneyland"], 42);

map.entry("poneyland")
   .and_modify(|e| { *e += 1 })
   .or_insert(42);
assert_eq!(map["poneyland"], 43);
```

[or_insert](https://doc.rust-lang.org/stable/std/collections/hash_map/enum.Entry.html#method.or_insert)

```rust
pub fn or_insert(self, default: V) -> &'a mut V
Ensures a value is in the entry by inserting the default if empty, and returns a mutable reference to the value in the entry.

Examples
use std::collections::HashMap;

let mut map: HashMap<&str, u32> = HashMap::new();

map.entry("poneyland").or_insert(3);
assert_eq!(map["poneyland"], 3);

*map.entry("poneyland").or_insert(10) *= 2;
assert_eq!(map["poneyland"], 6);
```
