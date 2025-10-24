structs can have data fields, methods and associated functions
```rust
struct RedFox {
	enemy: bool,
	life: u8,
}

let for = RedFox {
	enemy: true,
	life: 70,
}
```

typically we want to create an associated function for the struct, which serves like a constructor

```rust
impl RedFox {
	fn new() -> Self {
		Self {
			enemy: true,
			life: 80,
		}
	}
}

let fox = RedFox::new();
let life_left= fox.life;
fox.enemy=false;
fox.some_method();
```
new is just the conventional name given to a function used as constructor
Self is the return type of the function and Self also refers to the struct we are impl-ing
this above example is called an associated function because it doesnt take Self as a parameter, if it did then it would be a struct member method 

associated functions dont take self and are called with `::` instead of `.`

methods are also defined in the implementation block
methods must take some sort of Self as a parameter, other wise they are just associated functions
```rust
impl RedFox {
	fn move(self)
	fn burrow(&self)
	fn mut_burrow(&mut self)
}
```

there is a difference between the two main ways of taking parameters in the associated functions
```rust
struct Rect {
	width: u32,
	height: u32
}
impl Rect {
	fn new(h: u32, w: u32)->Self {
		Self {
			width: w,
			height: h
		}
	}
}
impl Rect {  
    fn area_takes_ownership(self) -> u32 {  
        self.width*self.height  
    }  
}
impl Rect {  
    fn area(&self) -> u32 {  
        self.width*self.height  
    }  
}
```

the function `fn area_takes_ownership(self)` takes ownership of the rect object and is destroyed after the function call.

the object on whom this function is called can not be used after this call

we dont have inheritance in structs, we have traits instead.

we can create unit structs, which are structs that dont have any data fields assigned to them, but are used to group associated functions together, useful if you want to implement the visitor pattern
then we can Impl the struct and add different visit methods to it

```rust
struct User;
```

![[Pasted image 20251017202054.png]]
the primitive fields are stored on the stack, while the fields that dont implement the copy trait are stored on the heap

since strings and vec dont implement copy trait, if my struct has those as fields then my struct can also not implement the copy trait

every type that implements `Copy` _must_ also implement `Clone`

## traits
traits are like interfaces
we do composition over inheritance
traits define required behaviour, functions and methods a struct must have for it to satisfy that trait

```rust
struct RedFox {
	enemy: bool,
	life: u32
}
trait Noisy {
	fn be_noisy(&self) -> &str;
}

impl Noisy for RedFox {
	fn be_noisy(&self) -> &str {"Meow?"}
}
```

the point of traits is that once our struct implements a trait we can start writing generic code that can accept any argument that implements a trait

```rust
fn make_noise<T: Noisy>(item: T) {
	println!("{}", item.be_noisy());
}

make_noise(RedFox {enemy:false, life: 80});
```
the generic T can be forced to implement multiple traits

```rust
fn make_noise<T: Noisy + Sound>(item: T){
	println!("{}", item.be_noisy());
}
```

we can actually impl any trait for any struct as long as either the struct or the trait is defined in our project
```rust
impl Noisy for u8 {
	fn be_noisy(&self) -> &str {"i am u8"}
}
make_noise(5_u8);
```

if we dont provide &self in the function then it becomes associated with struct instead of the instance of the struct, and essentially a static method that can be called from the struct instead of the object

```rust
struct RedFox {
	enemy: bool,
	life: u32
}
trait Noisy {
	fn be_noisy() -> &str;
}

impl Noisy for RedFox {
	fn be_noisy() -> &str {"Meow?"}
}

RedFox::be_noisy();

impl Noisy for u8 {
	fn be_noisy() -> &str {"i am u8"}
}

u8::be_noisy();
```

---
traits support inheritance
traits can inherit from other traits
for example we can have a root movement trait, and two child traits for it, fly and run 
if horse implements run trait then it must also implement the movement trait
all parent traits must be implemented if a child trait is implemented

traits can have default behaviour as well by having implementations associated with them, and then a struct implementing a child trait wont have to implement the parent trait
for the default behvaiour in the trait instead of just function signature, give entire definition
and then we dont need to provide a definition for the function when we implement it for our struct
```rust
trait Run {
	fn run(&self) {
		println!("I'm running!");
	}
}
struct Horse {}
impl Run for Horse {}
```

we cant have fields in out traits, only getters and setters

## Deriving
to print object like they would be printed in javascript
use the following annotation on top of you structs
```rust
#[derive(Debug)] // This enables using the debugging format string "{:?}"  
struct Carrot {  
    percent_left: f32,  
}
```
with this we can print an object of carrot like so
```rust
println!("{:?}", carrot);
```
and it will look like
`Carrot { percent_left: 80.0 }`

```rust
#[derive(Debug)]
struct Person {
    name: String,
    age: u32,
}

fn main() {
    let person = Person {
        name: String::from("Alice"),
        age: 30,
    };
    
    println!("{:?}", person);        // Debug format
    println!("{:#?}", person);       // Pretty-print Debug format
}
```

or we can implement the Display trait for the struct
```rust
use std::fmt;

struct Person {
    name: String,
    age: u32,
}

impl fmt::Display for Person {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "{} (age {})", self.name, self.age)
    }
}

fn main() {
    let person = Person {
        name: String::from("Alice"),
        age: 30,
    };
    
    println!("{}", person);  // Uses Display trait
}
```