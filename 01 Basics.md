```rust
fn main(){
	let bunnies = 2;
	let bunnies2: i32 = 2;
	let (cats, rats) = (6,5);
}
```

variables are immutable by default in rust

we can make variable mutable with `mut`
```rust
fn main() {  
    let mut bunnies: i32 = 5;  
    bunnies = 5;  
    println!("Hello, world!");  
}
```
### const
type definitions are required
use CAPS_SNAKE case
the value must be a constant expression that can be determined at compile time
```rust
const X: i32= 1;
```

benefits of const
- consts can be placed at module scope and be used anywhere we want
- consts are inlined at compile time so are very fast
### printing
```rust
println!("Hello, world!");
println!("{}, {}", x, y);
```
### scopes
when variables goes out of scope it is immediately dropped, without waiting for garbage collector

its possible to redeclare variables in the same scope
```rust
let x= 5;
let x=x+1;

let x=5;
let mut x=x;

let mut x=1;
let x=x;

let mut x=1;
let mut x=4;
```

its even possible to change the type of the variable when re declaring

variables must be initialized before they are used
```rust
fn main(){
	let enigma:i32;
	println!("{}", enigma); //error!
}
```

it wont work if conditionally it is being set to some value

```rust
fn main(){
	let enigma: i32;
	if true {
		enigma = 42;
	}
	println!("{}", enigma); //error!
}
```

but the following works, because the compiler can tell that the variable will have some value gauranteed
```rust
fn main(){
	let enigma: i32;
	if true {
		enigma = 42;
	} else {
		engima = 62;
	}
	println!("{}", enigma);
}
```
while in c we get default garbage value

the `mut` key word can go into pattermatching when declaring variables
```rust
let (mut a, b) = (1,2);
```

the type definitions can also be applied to the variables using similar pattern matching
```rust
let (mut a, b): (i32, i32) = (2,2);
```

### functions
lowercase snake case
and functions dont have to appear before they are called like in c
return types are like in python
```rust
fn do_stuff(qty: f64, oz: f64) -> f64 {
	return qty * oz;
}
```

short hand for return
omit the return keyword and the semicolon off the last expression
```rust
fn do_stuff(qty: f64, oz: f64) -> f64 {
	qty * oz
}
```
this means
`{return true}` === `{true}`

functions dont support variable number of arguments or different types for the same argument
but macros do like println!

### types
##### unsigned int
- u8 -> same as a byte in rust value between 0 and 255
- u16
- u32
- u64
- u128
- usize -> 
	- size of the platforms pointer size and can represent every memory address in the process
	- used to index into an array or vector
##### signed int
- i8
- i16
- i32 - > default
- i64
- i128
- isize ->
	- used to calculate differences between pointers
	- and to address every byte in a value like  a struct
##### floats
- f32
- f64 -> default
have inbuilt methods to them like powf and sqrt
```rust
pub fn print_distance((x,y): (f32, f32)) {  
    println!("Distance to the origin is {}",  
        ( x.powf(2.0) + y.powf(2.0) ).sqrt());  
}
```
so does the integers like abs
##### bools
- bool
can be type casted to integer types like so
```rust
true as u8
```
##### char 
- char is not just a char its any single unicode scalar value, which can be a character or emoji or sound byte or non printable control character
- is always 4 bytes
- an array of char is a utf-32 string but standard strings are utf-8
- strings dont use char internally
- when you want a char, its quite common to use a single character string of utf-8
### type definitions as suffixes
```rust
let x = 5u16;
let y = 3.14f32;
//or
let x = 5_u16;
let y = 3.14_f32;
```

especially useful when we have over loads of methods that take different types of floats or ints, we can then directly call them with the type we want
or to a generic function that can take many types
### tuples
```rust
let info: (u8, f64, i32) = (1,3.3,999);
```
can be de structures into variables
to access values from a tuple
```rust
info.0;
info.1;
info.2;
```
the other way is to destructure them
```rust
let (jets,fuel,ammo) = info;
```
we can de structure tuples in function parameters as well
```rust
pub fn print_distance((x,y): (f32, f32)) {}
```
### arrays
```rust
let buf = [1,2,3];
```
or through a expression [value, length]
```rust
let buf = [0;3]; // an array of size 3 filled with 0
```
type annotation for an array looks like the same as well
```rust
let buf: [u8; 3] = [1,2,3];
```
to index into an array
```rust
buf[0];
```
arrays are limited to a size of 32 because arrays live on the stack, for larger use cases use Vec
we can get the length of an array with
```rust
buf.len();
```

