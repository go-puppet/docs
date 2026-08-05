# go-puppet documentation

**A pure-Go (no cgo) implementation of the Puppet language.** `go-puppet/puppet`
parses Puppet 8 manifests into a faithful `Puppet::Pops`-style AST and compiles
them to a **catalog** — a resource graph with containment and relationship edges,
serializable to Puppet catalog JSON. The module path is
`github.com/go-puppet/puppet`.

The type system is delegated to **[go-pcore](https://github.com/go-pcore)**, data
binding to **[go-hiera](https://github.com/go-hiera)**, and facts to
**[go-facter](https://github.com/go-facter)** — never reimplemented.

!!! success "Status: complete"
    Lexer, parser, AST, evaluator and catalog compiler — at **100% coverage**,
    `gofmt` + `go vet` clean, CI green across the six 64-bit Go targets (amd64,
    arm64, riscv64, loong64, ppc64le, s390x). EPP/ERB templates, resource
    defaults/overrides/collectors, exported resources, the plan/apply language,
    an extensive stdlib and a Terraform-style HCL2 front-end are all implemented.
    Still in progress: Pcore type constructors beyond the scalar core, and the
    HCL2 front-end's v0.2 expression set.

## Install

```sh
go get github.com/go-puppet/puppet
```

## Quick taste

```go
package main

import (
	"fmt"

	"github.com/go-puppet/puppet/eval"
)

func main() {
	cat, logs, err := eval.EvalString(`
	  class nginx (Integer[1,65535] $port = 80) {
	    package { 'nginx': ensure => installed }
	    -> service { 'nginx': ensure => running, require => Package['nginx'] }
	  }
	  include nginx
	`)
	if err != nil {
		panic(err)
	}
	for _, l := range logs {
		fmt.Printf("[%s] %s\n", l.Level, l.Message)
	}
	fmt.Println(cat.JSON()) // Puppet catalog JSON
}
```

## Where to next

- [The language](language.md) — the grammar the lexer and parser cover.
- [Evaluation & catalog](evaluation.md) — scopes, functions, iteration, the catalog model.
- [Usage & API](api.md) — the Go surface, including the HCL2 front-end (`hcl.Parse`).
- [Roadmap](roadmap.md) — what is shipped and what is still in progress.
