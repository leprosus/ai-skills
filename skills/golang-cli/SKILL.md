---
name: golang-cli
description: Develop, extend, or review Go CLI applications using the standard library flag package. Apply when working on command routing, argument parsing, CLI configuration, version information, exit codes, signals, command output, or CLI tests.
license: MIT
metadata:
  author: leprosus
  version: "0.0.1"
---

# Go CLI Best Practices

Build CLI applications that are predictable, composable, and convenient
to use in scripts.

Follow explicit user instructions first, then applicable project
instructions and established CLI contracts, then these defaults.

## Existing Applications

Before making changes, inspect the entry point, command routing,
argument parsing, help output, and relevant tests.

If the requested task does not involve replacing an existing argument parser,
keep changes within scope and identify any conflict with the parsing rule.

Preserve existing command names, flag meanings, defaults, output formats,
and exit codes unless the requested change requires altering them.

## Commands and Flags

Do not use external packages to handle command-line arguments and flags.
Use the standard library `flag` package for CLI argument and flag parsing.

Use a separate `flag.FlagSet` for each command. Prefer
`flag.ContinueOnError` so parsing errors can be returned to the caller.

Route subcommands explicitly:

1. Identify the command from the arguments.
2. Select its handler.
3. Parse the remaining arguments using that command's flag set.
4. Validate positional arguments before performing the operation.

The standard `flag` parser stops at the first positional argument.
Document this behavior, or implement a different parsing contract only
when required by the application.

Provide useful root and command help. Treat an explicit help request
as successful completion.

## Project Structure

For new applications, use a small entry point and keep command handlers
separate from business logic.

A suitable starting structure is:

```text
app/
├── cmd/
│   └── app/
│       ├── main.go
│       ├── root.go
│       ├── serve.go
│       └── version.go
├── internal/
│   └── service/
└── go.mod
```

Keep `main.go` responsible for process setup, calling the application,
and selecting the final exit code.

Keep business logic in appropriate packages. Create additional command
packages or directories only when their size or responsibilities justify
them.

Preserve an existing project's layout unless restructuring is part of
the task.

## Configuration

When multiple configuration sources are needed, make their precedence
explicit. A usual default is:

command-line flags > environment variables > configuration file > defaults.

Distinguish an explicitly supplied flag from its default value when
merging configuration sources.

Do not add configuration files, environment variables, or dependencies
unless the application needs them.

## Output and Errors

Use stdout for command results and stderr for errors, diagnostics,
and progress.

Pass output destinations as `io.Writer` values so commands can be tested
with buffers. Configure each flag set's output and usage function
deliberately.

Return errors from command handlers. Keep `os.Exit` at the process
boundary, after required cleanup has completed.

Preserve the established meaning of output-related flags. Do not assume
that `--output` selects a format; it may designate a file or directory.

When machine-readable output is needed, provide a documented format
selection mechanism consistent with the application's existing interface.

Keep machine-readable output free from progress messages and color codes.

If terminal detection is needed, use a terminal-aware check such as
`golang.org/x/term.IsTerminal`. `os.ModeCharDevice` alone does not establish
that a stream is a terminal.

Add color, tables, or interactive UI dependencies only when the requested
interface benefits from them.

## Version and Build Information

Prefer injecting release information at build time.

For string variables declared in `package main`:

```go
var (
    version = "dev"
    commit  = "unknown"
    date    = "unknown"
)
```

Example build command:

```sh
go build \
  -ldflags "-X main.version=1.2.3 -X main.commit=$(git rev-parse --short HEAD) -X main.date=$(date -u +%Y-%m-%dT%H:%M:%SZ)" \
  -o bin/app \
  ./cmd/app
```

If the variables belong to another package, use that package's actual
import path instead of `main`.

The target variables must be strings compatible with linker `-X`
injection. Verify that the resulting binary reports the injected values.

Follow the project's existing release and versioning process.

## Exit Codes

Preserve existing exit-code contracts. For new applications, use these
defaults unless a documented interface requires otherwise:

| Code | Meaning |
|------|---------|
| 0 | Success, including an explicit help request |
| 1 | Runtime failure |
| 2 | Invalid command, flags, or arguments |

Use additional codes only when callers need to distinguish those cases.
Do not apply shell-specific codes indiscriminately to application errors.

## Cancellation

For commands that perform long-running or blocking work, use
`signal.NotifyContext` to propagate interruption through `context.Context`.

Call the returned stop function and ensure cancellation reaches network
requests, subprocesses, and other operations that support it.

Do not add signal-handling machinery to commands that do not need it.

## Testing

Test observable CLI behavior:

- Command routing and unknown commands.
- Flag parsing and positional argument validation.
- Help output and its successful exit status.
- Separation of stdout and stderr.
- Documented output formats and exit codes.
- Cancellation and cleanup when relevant.

Test handlers with injected arguments and output writers. Use subprocess
tests when the behavior under test depends on the real process boundary.

Run checks appropriate to the change and the project's existing workflow.
