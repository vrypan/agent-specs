# CLI Spec Prompts

## Full Implementation

```text
Use CLI.md as the CLI parser specification.

Implement:
- CORE-MODEL
- FLAGS-NAMES
- FLAGS-VALUES
- FLAGS-LONG-VALUES
- FLAGS-SHORT-VALUES
- FLAGS-REPEATABLE
- ARGS-POSITIONAL
- CMD-SUBCOMMANDS
- ERR-BEHAVIOR
- ERR-REPORTING
- HELP-GENERATION
- TESTS-CONFORMANCE

Do not implement:
- OPT-COLOR

Use example-spec.yaml as the sample command definition.
Use tests.yaml as conformance input.
Report any behavior that differs from CLI.md.
```

## Help Generation Only

```text
Using CLI.md, implement only HELP-GENERATION for the existing command
definition model.

Include:
- command descriptions
- usage lines
- argument sections
- option sections
- repeatable markers
- default markers
- 120-character wrapping

Do not add OPT-COLOR.
```

## Error Reporting Only

```text
Using CLI.md ERR-BEHAVIOR and ERR-REPORTING, replace generic parse errors with
specific user-facing errors.

Errors should include:
- active command
- offending token when available
- expected value name or kind
- usage line
- help hint
```

## Tests Only

```text
Translate tests.yaml into automated tests.
Every item under pass must parse successfully.
Every item under fail must fail with a parse error.
The tests should use example-spec.yaml as the command definition.
```

## Implementation Report

```text
When done, report:
- files changed
- commands run
- which CLI.md sections are implemented
- which CLI.md sections are not implemented
- conformance test results
- any deliberate deviations from CLI.md
```