## String
there are 6 types of string in rust, we care about 2
- &str 
	- borrowed string slice
	- `let m: &str = "literal";`
	- the data in a borrowed string slice can not be modified
	- its just a pointer to some bytes and the length
	- subset of String
- String
	- can be modified, use when you want to do manipulations
	- is a pointer to some bytes, a length and capacity
	- `let m: String = "somestring".to_string();`
	- `let m: String = String::from("some thing");`
	- we can index to get characters because strings are UTF-8 and stored as bytes in the memory, some characters are more than a single Byte, if we were to use the array notation we would get  a single byte
	- if we want to index into the bytes we can do
		- `word.bytes()` this returns an indexable vector of utf-8 bytes
		- `word.chars()` this returns an iterable over unicode scalars, but some characters are more than a single unicode scalars long, like alphabets with accents or Kanji characters
		- these unicode scalars when put together form a grapheme, kanji that looks like characters, and if coming from other language this is what you would assume an interation over word.chars() would get you but it doesnt
		- to get i-th character from chars
		- `let c: Option<char> = word.chars().nth(0);`
		- this returns an option of character
		- we can access a character from an option with
		- `print!("{}", c.unwrap());`
		- but unwrap throws runtime error if null
		- we can do pattern matching to handle it gracefully
```rust
let c: Option<char> = word.chars().nth(0);
match c{
	Some(ch) => print!("{}", ch),
	None => print!("none")
}
```

to work with graphemes we need to use unicode-segmentation crate
```rust
use unicode_segmentation::UnicodeSegmentation;

let s = "é🇺🇸";
for grapheme in s.graphemes(true) {
    println!("{}", grapheme);
}
```
Prints each user-perceived character correctly

## appending to string
```rust
let mut s = String::from("hello");
s.push_str(" world");
println!("{}", s); 

s.push('!'); 
println!("{}", s);

s += " world"; 
println!("{}", s);
```

ownership when appending strings
```rust
let s1 = String::from("hello");
let s2 = " world";
let s3 = s1 + s2; // s1 is moved here, can't use it anymore
println!("{}", s3); // "hello world"
```
s1 cant be used again after s3 declaration
because The `+` operator takes ownership of the left operand (`s1`) and moves it
if we want to preserve s1 
```rust
let s1 = String::from("hello");
let s2 = " world";
let s3 = s1.clone() + s2;

println!("{}", s1); // ✅ Works: "hello"
println!("{}", s3); // ✅ Works: "hello world"
```
## Control flow
##### Conditionals
```rust
if num == 5 {} else if num == 6 {} else {}
if ( num == 5 ){}
```
everything after the if and before curly brace is the conditional expressions, so no need of parenthesis, you can add if you want but it leads to a warning
if is an expression in rust, not a statement, expressions return a value, statements dont
this means we can do this
```rust
let m: &str = if num == 5 {"five"} else {"other"};
```
we have to use the return short hand, we cant use return statements
all the blocks must return the same type
there is a semicolon at the end of the if expressions but thats because its part of the declaration statement
braces are not optional in rust
there are no ternary expressions, if does its job
##### loops
1. unconditional loops

```rust
loop {
	break;
}
```

but what if we want to break out of multiple loops, then we need to use lables
```rust
'bob: loop{
	loop{
		loop{
			break `bob;
		}	
	}
}
```

when this hits the break statement we will break out of the loop labeled as bob
same is true of continue;

without the label we only break out of the inner most loop

```rust
loop{
	continue;
}
```

```rust
`bob: loop{
	loop{
		loop{
			continue `bob;
		}
	}
}
```

2. while loops
everything is similar except they also break out when the condition becomes false, and the condition must necessarily evaluate to true or false, rust wont coerce values to truthy or falsey
```rust
while dizz() {}
```

while is just syntactic sugar over
```rust
loop{
	if !dizzy() {break}
	//other stuff
}
```
3. for loops
over a list
```rust
for num in [7,8,9].iter(){}
```
we can do pattern matching when iterating
```rust
let array = [(1,2),(3,4)];
for (x,y) in array.iter(){}
```
iterating over range
```rust
for i in 0..50{} // 50 is exclusive
for i in 0..=50{} // 50 is inclusive
```
iterating over string
```rust
let s = "hello";
for c in s.chars() {
    println!("{}", c);
}
for (i,c) in s.chars().enumerate() {
	
}
let s = "hello";
for (i, c) in s.char_indices() {
    println!("Index {}: {}", i, c);
}

```