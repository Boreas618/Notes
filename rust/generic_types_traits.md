# Generic Data Types

```rust
fn largest<T>(list: &[T]) -> &T {
    let mut largest = &list[0];

    for item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}
```

This won't compile because not all possible types implemented `std::cmp::PartialOrd`.

If you're finding you need lots of generic types in your code, it could indicate that your code needs restructuring into smaller pieces.

We can use generic types on method definitions:

```rust
struct Point<T> {
  x: T,
  y: T,
}

impl<T> Point<T> {
  fn x(&self) -> &T {
    &self.x
  }
}
```

By declaring `T` as a generic type after `impl`, Rust can identify that the type in the angle brackets in `Point` is a generic type rather than a concrete type. We could have chosen a different name for this generic parameter than the generic parameter declared in the struct definition, but using the same name is conventional.

We can only implement a method for a specific `T`:

```rust
impl Point<f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}
```

There is no overhead when enforcing generics because we perform monomorphization of the code using generics at compile time.

# Traits

```rust
pub trait Summary {
  fn summarize(&self) -> String;
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}
```

The user must bring the trait into scope as well as the types if wanting call trait methods.

```rust
use aggregator::{Summary, Tweet};

fn main() {
    let tweet = Tweet {
        username: String::from("horse_ebooks"),
        content: String::from(
            "of course, as you probably already know, people",
        ),
        reply: false,
        retweet: false,
    };

    println!("1 new tweet: {}", tweet.summarize());
}
```

We can't implement external traits on external types. For example,  we can’t implement the `Display` trait on `Vec<T>` within our self-defined `aggregator` crate. `Display` and `Vec<T>` are both defined in the standard library and aren’t local to our `aggregator` crate. This restriction is part of a property called *coherence*, and more specifically the *orphan rule*, so named because the parent type is not present. This rule ensures that other people’s code can’t break your code and vice versa. Without the rule, two crates could implement the same trait for the same type, and Rust wouldn’t know which implementation to use.

We can have a default implementation.

```rust
pub trait Summary {
  fn summarize(&self) -> String {
    String::from("(Read more...)")
  }
}
```

To use the deafult implemetation, the `impl` cluase have a empty block between the curly braces.

Default implementation can call other methods in `trait` even if the method is not yet specified.

We can use traits to define functions that accept many different types. 

```rust
pub fn notify(item: &impl Summary) {
  println!("Breaking news ! {}", item.summarize());
}
```

A subtle difference between the trait bound syntax and the above definition:

```rust
pub fn notify(item1: &impl Summary, item2: &impl Summary) {}
```

```rust
pub fn notify<T: Summary>(item1: &T, item2: &T) {
```

We can specify multiple trait bounds with the `+` syntax:

```rust
pub fn notify(item: &(impl Summary + Display)) {
```

```rust
pub fn notify<T: Summary + Display>(item: &T) {
```

`where` clauses:

```rust
fn some_function<T, U>(t: &T, u: &U) -> i32
where
    T: Display + Clone,
    U: Clone + Debug,
{
```

The returning type can also use `impl syntax`:

```rust
fn returns_summarizable() -> impl Summary {
    Tweet {
        username: String::from("horse_ebooks"),
        content: String::from(
            "of course, as you probably already know, people",
        ),
        reply: false,
        retweet: false,
    }
}
```

However, you can only use `impl Trait` if you’re returning a single type. Even if `Tweet` and `NewsArticle` implement the same trait, it doesn't allow the possibility that a function can return either of them.

We can conditionally implement methods:

```rust
impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
}
```

