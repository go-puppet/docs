# Roadmap

## v0.1 — shipped

Lexer, parser and `Puppet::Pops` AST; the evaluator (scopes, expressions,
data-type checks via go-pcore, class/defined-type instantiation, iteration, a
built-in function set, `lookup()` via go-hiera, facts via go-facter); and the
catalog compiler (resource graph + containment/relationship/metaparameter edges,
Puppet catalog JSON) — at 100% coverage, CI green across the six 64-bit Go
targets, and a `RegisterFunction` seam for go-ruby-puppet.

## v0.2 — staged (parses today, evaluation returns a clear error — no fake stubs)

- **EPP / ERB templates** — `epp()` / `template()`.
- **Resource defaults, overrides and collectors** — `Type { }`, `Type[t] { }`,
  `<| |>` / `<<| |>>`.
- **Exported resources / PuppetDB** — collecting `@@` resources across nodes.
- **The full stdlib module** — beyond the language-core functions shipped today.
- **The plan / apply language** — Bolt-style plans.

Type constructors beyond `Integer` / `Float` / `String` / `Boolean` and Puppet's
match-variable capture (`$1` after `=~`) are also staged.
