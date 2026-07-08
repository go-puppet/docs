# Contributing

`go-puppet/puppet` is pure Go with **zero cgo** and a **100% coverage** CI gate.

## Ground rules

- **Pure Go, no cgo.** It must cross-compile to and pass under qemu on all six
  64-bit Go targets (amd64, arm64, riscv64, loong64, ppc64le, s390x).
- **100% coverage**, including every parse- and eval-error branch. Tests are
  table/fixture-driven: real `.pp` snippets → AST (`ast.Sexpr`) → catalog
  assertions.
- **No reinvention.** Types belong to [go-pcore](https://github.com/go-pcore),
  data to [go-hiera](https://github.com/go-hiera), facts to
  [go-facter](https://github.com/go-facter). Consume them as module deps.
- `gofmt` + `go vet` clean.

## Running the checks

```sh
go vet ./...
COVERPKG=$(go list ./... | paste -sd, -)
go test -race -coverpkg="$COVERPKG" -coverprofile=cover.out ./...
go tool cover -func=cover.out | tail -1   # must read 100.0%
```

BSD-3-Clause; copyright "the go-puppet/puppet authors". All public GitHub
content is in English.
