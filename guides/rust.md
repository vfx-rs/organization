Introduction
============

Rust code should always look like the C++ code (with a few exceptions). For example, Rust (mostly) does not implement function overloading.

Read https://doc.rust-lang.org/nomicon/ffi.html for more information about how to best write Rust code on the ffi boundary.

Formatting
==========

All code should be formatted with `rustfmt`, and preferably formatted on saving using the following rustfmt.toml:
```toml
edition = "2018"
max_width = 80
wrap_comments = true
```

Any new Rust project should have a copy of that rustfmt.toml placed in the project root.

Code Quality
============

All code must pass `clippy` without errors or warnings. Clippy will catch easy mistakes that can make the code unreadable.

Safety/Soundness
================

Even though there will be plenty of `unsafe {}` code, the Rust wrapper must make sure that the unsafe block will not produce unsound or undefined behaviour. This will often mean reading the C++ source and understanding the semantics at a fairly deep level.

In some cases, making a safe API will mean writing a lot of additional methods, or limiting the functionality of the C++ API to a subset that is representable in safe Rust. In these cases you might want to preserve the original API, but marked as `unsafe` in addition to the safe API. If you start finding yourself getting tied up in a tangled mess of lifetime annotations, this is a good indicator that you might need to split into safe and unsafe APIs.

Example
-------

### Bad

```rust
fn do_thing(thing: i32) {
    // If thing is greater than 12, then do_thing will access unallocated memory.
    unsafe { sys::do_thing(thing) }
}
```

### Good
```rust
use crate::error::Error;
type Result<T, E = Error> = std::result::Result<T, E>;

fn do_thing(thing: i32) -> Result<()> {
    if thing > 12 {
        return Err(Error::new("thing must be less than or equal to 12."));
    }

    unsafe { sys::do_thing(thing); }

    Ok(())
}
```

Module Structure
================

Try to preserve the structure from the C++ library where it makes sense, but don't be afraid to break structs and their impls into individual modules if the source files start to get very large.

## -sys crate import

Import the -sys crate in each module that needs it like so:
```rust
use openexr_sys as sys;

pub fn foo() {
    unsafe {
        sys::openexr_foo();
    }
}
```

## Create a prelude

Re-export the most commonly used structs, functions and traits in a `predule` module. Use your judgement as to what constitutes "most useful". Note that `Error` should NOT be included in the prelude but re-exported at the crate root to avoid polluting the global namespace. Then user code looks like this:

```rust
use openexr::prelude::*;
fn do_something() -> Result<(), openexr::Error> {
    // ...
}
```


Creating Structs
================

Always impl Drop for any C++ type that has a destructor

## Opaque Pointer
```rust
#[repr(transparent)]
pub struct Foo(pub(crate) *mut sys::Foo);

impl Drop for Foo {
    // ...
}

```

### Non-owning references
This creates an issue where the C++ layer returns a reference/pointer to an object that 
it owns - if you try and wrap that pointer in the struct above, then the Drop impl will call its destructor and you'll end up with a double free.

