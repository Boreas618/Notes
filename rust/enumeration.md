# Enumeration

## Defining an Enum

```rust
enum IpAddrKind {
  V4,
  V6
}

struct IpAddr {
  kind: IpAddrKind,
  address: String,
}

let home = IpAddr {
	kind: IpAddrKind::V4,
	address: String::from("127.0.0.1"),
};
```

We can also put data directly into each enum variant:

```rust
enum IpAddr {
	V4(String),
	V6(String),
}
let home = IpAddr::V4(String::from("127.0.0.1"));
let loopback = IpAddr::V6(String::from("::1"));
```

Each num variant we define also becomes a function that constructs an instance of the enum. That is, `IpAddr::V4()` is a function call that takes a `String` argument and returns an instance of the `IpAddr` type.

There’s another advantage to using an enum rather than a struct: each variant can have different types and amounts of associated data.

```rust
enum IpAddr {
	V4(u8, u8, u8, u8),
  V6(String),
}
```

Stand library way of defining:

```rust
struct Ipv4Addr {
    // --snip--
}

struct Ipv6Addr {
    // --snip--
}

enum IpAddr {
    V4(Ipv4Addr),
    V6(Ipv6Addr),
}
```

We can define method on enum using `impl`:

```rust
impl Message {
  fn call(&self) {
    // method body would be defined here
  }
}

left m = Message::Write(String::from("hello"));
m.call();
```

## The `Option` Enum and Its Advantages Over Null Values

The enum is defined by the standard library. The `Option` type encodes the very common scenario in which value could be something or it could be nothing.

Rust doesn’t have the null feature that many other languages have. *Null* is a value that means there is no value there. In languages with null, variables can always be in one of two states: null or not-null.

```rust
enum Option<T> {
  None,
  Some(T),
}
```

The `Option<T>` enum is so useful that it’s even included in the prelude. You can use `Some` and `None` directly without the `Option::` prefix.

```rust
let some_number = Some(5);
let some_char = Some('e');
let absent_number: Option<i32> = None;
```

Because `Option<T>` and `T` (where `T` can be any type) are different types, the compiler won’t let us use an `Option<T>` value as if it were definitely a valid value.

So a invalid number cannot take part in the calculation with a valid number.

## The `match` Control Flow Construct

```rust
fn value_in_cents(coin: Coin) -> u8 {
  match coin {
    Coin::Penny => {
      println("Lucky penny!");
      1
    } // the comma following the arm with curly braces is optional
    Coin::Nickel => 5,
    Coin::Dime => 10,
    Coin::Quater => 25,
  }
}
```

## Patterns That Bind to Values

```rust
enum UsState {
  Alabama,
  Alaska,
  //...
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quater(state) => {
          //do something to state
          25
      }
    }
}

```

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
	match x {
		None => None,
		Some(i) => Some(i + 1),
	}
}

let five = Some(5);
let six = plus_one(five);
let none = plus_one(None);
```

Matches in Rust are *exhaustive*: we must exhaust every last possibility in order for the code to be valid. Especially in the case of `Option<T>`.

## Catch-all Patterns and the _ Placeholder

```rust
other => move_player(other)
_ => move_player
_ => () //the unint value
```

