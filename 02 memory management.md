# ownership

1. every value has 1 owner
2. when owner gets out of scope the value is dropped immediately
```rust
let s1 = String::from("abc");
let s2 = s1;

print!("{}",s1); // Error
```

we cant print s1 because the ownership of the value is transferred over to s2
the ownership is moved because String does not implement the Copy trait
the rust compiler invalidates s1 as soon as we move the ownership to s2
and since its s1 is not even mutable we cant assign it a new value either, so its just garbage

if we didnt want to move its, but only a copy of it, use the clone() method of s1
```rust
let s1 = String::from("abc");
let s2 = s1.clone();

print!("{}",s1); 
```
clone creates a clone in the heap as well and then both pointers point to different memory locations on the heap
its called "copy" when only stack data is copied and clone when heap is copied as well

###### passing arguments
```rust
let s1 = String::from("abc");

do_stuff(s1);

fn do_stuff(s: String){
	
}
```

if we pass in String like above then s1 is moved into the local variable s
meaning we can use s1 anymore
one option is to move it back when we are done with it, make it mutable and return it back from the function do_stuff

```rust
let mut s1 = String::from("abc");
s1=do_stuff(s1);

fn do_stuff(s: String) -> String{
	s
}
```

but this is not the idiomatic way of writing rust, passing ownership to a function indicates that the function is going to "consume" it

if you want to use it afterwards pass it by referencing and borrowing
###### borrowing and references
we can make a function take a reference using the &
and pass in the reference similarly
```rust
let s1 = String::from("abc");
do_stuff(&s1);

fn do_stuff(s: &String){
	
}
```
do stuff borrows a reference to the value and s1 maintains the ownership
and we can use s1 as normal
![[Pasted image 20251014092734.png]]
rust makes sure the pointers are always valid using Lifetimes
its just a rule that references must always be valid
the compiler wont let you create a reference that outlives the data it is referencing
and we can never point to null

references are immutable even if the value being referenced is mutable
```rust
let mut s1 = String.from("world");
do_stuff(&s1);

fn do_stuff(s: &String){
	s.insert_str(0, "hi, "); //Error
}
```

the above code doesnt work because s is immutable

to make it work we need to make a ==mutable reference toa  mutable value==
like so
```rust
let mut s1 = String.from("world");
do_stuff(&mut s1);

fn do_stuff(s: &mut String){
	s.insert_str(0, "hi, ");
}
```

when working with the '.' operator it automatically dereferences the pointer for us
if we wanted to manually dereference s it would look like this
```rust
(*s).insert_str(0, "hi, ");
```
it has low precedence so make sure to use braces

for all other operators we will need to manually dereference the variable
```rust
fn do_stuff(s: &mut String){
	*s = String::from("new value");
}
```
---
if we have a variable x then or types
`&x` - immutable reference - `&i32`
`&mut x` - mutable reference - `&mut i32`
`*y` - manual dereference - gives immutable and mutable access to x depending on if y was a mutable reference or not
`y.` - automatic dereference

==at any time we can only have 1 mutable reference or any number of immutable references, and it applies across all threads, but we cant have mutable references when immutable ones exist==

all this is true for values that dont implement the copy trait, if some primitive is implementing it

then when we pass it rust automatically copies it

```rust
let x = 5;
let y = x;
println!("{}", x); // Works fine! x is still valid
```

## Heap-allocated types (don't implement Copy)

Types like `String`, `Vec`, `Box`, etc
# lifetimes