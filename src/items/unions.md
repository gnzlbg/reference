# Unions

> **<sup>Syntax</sup>**\
> _Union_ :\
> &nbsp;&nbsp; `union` [IDENTIFIER]&nbsp;[_Generics_]<sup>?</sup> [_WhereClause_]<sup>?</sup>
>   `{`[_StructFields_] `}`

A union declaration uses the same syntax as a struct declaration, except with
`union` in place of `struct`.

```rust
#[repr(C)]
union MyUnion {
    f1: u32,
    f2: f32,
}
```

The key property of unions is that all fields of a union share common storage.
As a result writes to one field of a union can overwrite its other fields, and
size of a union is determined by the size of its largest field.

## Initialization of a union

A value of a union type can be created using the same syntax that is used for
struct types, except that it must specify exactly one field:

```rust
# union MyUnion { f1: u32, f2: f32 }
#
let u = MyUnion { f1: 1 };
```

The expression above creates a value of type `MyUnion` and initializes the
storage using field `f1`. The union can be accessed using the same syntax as
struct fields:

```rust,ignore
let f = u.f1;
```

## Reading and writing union fields

Unions have no notion of an "active field". Instead, every union access just
interprets the storage at the type of the field used for the access. Reading a
union field reads the bits of the union at the field's type. Fields might have a
non-zero offset (except when `#[repr(C)]` is used); in that case the bits
starting at the offset of the fields are read. It is the programmer's
responsibility to make sure that the data is valid at the field's type. Failing
to do so results in undefined behavior. For example, reading the value `3` at
type `bool` is undefined behavior. Effectively, writing to and then reading from
a `#[repr(C)]` union is analogous to a [`transmute`] from the type used for
writing to the type used for reading.

Consequently, all reads of union fields have to be placed in `unsafe` blocks:

```rust
# union MyUnion { f1: u32, f2: f32 }
# let u = MyUnion { f1: 1 };
#
unsafe {
    let f = u.f1;
}
```

Writes to `Copy` union fields do not require reads for running destructors, so
these writes don't have to be placed in `unsafe` blocks

```rust
# union MyUnion { f1: u32, f2: f32 }
# let mut u = MyUnion { f1: 1 };
#
u.f1 = 2;
```

Commonly, code using unions will provide safe wrappers around unsafe union
field accesses.

## Pattern matching on unions

Another way to access union fields is to use pattern matching. Pattern matching
on union fields uses the same syntax as struct patterns, except that the pattern
must specify exactly one field. Since pattern matching is like reading the union
with a particular field, it has to be placed in `unsafe` blocks as well.

```rust
# union MyUnion { f1: u32, f2: f32 }
#
fn f(u: MyUnion) {
    unsafe {
        match u {
            MyUnion { f1: 10 } => { println!("ten"); }
            MyUnion { f2 } => { println!("{}", f2); }
        }
    }
}
```

Pattern matching may match a union as a field of a larger structure. In
particular, when using a Rust union to implement a C tagged union via FFI, this
allows matching on the tag and the corresponding field simultaneously:

```rust
#[repr(u32)]
enum Tag { I, F }

#[repr(C)]
union U {
    i: i32,
    f: f32,
}

#[repr(C)]
struct Value {
    tag: Tag,
    u: U,
}

fn is_zero(v: Value) -> bool {
    unsafe {
        match v {
            Value { tag: I, u: U { i: 0 } } => true,
            Value { tag: F, u: U { f: 0.0 } } => true,
            _ => false,
        }
    }
}
```

## References to union fields

Since union fields share common storage, gaining write access to one field of a
union can give write access to all its remaining fields. Borrow checking rules
have to be adjusted to account for this fact. As a result, if one field of a
union is borrowed, all its remaining fields are borrowed as well for the same
lifetime.

```rust,ignore
// ERROR: cannot borrow `u` (via `u.f2`) as mutable more than once at a time
fn test() {
    let mut u = MyUnion { f1: 1 };
    unsafe {
        let b1 = &mut u.f1;
                      ---- first mutable borrow occurs here (via `u.f1`)
        let b2 = &mut u.f2;
                      ^^^^ second mutable borrow occurs here (via `u.f2`)
        *b1 = 5;
    }
    - first borrow ends here
    assert_eq!(unsafe { u.f1 }, 5);
}
```

As you could see, in many aspects (except for layouts, safety and ownership)
unions behave exactly like structs, largely as a consequence of inheriting
their syntactic shape from structs. This is also true for many unmentioned
aspects of Rust language (such as privacy, name resolution, type inference,
generics, trait implementations, inherent implementations, coherence, pattern
checking, etc etc etc).

[IDENTIFIER]: ../identifiers.md
[_Generics_]: generics.md
[_WhereClause_]: generics.md#where-clauses
[_StructFields_]: structs.md
[`transmute`]: ../../std/mem/fn.transmute.html
