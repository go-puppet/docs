# Roadmap

## Shipped

Lexer, parser and `Puppet::Pops` AST; the evaluator (scopes, expressions,
data-type checks via go-pcore, class/defined-type instantiation, iteration, a
built-in function set, `lookup()` via go-hiera, facts via go-facter); and the
catalog compiler (resource graph + containment/relationship/metaparameter edges,
Puppet catalog JSON) — at 100% coverage, CI green across the six 64-bit Go
targets, and a `RegisterFunction` seam for go-ruby-puppet.

Also implemented and evaluated (no fake stubs):

- **EPP / ERB templates** — `epp()` / `inline_epp()` / `template()` /
  `inline_template()`, through an injectable template loader.
- **Resource defaults, overrides and collectors** — `Type { }`, `Type[t] { }`,
  `<| |>` / `<<| |>>`.
- **Exported resources** — collecting `@@` resources through an injectable
  exported-resource store.
- **The extensive stdlib** — string/array/hash/numeric, digests, encoding, path,
  time, TOML/JSON/PSON, `validate_*`, `pw_hash`, `shellwords`, and more.
- **The plan / apply language** — Bolt-style plans (`EvalPlanString`, `apply {}`).
- **Regex match-variable capture** — `$1` … `$n` after `=~`.
- **The Terraform-style HCL2 front-end** — `hcl.Parse` yields the same
  `ast.Program`, so an HCL2 manifest compiles to an identical catalog.

## In progress (returns a clear error today — no fake stubs)

- **Pcore type constructors** beyond the scalar core — `Timestamp()`, `SemVer()`,
  … currently report an "unsupported type constructor" error.
- **The HCL2 front-end's v0.2 expression set** — function calls, the `a ? b : c`
  conditional, `for` comprehensions, `%{…}` template directives and additional
  block types report a clear "unsupported in HCL2 v0.1" error.
