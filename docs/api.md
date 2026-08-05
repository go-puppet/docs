# Usage & API

The module path is `github.com/go-puppet/puppet`. Packages:

| Package | Role |
|---------|------|
| `github.com/go-puppet/puppet` | façade: `Parse`, `ParseExpression` |
| `…/lexer` | tokenizer (`Lex`) |
| `…/parser` | recursive-descent parser (`Parse`, `ParseExpression`) |
| `…/ast` | Puppet::Pops-style node model + `Sexpr` renderer |
| `…/eval` | evaluator (`EvalString`, `EvalPlanString`, `Evaluator`, `RegisterFunction`, `WithFacts`/`WithHiera`/`WithNodeName`/`WithTemplateLoader`/`WithExportedStore`/`WithPlanExecutor`) |
| `…/catalog` | catalog model + Puppet catalog JSON |
| `…/hcl` | Terraform-style HCL2 front-end (`Parse`) → the same `ast.Program` |

## Parse a manifest

```go
prog, err := puppet.Parse(`file { '/tmp/x': ensure => present }`)
// prog is an *ast.Program; walk it, transform it, or render it:
fmt.Println(ast.Sexpr(prog))
```

## Compile to a catalog

```go
cat, logs, err := eval.EvalString(src,
	eval.WithNodeName("web1"),
	eval.WithFacts(eval.MapFacts{"os": map[string]any{"family": "Debian"}}))
fmt.Println(cat.JSON())
```

## HCL2 front-end

`hcl.Parse` reads a Terraform-style HCL2 manifest and produces the same
`*ast.Program` the native Puppet parser produces, so it compiles to an identical
catalog through the same evaluator:

```go
prog, _ := hcl.Parse(`
	locals { mode = "0644" }
	resource "file" "app" {
		ensure  = "present"
		mode    = local.mode
		require = resource.package.app
	}
`)
cat, _ := eval.New().EvalProgram(prog)
fmt.Println(cat.JSON())
```

Constructs outside the v0.1 mapping (function calls, `a ? b : c`, `for`
comprehensions, `%{…}` directives, unknown block types) return a clear
`unsupported in HCL2 v0.1: …` error rather than a fake stub.

## Wire in Hiera

```go
h, _ := hiera.Load("hiera.yaml", hiera.MapScope{})
cat, _, _ := eval.EvalString(src, eval.WithHiera(h)) // backs lookup() and class data binding
```

## Custom functions

```go
e := eval.New()
e.RegisterFunction("double", func(c *eval.Context, args []eval.Value, _ *eval.Block) (eval.Value, error) {
	return args[0].(int64) * 2, nil
})
prog, _ := parser.Parse(`notice(double(21))`)
e.EvalProgram(prog) // logs "42"
```

## Value model

Values use the same representation as go-pcore: `bool`, `int64`, `float64`,
`string` for scalars; `[]any` for arrays; `map[string]any` for hashes;
`pcore.Undef` for undef; and the pcore wrapper types (`*pcore.Regexp`,
`pcore.Type`, …) for the rest.
