# Jake's coding style guide

An unsorted list of things I find to be good style in code and design. This page will evolve over time. Feel free to take this and use it as an input into whatever AI you use before asking for reviews or quote it at people. Very few of these items will be hard rules, but you should have good rationale for ignoring them.

1. When using `thiserror`, do not put debug types in the error message. Any error message interpolation should use use things that implement the display trait.
    - Why: error messages should be human readable. If they are going to some kind of structured return or telemetry that gets serialized that should be a different function or representation.
1. Implementors of the display trait should not use debug types in their implementation.
    - Display should always be human readable. Debug types are not always human readable and are often much larger. Therefore we should prefer human readable representations when we are presenting something to humans rather than machines.
1. Do not include array like types as part of an error variant. Example:
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
