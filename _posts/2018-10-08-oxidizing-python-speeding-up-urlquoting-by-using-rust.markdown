---
layout: single
title: "Oxidizing Python: Speeding up URL quoting by 10x using Rust"
date:   2018-10-08 12:48:12
tags: technology python rust
header:
  overlay_image: assets/images/2018-10-08-oxidizing-python-code.png
  overlay_filter: 0.5
  show_overlay_excerpt: false
author: Markus Klein
---

## Motivation

Recently a colleague of mine told me about a small bottleneck with url quoting since we are quoting a lot of storage keys at least once when loading or storing a dataset. To speed it up, we are going to write a C-Library in Rust and invoke it from Python. If you haven't heard of Rust yet, according to [rust-lang.org](https://wiki.blue-yonder.org/pages/createpage.action?spaceKey=PD&title=www.rust-lang.org&linkCreation=true&fromPageId=40734045)

> Rust is a systems programming language that runs blazingly fast, prevents segfaults, and guarantees thread safety.

In addition, Rust ensures memory safety without garbage collection and therefore requires essentially no runtime environment. Rust also performs static linking by default, so if the user systems' `libc` is as recent as the one of the system where the code has been built on, you are good to go. Deterministic memory management is also a nice property when embedding a language into Python since we do not have to worry about the interactions between two garbage collectors. In a nutshell, Rusts integration story is the same as for plain old C, yet Rust is a modern and ergonomic language. This blog article aims to show you _how_ to call Rust from Python and has a few benchmarks to motivate _why_ you may want to do so. 

## Percent encoding

