# CLI Parser Shape

This document describes the command-line interface shape used by this project.
It is language-agnostic: an implementation may be written in any language, but
should follow these parsing and help-generation rules.

The goal is a small, predictable parser where one command specification defines
both parsing behavior and help output.

## SPEC-METADATA: Specification Metadata

| Field | Value |
| --- | --- |
| Version | 1.0.0 |

## CORE-MODEL: Core Model

A CLI is described as:

- a root command
- zero or more subcommands
- flags/options for each command
- positional arguments for each command
- command and flag descriptions used to generate help

Each command has:

- `name`: command name, such as `list` or `ls`
- `description`: one-line command summary
- `usage`: usage string shown in help
- `flags`: ordered list of accepted flags
- `arguments`: ordered list of positional arguments
- `extra_help`: optional trailing help text for command-specific reference
  material

Each flag has:

- `name`: long name without `--`
- `short`: optional single-character short name without `-`
- `value`: value behavior
- `value_name`: placeholder shown in help, such as `VALUE`, `N`, or `REF`
- `description`: text shown in generated help
- `default_value`: optional display-only default text
- `repeatable`: whether the flag may appear multiple times
- `attached_short_value`: whether short form accepts an attached value, such as
  `-n10`

Each positional argument has:

- `name`: argument name shown in help, such as `FILE` or `REF`
- `description`: text shown in generated help
- `required`: whether the argument must be present
- `repeatable`: whether the argument may be used multiple times

## FLAGS-NAMES: Flag Names

Long flags use `--name`.

Short flags use `-x`, where `x` is a single character. Short aliases are not
required. A flag may have only a long form.

Short flag clusters are not supported. For example, `-abc` is not interpreted
as `-a -b -c`. If `-a` accepts an attached value and enables
`attached_short_value`, then `-abc` is interpreted as `-a bc`; otherwise it is
invalid.

## FLAGS-VALUES: Value Kinds

A flag declares one of these value kinds:

| Kind | Accepted forms | Help form |
| --- | --- | --- |
| `none` | `--flag`, `-f` | `--flag` |
| `string` | `--flag VALUE`, `--flag=VALUE`, `-f VALUE` | `--flag <VALUE>` |
| `int` | `--flag 10`, `--flag=10`, `-f 10` | `--flag <N>` |
| `bool_required` | `--flag true`, `--flag=false` | `--flag <BOOL>` |
| `bool_optional` | `--flag`, `--flag=true`, `--flag false` | `--flag[=BOOL]` |

Accepted boolean values are:

- true: `true`, `1`, `yes`
- false: `false`, `0`, `no`

Boolean matching is case-insensitive for words.

For `bool_optional`, a bare flag means `true`. If the next argument does not
start with `-`, it is consumed as the boolean value. If the next argument starts
with `-`, it is left as the next argument.

## FLAGS-LONG-VALUES: Long Flag Values

Long flags accept both separated and inline values:

```text
--number 10
--number=10
--attr val1
--attr=val1
```

Flags with `value = none` must not receive an inline value. For example,
`--reverse=true` is invalid when `reverse` is a value-less flag.

## FLAGS-SHORT-VALUES: Short Flag Values

Short flags with values accept separated values:

```text
-n 10
-a val1
```

They accept attached values only when `attached_short_value` is enabled:

```text
-n10
-aval1
```

Short flags without values must not have attached text. For example, `-rfoo` is
invalid when `-r` is a value-less flag.

## FLAGS-REPEATABLE: Repeated Flags

Repeatable flags may be used multiple times. Order is significant and must be
available to the command:

```text
--attr val1 --attr val2
-a val1 -a val2
```

If a non-repeatable flag is used multiple times, the final occurrence takes
precedence. This allows users to override an earlier option later in the
command line.

## ARGS-POSITIONAL: Positionals

Tokens that are not options are positional arguments for the active command.
Each command validates its own positional count and meaning.

Positional arguments appear in declaration order. A non-repeatable positional
may be used once. A repeatable positional may be used multiple times and must
be the final positional in the command specification.

Required positionals must be present. Optional positionals may be omitted.

Recommended usage notation:

- required positional: `<REF>`
- optional positional: `[REF]`
- repeatable positional: `[REF]...` or `<REF>...`

The `--` marker ends flag parsing. All following arguments are positionals,
even if they start with `-`:

```text
cmd -- -literal -values
```

## CMD-SUBCOMMANDS: Subcommands

A subcommand is selected by the command word immediately after the program
name.

Recommended behavior:

- `program --help` prints root help
- `program -h` prints root help
- `program --version` prints version
- `program -V` prints version
- `program subcommand --help` prints subcommand help
- `program subcommand -h` prints subcommand help

