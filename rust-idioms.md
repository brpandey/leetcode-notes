### Rust iteration Examples

```rust

    for i in 1..4 {
        println!("range notation - inclusive 1 to exclusive 4, so up to 3 {i}");
    }
    
    /* Output
    range notation - inclusive 1 to exclusive 4, so up to 3 1
    range notation - inclusive 1 to exclusive 4, so up to 3 2
    range notation - inclusive 1 to exclusive 4, so up to 3 3
    */
    
    println!("");
    
    for i in 1..=4 {
        println!("range notation - inclusive 1 to inclusive 4 {i}");
    }
    
    /* Output
    range notation - inclusive 1 to inclusive 4 1
    range notation - inclusive 1 to inclusive 4 2
    range notation - inclusive 1 to inclusive 4 3
    range notation - inclusive 1 to inclusive 4 4
    */
    
    println!("");
    
    for i in (1..4).rev() {
        println!("range notation - reverse! inclusive 1 to exclusive 4 (so up to 3) {i}");
    }
    
    /* Output
    range notation - reverse! inclusive 1 to exclusive 4 (so up to 3) 3
    range notation - reverse! inclusive 1 to exclusive 4 (so up to 3) 2
    range notation - reverse! inclusive 1 to exclusive 4 (so up to 3) 1
    */
    
    println!("");
    
    for i in (1..=4).rev() {
        println!("range notation - reverse! inclusive 1 to inclusive 4 {i}");
    }
    
    /*
    range notation - reverse! inclusive 1 to inclusive 4 4
    range notation - reverse! inclusive 1 to inclusive 4 3
    range notation - reverse! inclusive 1 to inclusive 4 2
    range notation - reverse! inclusive 1 to inclusive 4 1
    */
    
    println!("");
    
    for (e,i) in (1..=4).rev().enumerate() {
        println!("range notation - reverse w/ enumeration! inclusive 1 to inclusive 4 ({e},{i})");
    }
    
    /*
    range notation - reverse w/ enumeration! inclusive 1 to inclusive 4 (0,4)
    range notation - reverse w/ enumeration! inclusive 1 to inclusive 4 (1,3)
    range notation - reverse w/ enumeration! inclusive 1 to inclusive 4 (2,2)
    range notation - reverse w/ enumeration! inclusive 1 to inclusive 4 (3,1)
    */
```

Iterate through a matrix
```rust
    // Traverse through a matrix / board
    pub fn is_valid_sudoku(board: Vec<Vec<&str>>) -> bool {
        let rows: usize = board.len();
        let cols: usize = board[0].len();

        let mut row_sets = vec![HashSet::new(); rows];
        let mut col_sets = vec![HashSet::new(); cols];

        for r in 0..rows {
            for c in 0..cols {
                // references are copy types so we're fine
                let val = board[r][c];

                if val == "." { 
                    continue 
                }
        //...
        }
     }
     
     /*
      // board1 len is number of rows
      // board1[0].len is the length of first row => giving num of cols
          let board1 = vec![
            vec!["5","3",".",".","7",".",".",".","."],
            vec!["6",".",".","1","9","5",".",".","."],
            vec![".","9","8",".",".",".",".","6","."],
            vec!["8",".",".",".","6",".",".",".","3"],
            vec!["4",".",".","8",".","3",".",".","1"],
            vec!["7",".",".",".","2",".",".",".","6"],
            vec![".","6",".",".",".",".","2","8","."],
            vec![".",".",".","4","1","9",".",".","5"],
            vec![".",".",".",".","8",".",".","7","9"]];
     */
        
```

While loop 

```rust
    let values = vec![3, 4];
    
    // while loop iteration doesn't explicitly handling de-sugaring and calling iter.next()
    // so must do this explicitly
    let mut iter = values.iter();
    
    1)
    while let Some(a) = iter.next() {
        println!("while value is {}", *a);
    }
    
    /*
    while value is 3
    while value is 4
    */
    
    2)
    while !stack.is_empty() && stack[stack.len() - 1].0 < t {
        let (_v, v_index) = stack.pop().unwrap();
        result[v_index] = (i - v_index) as i32;
    }
    
    3)
    while r < nums.len() {
        max_jumped = 0;
        for i in l..=r {
            max_jumped = std::cmp::max(max_jumped, nums[i] as usize + i);
        }
    }
```
    
Rust for loop iteration styles: 
* into_iter (consume)
* iter_mut (&mut v)
* iter (&v)

