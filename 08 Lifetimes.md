![[Pasted image 20251030000759.png]]
better way to imagine lifetimes
![[Pasted image 20251030000712.png]]
r points to x shares the lifetime of x and so is only valid till x is valid

instead of thinking of when the reference is valid in lines of code
think of what can it point to as in which region of memory

![[Pasted image 20251030002047.png]]
![[Pasted image 20251030003104.png]]
`&mut self` → means “temporarily borrow the iterator to call next()”.
`&'a mut self` → means “borrow the iterator for as long as `'a` (the lifetime of the underlying string)”.
![[Pasted image 20251030004901.png]]
in the above example we want the returned str to outlive both s1 and s2

without constraints
![[Pasted image 20251030005015.png]]
what we want is 
![[Pasted image 20251030005035.png]]

s1 subset of out translates to
```rust
where
	's1: 'out,
	's2: 'out,
```

```rust
fn main(){
	let ans;
	let str1 = String::from("hi");
	{
		let str2 = String::from("hello");
		ans = longest(&str1, &str2);
	}
	
	println!("{}", ans);
}

fn longest(a: &str, b:&str) -> &str {
	if a.len() > b.len() {a} else {b}
}
```
the code above doesnt work

because the string hello is owned by str2

and since we only passed the strings as slices and dont move the ownership to the longest function

the original strings were owned by str1 and str2

but str2 goes out of scope

so if ans was suppose to point to str2 that is not valid, because str2's life time ends

and we are using ans outside of the scope/ lifetime of str2

rust solves this dangling pointer problem with lifetimes

the compiler says that we are returning one of the received values in branch a of the code and the other in branch b of the code

so the life time of a may not be the same as life time of b

and if you intend to use a in place b there may be a dangling pointer problem

so instead of allowing you to write such code it makes you specify which lifetime the return is associated to
the bottleneck typically is the shorter of the two lifetimes, so it makes you specify that in the return type

the compiler makes us change our function to 
```rust
fn longest<'a>(a: &'a str, b: &'a str) -> &'a str {
	if a.len() > b.len() {a} else {b}
}
```

and with the the return value can only be used where both the lifetimes are valid
because the return value and the parameters have the same label for the lifetime

the returned reference can only be used where both the input would have valid lifetimes

what this code does is 

```rust
ans = longest(&str1, &str2);
```

it binds the lifetime of ans to the return value of the function
so answer can only be used where the return values lifetime is valid

with this if we try to use ans outside of the scope of str 2 like below
```rust
fn main(){  
    let ans;  
    let str1 = String::from("hi");  
    {        let str2 = String::from("hello");  
        ans = longest(&str1, &str2);  
    }  
    println!("{}", ans);  
}  
  
fn longest<'a>(a: &'a str, b:&'a str) -> &'a str {  
    if a.len() > b.len() {a} else {b}  
}
```

the compiler is able to say that str2 does not live long enough

str2` does not live long enough
borrowed value does not live long enough

i think it should say that ans in println!(ans) points to something that has been dropped but i did not build the language so dont @ me

we only need to use lifetimes when we are dealing with references to objects
but we need to use lifetime annotations everywhere we can have references
```rust
struct User<'a> {
	name: &'a str
}
fn main(){
	let first_name = String::from("hi my name is chaitanya");
	let user = User { name: &first_name };
}
```
this ties the lifetime of user to the lifetime of first_name

if i wanted to return the longest string and both have different lifetimes
```rust
fn main(){  
    let ans;  
    let str1 = String::from("hi");  
    {        let str2 = String::from("hello");  
        ans = longest(&str1, &str2);  
        println!("{}", ans);  
    }
}  
  
fn longest<'a, 'b>(x: &'a str , y: &'b str) -> &'a str  
where  
    'b: 'a // this means b is subset of a
{  
    if x.len() > y.len() {  
        x  
    } else {  
        y  
    }  
}
```

b must outlive a because we are returning the lifetime of a and the shorter one is the bottleneck

the syntax to use lifetime annotation with Self is
```rust
struct Parser<'a> {  
    text: &'a str,  
}  
  
impl<'a> Parser<'a> {  
    fn longest<'b>(&'a self, other: &'b str) -> &'a str  
    where        
	   'b : 'a // this means b is subset of a
    {  
        if self.text.len() > other.len() {  
            self.text  
        } else {  
            other  
        }  
    }
}  
  
fn main() {  
    let text = String::from("parsing this");  
    let parser = Parser { text: &text };  
  
    let result;  
    {        
	    let other = String::from("a much longer string here");  
        result = parser.longest(&other);  
        println!("{}", result);  
    } 
}
```

String literals have `'static` lifetime - they live for the entire duration of the program

so when we write
```rust
fn main(){
	let s = "hi";
}
```

its owned by the process because string literals are stored in the binary and are just referenced by the variables

sometimes compiler can infer lifetimes
when it cant is when we need to specify lifetimes

```rust
struct ImportantExcerpt<'a>{
	part: &'a str,
}
```

the struct object must not outlive the reference we set into part

```rust
struct ImportantExcerpt<'a>{
	part: &'a str,
}

impl<'a> ImportantExcerpt<'a> {
	fn return_part(&self, announcement: &str) -> &str{
		println!("{}", announcement);
		self.part
	}
}
```

for a method, when ever the is a reference to self in the input parameters then its lifetime is automatically applied to the return value as well

which means the returned reference is valid till the object it was returned from is valid
