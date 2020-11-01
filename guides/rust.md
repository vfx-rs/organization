Introduction
============

Rust code should always look like the C++ code (with a few exceptions). For example, Rust (mostly) does not implement function overloading.

Read https://doc.rust-lang.org/nomicon/ffi.html for more information about how to best write Rust code on the ffi boundary.

Formatting
==========

All code should be formatted with `rustfmt`, and preferably formatted on saving.

Code Quality
============

All code must pass `clippy` without errors or warnings. Clippy will catch easy mistakes that can make the code unreadable.

Safety/Soundness
================

Even though there will be plenty of `unsafe {}` code, the Rust wrapper must make sure that the unsafe block will not produce unsound or undefined behaviour.

Example
-------

### Bad

```rust
fn do_thing(thing: i32) {
    // If thing is greater than 12, then do_thing will access unallocated memory.
    unsafe { ffi::do_thing(thing) }
}
```

### Good

```rust
fn do_thing(thing: i32) -> Result<(), Error> {
    if thing > 12 {
        return Err(Error::new("thing must be less than or equal to 12."));
    }

    unsafe { ffi::do_thing(thing); }

    Ok(())
}
```

Error Handling
==============

All functions that can have an error must return a `Result<T, E>` regardless of how the wrapped library handles errors.

Examples
--------

### C++

```cpp
void do_thing(Status* status);
```

### Rust

```rust
fn do_thing() -> Result<(), Error>;
```

### C++

```cpp
bool do_thing();

if (!do_thing()) {
    Error err = get_error();
}
```

### Rust

```rust
fn do_thing() -> Result<(), Error>;
```

### C++

```cpp
std::string err();
void do_thing(&err);

if (!err.empty()) {}
```

### Rust

```rust
fn do_thing() -> Result<(), Error>;
```

Panics
======

Panics from Rust must never go across the C boundary because it is undefined behaviour. If code can panic, then it should be wrapped. More information can be found at https://doc.rust-lang.org/nomicon/ffi.html#ffi-and-panics.

```rust
fn do_thing() -> Result<(), Error> {
    let result = std::panic::catch_unwind(|| {
        maybe_panic();
    })

    result
}
```
