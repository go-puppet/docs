# The language

The `lexer` and `parser` packages cover the Puppet 8 grammar and produce a
`Puppet::Pops`-style AST (the `ast` package). The `ast.Sexpr` renderer prints a
compact s-expression of any node, which is how the parser is asserted in tests.

## Literals & interpolation

- Integers (decimal, `0x` hex, `0` octal), floats, booleans, `undef`, `default`.
- Single-quoted strings (only `\'` and `\\` escapes) and double-quoted strings
  with `${…}` / `$var` interpolation and `\n \t \r \s \u{…}` escapes.
- Arrays `[…]` and hashes `{ k => v }`.
- Regular expressions `/…/`.

```puppet
$greeting = "hello ${upcase('world')} — ${1 + 2}"
```

Inside `${…}`, a **lower-case** leading word is an implicit variable
(`${os}` ≡ `$os`); an **upper-case** word is a type or resource reference
(`${File['x']}`); a word immediately followed by `(` is a function call.

## Heredocs

`@(TAG)` (no interpolation) or `@("TAG")` (interpolation on), with an optional
`:syntax` tag, a `|` indent margin and a `-` trailing-newline chomp:

```puppet
$config = @("EOT")
  server ${host}
  | EOT
```

## Data-type expressions

Data types such as `Integer[1,10]`, `Optional[String]`, `Enum['a','b']` and
`Array[Integer]` are parsed as AST and evaluated through
[go-pcore](https://github.com/go-pcore) — they are never reimplemented here.

## Operators

The full precedence ladder: unary `! - *`, `in`, `=~`/`!~`, `* / %`, `+ -`,
`<<`, `== !=`, `< > <= >=`, `and`, `or`, assignment `=`, and the relationship
operators `->` `~>` `<-` `<~`.

!!! note "Whitespace-sensitive `[`"
    Following Puppet, a `[` with preceding whitespace after a non-reference
    starts a new array/statement rather than indexing; `$x[0]` indexes, but
    `$x [0]` is two expressions.

## Control flow

`if`/`elsif`/`else`, `unless`, `case` (with value, `Regexp` and `Type` matching)
and selectors `$x ? { … }`.

## Resources & definitions

Resource declarations, defaults `Type { }`, overrides `Type[t] { }`, collectors
`<| |>` / `<<| |>>`, virtual `@` and exported `@@` forms; `class` (with
`inherits`), `define`, `node`, and `function` (with `>> ReturnType`)
definitions; function calls with lambdas `|params| { }`; and relationship
chaining.