Dispatch rules that are application-specific, such as treating a bare
`program` as a default subcommand, should live outside the generic parser.

## EXCLUDE: Not Included

This spec intentionally does not include:

- Optional non-boolean values, such as `--name` meaning a default value and
  `--name=full` overriding it. Optional string or enum values are ambiguous
  when followed by positionals, for example `command --name file.txt`.
- Short flag clusters, such as `-abc` meaning `-a -b -c`.
- Combined short flags with values, such as `-abc value` where one member of
  the cluster takes the value.
- Long-flag abbreviation or prefix matching, such as `--num` matching
  `--number`.
- Negated boolean aliases, such as `--no-color`, unless the application defines
  them as ordinary value-less flags.
- Environment variable defaults.
- Configuration-file defaults.
- Shell completion generation.
- Automatic validation of application-specific value syntax, such as splitting
  `key=value` strings.
- Nested subcommands beyond one command word after the program name.
- Automatic suggestions for mistyped command or flag names.

## OPTIONS: Options

This section records optional conventions that implementations may adopt. These
are not required by the parser shape, but they help separate common CLI policy
from application-specific code.

### OPT-COLOR: Color Support

If help generation supports color, color must be treated as presentation only.
It must not change wording, spacing, ordering, wrapping, or parse behavior.

Recommended behavior:

- Disable color by default when stdout is not a terminal.
- Disable color when `NO_COLOR` is set.
- Allow an application-level override to force color when needed.
- Do not emit ANSI escape sequences when color is disabled.
- Wrap help using visible character width, not byte length including ANSI
  escape sequences.
- Prefer applying color after deciding line breaks, or use a width function
  that ignores ANSI escape sequences.

Recommended styling:

- Headings such as `Usage:`, `Options:`, and `Commands:` may be bold.
- Flag labels such as `-h, --help` may be bold or colored.
- Value placeholders such as `<VALUE>` may be dim or colored.
- Descriptions should usually remain plain text for readability.
- Default and repeatable markers may be dim.
- Avoid using color as the only way to communicate meaning.

## ERR-BEHAVIOR: Error Behavior

The parser should reject:

- unknown flags
- missing required values
- invalid integers
- invalid booleans
- inline values for value-less flags
- attached short values when `attached_short_value` is disabled
- unsupported short clusters

The application may map parse errors to its own user-facing message, but parse
failures should be deterministic and non-zero.

### ERR-REPORTING: Error Reporting

Implementations should use command, flag, and argument definitions to produce
specific user-facing errors. Error messages should identify the active command,
the offending token when available, and the expected shape.

Recommended error details:

- unknown option name
- missing value and the expected `value_name`
- invalid value and the expected value kind
- too few or too many positional arguments
- the active command usage line
- a help hint for the active command

Examples:

```text
error: unknown option '--numbr'

Usage: command ls [options]

Try 'command ls --help' for more information.
```

```text
error: option '--number' requires <N>

Usage: command ls [options]
```

```text
error: invalid value for '--color': expected BOOL, got 'maybe'
```

```text
error: too many arguments for 'command ls'

Usage: command ls [options]
```

If an implementation supports additional metadata such as allowed values, it
should include those in the error:

```text
error: invalid value for '--color': expected one of auto, always, never
```

## HELP-GENERATION: Help Generation

Help output is generated from command and flag specifications, not copied from
static option text.

Generated command help should include:

```text
Description

Usage: program command [options]

Options:
  -x, --example <VALUE>  Description
  -a, --attr <ATTR>      Description [repeatable]
      --long-only        Description
  -h, --help             Print help
```

Rules:

- Preserve flag declaration order.
- Align descriptions in a readable column.
- Wrap descriptions so the total rendered line length is at most 120
  characters. Wrapped continuation lines should align with the description
  column.
- Use `value_name` for placeholders.
- Show optional boolean values as `--flag[=BOOL]`.
- Append ` [default: VALUE]` when `default_value` is set.
- Append ` [repeatable]` when a flag may be used multiple times.
- Add `-h, --help` automatically unless explicitly disabled by the
  implementation.
- Extra reference text may be appended after the generated options block.

Root help may also include a generated command list:

```text
Commands:
  ls   List entries
  cat  Print an entry
```

Command descriptions in that list should come from the same command
specification used for subcommand help.

## EXAMPLE-SPEC: Example Specification

This is an illustrative schema, not a required file format:

