# Chapter 3 - Type conversions

This chapter deals with passing (non-function) values between C and Rust. As we
will see, in order to write safe wrappers you will need some information that
is not encoded in C's type system.

### Simple (numeric) types

Some simple types are available in the standard library. These are (all in
`std::os::raw`)

 - c_char
 - c_double
 - c_float
 - c_int
 - c_long
 - c_longlong
 - c_schar
 - c_short
 - c_uchar
 - c_uint
 - c_ulong
 - c_ulonglong
 - c_ushort

These are basic integral types in C. Their width is not guaranteed at all,
except for some loose constraints (e.g. *sizeof(char) <= sizeof(short) <=
sizeof(int)* etc..). When wrapping these types you have a choice: either expose
c types, or wrap them in rust types and use casts. I tend to use the
latter in practice because I want my APIs to use rust types, but this means
that there is the possibility of casting errors.

When working with C libraries employing these types, portability is not a
given. Writing code that is truely flexible about the size of these types is 
hard, but in C it's even harder.

The art to writing a good ffi is to balance portability with useability (and
performance). There is `TryFrom` in the standard library (and libcore) that can 
do bounds checking for instance, but they come with a runtime
cost, so the decision whether to use them is down to the library (presently
they also require nightly rust). You also have
that integer overflows will cause a panic in debug mode (this is silent
undefined behavior for signed types in release) so you can see if they occur in
practice in your code. Unsigned types are guaranteed to wrap around.

Culturally, rust tends to emphaize safety, so you may want to consider allowing
the overhead of overflow checking. I struggle to think of a situation where a
possibly wrong answer is better than a correct answer, even if you got it
faster.

In practice in rust I work with a size that is "big enough", or the size on a
64 bit os, and then convert to the c type as the last thing before the external
call (using `as`), accepting the undefined behavior. Alternatively,
if you only want to target a specific architecture, you can just use the types
you know are correct, and the code won't compile on differing architectures
(because these types are type aliases, if you call the function `foo(i32)` as
`foo(c_int)` it will generate a types mismatch unless `type c_int = i32;`).

There are some types in `libc` that are more predictable than the standard c
types, and if you're lucky enough to be wrapping a C library using them, this
bit is much easier. For example `uint32_t` == `u32` on all architectures :D.
There is also `intptr_t` (or `ptrdiff_t`), which is guaranteed to be pointer
width (`isize` in rust), and `uintptr_t` (or `size_t`) which is the same thing
unsigned (`usize` in rust). If you write C, use these. All the above are part
of C99, so they will be available on every C platform.

### `enum`s
It's possible to create a Rust enum that matches a C enum, using the special
`repr` decorator. E.g.

```rust
#[repr(C)]
enum Example {
    Foo = 0,
    Bar,
    Baz
}
```

> **Don't** use `#[repr(C)]` on enums!

The compiler will not
check that the values of the Rust enum match the C equivalent, so you must do
this carefully. A mistake here leads to undefined behavior. If the C library
adds more variants, and these variants are not in the rust version, then this
is UB. Also, the width of enums in C is not specified. It is not even specified
that all enums must have the same width. Rust does not attempt to match the
behavior of C compilers, since it would have to implement a custom algorithm
for each compiler. As an aside, some C libraries add a dummy variant with a big
value (e.g. u32::MAX) to make sure the discriminant is at least 32 bits wide.

All of this means that it is safer to use integer types to represent C
enums in rust. Bindgen has just started doing this by default.

### `struct`s

The `struct` is a common data structure in C, where several fixed-width types
are laid out in memory next to each other, and the language facilitates
labeling and accessing the different fields.

Rust's default layout (`#[repr(Rust)]` or omitted) doesn't guarantee any
particular layout of the struct. This is to facilitate optimizations like
field reordering to save space. Therefore, there is no guaranteed ABI layout
for `repr(Rust)`.

Rust provides an alternative layout `#[repr(C)]` that lays structs out in a way
that is compatible with C. It can be thought of as saying "the  layout is part
of my API".

> **Do** use `#[repr(C)] on structs!

### `[n]` (C fixed-length arrays) -> `[T; n]`

Fixed-length arrays map nicely into rust. In this case, casting could be
expensive (if n is large) so it is more likely to be a good idea to use the
`c_*` types.

### `*

### `malloc(sizeof(T))` -> (`&'a T`, `Unique<T>`, Shared<T>, ...)

This is the first time we've seen allocation. Because the data is allocated in 

### `[T]`

Slices are easy enough to 
