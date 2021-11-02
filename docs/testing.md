# SixtyFPS tests

This documents describe the testing infrastructure of SixtyFPS

## Syntax tests

The syntax tests are testing that the compiler show the right error messages in case of error.

The syntax tests are located in `sixtyfps_compiler/tests/syntax/` and it is driven by the
[`syntax_tests.rs`](../sixtyfps_compiler/tests/syntax_tests.rs) file. More info in the comments of that file.

In summary, each .60 files have comments with `^error` like so:

```ingore
foo bar
//  ^error{parse error}
```

Meaning that there must be an error on the line above at the location pointed by the caret.

Ideally, each error message must be tested like so.

The syntax test can be run alone with

```sh
cargo test --test syntax_tests
```


## Driver tests

These tests make sure that feature in .60 behave as expected.
All the .60 files in the sub directories are going to be test by the drivers with the different
language frontends.

The `.60` code contains a comment with some block of code which is extracted by the relevant driver.

### Interpreter test

The interpreter test is the faster test to compile and run. It test the compiler and the eval feature
as run by the viewer or such. It can be run like so:

```
cargo test -p test-driver-interpreter --
```

You can add an argument to test only for particular tests.

If there is a property `test` in the last component of the file, the test will make sure this
property equal to bool.

example:

```60
Foo := Rectangle {
   // test would fail if that property was false
   property <bool> test: 1 + 1 == 2;
}
```

### Rust driver

The rust driver will compile each snippet of code and put it in a `sixtyfps!` macro in its own module
(we currently do not use these to test the sixtyfps-build crate).
In addition, if there are ```` ```rust ```` blocks in a comment, they are extracted into a `#[test]`
function in the same module. This is usefull to test the rust api.
This is all compiled in a while program, so the `SIXTYFPS_TEST_FILTER` environment variable can be
set while building to only build the test that matches the filter.
Example: to test all the layout test:

```
SIXTYFPS_TEST_FILTER=layout cargo test -p test-driver-rust
```

### C++ driver

The C++ test driver will take each .60 and generate a .h for it. It will also generate a .cpp that
includes it, and add the ```` ```cpp ```` block in the main function.
Each program is compiled separately. And then run.

Some macro like `assert_eq` are defined to look similar o the rust equivalent.

To run the test, you must make sure to first build the sixtyfps shared library:

```
cargo build --lib -p sixtyfps-cpp --features testing && cargo test -p  test-driver-cpp --
```

You can omit the first part that compiles sixtyfps-cpp if you only want to test the compiler and
the runtime library is uptodate.

Note that there are also C++ unit tests that can be run by CMake

### Node driver

This is used to test the NodeJS API. It takes the ```` ```js ```` blocks in comment and make .js file
with it that loads the .60 and runs node with it.
Each test is run in a different node process.
You need to build the node integration before running the tests, even if the change was on the compiler

```
cargo  build -p sixtyfps-node  && cargo  test -p test-driver-nodejs
```


## Doctests

```
cargo test -p doctests
```

The doctests extracts the ```` ```60 ````  from the files in the docs folder and make  sure that
the snippets can be build without errors