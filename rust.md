# Rust

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

### Scalar Types

Rust has four primary scalar types: integers, floating-point numbers, Booleans, and characters. 

Rust’s `char` type is four bytes in size and represents a Unicode Scalar Value

### Compound Types

To get the individual values out of a tuple, we can use pattern matching to destructure a tuple value.

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

Tree to access outside of the array will cause a runtime error.

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

## Understanding Ownership

The heap is less organized: when you put data on the heap, you request a certain amount of space. The memory allocator finds an empty spot in the heap that is big enough, marks it as being in use, and returns a pointer, which is the address of that location. This process is called allocating on the heap and is sometimes abbreviated as just allocating (pushing values onto the stack is not considered allocating). Because the pointer to the heap is a known, fixed size, you can store the pointer on the stack, but when you want the actual data, you must follow the pointer.

Pushing to the stack is faster.

Accessing data in the heap is slower than accessing data on the stack because you have to follow a pointer to get there.

The main purpose of ownership is to manage heap data

### Ownershio Rules

* Each value in Rust has an owner.
* There can only be one owner at a time.
* When the owner goes out of scope, the value will be dropped.

### The `String` Type

As opposed to string literals, a second string type `String` si introduced. This type manages data allocated on the heap and as such is able to store an amount of text that is unknown to us at compile time. You can create a String from a string literal using the from function

```rust
let mut s = String::from("hello");
s.push_str(", world!");
```

With the `String` type, in order to support a **mutable**, **growable** piece of text, we need to allocate an amount of memory on the heap, unknown at compile time, to hold the contents.

In languages with a garbage collector (GC), the GC keeps track of and cleans up memory that isn’t being used anymore, and we don’t need to think about it. In most languages without a GC, it’s our responsibility to identify when memory is no longer being used and to call code to explicitly free it, just as we did to request it.

Rust takes a different path: the memory is automatically returned once the variable that owns it goes out of scope. 

