# Reference

Rust has reference `&` and dereference operator `*`, seems identical to C++'s
ones. Well, there exists difference. I learned C++ before Rust, so it confused
me a lot at beginning. Here I would clarify the concept of reference in Rust.

## Pointer types

Reference is [Pointer
type](https://doc.rust-lang.org/reference/types/pointer.html) in Rust. And in
Rust, reference need to be dereferenced before use.

```rust
fn main() {
    let v = 10;
    let ref_v = &v;
    println!("{}", v == *ref_v);
}
```

In this example, compiler would give an error if we do not add dereference `*`
operator for `ref_v`, since `v` has type `i32` while `ref_v` has type `&i32`.
In C++, reference is considered as alias in most cases. But in Rust, reference
is a separate type compared to the type it reference to.

## Avoid ref/deref noisy with auto dereference

If explicit ref/deref is strictly needed, then code would be filled with `&`
and `*`. Since we only focus on accessing the referenced object, ref/deref
operator makes the code hard to read. So Rust offers some syntax sugars to
implicitly do ref/deref.

The problem is that Rust has lots of syntax sugars, especially for auto
dereference.  However, they work in different cases and Rust does not have
strictly explanation documents for each. Or some have one, but are distributed
in different places. So you may encounter the scenario that some code seems
like not following Rust syntax rules, but it works. Then you search for hours
and find the document says it is a syntax sugar, is a special case. And in many
cases you can only find someone talks about it in a forums and may even give a
link to a RFC, which as a official document says the syntax is valid. This is
really a bad experience.

Anyway, I would try to list them all.

### `Deref`/`DerefMut` trait

It's intuitive that dereference a `*T` or `&T` returns `T`, but we may want to
implement our own pointer type. To achieve this, Rust compiler only needs to
know the dereference policy when dereferencing. So here comes
[`Deref`](https://doc.rust-lang.org/std/ops/trait.Deref.html)/[`DerefMut`](https://doc.rust-lang.org/std/ops/trait.DerefMut.html)
trait.

`Deref` trait is really simple in design.

```rust
pub trait Deref {
    type Target: ?Sized;

    // Required method
    const fn deref(&self) -> &Self::Target;
}
```

It tells which type would be dereferenced to when dereferencing. And Rust
compiler would invoke the `deref` method when dereference happens.

`DerefMut` is similar to `Deref`, but it accepts and returns `mut` one instead.

### Deref coercion

`Deref coercion` is another magic built on `Deref`/`DerefMut` trait. It needs
the type's `Deref`/`DerefMut` trait is implemented with another `Target` type.
That is, accepts `&T` and returns `&U`. This is Rust style of implicit type
cast.  One of the most common one is casting `&String` to `&str`.

```rust
fn main() {
    let s: String = String::new();

    let str: &str = s.deref();
    let str: &str = &s;

    // let str: &str = s;
}
```

Here we has `s` with type `String`. First we explicitly cast it to `&str` by
manually invoking `deref`. Next we add reference sign `&` to `s`, and Rust
implicitly invokes `deref` (or say, implicitly cast it) to get `&str`.

In Rust, deref coercion can happens in two cases. The first is when the
receiver is marked as reference and the target type is another type of
reference, like the `let str: &str = &s;` shown above. The second is when
method call happens.

### Dot operator

Dot operator `.` is used for field accessing and method call.  Rust would try
to perform auto dereference(`autoderef`) if the receiver is a pointer type.  Or
say, add an additional dereference operator `*` explicitly.

```rust
struct Location {
    name: String,
}

fn main() {
    let location = Location {
        name: String::from("China"),
    };

    let ref_location = &location;

    println!("{}", (*ref_location).name);
    println!("{}", ref_location.name);
}
```

Here both print statements works. The latter is considered as the shorthand of
the former.

Since `autoderef` dereference the variable, if its struct implements
`Deref`/`DerefMut` trait, `Deref coercion` would happen.

```rust
struct Location {
    name: String,
}

fn main() {
    let lp = Box::new(Location {
        name: "me".to_string(),
    });

    println!("{}", lp.name);
}
```

The `lp` here as a `Box` type does not has `name` field. But Rust would work
like this(don't know how compiler works, but this can explain well under Rust
grammer):

1. Add a `*` to `lp` to dereference it via autoderef. `lp => *lp`
2. Add a `&` to `lp` so the type is converted from `&Box` to `&Location` via
   `Deref coercion`. `*lp => *&lp`
3. Returns `name` with type `&String`.
4. Dereference from `&String` to `String` with the `*` added in step 1.

The autoderef here is a little bit brutal. Rust would add as much `*` as
possible if necessary.

```rust
struct Location {
    name: String,
}

fn main() {
    let lp = Box::new(Location {
        name: "me".to_string(),
    });

    let reflp = &&&&&&&lp;

    println!("{}", reflp.name);
}
```

Here we has 7 `&`, but `reflp.name` still works. That's because Rust adds 8 `&`
for it.

The mechanism applies to method call as well.

Details can be checked
[here](https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/first-edition/deref-coercions.html).
