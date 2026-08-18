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

## Installation

### GitHub Copilot app on Windows

Copy `skills\guided-change-review\SKILL.md` to:

```text
%APPDATA%\com.github.githubapp\app-skills\guided-change-review\SKILL.md
```

PowerShell:

```powershell
$target = Join-Path $env:APPDATA 'com.github.githubapp\app-skills\guided-change-review'
New-Item -ItemType Directory -Force $target | Out-Null
Copy-Item .\skills\guided-change-review\SKILL.md $target
```

### Generic user-skill installation

For hosts that support the Agent Skills convention, copy the
`skills/guided-change-review` directory into the host's user-level skills
directory (commonly `~/.copilot/skills/`) so the resulting path is
`~/.copilot/skills/guided-change-review/SKILL.md`. Restart or reload the host if
required.

These installations are local snapshots and do not update automatically. Pull
changes from this repository and recopy the skill when publishing updates.

## License

Licensed under the [MIT License](LICENSE).