```yaml
name: ls
description: List entries
usage: command ls [options]
arguments:
  - name: REF
    description: Optional refs to list
    required: false
    repeatable: true
flags:
  - name: number
    short: n
    value: int
    value_name: N
    description: Limit number of entries shown
    attached_short_value: true
  - name: attr
    short: a
    value: string
    value_name: ATTR
    description: Filter or display attributes
    repeatable: true
    attached_short_value: true
  - name: color
    value: bool_required
    value_name: BOOL
    description: Color output
    default_value: "true"
  - name: reverse
    short: r
    value: none
    description: Show oldest first
extra_help: |
  Format tokens:
    %i       short ID
    %I       full ID
```

The same shape as Python data structures:

```python
from dataclasses import dataclass, field
from enum import Enum


class ValueKind(Enum):
    NONE = "none"
    STRING = "string"
    INT = "int"
    BOOL_REQUIRED = "bool_required"
    BOOL_OPTIONAL = "bool_optional"


@dataclass
class FlagSpec:
    name: str
    short: str | None = None
    value: ValueKind = ValueKind.NONE
    value_name: str | None = None
    description: str = ""
    default_value: str | None = None
    repeatable: bool = False
    attached_short_value: bool = False


@dataclass
class ArgumentSpec:
    name: str
    description: str = ""
    required: bool = False
    repeatable: bool = False


@dataclass
class CommandSpec:
    name: str
    description: str
    usage: str
    flags: list[FlagSpec] = field(default_factory=list)
    arguments: list[ArgumentSpec] = field(default_factory=list)
    extra_help: str | None = None


ls_spec = CommandSpec(
    name="ls",
    description="List entries",
    usage="command ls [options]",
    arguments=[
        ArgumentSpec(
            name="REF",
            description="Optional refs to list",
            repeatable=True,
        ),
    ],
    flags=[
        FlagSpec(
            name="number",
            short="n",
            value=ValueKind.INT,
            value_name="N",
            description="Limit number of entries shown",
            attached_short_value=True,
        ),
        FlagSpec(
            name="attr",
            short="a",
            value=ValueKind.STRING,
            value_name="ATTR",
            description="Filter or display attributes",
            repeatable=True,
            attached_short_value=True,
        ),
        FlagSpec(
            name="color",
            value=ValueKind.BOOL_REQUIRED,
            value_name="BOOL",
            description="Color output",
            default_value="true",
        ),
        FlagSpec(
            name="reverse",
            short="r",
            description="Show oldest first",
        ),
    ],
    extra_help="""Format tokens:
  %i       short ID
  %I       full ID
""",
)
```

That specification should accept:

```text
command ls -n10
command ls --number 10
command ls --number=10
command ls -a val1 -a val2
command ls --color=false
command ls -r
```

It should reject:

```text
command ls -rn10
command ls --reverse=true
command ls -n
command ls --number abc
```

## TESTS-CONFORMANCE: Conformance Tests

These commands use the illustrative `command ls` specification above. Any
implementation of this parser shape should produce the same pass/fail result.
The exact parsed data structure may vary by language, but accepted commands
must preserve the same option names, values, order for repeatable flags, and
positionals.

### TESTS-PASS: Should Pass

```text
command ls
command ls --
command ls ref1
command ls ref1 ref2
command ls -- ref1 -literal
command ls --number 10
command ls --number=10
command ls -n 10
command ls -n10
command ls --attr val1
command ls --attr=val1
command ls -a val1
command ls -aval1
command ls --attr val1 --attr val2
command ls -a val1 -a val2
command ls --color true
command ls --color=false
command ls --color YES
command ls --color 0
command ls --reverse
command ls -r
command ls --number 1 --number 2
command ls --attr val1 ref1 ref2
command ls ref1 --attr val1
command ls --number 10 --attr val1 --color=false --reverse ref1
```

Expected notes:

- `--number 1 --number 2` passes; the final non-repeatable value takes
  precedence.
- `--attr val1 --attr val2` passes; both values remain available to the
  command in order.
- `--` ends option parsing, so `-literal` is a positional.

### TESTS-FAIL: Should Fail

```text
command ls --unknown
command ls --num
command ls --number
command ls --number=
command ls --number abc
command ls --number 1.5
command ls --number -1
command ls -n
command ls -nabc
command ls --color
command ls --color maybe
command ls --color=maybe
command ls --reverse=true
command ls -rtrue
command ls -rn10
command ls -ra
command ls -abc
command ls --attr
command ls --attr val1 --unknown
```

Expected notes:

- `--num` fails; long flag abbreviation is not supported.
- `--number=` fails because the value is not a valid integer.
- `--color` fails because `color` is `bool_required`, not `bool_optional`.
- `-rn10`, `-ra`, and `-abc` fail because short flag clusters are not
  supported.
- `--reverse=true` fails because `reverse` is value-less.