Read more about url encoding [here](https://www.w3schools.com/Tags/ref_urlencode.asp). But this blog post is not about this at all. We use the `percent-encoding` crate (Rust packages are called crates) for us. Python 3 also comes with `urllib.parse.quote` to perform exactly this job. 
    
```python
from urllib.parse import quote
quote('/El Niño/')
```

output: `'/El%20Ni%C3%B1o/`

## Benchmarking `urllib.parse.quote`

`urllib.parse.quote` can handle both, strings and byte sequences. We want to optimize performance for byte sequences; Incidentally this also eases the comparison to Rust, since strings in Rust are usually represented as UTF-8 encoded bytes. 
    
```python
from timeit import timeit
timeit("quote(input)",
        setup = "from urllib.parse import quote\ninput = '/El Niño/'.encode('utf-8')",
        number = 10000)
```

yields about 0.03 seconds for 10000 repetitions on my machine. This means an individual run takes roughly `3μs` (microseconds). `'/El Niño/'` is a bit short; Let's see how performance scales with a larger string: 
    
```python
from timeit import timeit
LOREM_IPSUM = "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod   tempor\
    incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation\
    ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in\
    voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non\
    proident, sunt in culpa qui officia deserunt mollit anim id est laborum."
SETUP = f"from urllib.parse import quote\ninput = '{LOREM_IPSUM}'.encode('utf-8')"
timeit("quote(input)", setup = SETUP, number = 10000)
```

Averaging to 40μs per run for the much larger string. Take these benchmarks with a grain of salt (both strings have a different density of characters which actually need quoting). Their only purpose is to motivate you to call into Rust from Python, not to precisely quantify the speedup. 

## Benchmarking Rust

Before we wrap the `percent-encoding` crate in a C-Interface, we should verify if the endeavor is worthwhile by looking at the time we need to encode a String in Rust. Our top level directory is going to hold the `setup.py`, it should look like more or less any other Python project. To set up our benchmark, we are going to create a subdirectory holding all rust related stuff. We do this by calling:  `cargo new --lib --name urlquote rust`

to create a crate named `urlquote` in a directory named `rust`. If you want to try this out and have `cargo` not installed you can get it [here](https://www.rust-lang.org/en-US/install.html). We want to use the `percent-encoding` crate so we need to update our dependencies in the `Cargo.toml`.
    
```ini
[dependencies]
percent-encoding = "1.0"
```

Rust's nightly toolchain comes with a built-in way to write benchmarks. To use it, we create a `benches` subdirectory within `rust` and a `lib.rs` file within it.
    
```rust
#![feature(test)]
extern crate percent_encoding;
extern crate test;
    
use test::{black_box, Bencher};
use percent_encoding::{utf8_percent_encode, DEFAULT_ENCODE_SET};
    
const LOREM_IPSUM : &str = "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod\
    tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud\
    exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in\
    reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint\
    occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.";
    
#[bench]
fn encode_el_nino_collect_to_string(b: &mut Bencher) {
    b.iter(|| {
        utf8_percent_encode(black_box("/El Niño/"), DEFAULT_ENCODE_SET).collect::<String>()
    })
}
    
#[bench]
fn encode_lorem_ipsum_collect_to_string(b: &mut Bencher) {
    b.iter(|| {
        utf8_percent_encode(black_box(LOREM_IPSUM), DEFAULT_ENCODE_SET).collect::<String>()
    })
}
```

Rather than performing the complete encoding the `utf8_percent_encode` function returns an iterator. To get a fair comparison to `urllib` we collect the iterator into an utf8 encoded owned and heap allocated `String`. The `black_box` prevents compiler optimization based on the fact that the input is constant and therefore known at compile time. Running `cargo +nightly bench` shows us the following results: 
    
```bash
test encode_el_nino_collect_to_string     ... bench:         179 ns/iter (+/- 71)
test encode_lorem_ipsum_collect_to_string ... bench:       2,444 ns/iter (+/- 164)
```

`0.18μs` and `2.4μs` is fast enough compared to `3μs` and `40μs` (a 16.7x speedup in both cases) to hint that making `utf8_percent_encode` accessible from Python may indeed be worthwhile.

## Deciding how to integrate Rust with Python

When integrating Python with Rust, or any native language for this matter, there are two basic ways to go:

  * Use the C interface of the Python interpreter. This means that Python’s API is used to define Python modules and functions and to handle conversion from native data types to Python objects. This leads to special symbols in the compiled binary library. It can be used in Python with a simple import statement, just like libraries that are written in pure Python. Crates like cpython or pyO3 support this approach.
  * Provide a C interface to your native library and integrate it into Python using cffi. This means our dynamic library does not know anything about libpython or the Python interpreter. Instead we ship a header file together with our dynamic library and use cffi to invoke the functions from Python. This approach is supported by [milksnake](https://github.com/getsentry/milksnake).

Option 1 offers great flexibility and performance in manipulating existing Python objects without the need to write unsafe code (i.e. code there the Rust compiler does not prevent you from accessing invalid memory). However, this comes at the price of linking against [libpython.so](http://libpython.so/), and so binary versions need to be created and distributed for any major Python version that needs to be supported.

Option 2 Usually requires to write unsafe code, but only requires to distribute one library for all Python versions, including the PyPy implementation of Python. Also, the resulting C-library is easily integrated in other languages, should the need arise.

Hence, I prefer option 2 for my open source projects.

## Building a dynamic C-Library

By default, Rust builds static libraries intended to be linked against other static libraries written in Rust. Since Rust does not provide a stable ABI (Application Binary Interface) we need to instruct Rust to build a dynamic C-Library instead. We can achieve this easily by editing our Cargo.toml.

```ini
[lib]
crate-type = ['cdylib']
```

## Implementing the C-Interface

To keep memory management simple, I decided to perform all the relevant allocations on the Python side. Our interface for quoting will consist of one function taking an input and an output buffer. We will also return the quoted length of the input string. This way the caller can decide if the output buffer he passed has been large enough.

In order to make our function a C function we do several things. We

  * pass the buffers as raw pointers and length, rather than Rust’s native slice type.
  * declare it with `extern "C"` to conform to the C calling conventions.
  * give it the `#[no_mangle]` attribute so the function name is identical to the symbol name for linking.

Here is the complete implementation:

```rust
extern crate percent_encoding;
use percent_encoding::{percent_encode, DEFAULT_ENCODE_SET};
use std::slice;
 
/// Fill the provided output buffer with the quoted string.
///
/// # Parameters
///
/// * input_buf: Non Null pointer to utf-8 encoded character sequence to be quoted. A terminating
///              zero is not required.
/// * input_len: Number of bytes in input_buf (Without terminating zero).
/// * output_buf: Non Null pointer to buffer which will hold the utf-8 encoded output string. The
///               buffer should be big enough to hold the quoted string. This function is not going
///               to write beyond the bounds specified by `output_len`.
/// * output_len: Length of the output buffer.
///
/// # Return value
///
/// The number of bytes requiered to hold the quoted string. By comparing `output_len` with the
/// returned value one can determine, if the provided output buffer has been sufficient.
#[no_mangle]
pub unsafe extern "C" fn quote(
    input_buf: *const u8,
    input_len: usize,
    output_buf: *mut u8,
    output_len: usize,
) -> usize {
    let input = slice::from_raw_parts(input_buf, input_len);
    let output = slice::from_raw_parts_mut(output_buf, output_len);
 
    let mut index = 0;
    let mut quoted_bytes = percent_encode(input, DEFAULT_ENCODE_SET).flat_map(str::bytes);
 
    for byte in (&mut quoted_bytes).take(output_len) {
        output[index] = byte;
        index += 1;
    }
    // Number of bytes required to hold the quoted string
    index + quoted_bytes.count()
}
```

The `unsafe` keyword indicates that calling this function with invalid arguments (e.g. an invalid pointer, or a wrong buffer length) violates memory safety and leads to undefined behaviour. Functions exported to C do not necessarily have to be unsafe. Since many implementations require dereferencing pointers, they often are.

All that is missing now is a header file. We use `cbindgen` to generate the header file from Rust code. `cbindgen` is a tool to generate headers from Rust code. It has seen a fair amount of battle testing as it is used by Mozilla to generate C++ bindings for the Rust code they introduce into Firefox. `cbindgen` can either be invoked from the command line, or be called as a library from our `build.rs` script. The `milksnake` template introduces an extra build step in `build.rs`, but I rather invoke `cbindgen` from the command line and check the header file into source control. In principle there is little wrong with having the header generated via an extra build step, but the `build.rs` is executed before `rustc` and syntax errors in Rust code preventing the header to be generated also cause `rustc` never to be executed at all and in turn prevent the developer from seeing the syntax error which caused the error in the first place. If we invoke it from the command line, we have to install the CLI tool.

```bash
cargo install cbindgen
```

Now we can run:

```bash
cbindgen --lang C
```

This automatically picks up our quote function and generates the header code below. The `lang -C` option is needed, otherwise the header will be generated for C++ and Python’s `cffi` will get confused by the `extern "C"` syntax introduced to the header file.

```c
#include <stdint.h>
#include <stdlib.h>
#include <stdbool.h>
 
/*
 * Fill the provided output buffer with the quoted string.
 *
 * # Parameters
 *
 * * input_buf: Non Null pointer to utf-8 encoded character sequence to be quoted. A terminating
 *              zero is not required.
 * * input_len: Number of bytes in input_buf (Without terminating zero).
 * * output_buf: Non Null pointer to buffer which will hold the utf-8 encoded output string. The
 *               buffer should be big enough to hold the quoted string. This function is not going
 *               to write beyond the bounds specified by `output_len`.
 * * output_len: Length of the output buffer.
 *
 * # Return value
 *
 * The number of bytes requiered to hold the quoted string. By comparing `output_len` with the
 * returned value one can determine, if the provided output buffer has been sufficient.
 */
uintptr_t quote(const uint8_t *input_buf,
                uintptr_t input_len,
                uint8_t *output_buf,
                uintptr_t output_len);
```

I have been suprised finding out that `usize` is translated to `uintptr_t` rather than `size_t`, but `cbindgen` is not mistaken. Rusts usize is actually defined as a primitive large enough to reference any location in memory. `size_t` on the other hand only needs to be large enough to hold the size of the largest object. A minor point surely, since the two are identical on most platforms, but I am happy I did not write the C header myself, if only to save me some typing.

## Integrating cargo into our build process

To execute our pure Rust benchmarks we invoked cargo directly, but now we would like to invoke it via `pip`. To this end we have to edit / create our `setup.py` file. A good starting point for setting up a Python wheel that builds Rust code is Armin Ronacher’s `milksnake`. Following this template we end up with a top level directory containing a Python wheel and a `rust` subdirectory containing the Rust crate. The top level `setup.py` invokes cargo to build the Rust crate. Here is the `setup.py` for this example:

```python
from setuptools import setup
def build_native(spec):
    # build rust library
    build = spec.add_external_build(
        cmd=['cargo', 'build', '--release'],
        path='./rust'
    )
    spec.add_cffi_module(
        module_path='urlquote._native',
        dylib=lambda: build.find_dylib('urlquote', in_path='target/release'),
        header_filename=lambda: build.find_header('native.h', in_path='.'),
        rtld_flags=['NOW', 'NODELETE']
    )
setup(
    name='urlquote',
    packages=['urlquote'],
    zip_safe=False,
    platforms='any',
    setup_requires=['milksnake', 'setuptools_scm'],
    install_requires=['milksnake'],
    use_scm_version=True,
    url='https://github.com/blue-yonder/urlquote',
    milksnake_tasks=[
        build_native
    ],
    author='Blue Yonder',
    author_email='oss@blue-yonder.com',
    description='Fast quoting and unquoting of urls.',
```

As mentioned above, we make use of milksnake, which allows us to execute this command:

```bash
pip install -e .
```

After executing it we end up with a the dynamic library and a bit of wrapper code next to our `__init__.py` in the package directory.

    .rw-r--r-- 2.1k mklein  2 Aug 15:33 __init__.py
    drwxr-xr-x    - mklein  2 Aug 15:51 __pycache__
    .rw-r--r--  186 mklein  2 Aug 15:51 _native.py
    .rw-r--r--  322 mklein  2 Aug 15:51 _native__ffi.py
    .rwxr-xr-x 268k mklein  2 Aug 15:51 _native__lib.so

We now have everything in place to actually invoke the Rust code from Python.

## Using `cffi` to consume the C-Interface

It’s time to call our C function from Python. Luckily `cffi` makes this fairly easy. We keep the buffer between function calls. Not only does this avoid one allocation, it also prevents us from having to encode the string twice (once to determine the buffer length and once to fill the buffer).

```python
from urlquote._native import ffi, lib
import six
 
# This buffer is passed to the C-Interface in order to obtain the quoted string. It will be
# reallocated automatically by `_native_quote`, if its size should not be large enough. It is ok to
# reset this buffer to a smaller value, but it always needs to be a valid buffer.
buffer = ffi.new('uint8_t[]', 1)
 
def _native_quote(value):
    """
    Urlencodes the given bytes
    """
    global buffer
    buffer_len = len(buffer)
    quoted_len = lib.quote(value, len(value), buffer, buffer_len)
    if quoted_len > buffer_len:
        # Our buffer has not been big enough to hold the quoted url. Let's allocate a buffer large
        # enough and try again.
        buffer = ffi.new('uint8_t[]', quoted_len)
        lib.quote(value, len(value), buffer, quoted_len)
 
    return ffi.string(buffer, quoted_len)
 
def quote(value):
    """
    Performs string encoding and urlencodes the given string. Always returns utf-8 encoded bytes.
    """
    if not isinstance(value, six.binary_type):
        if not isinstance(value, six.text_type):
            value = str(value)
        value = value.encode('utf-8')
 
    return _native_quote(value)
```

## Final Benchmark

Comparing the speed of Rust and Python is all fun and games, but at the end of the day, we have to compare the speed of a Python function with the speed of a Python function. Let’s see how fast we went.

```python
from timeit import timeit
timeit("quote(input)",
       setup = "from urlquote import quote\ninput = '/El Niño/'.encode('utf-8')",
       number = 10000)
```

Averaging to `1.3μs` for each execution. This is roughly a 2x speedup for the short `'/El Niño/'`. Repeating the same test with the much larger `LOREM_IPSUM` string as an input yields us an average execution time of `3.4μs` a speedup in the 10x order of magnitude.

## Summary

In short to call Rust from Python we:

  * Exported the functionality in Rust code using `#[no_mangle]` and `extern "C"`.
  * Changed the crate type to `cdylib` in `Cargo.toml`.
  * Generated header files using `cbindgen`.
  * Used `milksnake` to integrate the `cargo` into your build process.
  * Called into the generated native code using `cffi`.

Rust is a true alternative to C/C++ in the systems programming space, with many modern conveniences like a central package manager and additional security guarantees. I hope I could demonstrate that it is easy enough to integrate with your Python code so you may want to give it a try. The example presented in this blog post is open source and you can find the complete code on Github.