# Statements

> **<sup>Syntax</sup>**\
> _Statement_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; `;`\
> &nbsp;&nbsp; | [_Item_]\
> &nbsp;&nbsp; | [_LetStatement_]\
> &nbsp;&nbsp; | [_ExpressionStatement_]\
> &nbsp;&nbsp; | [_MacroInvocationSemi_]


A *statement* is a component of a [block], which is in turn a component of an
outer [expression] or [function].

Rust has two kinds of statement: [declaration
statements](#declaration-statements) and [expression
statements](#expression-statements).

## Declaration statements

A *declaration statement* is one that introduces one or more *names* into the
enclosing statement block. The declared names may denote new variables or new
[items][item].

The two kinds of declaration statements are item declarations and `let`
statements.

### Item declarations

An *item declaration statement* has a syntactic form identical to an
[item declaration][item] within a [module]. Declaring an item within a statement
block restricts its scope to the block containing the statement. The item is not
given a [canonical path] nor are any sub-items it may declare. The exception to
this is that associated items defined by [implementations] are still accessible
in outer scopes as long as the item and, if applicable, trait are accessible.
It is otherwise identical in meaning to declaring the item inside a module.

There is no implicit capture of the containing function's generic parameters,
parameters, and local variables. For example, `inner` may not access
`outer_var`.

```rust
fn outer() {
  let outer_var = true;

  fn inner() { /* outer_var is not in scope here */ }

  inner();
}
```

### `let` statements

> **<sup>Syntax</sup>**\
> _LetStatement_ :\
> &nbsp;&nbsp; [_OuterAttribute_]<sup>\*</sup> `let` [_Pattern_]
>     ( `:` [_Type_] )<sup>?</sup> (`=` [_Expression_] )<sup>?</sup> `;`

A *`let` statement* introduces a new set of [variables], given by a [pattern]. The
pattern is followed optionally by a type annotation and then optionally by an
initializer expression. When no type annotation is given, the compiler will
infer the type, or signal an error if insufficient type information is
available for definite inference. Any variables introduced by a variable
declaration are visible from the point of declaration until the end of the
enclosing block scope.

## Expression statements

> **<sup>Syntax</sup>**\
> _ExpressionStatement_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [_ExpressionWithoutBlock_][expression] `;`\
> &nbsp;&nbsp; | [_ExpressionWithBlock_][expression]

An *expression statement* is one that evaluates an [expression] and ignores its
result. As a rule, an expression statement's purpose is to trigger the effects
of evaluating its expression.

An expression that consists of only a [block expression][block] or control flow
expression, if used in a context where a statement is permitted, can omit the
trailing semicolon. This can cause an ambiguity between it being parsed as a
standalone statement and as a part of another expression; in this case, it is
parsed as a statement. The type of [_ExpressionWithBlock_][expression]
expressions when used as statements must be the unit type.

```rust
# let mut v = vec![1, 2, 3];
v.pop();          // Ignore the element returned from pop
if v.is_empty() {
    v.push(5);
} else {
    v.remove(0);
}                 // Semicolon can be omitted.
[1];              // Separate expression statement, not an indexing expression.
```

When the trailing semicolon is omitted, the result must be type `()`.

```rust
// bad: the block's type is i32, not ()
// Error: expected `()` because of default return type
// if true {
//   1
// }

// good: the block's type is i32
if true {
  1
} else {
  2
};
```

## Attributes on Statements

Statements accept [outer attributes]. The attributes that have meaning on a
statement are [`cfg`], and [the lint check attributes].

[block]: expressions/block-expr.md
[expression]: expressions.md
[function]: items/functions.md
[item]: items.md
[module]: items/modules.md
[canonical path]: paths.md#canonical-paths
[implementations]: items/implementations.md
[variables]: variables.md
[outer attributes]: attributes.md
[`cfg`]: conditional-compilation.md
[the lint check attributes]: attributes/diagnostics.md#lint-check-attributes
[pattern]: patterns.md
[_ExpressionStatement_]: #expression-statements
[_Expression_]: expressions.md
[_Item_]: items.md
[_LetStatement_]: #let-statements
[_MacroInvocationSemi_]: macros.md#macro-invocation
[_OuterAttribute_]: attributes.md
[_Pattern_]: patterns.md
[_Type_]: types.md
