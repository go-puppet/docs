# Evaluation & catalog

The `eval` package evaluates a parsed manifest into a `catalog.Catalog`.

## Scopes

Top, node and local scopes with a lexical parent chain. Variables are immutable
once set in a scope. `$facts` and each top-level fact are injected into the top
scope from the configured [FactsProvider](#facts); `$::name` and `a::b` resolve
from the top scope.

## Expressions

Arithmetic (integer and float, with array concat and hash merge on `+`, append
on `<<`), comparison, boolean short-circuit (`and`/`or`), `=~`/`!~` against a
`Regexp` or a `Type`, `in`, indexing and slicing, selectors, and
`if`/`unless`/`case` (value, `Regexp` and `Type` matching). Type checks and the
`assert_type` / `type` functions delegate to [go-pcore](https://github.com/go-pcore).

## Classes, defined types & nodes

`include` / `require` / `contain`, the `class { 'name': … }` resource form,
`inherits`, and defined-type instantiation binding `$title` / `$name`. Class and
defined-type parameters are bound from the declaration, then from **automatic
Hiera data binding** (`name::param` via [go-hiera](https://github.com/go-hiera)),
then from their default. `node` definitions match the compiling node by string
or regular expression, falling back to `default`.

## Iteration & built-in functions

`each`, `map`, `filter`, `reduce`, `with` and `slice` over arrays and hashes;
plus a core built-in set: logging (`notice`/`info`/`warning`/`err`/`debug`/…),
`fail`, `include`/`require`/`contain`, `lookup` (Hiera), `assert_type`, `type`,
and string/array/hash/numeric helpers (`upcase`, `split`, `join`, `sprintf`,
`keys`, `values`, `merge`, `reverse`, `size`, `empty`, `abs`, `min`, `max`, …).

## Facts

`WithFacts` accepts any `FactsProvider`. Use `eval.MapFacts` for deterministic
facts, or `eval.FacterFacts()` for the live host inventory via
[go-facter](https://github.com/go-facter):

```go
cat, _, _ := eval.EvalString(src,
	eval.WithNodeName("web1.example.com"),
	eval.WithFacts(eval.FacterFacts()))
```

## The registry seam

`Evaluator.RegisterFunction(name, fn)` adds or overrides a function at runtime.
This is the seam **go-ruby-puppet** plugs into to contribute Ruby-defined custom
functions and types:

```go
e := eval.New()
e.RegisterFunction("greet", func(c *eval.Context, args []eval.Value, b *eval.Block) (eval.Value, error) {
	c.Log("notice", "hello "+args[0].(string))
	return nil, nil
})
```

## The catalog

Evaluation produces a `catalog.Catalog`: resources (typed, titled parameter bags
with tags and virtual/exported flags) plus directed edges for **containment**
(`Class[main] → Class[Web] → File[…]`), **relationships**
(`->` `~>` `<-` `<~`) and **metaparameters** (`require` / `before` / `notify` /
`subscribe`). `Catalog.JSON()` serializes it to the Puppet catalog JSON shape.
