# Python Example

This mirrors the illustrative command shape from `CLI.md`.

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
