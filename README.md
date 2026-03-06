# Agent Skills Collection

A curated collection of **[Agent Skills](https://github.com/agent-skills)** — modular knowledge packs that teach AI coding assistants domain-specific conventions, best practices, and architectural patterns. Each skill is self-contained and can be installed independently into any project.

---

## Available Skills

| Skill | Description | Version | License |
|-------|-------------|---------|---------|
| [SAP Commerce Cloud Development](sap-commerce-development/) | Guides SAP Commerce Cloud backend development: type system, ImpEx, Spring, OCC REST API, extensions, builds, and testing | 2.0 | Apache-2.0 |

> **More skills coming soon.** This collection is actively growing — check back for new additions or [contribute your own](#contributing).

---

## 📦 SAP Commerce Cloud Development

Currently the first and only skill in this collection. It teaches AI agents how to work with SAP Commerce Cloud (Hybris) backend codebases.

### Highlights

- 🏗️ **Architect** extensions using the proper layered pattern (Type System → Services → Facades → OCC API)
- 📋 **Define types** in `items.xml` with correct deployment, typecodes, and modifiers
- 🔧 **Configure Spring** beans, aliases, and overrides in `*-spring.xml`
- 🌐 **Build OCC REST endpoints** with controllers, WsDTOs, and `DataMapper`
- 📦 **Manage data** through the OOTB patches framework and ImpEx
- ✅ **Write tests** following the AAA pattern with JUnit 5 and Mockito
- 🚀 **Build & deploy** using the correct Ant lifecycle commands

### Key Conventions Enforced

| Rule | Why |
|------|-----|
| Never modify `hybris/bin/platform/` or `hybris/bin/modules/` | These are SAP core — changes will be overwritten on upgrade |
| Config goes in `core-customize/hybris/config/` | Project-specific config location |
| `ant clean all` after `items.xml` changes | Regenerates model classes from the type system |
| `ant updatesystem` for DB schema changes | Applies type and patch changes to the database |
| Typecodes are permanent | Never reuse or change a typecode once assigned |
| Override beans via `<alias>`, not by redefining | Preserves SAP's bean wiring while allowing customization |

### Reference Docs

- **[Type System](sap-commerce-development/reference/type-system.md)** — items.xml: types, enums, relations, deployment
- **[Patches & ImpEx](sap-commerce-development/reference/patches.md)** — Patches framework, ImpEx syntax, macros, lifecycle
- **[Spring Configuration](sap-commerce-development/reference/spring-config.md)** — Beans, aliases, overrides, list/map merge
- **[OCC REST API](sap-commerce-development/reference/occ-api.md)** — Controllers, WsDTOs, beans.xml, DataMapper
- **[Testing](sap-commerce-development/reference/testing.md)** — Unit tests, Mockito, AAA pattern

### Agent Compatibility

This skill is compatible with any agent that supports filesystem-based skills or prompt packs, as long as the folder is installed in the path that agent expects and `SKILL.md` remains the entrypoint.

| Agent / Scope | Expected Path | Notes |
|---------------|---------------|-------|
| Claude Code (personal/global) | `~/.claude/skills/sap-commerce-development/SKILL.md` | Standard personal skill location |
| Claude Code (project-local) | `.claude/skills/sap-commerce-development/SKILL.md` | Shared with a specific repository |
| OpenAI-compatible setups | [`sap-commerce-development/agents/openai.yaml`](sap-commerce-development/agents/openai.yaml) | Optional adapter config for platforms that use OpenAI agent manifests |
| Other agents | Agent-specific skill directory | Install this skill folder wherever that agent loads custom skills |

---

## Repository Structure

```
.
├── .gitignore                             # Repository-level ignores
├── README.md                              # This file
└── sap-commerce-development/              # Skill package
    ├── SKILL.md                           # Main skill definition
    ├── LICENSE.txt                        # Apache 2.0 license
    ├── agents/
    │   └── openai.yaml                    # OpenAI agent config
    └── reference/
        ├── type-system.md
        ├── patches.md
        ├── spring-config.md
        ├── occ-api.md
        └── testing.md
```

## Installation

Each skill can be installed individually. Copy or symlink the skill folder into the directory your agent uses for custom skills.

```bash
# Claude Code (personal/global)
mkdir -p ~/.claude/skills
cp -r /path/to/this-repo/sap-commerce-development ~/.claude/skills/
```

```bash
# Claude Code (project-local)
mkdir -p .claude/skills
cp -r /path/to/this-repo/sap-commerce-development .claude/skills/
```

Or add the entire collection as a Git submodule:

```bash
git submodule add <repo-url> agent-skills
ln -s agent-skills/sap-commerce-development .claude/skills/sap-commerce-development
```

## Contributing

Contributions are welcome — whether improving existing skills or adding entirely new ones.

### Improving an Existing Skill

1. **Fork** the repository
2. **Edit** the relevant files inside the skill's directory
3. **Test** by using the skill with your AI assistant on a real project
4. **Submit** a pull request

### Adding a New Skill

1. Create a new folder under the root directory with a descriptive name
2. Add a `SKILL.md` with YAML frontmatter (`name`, `description`, `license`) and instructions
3. Add a `LICENSE.txt` for the skill
4. Optionally add agent-specific configs in an `agents/` subdirectory
5. Update this root `README.md` to list the new skill in the **Available Skills** table

### Guidelines

- Keep instructions concise and example-driven
- Use generic/anonymized names in code samples
- Include both the *what* and the *why* for every rule
- Each skill should be fully self-contained

## License

Individual skills may have their own licenses — see each skill's `LICENSE.txt` for details. The SAP Commerce Cloud Development skill is licensed under the **Apache License 2.0**.
