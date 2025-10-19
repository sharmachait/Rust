![[Pasted image 20251011202529.png]]
for debug version
> cargo run

this above binary is stored in `target/debug`

>cargo run --release

this binary is stored in `target/release`

> cargo clippy

tells you what can be removed like return as the last statement of the function

cargo run with run time argument
> cargo run apple banana

and we can read in run time arguments like so
```rust
let args: Vec<String> = std::env::args().skip(1).collect();
```

we can skip the first element  because thats just the path of the binary in the cargo run command
which we dont have to provide
## module system
1. main.rs is the root of the exe
2. lib.rs if the root of the library
when we compile our code the name of the project becomes the name of the library and then anything in the lib.rs can be referenced with project_name::

like so

main.rs
```rust
fn main(){
	hello::greet();
} 
```
where lib.rs
```rust
pub fn greet(){
	println!("Hi!");
}
```

to refer the function other modules it should be public as all items in a  library are private by default

its cumbersome to give library path everytime, use the `use`  keyword
use brings some item from some path into some scope

main.rs
```rust
use hello::greet;

fn main(){
	greet();
} 
```

it is also how we need to bringing the std library

we can bring in crates by adding them to dependencies in the TOML file and using them like libraries
```toml
[package]  
name = "untitled"  
version = "0.1.0"  
edition = "2024"  
  
[dependencies]  
rand="0.9.2"
```

```rust
use rand::rng;

fn main(){
	greet();
} 
```