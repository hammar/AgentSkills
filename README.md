# Agent Skills

Public source repository for reusable agent skills maintained by Karl Hammar.
Installed copies are machine-local; treat this repository as the source of truth
for updates and redistribution.

## Guided change review

`guided-change-review` walks through an existing branch, pull request, commit, or
working-tree diff in a teaching order derived from runtime, data, or command flow.
It analyzes the complete change set first, explains exactly one major file at a
time, pauses for discussion after each file, and advances only when explicitly
asked. The walkthrough is read-only unless the user separately requests edits.

Examples:

```text
/guided-change-review
/guided-change-review --base main
/guided-change-review --pr 1234
/guided-change-review --commit <sha>
/guided-change-review --working-tree
```

## Conceptual naming review

`conceptual-naming-review` examines changed code as a domain model, identifies
names that blur important distinctions or use inconsistent terminology, and
builds a concept map before proposing focused renames. It remains read-only
until the user approves the proposal, then updates usages and validates the
result.

Examples:

```text
/conceptual-naming-review
/conceptual-naming-review --base main
/conceptual-naming-review --working-tree
/conceptual-naming-review path/to/subsystem
```

## Installation

### GitHub Copilot app on Windows

Copy the desired skill directory from `skills\` to:

```text
%APPDATA%\com.github.githubapp\app-skills\<skill-name>\SKILL.md
```

PowerShell examples:

```powershell
$skills = 'guided-change-review', 'conceptual-naming-review'
foreach ($skill in $skills) {
    $target = Join-Path $env:APPDATA "com.github.githubapp\app-skills\$skill"
    New-Item -ItemType Directory -Force $target | Out-Null
    Copy-Item ".\skills\$skill\SKILL.md" $target
}
```

### Generic user-skill installation

For hosts that support the Agent Skills convention, copy the desired directory
under `skills/` into the host's user-level skills directory (commonly
`~/.copilot/skills/`). Restart or reload the host if required.

These installations are local snapshots and do not update automatically. Pull
changes from this repository and recopy the skill when publishing updates.

## License

Licensed under the [MIT License](LICENSE).
