```rust
enum Color {
	Red,
	Green,
	Blue
}

let color = Color::Red;
```

in rust we can associated data and functions with the enum values
```rust
enum DispenserItem {
	Empty, // no data
	Ammo(u8), // single type of data
	Things(String, i32), // tuple of data
	Place{x: i32, y:i32} // anonymous struct of data
}

let empty = DispenserItem::Empty;
let ammo = DispenserItem::Ammo(69);
let things = DispenserItem::Things("hat".to_string(), 7);
let place = DispenserItem::Place{x: 24, y: 3};
```

we can eve implement functions and methods for enums
```rust
impl DispenserItem {
	fn display(&self){
		// do something
	}
}
```

Option is a generic enum in the standard library

```rust
enum Option<T>{
	Some(T),None
}
```
similarly we have result
```rust
#[must_use]
enum Result<T,E>{
	Ok(T), Err(E)
}
```

the must use means it throws a warning if we dont look at all the branches of this enum in our code, its recommended to always handle all branches of Result

we can simply check if result is ok or not with
```rust
let res = File::open("foo");
if res.is_ok() {
	let f = res.unwrap();
}

if let Ok(f) = res {
	// do stuff with f
}
```

we can then match on these patterns to do conditional logic on different enum values
```rust
match v {
	Some(t) => {},
	None =>
}

match res {
	Ok(r) => {},
	Err(e) => {}
}
```

but if we want to write logic for only one of those branches

```rust
if let Some(x) = v {
	println!("{}", x);
}
```

this works only when the Some(x) matches by pattern one of the values of the enum v

match is kinda like a switch case, but we dont have a default branch for the switch case
we can have that with and `_`
```rust
match v {
	_ => {
		println!("who cares brother who cares");
	}
}
```

this becomes use ful when you are pattern matching on things other than enums like numbers or tuples or strings
```rust
match number {
    0 => println!("zero"),
    1 | 2 => println!("one or two"),
    3..=9 => println!("three to nine"),
    _ => println!("something else"),
}

match (x, y) {
    (0, 0) => println!("origin"),
    (0, _) => println!("on x-axis"),
    (_, 0) => println!("on y-axis"),
    _ => println!("somewhere else"),
}

match command {
    "start" => start_game(),
    "quit" => quit_game(),
    _ => println!("unknown command"),
}
```

we can also do pattern matching on structure objects with de structuring

```rust
match point {
    Point { x: 0, y: 0 } => println!("origin"),
    Point { x, y: 0 } => println!("on x-axis at {}", x),
    Point { x, y } => println!("at ({}, {})", x, y),
}
```

we can return from match branches as long as all the branches return the same type

```rust
let x = match v {
	Some(t) => 3,
	None => 4
};
```

we can use option Enum values in our own code instead of just seeing them in the std library and we also have a helper method that lets us check if a variable is `Some` from and `Option`
```rust
let mut x: Option<i32> = None;
x = Some(4);

x.is_some();// true
x.is_none();// true
```

we can match patterns and do some logic for the value associated with the variant using `guards`

```rust
enum Shot {  
    Bullseye, Hit(f64), Miss  
}  
impl Shot {  
    fn points(self) -> i32 {  
        match self {  
            Shot::Bullseye => 5_i32,  
            Shot::Hit(x) if x < 3.0 => 2,  
            Shot::Hit(x) => 1,  
            Shot::Miss => 0  
        }  
    }
}
```

the second only matches if x < 3.o other wise the control goes to the third match