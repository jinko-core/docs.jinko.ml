# Error handling in jinko

This document should not cover the usage of `Maybe[T]` as that has been previously established.

## The `Error` type

```rust
type Error[T = string](from: T);

/// The error type is generic, as long as the contained type
//implements the `as_error()` method
fn emit[T](err: Error[T]) {
    println_err(err.from.as_error())
}

// Specialization for the default implementation
fn emit[string](err: Error[string]) {
    println_err(err.from)
}
```

## Without a `Result` type

```rust
// We can use the default `string` type contained in an `Error`

// Fails if a division by 0 is performed
func div_can_fail(lhs: int, rhs: int) -> int | Error {
    if rhs == 0 {
        Error(from: "Attempting division by 0")
    } else {
        lhs / rhs
    }
}

switch div_can_fail(165, 0) {
    value: int => println("Result is {value}, yipee"),
    e: Error => e.emit(),
}

// We can also use a complex type as the error's inner
// type
type DivError;

fn as_error(err: DivError) -> string {
    "Division error"
}

func div_can_fail2(lhs: int, rhs: int) -> int | Error[DivError] {
    if rhs == 0 {
        Error(from: DivError)
    } else {
        lhs / rhs
    }
}
```

## With a `Result` type

```rust
type Result[T, E = string](Ok[T] | Error[E]);

// Using the default string as error type again
func div_can_fail3(lhs: int, rhs: int) -> Result[int] {
    if rhs == 0 {
        Error(from: "Attempting division by 0")
    } else {
        Ok(with: lhs / rhs)
    }
}

switch div_can_fail(165, 0) {
    value: Ok => println("Result is {value.get()}, yipee"),
    e: Error => e.emit(),
}
```

## Propagating the errors

Since `jinko` currently does not have a concept of postfix operators, we will
need to rely on interpreter magic to propagate errors properly.

```rust
// Assuming we have the same Result type before

func try[T, E](result: Result[T, E]) -> T {
    switch result {
	  ok: Ok => ok.get(),
	  err: Error => Jinko.caller().return_with(err), // return *from* the caller
    }
}

func faillible_fn() -> Result[int, DivError] {
    Ok(with: 3);
}

func another_faillible_fn() -> Result[int, DivError] {
    Err(from: DivError);
}

func propagate_result() -> Result[int, DivError] {
    a = faillible_fn().try();
    //  ^ Ok(int)      ^ int
    b = another_faillible_fn().try();
    //  ^ Err(...)
    // 
    // this will cause a return from the current function: try's caller

    // This instruction will never be reached, since b will always
    // be an Err[] and cause an early return. If b was valid, we would otherwise
    // just have extracted the integer and computed an addition here
    Ok(with: a + b)
}
```
