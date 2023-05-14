# Packages, Crates and Modules

## **Packages and Crates**

A crate is the smallest amount of Rust code that the compiler considers at a time. It may contain modules that the compiler compiles with the crate.

- Binary crate

  Binary crates require a function called `main` that defines what happens when the program runs.

- Library crate

  Most Rustaceans use the term "crate" to refer to library crates, which are also called "libraries".

The crate root is a source file that the Rust compiler starts from and makes up the root module of your crate.

A package is a bundle of one or more crates that provides a set of functionality. A package contains a `Cargo.toml` file that describes how to build those crates.

A package can contain multiple binary crates, but only one library crate. A package must contain at least one crate, whether that is a library or binary crate.

The library crate defines the public interface of the package. 

Cargo follows a convention that `src/main.rs` is the crate root of a binary crate with the same name as the package. If the package directory contains `src/lib.rs`, then the package contains a library crate with the same name as the package and `src/lib.rs` is its crate root.

A package can have multiple binary crates by placing files in the `src/bin` directory. Each file will be a separate binary code.



