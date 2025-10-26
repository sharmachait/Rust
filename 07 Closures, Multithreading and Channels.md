closures are anonymous functions that can borrow variables from its enclosing scope
```rust
|x,y|{x+y}
```

the types of the arguments and the return type are all inferred based on how we call this function

```rust
let add = |x,y|{x+y};

add(1,2); // returns 3
```
we can have closures without parameters as well and we can leave the block empty as well
```rust
||{}
```
and it can borrow references for values in its enclosing scope

```rust
let s = "strawberry".to_string();
let f = ||{ println!("{}",s); };

f();
```

this works because when we declared the closure it has access to variable s

this works all fun and dandy as long as the closure lives as long as the variable s does

but the compiler wont let us pass that closure to another thread because that thread may live longer than the thread the closure was created in

to do something like that 

we will need to move the ownership of the variables to the closure
this takes ownership of the variable along with it
now s is owned by the closure and will live as long as the closure lives
```rust
let s = "strawberry".to_string();
let f = move ||{ println!("{}",s); };

f();
```

closures are used to do functional style programming
```rust
let mut v: Vec<i32> = vec![1,2,3,4];
v.iter().map(|x: &i32| x*3)
.filter(|x: &i32| *x>10)
.fold(0, |acc: i32,x: i32| acc+x);
```

fold reduces the dimensions of the iterator
if you just want to turn an iterator into a vector use .collect instead

## Multithreading
```rust
use std::thread;
fn main(){
	let handle = thread::spawn(move ||{  
	    // do stuff  
	});  
	// do stuff simultaneously  
	match handle.join() {  
	    Ok(_) => {}  
	    Err(_) => {}  
	}
}
```
this is an example of OS threads which are heavy
async await are better suited for a IO tasks

the .join() method waits for the thread to finish

### Channels and Message passing between threads
in the above example we passed data from creator thread to the created thread, but how to pass data back ?
done through Channels
a channel consists of a transmitter and a receiver

```rust
fn main(){  
    let (tx, rx) = std::sync::mpsc::channel();  
  
    std::thread::spawn(move ||{  
        let val = String::from("HI");  
        tx.send(val).unwrap();  
    });  
    let received = rx.recv().unwrap();  
    println!("{:?}", received);  
}
```
rx.recv is thread blocking
after receiving the string, the main thread owns the string

if we want to produce from multiple threads, we can tx.clone() the transmitter

and we can wait for all the transmission with
```rust
for received in rx {}
```

if the original transmitter (not the clone) is not moved to any specific thread
then it will infinitely block the receiver thread
because the transmitter is not dropped from scope and rust doesnt know how many more messages are gonna be produced until there is even one transmitter still in memory
if the OG transmitter is not moved over to any other thread then we need to drop it manually to unblock the main thread at receive with
drop(tx);
after passing its clones to other threads