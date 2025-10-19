```rust
serde_json::Value::Number(serde_json::Number::from(v))
```
is the same as
```rust
v.into()
```
`.into()` in Rust is used to **convert a value from one type into another** when there’s an implementation of the `From` or `Into` trait for those types.

So if a type `B` implements `From<A>`, you can call `.into()` on an `A` to get a `B`.
example
```rust
pub fn decode_bencoded_ints(encoded_value: &str) -> serde_json::Value{  
    let val = encoded_value.parse::<isize>();  
    match val {  
        Ok(v) => {  
            //serde_json::Value::Number(serde_json::Number::from(v))  
            v.into()
        },        
        Err(e) => {  
            eprintln!("{}",e);  
            panic!("not a number: {}", encoded_value)  
        }    
	}
}
```
in the above example serde_json::Value must implement `From<isize>`

## assertion
```rust
fn find_first_a(s: String) -> Option<u32> {  
    for (i,c) in s.chars().enumerate(){  
        if c=='a' {  
            return Some(i as u32);  
        }    
	}    
	None  
}
```