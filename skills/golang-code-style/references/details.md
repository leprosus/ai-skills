# Go Code Style Details

Read the relevant section when the task involves complex conditions
or choosing between value and pointer parameters.

## Complex Conditions and Short-Circuit Evaluation

Extract conditions into named booleans when their names communicate
meaning more clearly than the original expressions.

```go
isAdmin := user.Role == RoleAdmin
isOwner := resource.OwnerID == user.ID

if isAdmin || isOwner || expensivePermissionCheck(user, resource) {
    allow()
}
```

Keep the expensive check inline when it should run only if earlier
conditions do not satisfy the expression.

This rewrite changes evaluation behavior:

```go
isAdmin := user.Role == RoleAdmin
isOwner := resource.OwnerID == user.ID
hasPermission := expensivePermissionCheck(user, resource)

if isAdmin || isOwner || hasPermission {
    allow()
}
```

The permission check now runs unconditionally. That may increase cost
or change behavior if it has side effects.

Preserve evaluation order and conditional execution when extracting
expressions. Apply the same care to checks that rely on an earlier
nil or bounds check.

Do not introduce names that merely repeat the syntax without making
the condition easier to understand.

## Value and Pointer Parameters

Choose based on semantics first.

Use values when the function needs an independent value and copying
is appropriate:

```go
func FormatUser(name string, age int, createdAt time.Time) string
```

Use pointers when the function changes the caller's value:

```go
func PopulateDefaults(cfg *Config)
```

Use pointers when nil represents an intentional state:

```go
func UpdateUser(ctx context.Context, id string, name *string) error
```

Also consider shared identity and types that must not be copied after
use, such as structs containing an active mutex.

Passing a struct by value is a shallow copy. Its slices, maps, and
pointers may still refer to shared underlying data.

A pointer does not automatically reduce memory usage or improve
performance. Escape analysis, allocation, copying, and access patterns
all affect the result.

For a demonstrated performance concern, benchmark representative
workloads. Avoid treating a fixed struct size as a universal boundary
between values and pointers.

## Method Receivers

Use a pointer receiver when methods mutate the receiver or the type
must not be copied.

For small value-like types, value receivers may be appropriate.

Keep receiver choices consistent where practical, and consider their
effect on method sets and interface implementation.
