#### no-std-tester
Originally used internally as the testing framework for my microkernel project. Before this, I tried follow another os dev [blog](https://os.phil-opp.com) to incorporate `#![feature(custom_test_framework)]`
to customize my `[no_std]` test. Unfortunately, that RFC was closed, see [discussion](https://github.com/rust-osdev/bootloader/issues/366).

#### Workflow
This little crate uses linkme to allow user to distributedly submit unit test within the crate. 
1. Define registry at crate root with `declare_registry!(global)`
2. Submit unit test anywhere within the crate. The current registry boundary is crate. Different crate can share the same registry name.
```rust
#[kernel_test(global)]
fn testme() -> bool {...}
```
3. run test: `let test_result: (usize, Vec<TestFnInfo>) = test_all!(global)`

#### Flexibility
1. if not specified, fn to be registered should have prototype `fn() -> bool`, where `true` indicates that test is passed. User can declare their own fn registry
with different prototype, for example `declare_registry(my_registry: [fn() -> Option<()>])`
2. User can also provide their own runner routine (either in form of fn path or closure) as parameter of `test_all` macro, for example
`test_all!(my_registry, |f: fn() -> Option<()>| f().is_some())`. My own usage of custom runner is that I use the catch_unwind function within my kernel to capture the
potential panic from the unit test, so that I may write `assert!` in unit test fns as people usually with std support.

#### Dependencies
`linkme` and `extern crate alloc`
