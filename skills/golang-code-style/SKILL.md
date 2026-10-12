---
name: golang-code-style
description: Apply the user's Go code style preferences when writing or reviewing Go code, especially line wrapping, variable declarations, control flow, function design, and blank-line placement. Focus on readability rather than introducing linter configuration or broad architectural changes.
license: MIT
metadata:
  author: leprosus
  version: "0.0.1"
---

# Go Code Style

These are the user's style preferences, not universal requirements of Go.

Follow explicit user instructions first, then applicable project
instructions and established conventions, then these defaults.

Keep changes scoped to the task. Do not reformat unrelated code during
a functional change.

Use `gofmt` for Go formatting and follow the project's configured checks.

## Line Length and Wrapping

Prefer lines within approximately 120 characters. Break longer lines
at semantic boundaries when doing so improves readability.

By default, put each argument on its own line in calls with four or more
arguments. Respect an explicit request for a different presentation.

For multiline function signatures, put each parameter on its own line
when that makes the signature easier to read.

Do not introduce an options struct solely to shorten a line. Consider one
when several parameters form a coherent configuration.

## Variable Declarations

Use `:=` for values supplied by an expression only when every non-blank
variable on its left-hand side is newly declared. Use `var` when the zero
value is the intended starting state or when an explicit declaration is
needed before assignment.

Do not mix declaration of new variables with reassignment of existing
variables in a short declaration. This includes named return variables,
which are already declared in the function body's scope. Mixed `:=`
statements make reassignment easy for a reader to miss and can cause
defects or incorrect interpretation of the code.

Declare new variables explicitly, then use `=` to assign the results:

```go
func List(ctx context.Context) (err error) {
    var versionList []Version

    versionList, err = getSortedVersionList(ctx)
    if err != nil {
        return err
    }

    return nil
}
```

Avoid the mixed declaration in this example:

```go
versionList, err := getSortedVersionList(ctx) // err already exists
```

Explicit reassignment with `=` is allowed. This rule makes reassignment
visible; it does not prohibit updating variables. The blank identifier
`_` is not a variable and does not count as an existing declaration.
Preserve intentional shadowing in a nested scope when it is needed;
review it separately from reassignment in the same scope.

```go
var count int
var buf bytes.Buffer

name := "default"
```

For slices, prefer the zero value when an empty, nil slice is appropriate:

```go
var users []User
```

Use an explicitly empty slice when nil and empty have different observable
meanings, such as in a serialization contract.

Preallocate slices or maps when the expected size is known and doing so
serves the operation. Avoid arbitrary speculative capacity.

Initialize maps before writing to them. Nil maps are valid for reading,
iteration, and operations that do not require insertion.

```go
users := make([]User, 0, len(ids))
counts := make(map[string]int, len(items))
```

## Struct Literals

Prefer keyed fields in struct literals, especially for exported types
or types that may evolve.

```go
srv := &http.Server{
    Addr:         ":8080",
    ReadTimeout:  5 * time.Second,
    WriteTimeout: 10 * time.Second,
}
```

This preference concerns struct literals; it does not require field names
in slice or array literals.

## Control Flow

Handle errors and edge cases early. Keep the main successful path
at minimal indentation.

Drop an unnecessary `else` when the preceding branch ends with
`return`, `break`, or `continue`.

For simple conditional assignments, prefer a clear default followed
by overrides. Use `switch` when it makes mutually exclusive cases
or repeated comparisons easier to understand.

Extract complex conditions into named booleans when the names clarify
their meaning. Three or more operands are a useful signal to review
readability, not an automatic reason to rewrite the expression.

Preserve short-circuit behavior. Do not eagerly evaluate expensive
or side-effecting checks merely to give them a name.

Prefer declaring an error separately before checking it:

```go
err := validate(input)
if err != nil {
    return err
}
```

This is a readability preference. Declaring a variable in an `if`
initializer does not by itself cause a heap allocation.

## Function Design

Keep functions focused on one coherent responsibility.

Prefer a small number of parameters. When a function needs more than
four, consider whether some represent a cohesive options value.

Put `context.Context` first, followed by inputs and output destinations,
when applicable.

Use named return values when their names clarify the result or when
the function needs to modify them in deferred cleanup.

Prefer explicit return expressions. Naked returns are acceptable
in very short functions when their meaning is immediately clear.

Prefer `range` when it expresses the iteration clearly. Use integer
`range` only when the project's Go version supports it.

## Exported Names and Package Context

Choose exported names with their package-qualified form in mind. Avoid
prefixing an exported type, variable, constant, or function with the package
name when the qualifier already supplies that context. Repeating the package
name creates unnecessary stuttering at call sites.

For example, in package `version`, prefer:

```go
package version

type List []Version
```

Callers then use `version.List` rather than `version.VersionList`. Likewise,
prefer `version.Default` over `version.DefaultVersion` when both names convey
the same meaning. Review repetition in suffixes as well as prefixes.

