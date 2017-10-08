# Chapter 2 - FFI

This chapter deals with creating a wrapper around a C library, simply
representing the contents of the C header in Rust.

## Workflow

There are a few choices for how to proceed here. The [servo] team have crated
the [bindgen] library (and a [tutorial][bindgen_tutorial], that parses a header 
file and generates the
corresponding Rust ffi. It also works for C++, although I've never looked at
this functionality.  If you use bindgen, you don't need to read any further :).

The alternative is to read the header file, and manually copy the C types and
functions into Rust. I've done this in the past, as it helps me to understand
the C api, and I can copy docs (which is difficult for bindgen to do since docs
are not syntactically specified), but I can rely on
bindgen to generate accurate ffi, whereas by hand you can make mistakes.

There are a few options on bindgen that you will want to set, as by default it
will generate bindings for everything in the header, including other headers
that have been substituted in (e.g. you don't want libc headers - you want to
use the crate!). See [the tutorial][bindgen_tutorial] for more info.


[servo]: https://servo.org/
[bindgen]: https://github.com/rust-lang-nursery/rust-bindgen
[bindgen_tutorial]: https://rust-lang-nursery.github.io/rust-bindgen/introduction.html
