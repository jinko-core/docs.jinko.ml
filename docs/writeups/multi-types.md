# Desugaring multi types in jinko

```rust
type Red;
type Blue;
type Green(is_light: bool);

type Color(Red | Green | Blue);

color: Color = Green(is_light: true);

// can get compiled/lowered to 
(color_type_mark, color_value) = (1, Green(is_light: true));

// then, when matching on `color`
switch color {
    _: Red -> {},
    _: Blue -> {},
    _: Green -> {},
}

// could get compiled/lowered to
switch color_type_mark {
    0: Red -> { /* we use color_value */ },
    1: Blue -> { /* we use color_value */ },
    2: Green -> { /* we use color_value */ },
}
```

Likewise for anonymous multi-types:

```rust
func takes_red_or_blue(color: Red | Blue, some_arg: int) {
    switch color {
        /* ... */
    }
}

// becomes
func takes_red_or_blue(color_type_mark: int, color_value: Red | Blue, some_arg: int) {
    switch color_type_mark {
        /* ... */
    }
}
```
