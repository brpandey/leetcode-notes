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
    
    let values = vec![3, 4];
    
    // while loop iteration doesn't explicitly handling de-sugaring and calling iter.next()
    // so must do this explicitly
    let mut iter = values.iter();
    while let Some(a) = iter.next() {
        println!("while value is {}", *a);
    }
    
    /*
    while value is 3
    while value is 4
    */
    
    println!("");

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
