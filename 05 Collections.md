## `Vec<T>`
```rust
let mut v: Vec<i32> = Vec::new();  
  
v.push(1);  
let l = v.len();  

println!(v[0]); // if index is out of bounds rust panics!

for next in v.iter(){  
    println!("{}", next);  
}

let popped = v.pop(); 
```

if we know all the values we want in the vector we can use `vec!` macro to create the vector
```rust
let mut v = vec![1,2,3,4];
```

## `HashMap<K,V>`
```rust
let mut h: HashMap<u8, bool> = HashMap::new();
h.insert(5, true);
h.insert(1,false);

let visited_five = h.remove(&5).unwrap(); // returns an Option
```

other collections are
VecDeque
HashSet
LinkedList
BinaryHeap
BTreeMap - sorted
BTreeSet - sorted