Keep names descriptive enough to distinguish different concepts within the
package. Retain a qualifier when it adds meaning beyond the package name;
do not shorten names mechanically or rename an established public API without
considering compatibility and the authorized scope of the change.

## Methods for Values and Collections

When an operation belongs to a concrete type and its behavior is determined
by that type's data, prefer a method over a standalone function. Keep the
method near the type definition.

For example, formatting and writing a single commit belongs to `CommitInfo`:

```go
func (c CommitInfo) Write(stdout io.Writer) (err error)
```

When a collection has its own behavior, define a named collection type and
implement that behavior as a method:

```go
type CommitInfoList []CommitInfo

func (commits CommitInfoList) Write(stdout io.Writer) (err error)
```

Name methods after the operation without repeating the receiver type's
name. The receiver already identifies the value or collection being operated
on. Prefer `CommitInfoList.Write` over `CommitInfoList.WriteCommitInfoList`
or `CommitInfoList.WriteCommitList`. Both `CommitInfo.Write` and
`CommitInfoList.Write` can use the same method name because their receivers
provide the distinction. Add qualifiers only when they distinguish meaningful
operations or output formats, not merely to restate the receiver type.

The collection method handles iteration and list presentation, such as
separators between entries. Delegate each element's operation to its method
instead of duplicating the element's logic:

```go
err = commit.Write(stdout)
```

Use the named collection type consistently in return values, variables,
and call sites. Remove standalone functions fully replaced by these methods.

Do not create a named type for every slice automatically. Introduce one
when the collection has meaningful behavior. Keep functions that coordinate
independent objects as standalone functions when the operation does not
belong to one type. Preserve established package boundaries and avoid
introducing import cycles when moving behavior.

## Values and Pointers

Pass simple values and small read-only structs by value by default.

Use pointers when mutation, shared identity, meaningful nil values,
or avoiding significant copying requires them.

Do not select pointers using a universal size threshold. For performance
decisions, consider the type's semantics and measure representative code.

Read [references/details.md](references/details.md) when reviewing
short-circuit behavior or choosing between values and pointers.

## File Organization

Group related types, constructors, and methods.

A useful default order is:

1. Package documentation and imports.
2. Constants and variables.
3. Types.
4. Constructors and methods.
5. Helpers.

Keep a significant primary type with its related methods where practical.
Do not split a small cohesive file solely to enforce one type per file.

Keep side-effect imports near application setup where practical.
Preserve imports required by library or framework integration.

Avoid dot imports in library code.

Export only the names that callers need.

## Unit Test File Location

Keep each Go source file and its corresponding unit test file in the same
package directory. Do not split them across different directories or maintain
a separate directory tree for unit tests.

For example, `commit_info.go` and `commit_info_test.go` must be stored together:

```text
internal/git/
    commit_info.go
    commit_info_test.go
```

This applies whether the test uses the package itself (`package git`) or an
external test package (`package git_test`). An external test package does not
require a separate directory.

Name the unit test file after the source file using the `_test.go` suffix.
When moving a source file, move its corresponding unit tests to the same
package directory and update package declarations and imports as needed.
Keep tests that exercise a command or integration boundary beside the code
for that boundary; they do not replace unit tests for lower-level types.

## Strings and Conversions

Prefer `strconv` for straightforward numeric conversions and
`fmt.Sprintf` for formatting that benefits from it.

Use `%q` when quoting a value makes an error message clearer.

Use `+` for simple string concatenation. Consider `strings.Builder`
for repeated construction in loops.

Prefer concrete types. Use generics when they express useful type
relationships or reusable operations without unnecessary abstraction.

## Dependencies and Abstraction

Prefer standard library functionality when it meets the requirements.

Avoid speculative abstractions and unnecessary reflection.

Introduce interfaces when they express a useful boundary, support
multiple implementations, or enable substitution needed by callers
or tests.

Do not create an interface automatically for every concrete type.

## Blank Lines

Separate distinct logical steps with blank lines.

Keep an operation immediately adjacent to its error check:

```go
err := process(input)
if err != nil {
    return err
}
```

Separate an unrelated declaration from the following control-flow block:

```go
someInt := 1

if isAllowed {
    use(someInt)
}
```

When a declaration is followed by a separate assignment that may fail,
separate the declaration from that operation:

```go
var result int

result, err := calculate(first, second)
if err != nil {
    return err
}
```

Do not insert blank lines mechanically around every brace.

## Comments and Exceptions

Write comments when they explain intent, constraints, or a non-obvious
tradeoff.

Do not add a comment for every style exception. Explain an exception
only when future readers need the reason to understand the code.

## Reviews

Prioritize readability issues that materially affect understanding
or maintenance.

Distinguish correctness problems from style preferences. Keep proposed
changes proportional to their benefit and within the requested scope.
