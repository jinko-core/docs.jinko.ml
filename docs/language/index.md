# The jinko programming language

`jinko` is an interpreted strongly and statically typed programming language.
It relies on a strong typechecker and a low amount of concepts in order to keep
scripts simple and correct.

The goal of `jinko` is to stay simple: The language specification should stay
tiny, with few keywords and few builtins. If possible, a lot of the
functionality should be baked-in the standard library: This helps in having
multiple possible implementations, which could implement a complete language by
respecting a small specification. The standard library should rely on the type
system as much as possible.

This chapter contains information regarding *how* to program in `jinko`.
Inside, you will find out how the type-system works, how it compares to other
languages, how to do conditionals, for loops, or any other language construct.

Keep in mind that `jinko` is still being developed: If something in this
chapter is not clear enough or seems invalid, please let us know via a
[github issue](repo.jinko.ml) or on our [matrix chat](https://matrix.to/#/#jinko-lang:matrix.org).

- [Types](types/0_builtin_types.md) 
- [Variables](variables.md)
- [Functions](functions.md)
- [Conditionals](conds.md)
- [Loops](loops.md)
- [Blocks](blocks.md)
- [Code inclusion](incl.md)
