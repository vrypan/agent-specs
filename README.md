# Agent Specs

Reusable specifications designed to be handed to AI coding agents.

Each spec is written for two audiences:

- humans who need to decide what behavior they want
- agents that need stable, explicit implementation instructions

Specs should include stable section IDs, required behavior, optional behavior,
exclusions, examples, and conformance tests.

## Available Specs

- [`specs/cli`](specs/cli): command-line parser, help-generation, and error
  reporting behavior

## Repository Shape

```text
agent-specs/
  README.md
  specs/
    cli/
      README.md
      CLI.md
      example-spec.yaml
      tests.yaml
      prompts.md
      examples/
        python.md
```

## How To Use A Spec

1. Give an agent the relevant spec directory.
2. Tell it which section IDs are required.
3. Tell it which optional section IDs to skip.
4. Ask it to implement the spec and report deviations.
5. Ask it to run or translate the conformance tests.

Example:

```text
Use specs/cli/CLI.md as the CLI parser specification.
Implement all required sections.
Do not implement OPT-COLOR.
Translate specs/cli/tests.yaml into automated tests.
Report any deviations.
```

## Spec Conventions

- Stable section IDs are part of the public interface.
- Optional features use the `OPT-` prefix.
- Exclusions are explicit.
- Conformance tests should be machine-readable when practical.
- A spec should say what is deliberately not included.
- Breaking changes should be called out before a spec is reused elsewhere.

## License

Licensed under the MIT License. See [`LICENSE`](LICENSE).
