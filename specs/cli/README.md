# CLI Spec

This directory contains an agent-friendly CLI parser specification.

- [`CLI.md`](CLI.md): normative human-readable spec
- [`CHANGELOG.md`](CHANGELOG.md): version history
- [`example-spec.yaml`](example-spec.yaml): illustrative command definition
- [`tests.yaml`](tests.yaml): pass/fail conformance commands
- [`prompts.md`](prompts.md): copy-paste prompts for agents
- [`examples/python.md`](examples/python.md): Python data structure example

## Quick Prompt

```text
Use CLI.md as the CLI parser specification.
Implement CORE-MODEL, FLAGS-*, ARGS-POSITIONAL, CMD-SUBCOMMANDS,
ERR-BEHAVIOR, ERR-REPORTING, HELP-GENERATION, and TESTS-CONFORMANCE.
Do not implement optional sections unless requested.
Use tests.yaml for conformance tests.
```

## Section IDs

Core parser sections:

- `CORE-MODEL`
- `FLAGS-NAMES`
- `FLAGS-VALUES`
- `FLAGS-VALUE-NAMES`
- `FLAGS-LONG-VALUES`
- `FLAGS-SHORT-VALUES`
- `FLAGS-REPEATABLE`
- `ARGS-POSITIONAL`
- `CMD-SUBCOMMANDS`

Behavior and output sections:

- `ERR-BEHAVIOR`
- `ERR-REPORTING`
- `HELP-GENERATION`

Optional sections:

- `OPT-COLOR`
- `OPT-COMPLETION`

Reference sections:

- `EXCLUDE`
- `EXAMPLE-SPEC`
- `TESTS-CONFORMANCE`
- `TESTS-PASS`
- `TESTS-FAIL`

## Review Checklist

Before accepting an implementation, check that:

- help text is generated from command and parameter definitions
- repeatable flags show `[repeatable]`
- default values show `[default: VALUE]`
- unrecognized value names render as `<VALUE>`
- help lines wrap at 120 visible characters
- `--` stops option parsing
- short flag clusters fail
- unknown long flag prefixes fail
- non-repeatable repeated flags use the final value
- repeatable flag values preserve order
- positional arguments follow the command definition
- errors include usage and a help hint
- optional non-boolean values are not supported
- optional sections such as `OPT-COLOR` and `OPT-COMPLETION` were only
  implemented if requested