```rust

    /**** Rust iter for loop styles: into_iter (consume), iter_mut (&mut v), iter (&v) ****/
    
    let mut values = vec![41,33,19,99];

    for x in values.iter_mut() {
        *x += 1;
    }

    assert_eq!(values, vec![42,34,20,100]); // `values` is still owned by this function.

    for (i,x) in values.iter().enumerate() {
        if i == 0 { assert_eq!(*x, 42); }
    }
    
    assert_eq!(values.len(), 4); // `values` is still owned by this function.
    
    println!("");

    for x in values.into_iter() { // values is moved into this block/func
        println!("x is {x}");
    }
    
    /*
    x is 42
    x is 34
    x is 20
    x is 100
    */
    
    // assert_eq!(values.len(), 4); // `values` has been consumed, statement will error
    
    /*
    38 |     for x in values.into_iter() {
       |                     ----------- `values` moved due to this method call
        ...
    42 |     assert_eq!(values.len(), 4); // `values` has been consumed
       |                ^^^^^^^^^^^^ value borrowed here after move
    */
```

[Iterator - map]()
```rust
    let a = [1, 2, 3];

    let mut iter = a.iter().map(|x| 2 * x);

    assert_eq!(iter.next(), Some(2));
    assert_eq!(iter.next(), Some(4));
    assert_eq!(iter.next(), Some(6));
    assert_eq!(iter.next(), None);
    
    //If you’re doing some sort of side effect, prefer *for* to map():

    // don't do this:
    (0..2).map(|x| println!("A {x}")); // it won't even execute, as it is lazy. Rust will warn you about this.

    // Instead, use for:
    for x in 0..2 {
        println!("B {x}");
    }

    println!("");

    // Or also:
    (0..2).for_each(|x| println!("C {x}"))
    
    /* Output
    
        B 0
        B 1

        C 0
        C 1
    */
```

[Iterator - fold](https://doc.rust-lang.org/stable/std/iter/trait.Iterator.html#method.fold)
```rust
    // Fold
    
    let a = [1, 2, 3];

    // the sum of all of the elements of the array
    let sum = a.iter().fold(0, |acc, &x| acc + x);

    assert_eq!(sum, 6);
 ```
 
 Fold: [ 347 top_k_frequent_elements](https://github.com/brpandey/leetcode/blob/master/rust/src/p0347_top_k_frequent_elements.rs)
 ```rust    
    // Given count values, store in another hashmap which is indexed by counts, 
    // values being the collection of keys with that count
    let mut flipped_counts = counts.into_iter().fold(HashMap::new(), |mut acc, (k,v)| {
        //acc.entry(v).and_modify(|x: &mut Vec<&i32>| x.push(k)).or_insert(vec![k]);
        acc.entry(v).or_insert_with(|| vec![]).push(k);
        acc
    });
 ```
 
 [Iterator - sum](https://doc.rust-lang.org/stable/std/iter/trait.Iterator.html#method.sum)
 ```rust
    let a = [1, 2, 3];
    let sum: i32 = a.iter().sum();

    assert_eq!(sum, 6);
 ```
 
 [Iterator - zip](https://doc.rust-lang.org/stable/std/iter/trait.Iterator.html#method.zip)
 ```rust
    let a1 = [1, 2, 3];
    let a2 = [4, 5, 6];

    let mut iter = a1.iter().zip(a2.iter());

    assert_eq!(iter.next(), Some((&1, &4)));
    assert_eq!(iter.next(), Some((&2, &5)));
    assert_eq!(iter.next(), Some((&3, &6)));
    assert_eq!(iter.next(), None);
 ```
 
 [Iterator - rev](https://doc.rust-lang.org/stable/std/iter/trait.Iterator.html#method.rev)
 ```rust
    let a = [1, 2, 3];

    let mut iter = a.iter().rev();

    assert_eq!(iter.next(), Some(&3));
    assert_eq!(iter.next(), Some(&2));
    assert_eq!(iter.next(), Some(&1));

    assert_eq!(iter.next(), None);
 ```
 
 ### String Handling Examples

 * [String - as_bytes](https://doc.rust-lang.org/std/string/struct.String.html#method.as_bytes) -> &[u8]
 * [String - into_bytes](https://doc.rust-lang.org/std/string/struct.String.html#method.into_bytes) -> Vec<u8>
 * [String - bytes](https://doc.rust-lang.org/std/string/struct.String.html#method.bytes) -> Bytes (an iterator over the bytes of a string slice)
    
 ```rust
    // or
    pub fn as_bytes(&self) -> &[u8] {
        // Returns a byte slice of this String’s contents.

        // The inverse of this method is from_utf8.

        // Examples
        // Basic usage:

        let s = String::from("hello");

        assert_eq!(&[104, 101, 108, 108, 111], s.as_bytes());
    }
```
   
```rust
    pub fn is_anagram(s: String, t: String) -> bool {
        // Reduce string to source map
        let mut source = s.as_bytes().iter().fold(HashMap::new(), |mut acc, b| {
            acc.entry(b).and_modify(|c| *c += 1).or_insert(1);
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
