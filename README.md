# Jake's coding style guide

An unsorted list of things I find to be good style in code and design. This page will evolve over time. Feel free to take this and use it as an input into whatever AI you use before asking for reviews or quote it at people. Very few of these items will be hard rules, but you should have good rationale for ignoring them.

1. When using `thiserror`, do not put debug types in the error message. Any error message interpolation should use use things that implement the display trait.
    - Why: error messages should be human readable. If they are going to some kind of structured return or telemetry that gets serialized that should be a different function or representation.
1. Implementors of the display trait should not use debug types in their implementation.
    - Display should always be human readable. Debug types are not always human readable and are often much larger. Therefore we should prefer human readable representations when we are presenting something to humans rather than machines.
1. Do not use array like types to nest Error-like objects in error variants. This bloats the size of the error object, can make them unreadable, and often indicates poor error handling. Example:

   ```rs
   #[derive(thiserror::Error, Debug)]
   pub enum MyError {
       #[error("I'm going to output every single thing in {field} ")]
       VariantA {
           field: Vec<tonic::Status>
       }
   }
   ```
1. Do not use tuples as return types unless the returned object holds meaning as a tuple itself. So that's to say

   ```rs
   fn get_point() -> (u8, u8) {
       // ..
   }
   ```

is valid because `(u8, u8)` can be well understood to be a point. However the below is is not valid.

   ```rs
   fn pour_coffee() -> (JoinHandle<()>, bool) {
       // ..
   }
   ```

because a tuple of `JoinHandle` and `bool` are disjoint and don't have a commonly understood meaning as a tuple. If the two really are related define an object that makes the grouping obvious and return that instead.
1. All public types and functions should have rustdoc.
1. Do not add rustdoc to implementors of a trait.
    - Why: trait implementors use the trait definintions rustdoc as its own. If you are redefining the rustdoc then you are describing functionality that should not be a behavior of that trait or leaking implementation details that should not impact the caller.
1. Avoid "Section" comments in code. If it's a section, it should probably be a private function.
1. Crates in rust (`lib` and `binary`) should use kebab casing (`this-is-a-crate`) for their names.
    - This is the only part of the [rust naming spec](https://rust-lang.github.io/api-guidelines/naming.html) that is unclear.
    - I elect to follow some of the well defined crates in the rust ecosystem like `tokio-core`.
    - This is true for both `lib` and `binary` crates because you can have a lib nested in a binary. Therefore to avoid a further lack of clarity these two should be the same.
1. Avoid `debug` representation in tracing logs at the `INFO` and `ERROR` levels. These representations should be reserved for `DEBUG` level traces.
    - Why: Debug representation are not always human readable and can lead to a lot of log bloat.
    - If you really need something that is currently "Debug only"  in a log implement either the `std::fmt::Display` trait, or a custom to string [extension trait](http://xion.io/post/code/rust-extension-traits.html).
1. When using tracing, avoid re-logging errors at every level. Errors should only be logged at the lowest level. This prevents log spew. From the below example you can see only "function b" is logging its error.

   ```rs
   fn a() -> Result<(), MyError> {
       b()
   }

   fn b() -> Result<(), MyError> {
       //something fails
       error!("I called an API that failed");
   }

   // as an extension to this, this only applies to functions you own. If you own `mod A`, but not `mod B` even if it is logging internally, you should not make the assumption that is. You should log something when it errors and if nessecary convert the error to your own owned type.
   mod b {
       fn lib_crate_fn() -> Result<(), std::io::Error> {
           // Am I logging? Who knows
           return // some kind of io::Error;
       }
   }
   mod a {
       fn my_fn() -> Result<(), MyError> {
           if let Err(e) = lib_crate_fn() {
               error!("yep, i hit an error");
               return Err(MyError::from(e));
           }
           // ...
       }
   }
   ```
1. I just want more to see how things will format.
