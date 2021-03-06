<p align="center">
    <img src="./resource/Gutenberg_logo.png" width="250px" alt="The Gutenberg logo" />
</p>

---

# The Gutenberg post parser

[Gutenberg] is a new post editor for the [WordPress] ecosystem. A post
has always been HTML, and it continues to be. The difference is that
the HTML is now annotated. Like most annotation languages, it is
located in comments, like this:

```html
<h1>Famous post</h1>

<!-- wp:component {attributes: "as JSON"} -->
lorem ipsum
<!-- /wp:component -->
```

The parser analyses a post and generates an Abstract Syntax Tree (AST)
of it. The AST is then accessible to many languages through bindings.

## Platforms and bindings, aka targets

The parser aims at being used on different platforms, such as: the Web
(within multiple browsers), Web applications like [Electron], native
applications like macOS, iOS, Windows, Linux etc.

Thus, the parser can be compiled as:

  * A [binary](#binary),
  * A [static library](#static-library),
  * Can be embedded in any Rust projects,
  * A [WebAssembly binary](#webassembly),
  * An [ASM.js module](#asmjs),
  * A [NodeJS native module](#nodejs),
  * A [C header](#c),
  * A [PHP extension](#php),
  * And soon more.

This project uses [Justfile] as an alternative to Makefile. Every
following command will use `just`, you might consider to install
it. To learn about all the commands, just `just --list`.

**Note**: Right now, this project needs `rustc` nightly to compile the
WASM target. This target should switch to stable in a couple of
months. Since then, be sure to run the latest nightly version with
`rustup update nightly`.

### Binary

To compile the parser to a binary, run:

```sh
$ just build-binary
$ ./target/release/gutenberg-post-parser --emit-json tests/fixtures/gutenberg-demo.html
```

### Static library

To compile the parser to a static library, run:

```sh
$ just build-library
$ ls target/release/
```

### WebAssembly

To compile the parser to a [WebAssembly] binary, run:

```sh
$ just build-wasm
$ ./bindings/wasm/bin/gutenberg-post-parser --emit-json tests/fixtures/gutenberg-demo.html
```

If you would like to test directly in your browser, run:

```sh
$ just build-wasm
$ just start-wasm-server
$ open localhost:8888
```

Learn more about the [WebAssembly binding](./bindings/wasm/).

### ASM.js

To compile the parser to an [ASM.js] module, run:

```sh
$ just build-asmjs
$ just start-asmjs-server
$ open localhost:8888
```

The ASM.js module is slower than the WebAssembly binary, but it is
useful for Internet Explorer compatibility, or any browser that does
not support WebAssembly. Remember that ASM.js is just a JavaScript
file.

Learn more about the [ASM.js binding](./bindings/asmjs/).

### NodeJS

To compile the parser to a [NodeJS] native module, run:

```sh
$ just build-nodejs
$ ./bindings/nodejs/bin/gutenberg-post-parser --emit-json tests/fixtures/gutenberg-demo.html
```

Learn more about the [NodeJS binding](./bindings/nodejs/).

### C

To compile the parser to a [C header][C], run:

```sh
$ just build-c
$ ./bindings/c/bin/gutenberg-post-parser tests/fixtures/gutenberg-demo.html
```

### PHP

To compile the parser to a [PHP extension][PHP], run:

```sh
$ just build-php
$ ./bindings/php/bin/gutenberg-post-parser --emit-debug tests/fixtures/gutenberg-demo.html
```

To load the extension, add `extension=gutenberg_post_parser` in the
`php.ini` file (hint: Run `php --ini` to locate this configuration
file), or run PHP such as `php -d extension=gutenberg_post_parser
file.php`.

Learn more about the [PHP binding](./bindings/php/).

## Performance and guarantee

The parser guarantees to never copy the data in memory while
analyzing, which makes it fast and memory efficient.

### WASM binary

[A yet-to-be-official benchmark][gutenberg-parser-comparator] is used
to compare the performance of the actual Javascript parser against the
Rust parser compiled as a WASM binary so that it can run in the
browser. Here are the results:

| file | Javascript parser (ms) | Rust parser as a WASM binary (ms) | speedup |
|-|-|-|-|
| [`demo-post.html`] | 13.167 | 0.43 | × 31 |
| [`shortcode-shortcomings.html`] | 26.784 | 0.476 | × 56 |
| [`redesigning-chrome-desktop.html`] | 75.500 | 1.811 | × 42 |
| [`web-at-maximum-fps.html`] | 88.118 | 1.334 | × 66 |
| [`early-adopting-the-future.html`] | 201.011 | 3.171 | × 63 |
| [`pygmalian-raw-html.html`] | 311.416 | 2.894 | × 108 |
| [`moby-dick-parsed.html`] | 2,466.533 | 23.62 | × 104 |

The WASM binary of the Rust parser is in average 67 times faster than
the actual Javascript implementation. The median of the speedup is 63.

### ASM.js module

ASM.js is a fallback for environments that do not support WebAssembly,
like Internet Explorer. The same benchmark is used for ASM.js than for
WASM, and compares the performance of the actual Javascript parser
against the Rust parser compiled as a ASM.js module so that it can run
in the browser. Here are the results:

| file | Javascript parser (ms) | Rust parser as an ASM.js module (ms) | speedup |
|-|-|-|-|
| [`demo-post.html`] | 15.368 | 8.556 | × 2 |
| [`shortcode-shortcomings.html`] | 31.022 | 12.146 | × 3 |
| [`redesigning-chrome-desktop.html`] | 106.416 | 388.438 | × 0.27 |
| [`web-at-maximum-fps.html`] | 82.92 | 97.898 | × 0.84 |
| [`early-adopting-the-future.html`] | 119.88 | 177.89 | × 0.67 |
| [`pygmalian-raw-html.html`] | 349.075 | 24 | × 15 |
| [`moby-dick-parsed.html`] | 2,543.75 | 1360.5 | × 2 |

The ASM.js module version of the Rust parser is in average 3 times
faster than the actual Javascript implementation. The median of the
speedup is 2.

### PHP native extension

Another benchmark has been used to compare the performance of the
actual PHP parser against the Rust parser compiled as a PHP native
extension. Here are the results:

| file | PHP parser (ms) | Rust parser as a PHP extension (ms) | speedup |
|-|-|-|-|
| [`demo-post.html`] | 30.409 | 0.127 | × 239 |
| [`shortcode-shortcomings.html`] | 76.39 | 0.220 | × 347 |
| [`redesigning-chrome-desktop.html`] | 225.824 | 0.911 | × 248 |
| [`web-at-maximum-fps.html`] | 173.495 | 0.647 | × 268 |
| [`early-adopting-the-future.html`] | 280.433 | 0.707 | × 397 |
| [`pygmalian-raw-html.html`] | 377.392 | 0.051 | × 7400 |
| [`moby-dick-parsed.html`] | 5,437.630 | 11.113 | × 489 |

The PHP extension of the Rust parser is in average 1341 times faster
than the actual PHP implementation. The median of the speedup is 347.

Note that memory limit has been hit very quickly with the PHP parser,
while the Rust parser as a PHP native extension has a small memory
footprint.

## License

The license is a classic `BSD-3-Clause`:

> New BSD License
>
> Copyright © Ivan Enderlin. All rights reserved.
>
> Redistribution and use in source and binary forms, with or without
> modification, are permitted provided that the following conditions are met:
>
>   * Redistributions of source code must retain the above copyright
>     notice, this list of conditions and the following disclaimer.
>
>   * Redistributions in binary form must reproduce the above copyright
>     notice, this list of conditions and the following disclaimer in the
>     documentation and/or other materials provided with the distribution.
>
>   * Neither the name of this project nor the names of its contributors may be
>     used to endorse or promote products derived from this software without
>     specific prior written permission.
>
> THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
> AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
> IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
> ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDERS AND CONTRIBUTORS BE
> LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
> CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
> SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
> INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
> CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
> ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
> POSSIBILITY OF SUCH DAMAGE.

[Gutenberg]: https://github.com/WordPress/gutenberg/
[WordPress]: https://wordpress.org/
[Electron]: https://github.com/electron/
[Justfile]: https://github.com/casey/just/
[WebAssembly]: http://webassembly.org/
[ASM.js]: http://asmjs.org/spec/latest/
[NodeJS]: https://nodejs.org/
[C]: https://en.wikipedia.org/wiki/C_(programming_language)
[PHP]: https://php.net/
[gutenberg-parser-comparator]: https://github.com/dmsnell/gutenberg-parser-comparator
[`demo-post.html`]: https://raw.githubusercontent.com/dmsnell/gutenberg-document-library/master/library/demo-post.html
[`shortcode-shortcomings.html`]: https://raw.githubusercontent.com/dmsnell/gutenberg-document-library/master/library/shortcode-shortcomings.html
[`redesigning-chrome-desktop.html`]: https://raw.githubusercontent.com/dmsnell/gutenberg-document-library/master/library/redesigning-chrome-desktop.html
[`web-at-maximum-fps.html`]: https://raw.githubusercontent.com/dmsnell/gutenberg-document-library/master/library/web-at-maximum-fps.html
[`early-adopting-the-future.html`]: https://raw.githubusercontent.com/dmsnell/gutenberg-document-library/master/library/early-adopting-the-future.html
[`pygmalian-raw-html.html`]: https://raw.githubusercontent.com/dmsnell/gutenberg-document-library/master/library/pygmalian-raw-html.html
[`moby-dick-parsed.html`]: https://raw.githubusercontent.com/dmsnell/gutenberg-document-library/master/library/moby-dick-parsed.html

