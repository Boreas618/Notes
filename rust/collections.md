# Vec

```rust
let mut v1: Vec<i32> = Vec::new();
let v2 = vec!([1,2,3]); // macro

v1.push(4); // This is a mutable borrow

// Once we have a mutable borrow, we should not have any immutable borrow

let e1 = &v[0];
let e2:Option<&i32> = v.get(0);

// The latter have the used for preventing ileagal index
match e2 {
  Some(e2) => /*...*/ ,
  None => /*...*/
}

for i in &mut v1 {
  *i += 50;
  // This is a dereferncing. The += requires the left value to mutable and owned
}
```

# String

```rust
let s = "nmsl";

let mut s1: String = String::from(s);

s1.push_str("!");

s1.push('6');
```

The `+` operator uses the `add` method whose signature looks something like this:

```rust
fn add(self, s: &str) -> String {
```

```rust
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");

let s = s1 + "-" + &s2 + "-" + &s3;
```

```rust
let s = format!("{s1}-{s2}-{s3}");
```

It's ilegal to index into strings. If you do need to index into a string, go slicing it.

```rust
let hello = "Здравствуйте";
let s = &hello[0..4];

// This is sliced based on the types
```

```rust
for c in "Зд".chars() {
    println!("{c}");
}

for b in "Зд".bytes() {
    println!("{b}");
}
```

# Hash Map

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

let team_name = String::from("Blue");
let score = scores.get(&team_name).copied().unwrap_or(0);

for (key, value) in &scores {
  println!("{key}:{value}");
}
```

`get` returns an `Option<&V>`. `copied` returns a `Option<V>` and `unwarsp_or(0)` set the score to 0 if `scores` doesn't have an entry for the key.

For types that implement the `Copy` trait, like `i32`, the values are copied into the hash map. For owned values like `String`, the values will be moved and the hash map will be the owner of those values.

We can use `insert` to overwrite values.

```rust
scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Blue"), 25);
```

We can conditionally add a key and value only if a key isn't present.

```rust
scores.entry(String::from("Yellow")).or_insert(50);
scores.entry(String::from("Blue")).or_insert(50);
```

The `or_insert` method on `Entry` is defined to return a mutable reference to the value for the corresponding `Entry` key if that key exists, and if not, inserts the parameter as the new value for this key and returns a mutable reference to the new value. 

We can update the values based on the old values:

```rust
use std::collections::HashMap;

let text = "hello world wonderful world";

let mut map = Hash::new();

for word in text.split_whitespace() {
  let count = map.entry(word).or_insert(0);
  *count += 1;
}

println!("{:?}", map);
```

