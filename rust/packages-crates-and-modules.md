# Packages, Crates and Modules

## Packages and Crates

A crate is the smallest amount of code that the Rust compiler considers at a time.

Crate can contain modules, and the modules may be defined in other files that get compiled with the crate.

* Binary crate

  Each must have a function called `main` that defines what happens when the executable runs.

* Library crate

  Most of the time when Rustaceans say "crate", they mean library crate. Same meaning as "Library"

The crate root is a source file that the Rust compiler starts from and makes up the root module of your crate.

Cargo is actually a package that contains the binary crate for the command-line tool you’ve been using to build your code. The Cargo package also contains a library crate that the binary crate depends on. Other projects can depend on the Cargo library crate to use the same logic the Cargo command-line tool uses.

A package is a bundle of one or more crates that provides a set of functionality. A package contains a `Cargo.toml` that describes how to build those crates.

A package can contain as many binary crates as you like, but at most only one library crate. A package must contain at least one crate, whether that’s a library or binary crate.

Cargo follows a convention  that `src/main.rs` is the crate root of a binary crate with the same name as the package. Likewise, Cargo knows that if the package directory contains `src/lib.rs`, the package contains a library crate with the same name as the package, and `src/lib.rs` is its crate root .

A package can have mutiple binary crates by placing files in the `src/bin` directory: each file will be a separate binary code.

