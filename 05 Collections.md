## `Vec<T>`
```rust
let mut v: Vec<i32> = Vec::new();  
  
v.push(1);  
let l = v.len();  

println!(v[0]); // if index is out of bounds rust panics!

for next in v.iter(){  
    println!("{}", next);  
}

println!("{:?}", v); // to print a vector use the debug syntax

let popped = v.pop(); 
```
we can iterate over a vector directly as well but that takes ownership of the vector and it cant be used again
```rust
for next in v{}
```
we can iterate over a reference of v and that wont take the ownership
```rust
for nect in &v{}
```
if we know all the values we want in the vector we can use `vec!` macro to create the vector
```rust
let mut v = vec![1,2,3,4];
```
we can slice vectors as well
```rust
let a = vec![1, 2, 3, 4, 5, 5];
let b = &a[0..3];  // b is a slice: &[i32]
println!("{:?}", b); // prints [1, 2, 3]
```

a vector of arrays and each array is fixed to be of size 20 bytes
```rust
Vec<[u8; 20]>
```
## `HashMap<K,V>`
```rust
let mut h: HashMap<u8, bool> = HashMap::new();
h.insert(5, true);
h.insert(1,false);

let falseOption: Option<bool> = h.get(&1);

let visited_five = h.remove(&5).unwrap(); // returns an Option

let map = HashMap::from([
        ("apple", 3),
        ("banana", 2),
        ("orange", 5),
    ]);
    
for (key, value) in &map {
    println!("{}: {}", key, value);
}

for key in map.keys() {
    println!("key: {}", key);
}

for value in map.values() {
    println!("value: {}", value);
}
```

we can iterate over a hashmap mutably to update the values when iterating
```rust
for (key, value) in map.iter_mut() {
    *value += 1;
    println!("{}: {}", key, value);
}
```

we can iterate over a map without the reference but that takes ownership of the map and cant be used afterwords
```rust
for (key, value) in map {
    println!("{}: {}", key, value);
}
```

other collections are
VecDeque
HashSet
LinkedList
BinaryHeap
BTreeMap - sorted
BTreeSet - sorted