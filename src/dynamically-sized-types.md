# Dynamically Sized Types

Most types have a fixed size that is known at compile time and implement the
trait [`Sized`][sized]. A type with a size that is known only at run-time is
called a _dynamically sized type_ (_DST_) or, informally, an unsized type.
[Slices] and [trait objects] are two examples of <abbr title="dynamically sized
types">DSTs</abbr>. Such types can only be used in certain cases:

* [Pointer types] to <abbr title="dynamically sized types">DSTs</abbr> are
  sized but have twice the size of pointers to sized types
    * Pointers to slices also store the number of elements of the slice.
    * Pointers to trait objects also store a pointer to a vtable.
* <abbr title="dynamically sized types">DSTs</abbr> can be provided as
  type arguments when a bound of `?Sized`. By default any type parameter
  has a `Sized` bound.
* Traits may be implemented for <abbr title="dynamically sized
  types">DSTs</abbr>. Unlike type parameters `Self: ?Sized` by default in trait
  definitions.
* Structs may contain a <abbr title="dynamically sized type">DST</abbr> as the
  last field, this makes the struct itself a
  <abbr title="dynamically sized type">DST</abbr>.

Notably: [variables], function parameters, [const] and [static] items must be
`Sized`.

[sized]: special-types-and-traits.md#sized
[Slices]: types/slice.md
[trait objects]: types/trait-object.md
[Pointer types]: types/pointer.md
[variables]: variables.md
[const]: items/constant-items.md
[static]: items/static-items.md
