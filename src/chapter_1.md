# Chapter 1 - Introduction

This chapter deals with the basics of wrapping a C library (or library with C
interface) in Rust.

It is recommended to read all the way through this document. Whilst it is
boring (hopefully not, but sorry if it is), knowing all the pitfalls is
important because safety is all or nothing. There's no TL;DR :(.

## Linking to C libraries

You'll need to tell cargo what it needs to do to link to the C library. This
might involve building a C archive, or linking to a shared lib. The [build
script guide][build_script_guide] at crates.io covers this better than I could.

## `libc` bindings

When working with C code, you will almost certainly need to use
[`libc`][libc_crate]. This
maps the operating system's implementation of libc into rust. The library
provides no code wrapping the C types and functions, it merely exposes them as
unsafe rust functions. It's up to the caller to know how to use these functions
whilst maintaining rust's safety guarantees, hence the requirement to mark
their use inside an unsafe block. Some simple types are exposed in libstd (in
[`std::os::raw`][std_os_raw]) but most will require the use of the `libc`
crate.

## Aside: Portability

If you're wrapping a C library, it is possible that it will only be available
on one platform, or offer different features depending on the platform (e.g.
Windows uses a different async model to Linux). If you need to enable or
disable sections of code depending on the platform, you can use `#[cfg(..)]` 
syntax.

## FFI Library

The first thing to do is create a library that maps entities in C to 
equivalents in rust. At this point we're faithfully representing C objects in
rust, without attempting to make them safe. This is often done in a separate
crate, so multiple rust libraries can use the same bindings. By convention, the
name of the crate will be `<name_of_c_lib>-sys`.

This is covered in chapter [2][chapter 2].

## Safe wrapper

The final step is to write a wrapper API around the C api that encodes
ownership and lifetime constraints in the type system, meaning that programs
that attempt to use C code incorrectly won't compile. Chapters 3 and onward
cover this.

[build_script_guide]: http://doc.crates.io/build-script.html
[std_os_raw]: https://doc.rust-lang.org/std/os/raw/index.html
[libc_crate]: https://crates.io/crates/libc
[chapter 2]: ./chapter_2.html
[chapter 3]: ./chapter_3.html
