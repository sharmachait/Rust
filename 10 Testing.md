run tests with
> cargo test

```rust
#[test]
fn large_constant_pool(){
	assert_eq!(val1, val2);
	assert_ne!(val1, val2);
	assert!(); //for booleans
}

#[test]
#[should_panic]
fn testing_if_panics(){
	//write code that panics
}
```
but this panic case is not very useful maybe it paniced because of something other than what we were testing
we can test the panic message as well
```rust
#[test]
#[should_panic(expected="expected message")]
fn testing_if_panics(){
	//write code that panics
	panic!("expected message");
}
```