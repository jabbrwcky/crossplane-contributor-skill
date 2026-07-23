# Coding style, in depth

Source: [crossplane contributing guide][contributing]. Applies to all
`github.com/crossplane` and `github.com/crossplane-contrib` repositories
unless a repo states otherwise.

The project doesn't maintain its own style guide beyond this — it defers to
[Effective Go], the Go [code review comments], and the Go [test review
comments]. What follows are the points most frequently raised in review.

## Explain every `//nolint`

`golangci-lint` runs everywhere; false positives are tolerated over missed
real issues, so overriding it is sometimes necessary. When you do:

1. Scope it as tightly as possible — `//nolint:<linter>` on the exact line,
   never a blanket file/function-level suppression.
2. Say why, inline, on the same comment.

```go
func hash(s string) string {
	h := fnv.New32()
	_ = h.Write([]byte(s)) //nolint:errcheck // Writing to a hash never returns an error.
	return fmt.Sprintf("%x", h.Sum32())
}
```

## Short variable names

> Variable names in Go should be short rather than long. This is especially
> true for local variables with limited scope. Prefer `c` to `lineCount`.
> Prefer `i` to `sliceIndex`. The basic rule: the further from its
> declaration that a name is used, the more descriptive the name must be.

A one-or-two-letter receiver name is fine. A package-level export used many
files away needs a descriptive name. Match the name's length to how far a
reader has to carry it in their head.

## Don't wrap function signatures

If a signature is long enough that you're wrapping it across lines, that's
usually a sign the parameter names are too verbose or the function does too
much. For many optional parameters (e.g. dependency injection), prefer a
functional-options pattern instead of a long positional signature:

```go
type Option func(w *Wrangler)

func WithFowlWrangler(fw fowl.Wrangler) Option {
	return func(w *Wrangler) { w.fw = fw }
}

func NewWrangler(looseGeese int, o ...Option) *Wrangler {
	w := &Wrangler{fw: fowl.DefaultWrangler{}, loose: looseGeese}
	for _, fn := range o {
		fn(w)
	}
	return w
}
```

## Return early

Handle terminal cases (usually errors) first, at the lowest indentation,
so the "main" logic reads at function scope, not nested in a conditional.
This is also why `else` is rare in Crossplane code — per [Effective Go]:

> when an if statement doesn't flow into the next statement — that is, the
> body ends in break, continue, goto, or return — the unnecessary else is
> omitted.

```go
// Prefer this:
func example() error {
	v := fetch()
	if v != 42 {
		return errors.New("v was a bad number")
	}
	b := embiggen(v)
	for k, ok := range lookup(b) {
		if !ok {
			remove(k)
			continue
		}
		store(k)
	}
	return nil
}
```

## Wrap errors with inline strings, not constants

```go
import "github.com/crossplane/crossplane-runtime/pkg/errors"

func example() error {
	v, err := fetch()
	if err != nil {
		return errors.Wrap(err, "could not fetch the thing")
	}
	store(embiggen(v))
	return nil
}
```

The project **used to** recommend error constants (`const errFetch = "..."`)
but no longer does — inline strings are preferred now. Wrapping with
`crossplane-runtime/pkg/errors` (or `github.com/pkg/errors`) lets logs and
events surface a useful, specific chain without plumbing loggers deep into
the codebase.

## Scope errors narrowly

Declare `err` inside the conditional that handles it, not at function scope,
to avoid an old `err` being silently reused/shadowed as the function grows:

```go
// Prefer this — err only exists inside the block that handles it:
func example() error {
	if err := enable(); err != nil {
		return errors.Wrap(err, "could not enable the thing")
	}
	return errors.Wrap(emit(), "could not emit the thing")
}
```

The "return early" rule trumps this one where it would force an `else`:
declaring at function scope to keep the main logic unindented is fine.

## Test error properties, not error strings

Use `cmpopts.EquateErrors()` — it treats an error `errors.Is` another as
equal to it, so you're testing the error's *identity*, not its exact text.
For a simple pass/fail check, `cmpopts.AnyError` keeps this consistent
across a whole test table:

```go
cases := map[string]struct {
	input string
	want  want
}{
	"BadInput": {
		input: "Hello!",
		want:  want{err: cmpopts.AnyError},
	},
	"GoodInput": {
		input: "Quack!",
		want:  want{output: "Quack!"},
	},
}
```

For functions with many distinct error cases (e.g. `Reconciler` methods),
inject dependencies that return a specific sentinel error, and assert the
returned error wraps *that* sentinel — not just "some error occurred":

```go
errBoom := errors.New("boom")

"BadQuackModulator": {
	q: &ComplicatedQuacker{
		QuackModulator: func() (int, error) { return 0, errBoom },
	},
	want: want{err: errBoom}, // want an error that errors.Is(errBoom)
},
```

## Prefer table-driven tests

No Ginkgo, Gomega, or Testify — the project follows the Go [test review
comments] guidance directly. The canonical shape:

```go
// Test function names are always PascalCase. No underscores.
func TestExample(t *testing.T) {
	type args struct {
		ctx   context.Context
		input string
	}
	type want struct {
		output int
		err    error
	}

	cases := map[string]struct {
		reason string
		args   args
		want   want
	}{
		// The summary is always PascalCase. No spaces, hyphens, underscores.
		"BriefTestCaseSummary": {
			reason: "A longer summary of what we're testing — printed on failure.",
			args:   args{ctx: context.Background(), input: "some input value"},
			want:   want{output: 1, err: nil},
		},
	}

	for name, tc := range cases {
		t.Run(name, func(t *testing.T) {
			got, err := Example(tc.args.ctx, tc.args.input)

			if diff := cmp.Diff(tc.want.err, err, cmpopts.EquateErrors()); diff != "" {
				t.Errorf("%s\nExample(...): -want, +got:\n%s", tc.reason, diff)
			}
			if diff := cmp.Diff(tc.want.output, got); diff != "" {
				t.Errorf("%s\nExample(...): -want, +got:\n%s", tc.reason, diff)
			}
		})
	}
}
```

Use `github.com/google/go-cmp` even for trivial comparisons, to keep test
output format consistent across the codebase. Crossplane-specific `cmp`
options live in `crossplane-runtime/pkg/test`.

[contributing]: https://github.com/crossplane/crossplane/blob/main/contributing/README.md
[Effective Go]: https://golang.org/doc/effective_go
[code review comments]: https://go.dev/wiki/CodeReviewComments
[test review comments]: https://go.dev/wiki/TestComments