When `s` goes out of scope. When a variable goes out of scope, Rust calls a special function for us. This function is called [`drop`](https://doc.rust-lang.org/stable/std/ops/trait.Drop.html#tymethod.drop), and it’s where the author of `String` can put the code to return the memory. Rust calls `drop` automatically at the closing curly bracket.

**Variables and Data Interacting with Move**

```rust
let s1 = String::from("hello");
let s2 = s1;
```

A `String ` is made up of three parts stored in the stack. The heap holds the content.

<img src="https://p.ipic.vip/evs16y.png" alt="Screenshot 2023-04-25 at 2.51.20 AM" style="zoom:25%;" />

When `s2` and `s1` go out of scope, they will both try to free the same memory. This is known as a *double free* error and is one of the memory safety bugs we mentioned previously. Freeing memory twice can lead to memory corruption, which can potentially lead to security vulnerabilities.

To ensure memory safety, after the line let s2 = s1;, Rust considers s1 as no longer valid.

Beyond shallow copy, Rust invalidates the first variable. It is called a move.

Rust will never automatically create “deep” copies of your data. Therefore, any automatic copying can be assumed to be inexpensive in terms of runtime performance.

**Variables and Data Interacting with Clone**

```rust
let s1 = String::from("hello");
let s2 = s1.clone();

println!("s1 = {}, s2 = {}", s1, s2);
```

**Stack-Only Data: Copy**

If a type implements the Copy trait, variables that use it do not move, but rather are trivially copied, making them still valid after assignment to another variable.

* All the integer types, such as `u32`.
* The Boolean type, `bool`, with values `true` and `false`.
* All the floating-point types, such as`f64`.
* The character type, `char`.
* Tuples, if they only contain types that also implement `Copy`. For example, `(i32, i32)` implements Copy, but `(i32, String)` does not.

Passing a variable to a function will move or copy, just as assignment does.

**Ownership and Functions**

```rust
fn main() {
    let s = String::from("hello");  // s comes into scope

    takes_ownership(s);             // s's value moves into the function...
                                    // ... and so is no longer valid here

    let x = 5;                      // x comes into scope

    makes_copy(x);                  // x would move into the function,
                                    // but i32 is Copy, so it's okay to still
                                    // use x afterward

} // Here, x goes out of scope, then s. But because s's value was moved, nothing
  // special happens.

fn takes_ownership(some_string: String) { // some_string comes into scope
    println!("{}", some_string);
} // Here, some_string goes out of scope and `drop` is called. The backing
  // memory is freed.

fn makes_copy(some_integer: i32) { // some_integer comes into scope
    println!("{}", some_integer);
} // Here, some_integer goes out of scope. Nothing special happens.
```

```rust
fn main() {
    let s1 = gives_ownership();         // gives_ownership moves its return
                                        // value into s1

    let s2 = String::from("hello");     // s2 comes into scope

    let s3 = takes_and_gives_back(s2);  // s2 is moved into
                                        // takes_and_gives_back, which also
                                        // moves its return value into s3
} // Here, s3 goes out of scope and is dropped. s2 was moved, so nothing
  // happens. s1 goes out of scope and is dropped.

fn gives_ownership() -> String {             // gives_ownership will move its
                                             // return value into the function
                                             // that calls it

    let some_string = String::from("yours"); // some_string comes into scope

    some_string                              // some_string is returned and
                                             // moves out to the calling
                                             // function
}

// This function takes a String and returns one
fn takes_and_gives_back(a_string: String) -> String { // a_string comes into
                                                      // scope

    a_string  // a_string is returned and moves out to the calling function
}
```

Rust has a feature for using a value without transferring ownership, called references.

## References and Borrowing

```rust
fn main() {
    let s1 = String::from("hello");

    let len = calculate_length(&s1);

    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &String) -> usize {
    s.len()
}
```

![Screenshot 2023-04-25 at 3.12.00 AM](https://p.ipic.vip/gc90t5.png)

The `&s1` syntax lets us create a reference that *refers* to the value of `s1` but does not own it

`s` is dropped when `calculate_length` ends. However, `s1` will not be dropped because `s` only refers to `s1` rather than owns it. We call the action of creating a reference *borrowing*.

But **we’re not allowed to modify something we have a reference to**.

### Mutable References

```rust
fn main() {
    let mut s = String::from("hello");

    change(&mut s);
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```

Mutable references have one big restriction: if you have a mutable reference to a value, you can have no other references to that value. 

It prevents from data races.

```rust
let mut s = String::from("hello");

let r1 = &s; // no problem
let r2 = &s; // no problem
let r3 = &mut s; // BIG PROBLEM

println!("{}, {}, and {}", r1, r2, r3);
```

**But what if `s` changes itself?** 

```rust
fn main() {
    let mut x = String::from("nmsl");
    let y = & x;
    let z = & x;
    x.push_str(",ll");
    println!("{}, {}", y, z);
}
```

```rust
cannot borrow `x` as mutable because it is also borrowed as immutable
```

Note that a reference’s scope starts from where it is introduced and continues through the last time that reference is used. For instance, this code will compile because the last usage of the immutable references, the println!, occurs before the mutable reference is introduced:

```rust
let mut s = String::from("hello");

let r1 = &s; // no problem
let r2 = &s; // no problem
println!("{} and {}", r1, r2);
// variables r1 and r2 will not be used after this point

let r3 = &mut s; // no problem
println!("{}", r3);
```

### Dangling References

```rust
fn dangle() -> &String { // dangle returns a reference to a String

    let s = String::from("hello"); // s is a new String

    &s // we return a reference to the String, s
} // Here, s goes out of scope, and is dropped. Its memory goes away.
  // Danger!
```

The solution here is to return the `String` directly.

## The Slice Type

To get the length of the first word in a `String`:

```rust
fn first_word(s: &String) -> usize {
    let bytes = s.as_bytes();
  
  	// we get a reference to the element in .enumerate()
  	// as a result, we need to use &item to destructure the tuple
    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return i;
        }
    }

    s.len()
}
```

However, the returned value is not in sync with the `String` itself. If the `String` is `clear`ed, the value has no meaning.

### String Slices

```rust
let s = String::from("hello world");

let hello = &s[0..5];
let world = &s[6..11];
```

<img src="https://p.ipic.vip/j0lah0.png" alt="Screenshot 2023-04-25 at 11.23.13 AM" style="zoom:25%;" />

We can rewrite the `first_word`:

```rust
fn first_word(s: &String) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}
```

Thus, the `word` is sync witn `s`. If `s` is `clear`ed, there's an error with `word`.

```rust
fn main() {
    let mut s = String::from("hello world");

    let word = first_word(&s);

    s.clear(); // error!
  	// Because clear needs to truncate the String, it needs to get a mutable reference. Hence, there are both mutable reference and immutable reference

    println!("the first word is: {}", word);
}
```

### String Literals as Slices

```rust
let s = "Hello, world";
```

The type of `s` here is `&str`: it’s a slice pointing to that specific point of the binary. This is also why string literals are immutable; `&str` is an immutable reference.

In Rust, string literals are of type `&str`, which stands for "string slice". A string slice is a reference to a sequence of UTF-8 encoded bytes in memory, which represents a string. 

We can create `String` from `&str`:

```rust
let my_string: &str = "Hello, world!";
```

## Structs

```rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}

fn main() {
    let user1 = User {
        active: true,
        username: String::from("someusername123"),
        email: String::from("someone@example.com"),
        sign_in_count: 1,
    };
}
```

Note that the entire instance must be mutable; Rust doesn’t allow us to mark only certain fields as mutable. As with any expression, we can construct a new instance of the struct as the last expression in the function body to implicitly return that new instance.

The `email` and `sign_in_count` is of type `String`. We want each instance of the struct owns all of its data. If we use a `&str`, the variables are not owned by the struct instance.

```rust
fn build_user(email: String, username: String) -> User {
  User {
    active: true
    username,
    email,
    sign_in_count: 1,
  }
}
```

It uses  *field init shorthand* syntax.

```rust
fn main() {
    // --snip--

    let user2 = User {
        email: String::from("another@example.com"),
        ..user1
    };
}
```

This is struct update syntax. The `..user1` must come last to specify that any remaining fields should get their values from the corresponding fields in `user1`, but we can choose to specify values for as many fields as we want in any order, regardless of the order of the fields in the struct’s definition.

Note that this is a **move**. We can no longer use `user1` as a whole as a whole after creating `user2`.

In this example, we can no longer use `user1` as a whole after creating `user2` because the `String` in the `username` field of `user1` was moved into `user2`. If we had given `user2` new `String` values for both `email` and `username`, and thus only used the `active` and `sign_in_count` values from `user1`, then `user1` would still be valid after creating `user2`. Both `active` and `sign_in_count` are types that implement the `Copy` trait, so the behavior we discussed in the [“Stack-Only Data: Copy”](https://doc.rust-lang.org/stable/book/ch04-01-what-is-ownership.html#stack-only-data-copy) section would apply.

### Tuple Structs

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

fn main() {
    let black = Color(0, 0, 0);
    let origin = Point(0, 0, 0);
}
```

## An Example Program Using Structs

```rust
struct Rectangle {
  width: u32,
  height: u32,
}

fn main() {
  let rect1 = Rectangle {
    width: 30,
    height: 50,
  };
  
  println!(
  	"The area of the rectangle is {} square pixels.",
     area(&rect1)
  );
}

fn area(rectangle: &Rectangle) -> u32 {
  rectangle.width * rectangle.height
}
```

To print out the structure we can use an output format called `Debug`.

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!("rect1 is {:?}", rect1);
}
```

We can use `{:#?}` instead of `{:?}` in the `println!` string for larger structs.

