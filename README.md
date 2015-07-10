# Hierachical logger

Simple wrapper for the log crate that enables adding arbitrary information at
runtime for future calls to the debugging macros.

That magic behind this is a `thread_local RefCell<Vec<String>>` which when
populated will include the included identifiers, seperated with ": ".

```rust
#[macro_use] extern crate log;
#[macro_use] extern crate scoped_log;
extern crate env_logger;

fn main () {
    let _ = env_logger::init();

    scoped_info!("1");
    {
        push_log_scope!("outer-scope");
        scoped_info!("2: {}", "some args");
        if false { // Enable to test assert
            scoped_assert!(false);
            scoped_assert!(false, "I failed!");
        }
        push_log_scope!("inner-scope");
        scoped_info!(target: "some-target", "2: {}", "some args");
        Foo.foo();
    }
    scoped_info!("4");
}

#[derive(Debug)]
struct Foo;

impl Foo {
    fn foo(&self) {
        push_log_scope!("{:?}-scope", self);
        scoped_info!("3");
    }
}
```

Sample output from: `cargo run --example example && RUST_LOG=example=trace target/debug/examples/example`

```
     Running `target/debug/examples/example`
INFO:example: 1
INFO:example: outer-scope: 2: some args
INFO:example: outer-scope: inner-scope: Foo-scope: 3
INFO:example: 4
```