Use the following wrapper to create a non-owning reference to an opaque pointer type [refptr.rs](https://github.com/vfx-rs/openexr-bind/blob/main/openexr-rs/src/core/refptr.rs) (just copy this module into your project).


## Opaque Bytes
```rust
#[repr(transparent)]
pub struct Foo(pub(crate) sys::Foo);

impl Drop for Foo {
    // ...
}

```

## Value Type

Currently we need to recreate the value type in the -rs layer. We need to figure a way of adding the impl directly and just importing it from the -sys layer to save the redefinition.

```rust
pub struct Foo {
    a: i32,
    b: f32,
}

impl From<Foo> for sys::Foo {
    // ...
}

```




Methods
=======

In general, keep method names the same as the C++ methods you're wrapping. Obviously the main exception to this is that Rust methods are `snake_case` as opposed to whatever convention the target library is using. Other changes we want to make to make the Rust code idiomatic:

## Getters/setters
Infallible getters/setters should always be called `name()` and `set_name()`, respectively, regardless of what the C++ equivalents are called.

Falllible getters, i.e. ones that return a "lookup" that might fail with an `Option` or `Result` should be called `get_name()`.

## Boolean queries

Should always be called `is_thing() -> bool` rather than just `thing()` as is common in C++. For instance `is_empty()` on a Vec instead of `empty()`.

## Getting sizes/lengths

Container types should have methods for getting their size named `len()`:
```rust
pub fn len(&self) -> usize {
    let result = 0;
    unsafe {
        sys::Foo_size(self.0, &mut result);
    }
    result
}
```

Getting the `number of things` on an object should be called `num_things()`:
```rust
pub fn num_channels(&self) -> usize {
    let result = 0;
    unsafe {
        sys::OIIO_ImageSpec_nchannels(self.ptr, &mut result);
    }
    result
}
```

Enums 
=====

Strip any common prefix/suffix from enum variants to make them more readable. e.g.
```c++
enum WrapMode {
    WrapMode_CLAMP,
    WrapMode_PERIODIC
};
```
should become:
```rust
// don't forget to derive all these 
#[derive(Debug, Copy, Clone, PartialEq, Eq, PartialOrd, Ord)]
pub enum WrapMode {
    Clamp,
    Periodic,
}
```

For top-level enums you can use the cppmm attributes `CPPMM_RUSTIFY_ENUM`, `CPPMM_ENUM_PREFIX` and `CPPMM_ENUM_SUFFIX` to make this easy:
```c++
// in the binding file
// this will automtically generate a Rust enum that matches the one above
// doesn't work for enums declared in classes currently (because you can't attach attributes to them)
enum WrapMode {
    WrapMode_CLAMP,
    WrapMode_PERIODIC
} CPPMM_RUSTIFY_ENUM CPPMM_ENUM_PREFIX(WrapMode_);
```

Sentinels
=========
C++ libraries often contain sentinel values. For example, enums might be declared as:
```c++
enum WrapMode {
    WrapMode_CLAMP,
    WrapMode_PERIODIC,
    NumWrapModes
};
```
How to handle these depends on the target library. In most cases they will just be used internally for iteration. Check with the library authors (or read the code) - if the sentinels are never expected to be visible in the public API, just ignore them in the -rs layer and panic if they're encountered during conversion. 

If they are used in the public API, e.g. to signal invalid values, then it will normally be preferable to keep them out of the -rs enum and if found, convert them to an `Option` or `Result`.

The same goes for integer values, where e.g. -1 might be used to specify a value that is invalid or to be ignored. Again, convert these to use `Option`/`Result` containing only vald values instead.

Integer types
=============

Promote all integer types that represent a size or index and conceptually cannot be negative to `usize`. This entails panicking when the value is outside the range of the C++ integer. 

This does mean extra overhead for the conversions and checks, but we consider the improved ergonomics on the user side and not having to worry about overflow worth it. If profiling indicates that this is a significant cost in a particular situation, this can be reconsidered.

```rust
pub fn width(&self) -> usize {
    let mut result = 0i32;
    unsafe {
        sys::Foo_get_width(elf.ptr, &mut result);
    }
    result as usize
}

/// Sets the width.
/// 
/// # Panics
/// * If `value` is not representable as an `i32`.
/// 
pub fn set_width(&mut self, value: usize) {
    let value = value.try_into::<i32>().expect("value not representable as i32");
    unsafe {
        sys::Foo_set_width(self.ptr, value);
    }
}
```

Error Handling
==============

All functions that can have an error must return a `Result<T, E>` regardless of how the wrapped library handles errors.

## Error structure

1. Create a single error enum called `Error` in a module called `error.rs` at the top level of the crate:
```
 .
├──  Cargo.toml
├──  rustfmt.toml
├──  src
│  ├──  lib.rs
│  ├──  error.rs
```

If the crate is complex enough to warrant it, you can use the `ErrorKind` pattern, but there should always be a single top-level user-facing `Error` type.

2. Implement `std::error::Error` for the `Error` enum. You can use `thiserror` for convenience.
```rust
// src/error.rs
#[derive(thiserror::Error)]
pub enum Error {
    #[error("Something bad happened")]
    SomethingBad
}
```

You can use `anyhow` for self-contained examples if they make the code easier to understand, but only for those examples in the `examples` directory. Doc examples and tests should always use the main `Error` type.

3. Re-export the `Error` enum at crate level.
```rust
// src/lib.rs
pub use error::Error;
```

4. Create a private `Result` alias in all source files that need to use `Error`:
```rust
// src/foo.rs
use crate::error::Error;
type Result<T, E = Error> = std::result<T, E>;

// now we can just use `Result<>`
pub fn foo() -> Result<i32> {
    42
}
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
