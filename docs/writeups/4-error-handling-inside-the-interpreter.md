# Error handling inside jinko's interpreter

Since we are designing a programming language, we must take particular
attention in reporting the errors that users are encountering. This means
delivering clear and beautiful error messages, which is not the point of this
write-up, as well as handling their emission properly. Most errors are not
*fatal* to the programmer: We do not want them to stop the flow of typechecking
completely and stop the interpreter. Instead, we want to skip over the
offending instruction and check the next one. This allows for emitting multiple
errors per interpreter invocation, making for a better development experience.

However, the Rust way of handling errors is usually to stop the flow of
execution if an error arises, propagating it back to the caller using the `?`
operator. Obviously, this cannot work really well with emitting multiple
errors: As soon as one is detected, it will be propagated all the way up to the
caller, eventually short-circuiting the `main` function.

To avoid this, we currently store errors inside the various `Contexts` in the
form of an `ErrorHandler`, which is basically a glorified `Vec<jinko::Error>`
type. `jinko::Error`s are built using a builder pattern, and then stored in the
context's error handler. This causes a lot of code to look like the following:

```rust
if something_bad() {
    ctx.error(Error::new(ErrKind::TypeChecker)
		    .with_msg("typechecking error!")
		    .with_loc(self.location())
		    .with_hint(Error::hint()
				    .with_loc(dec.location())
				    .with_msg("something helpful")));
    return;
    // return nothing, since the function does not return a Result: Errors are
    // kept in the context
}
```

This produces a lot of noise, and makes for nasty code sprinkled in a lot of
places in the interpreter.

I believe we should be able to clean that up by categorizing jinko nodes as
either "Aggregation sites" or "Emission sites". Ideally, individual
instructions can simply return an error: If they are invalid, there's a good
chance that they do not want to keep typechecking and instead want to return early.

Let's take the following jinko instruction

```rust
func add(lhs: int, rhs: int) -> int { lhs + rhs }

a = 15.add(14).add("5").add(27)
//             ^ type error
```

In the above example, there's a type error: We're trying to give a string to a
function that only takes two integers as arguments. Why should we go further
down the line and try to type-check the third call to add? Since the second one
is inerently incorrect, there is absolutely nothing to be gained from analyzing
the third one. The user might have tried to give an integer instead, and will
change `"5"` to `5`, or they might have wanted to instead call an entirely
different function. Hell, maybe they forgot a call to `parse[int]()` in the
middle. That's a lot of possibilities, which we cannot reasonnably account for.
Instead, the second method call in there should return an error, which will be
propagated by the first method call back to the assignment expression, which
should then propagate it as well.

Likewise, if we turned the above assignment in an arithmetic operation like so

```rust
a = 15.add(14).add("5").add(27) * 'k'
```

There'd be no point in typechecking that the operation can actually happen. The
left hand side of the operator is in an error state, so there is absolutely no
way that this code will ever be valid.

We will thus refer to the assignment expression, the method call or the binary
operation as an *Emission Sites*. When going through the various static
analysis passes, they cannot accumulate errors: They should just propagate them
when they happen. Binary operations are a bit special in that they contain
*two* expressions to typecheck, so we might as well go and type check the right
hand of the operation for more feedback to the user.

This is opposed to *Aggregation Sites*, such as blocks of code: In a block,
multiple separate instructions need to be checked for errors. This causes the
user to want multiple errors to be reported:

```rust
{
  i1 = 15.add('5');
  i2 = println(not_a_string);
  i3 = add(invalid, amount, of_args);
}
```

These 3 instructions have completely separate error messages, which the user
would want to be reported all at once. `i1` should emit an error regarding the
type of the argument given to `add()`, just like `i2` for a different function
and different type. Finally, `i3` should error out because of the invalid amount
of arguments given to the `add()` function.

A block is thus an aggregation site. Upon encountering an error when checking
one of its instruction, it should store it, but still check the following
instruction it contains before returning in an error state.

Since blocks are the base upon which the entirety of jinko is built (they are
present in function definitions, loops, if-else blocks...), aggregating errors
only in a few places should easily offer a lot of useful feedback to the user
while limiting noise in the interpreter's code.
