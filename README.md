# grimoire

A personal Claude Code plugin marketplace.

## Plugins

- **github-workflow** — Generic PR workflow: `/create-pr`, `/review-pr`, `/merge-pr` with CHANGELOG and label management. Language-agnostic; reads project conventions from `CLAUDE.md`.
- **cpp-expert** — C++ best practices skill (`cpp-coding-standards`) derived from the C++ Core Guidelines. Auto-loads when reviewing or writing C++.

## Install

```
/plugin marketplace add riasc/grimoire
/plugin install github-workflow@grimoire
/plugin install cpp-expert@grimoire
```

Both install at user level (`~/.claude/plugins/`) and apply to every project on the machine.

## Update

```
/plugin marketplace update grimoire
```

## Layout

```
grimoire/
├── .claude-plugin/marketplace.json    # catalog
└── plugins/
    ├── github-workflow/
    │   ├── .claude-plugin/plugin.json
    │   └── commands/
    │       ├── create-pr.md
    │       ├── review-pr.md
    │       └── merge-pr.md
    └── cpp-expert/
        ├── .claude-plugin/plugin.json
        └── skills/
            └── cpp-coding-standards/SKILL.md
```