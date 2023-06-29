# Packages and Crates

In Rust, a package is the highest level of organization. It is a collection of files in a directory that contains a `Cargo.toml` file. The `Cargo.toml` file is a manifest file that contains metadata about the package, including its name, version, authors, and dependencies, and instructions on how to build the package.

A package can contain one or more crates, which are units of compilation in Rust. In other words, a crate is a collection of Rust source code files that are compiled together.

There are two types of crates:

- Binary crates: These are executable programs. They require a `main` function, which is the entry point of the program. The `main` function defines what happens when the program runs. A binary crate is typically used to create applications.

- Library crates: These are collections of code that are intended to be used by other programs. They do not have a `main` function and cannot be run directly. Instead, other crates (either binary or library) can link to and use the functions and types defined in a library crate. Most of the time, when Rustaceans refer to a "crate", they mean a library crate.

The crate root is the source file that the Rust compiler starts from. It makes up the root module of your crate. For a binary crate, this is often `src/main.rs`. For a library crate, this is often `src/lib.rs`.

A package can contain multiple binary crates (each with its own crate root in the `src/bin` directory), but it can only contain one library crate. This is because the library crate is intended to define the public interface of the package, i.e., the functionality that is exposed for other packages to use.

# Modules

A module in Rust is a namespace that contains definitions of functions, types, constants, and other items. The purpose of modules is to help organize code and control the privacy of items, i.e., whether they can be used by outside code.

A crate can contain multiple modules, which can be defined in the crate root or in other modules. The module system in Rust allows you to structure your code in a hierarchical way, where each module forms a separate namespace.

For example, you could have a crate that contains a `math` module and a `physics` module. The `math` module could contain definitions for functions like `add` and `multiply`, and the `physics` module could contain definitions for functions like `calculate_force`.

In summary, Rust code is organized into packages, which contain crates, which contain modules. This structure helps to organize code, manage dependencies, and control the visibility and reuse of code.

- **Start from the crate root**: When compiling a crate, the compiler first looks in the crate root file (usually *src/lib.rs* for a library crate or *src/main.rs* for a binary crate) for code to compile.
- **Declaring modules**: In the crate root file, you can declare new modules; say, you declare a “garden” module with`mod garden;`. The compiler will look for the module’s code in these places:
  - Inline, within curly brackets that replace the semicolon following `mod garden`
  - In the file *src/garden.rs*
  - In the file *src/garden/mod.rs*
- **Declaring submodules**: In any file other than the crate root, you can declare submodules. For example, you might declare`mod vegetables;` in src/garden.rs. The compiler will look for the submodule’s code within the directory named for the parent module in these places:
  - Inline, directly following `mod vegetables`, within curly brackets instead of the semicolon
  - In the file *src/garden/vegetables.rs*
  - In the file *src/garden/vegetables/mod.rs*
- **Paths to code in modules**: Once a module is part of your crate, you can refer to code in that module from anywhere else **in that same crate**, as long as the privacy rules allow, using the path to the code. For example, an `Asparagus` type in the garden vegetables module would be found at `crate::garden::vegetables::Asparagus`.
- **Private vs public**: Code within a module is private from its parent modules by default. To make a module public, declare it with `pub mod` instead of `mod`. To make items within a public module public as well, use `pub` before their declarations.
- **The `use` keyword**: Within a scope, the `use` keyword creates shortcuts to items to reduce repetition of long paths. In any scope that can refer to `crate::garden::vegetables::Asparagus`, you can create a shortcut with `use crate::garden::vegetables::Asparagus;` and from then on you only need to write `Asparagus` to make use of that type in the scope.

For a module tree:

```pse
crate
 └── front_of_house
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment
```

The entire module tree is rooted under the implicit module named `crate`.

## Paths for referring

This the crate root. We can refer to a function by its absolute path or relative path.

- An *absolute path* is the full path starting from a crate root; for code from an external crate, the absolute path begins with the crate name, and for code from the current crate, it starts with the literal `crate`.
- A *relative path* starts from the current module and uses `self`, `super`, or an identifier in the current module.

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // Absolute path
    crate::front_of_house::hosting::add_to_waitlist();

    // Relative path
    front_of_house::hosting::add_to_waitlist();
}
```

Our preference in general is to specify absolute paths because it’s more likely we’ll want to move code definitions and item calls independently of each other.

Items in a parent module can’t use the private items inside child modules, but items in child modules can use the items in their ancestor modules. 

In order to access the `add_to_waitlist` function, we should not only mark the modules all the way to the function as `pub`, but also mark the module itself as `pub`.

### `super` 

```rust
fn deliver_order() {}

mod back_of_house {
  fn fix_incorrect_order(){
  	super::deliver_order();
	}
  
  fn cook_order() {}
}
```

### Making Structures and Enums Public

We can make part of the fields of a structure public.

```rust
mod back_of_house {
    pub struct Breakfast {
        pub toast: String,
        seasonal_fruit: String,
    }

    impl Breakfast {
        pub fn summer(toast: &str) -> Breakfast {
            Breakfast {
                toast: String::from(toast),
                seasonal_fruit: String::from("peaches"),
            }
        }
    }
}

pub fn eat_at_restaurant() {
    // Order a breakfast in the summer with Rye toast
    let mut meal = back_of_house::Breakfast::summer("Rye");
    // Change our mind about what bread we'd like
    meal.toast = String::from("Wheat");
    println!("I'd like {} toast please", meal.toast);

    // The next line won't compile if we uncomment it; we're not allowed
    // to see or modify the seasonal fruit that comes with the meal
    // meal.seasonal_fruit = String::from("blueberries");
}
```

In contrast, we can simply make all of the variants of a ennunm public by specifying `pub` in front of the `enum`. However, the default for enum variants is to be public.

## `use`

 `use` only creates the shortcut for the particular scope in which the `use` occurs.

To indicate that the function brought in is not locally defined, we have to use the parent module can call the function with `parent_mod:function`. On the other hand, when bringing in structs, enums, and other items with `use`, it’s idiomatic to specify the full path.

If the two items we intend to bring in have the same name, then we should only bring in their parents in order to distinguish them.

We can use `as` to specify a alias.

### Re-exporting

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```

Before this change, external code would have to call the `add_to_waitlist` function by using the path `restaurant::front_of_house::hosting::add_to_waitlist()`. Now that this `pub use` has re-exported the `hosting` module from the root module, external code can now use the path `restaurant::hosting::add_to_waitlist()` instead.

### Using External Packages

```rust
use rand::Rng;
use std::collections::HashMap;
```

### Nested Path

```rust
use std::{cmp::Ordering, io};
use std::io::{self, Write};
```

### The Glob Operator

```rust
use std::collections::*;
```

The glob operator is often used when testing to bring everything under test into the `tests` module;

