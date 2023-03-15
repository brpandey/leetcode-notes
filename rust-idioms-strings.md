 
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
