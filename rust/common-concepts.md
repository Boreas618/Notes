# Common Concepts

`cargo`

```rust
cargo run
cargo build
cargo check
```

3 ways of defining dependencies:

```
[dependencies]
rand = "0.3"
hammer = { version = "0.5.0"}
color = { git = "https://github.com/bjz/color-rs" }
geometry = { path = "crates/geometry" }
```

## Common Programming Concepts

```rust
fn main() {
  // immutable
  // this is a variable binding
  let a = 10;
  // mutable
  let mut b: i32 = 20;
  let mut c = 30_i32;
  // variable deconstrcuting
  let (d, mut e): (bool, bool) = (true, false);
  // const, the type should be specified
  const MAX_POINTS: u32 = 100_000;
  
  // variable shadowing
  let x = 5;
  // this is a new variable, which is also called x
  let x = x + 1;
  {
    let x = x * 2;
  }
}
```

If `a` variable is immutable, then we shouldn't change `a`.

A variable binding is a statement that binds a variable to a name. After this statement you may refer to `a` as a **variable** or a **binding** (but not a variable binding).

### Scalar Types

Rust has four primary scalar types: integers, floating-point numbers, Booleans, and characters.

Rust’s `char` type is four bytes in size and represents a Unicode Scalar Value

### Compound Types

To get the individual values out of a tuple, we can use pattern matching to decapsulate a tuple value.

```rust
let (x, y, c) = (500, 6.4 1)
```

Or

```rust
let five_hundred = x.0;
```

**Array**

```rust
let a: [i32, 5] = [1, 2, 3, 4, 5]
let a = [3; 5];
```

Try to access outside of the array will cause a runtime error.

## Functions

```rust
fn main() {
    let x = (let y = 6);
}
```

An error will yield here. Not the same as C.

## Control Flow

```rust
let number = if condition { 5 } else { 6 }
```

Return value from `loop`

```rust
let result = loop {
  counter += 1;
  if counter == 10 {
   break counter * 2;
  }
};
```

Labeled loop

```rust
'counting_up': loop {
	break 'counting_up';
}
```

`for` loop

```rust
fn main() {
  for number in (1..4).rev() {
    println!("{number}!");
  }
  println!("LIFTOFF!!!");
}
```
