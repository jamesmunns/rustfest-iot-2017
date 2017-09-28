# Cross Compilation

When developing software in a desktop or server context, the **Target** environment (where the software runs) is usually the same, or similar, to the **Host** environment (where the software is built/compiled). For interpreted languages, like Python, this is not a problem, as the local interpreter manages the differences. For compiled languages, it is necessary to have a compiler capable of running on the **Host** environment, but produce a binary usable on the **Target** environment.

For compilers like GCC, it is necessary to have a specific set of compiler tools built for the (**Host**, **Target**) set. Other compilers like LLVM (which Rust is built on top of), support multiple **Target**s with the same compiler.

Rust officially supports a number of platforms, with different levels of guaranteed functionality. A list of these platforms can be found in the official [Rust Platform Documentation](https://forge.rust-lang.org/platform-support.html).

## Triples

When talking about Hosts and Targets, we typically refer to each of them based on a "Triple", which is a short description of the CPU Architecture, the operating system, and any specific capabilities the platform supports. Definitions of triples vary a bit between LLVM and GCC, though we will mainly use the LLVM version.

A Host with an x86-64 CPU processor, running Linux, using glibc would have a triple of `x86_64-unknown-linux-gnu` (the "unknown" part here means there is no distribution specific assumptions made, and should work on any Linux system). If the Host were to use MUSL instead of glibc, it would have a triple of `x86_64-unknown-linux-musl`.

A target with an ARMv7 processor, running Linux, using glibc with hardware float support would have a triple of `armv7-unknown-linux-gnueabihf`.

## Something Similar - ARMv7HF

When compiling for the Raspberry Pi, our Host and Target are generally similar. They are both running Linux, which abstracts away much of the low level interactions with the system. Rust and LLVM know how to compile code for the Raspberry Pi, however, compiling your code is only one part of the story.

### What Rust can do, and what it can't

#### Rust Toolchains

When compiling your code, there are typically two "implicit" libraries used. `core`, and `std`.

`core` contains very low level abstractions for interacting with primitive types like `u32` and `i64`, use of types like `str` and `slice`s, `Result` and `Option` types, and more.

`std` contains higher level functionality, such as Heap allocated datatypes including `String` and `Vec`, `collection`s like `HashMap` and `BTree`, `thread`s, and even macros like `println!()`.

Instead of compiling components from `core` and `std` on every build, these components are generated as pre-built libraries, and linked into your application as necessary to save time. These components, when put together are called a `toolchain`.

In order to compile for a target, you will need to obtain a toolchain for that target. The toolchain is only specific to the target, so different hosts can use the same target toolchain.

If you have installed Rust using `rustup`, adding a an officially supported target is as simple as:

```bash
rustup target add armv7-unknown-linux-gnueabihf
```

#### Linking

Currently, Rust does not use LLVM's linker, called `lld`. There is [current work] to change this, but it is still a work in progress. In the meantime, Rust uses the GCC linker, `ld`. Unfortunately as mentioned above, GCC tools (including `ld`) are specfic to a set of (`Host` and `Target`). This means it is necessary to install a GCC toolchain for the Host and Target for your cross compilation.

[current work]: https://github.com/rust-lang/rust/issues/39915

### Not every crate is pure, statically linked Rust!

Some crates are used for wrapping existing C or C++ libraries. When compiling these crates, it is necessary to have a C/C++ compiler for the core of those libraries.

Other crates are used for wrapping Dynamically Linked Libraries, like OpenSSL or DBus. Care must be taken when compiling these libraries that the Host Environment and the Target Environment match, or the host is properly configured for the target.

## Something Different - ARM Cortex-M

Cross compiling for embedded targets, like our Hail board, is a bit different.

### `std` vs `core`

For embedded

### xargo - building `core` from scratch
### cross + xargo

- pain free embedded compilation
