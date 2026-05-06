# Agent Skills Collection

A curated collection of **[Agent Skills](https://github.com/agent-skills)** — modular knowledge packs that teach AI coding assistants domain-specific conventions, best practices, and architectural patterns. Each skill is self-contained and can be installed independently into any project.

---

## Available Skills

| Skill | Description | Version | License |
|-------|-------------|---------|---------|
| [SAP Commerce Cloud Development](sap-commerce-development/) | Guides SAP Commerce Cloud backend development: type system, ImpEx, Spring, OCC REST API, extensions, builds, and testing | 2.0 | Apache-2.0 |
| [SAP Commerce Cloud CLI](sap-cloud-cli/) | Manage SAP Commerce Cloud Portal (CCV2) builds, deployments, data backups, data restores, and service properties via the `sapccm` CLI | 1.0 | Apache-2.0 |
| [Personal Knowledge Wiki](wiki/) | Compiles personal data into a personal knowledge wiki for ingest, absorb, query, cleanup, and expansion workflows | - | - |

> **More skills coming soon.** This collection is actively growing — check back for new additions or [contribute your own](#contributing).

---

## 📦 SAP Commerce Cloud Development

This skill teaches AI agents how to work with SAP Commerce Cloud (Hybris) backend codebases.

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

## 🛠️ SAP Commerce Cloud CLI

This skill teaches AI agents how to operate the `sapccm` CLI tool to manage a CCV2 subscription on SAP Commerce Cloud Portal.

### Highlights

- 🏗️ **Create and monitor builds** from any Git branch with live log streaming
- 🚀 **Deploy builds** to d1 / s1 / p1 environments with full control over update mode and deployment strategy
- 💾 **Back up and restore data** across environments with status polling
- 🔧 **Inspect and update service properties** (customer-properties, initial passwords, encryption keys)
- 🔐 **Auth-failure recovery** — prompts for updated client-id, client-secret, and token-url when credentials expire

### Key Conventions Enforced

| Rule | Why |
|------|-----|
| Load `.env` before any command | Keeps subscription/env codes out of history and code |
| Confirm before `cancel` and `delete` | Cancelling a deployment or deleting a backup is irreversible |
| `database-update-mode` and `strategy` are case-sensitive | CLI rejects lowercase values silently |
| After `service-properties put`, click **Apply Configurations** in Cloud Portal | A per-service Restart does not pick up property changes |
| Set `subscription-code` globally via `sapccm config set` | Avoids repeating it on every command |

### Reference Docs

- **[Commands](sap-cloud-cli/reference/commands.md)** — Full syntax cheat sheet for every subcommand
- **[Service Codes](sap-cloud-cli/reference/service-codes.md)** — All service codes and property codes with editability notes

### Agent Compatibility

| Agent / Scope | Expected Path | Notes |
|---------------|---------------|-------|
| Claude Code (personal/global) | `~/.claude/skills/sap-cloud-cli/SKILL.md` | Standard personal skill location |
| Claude Code (project-local) | `.claude/skills/sap-cloud-cli/SKILL.md` | Shared with a specific repository |
| OpenAI-compatible setups | [`sap-cloud-cli/agents/openai.yaml`](sap-cloud-cli/agents/openai.yaml) | Optional adapter config |
| Other agents | Agent-specific skill directory | Install the skill folder wherever that agent loads custom skills |

> **Security:** The `.env` file in the skill directory holds your subscription code and is git-ignored. Never commit it. If authentication fails, update the CLI credentials via `sapccm config set client-id / client-secret / token-url`.

---

## 📚 Personal Knowledge Wiki

This skill teaches AI agents how to compile journals, notes, messages, and other personal data into a structured personal knowledge wiki.

### Highlights

- 📝 **Ingest source data** from formats like Day One, Apple Notes, Obsidian, Notion, email, CSV, and plain text
- 🧠 **Absorb entries into articles** by turning raw entries into connected wiki pages instead of flat archives
- 🔎 **Query the wiki** to answer questions from synthesized knowledge rather than re-reading source files
- 🧹 **Clean up and expand** articles with audits, backlink rebuilding, and missing-page detection
- 🗂️ **Organize knowledge** into a durable wiki structure with index, backlinks, and absorb logs

### Key Workflows

| Command | Purpose |
|---------|---------|
| `/wiki ingest` | Convert source data into normalized markdown entries |
| `/wiki absorb [date-range]` | Compile raw entries into richer wiki articles |
| `/wiki query <question>` | Answer questions by navigating the generated wiki |
| `/wiki cleanup` | Audit and improve article quality and structure |
| `/wiki breakdown` | Detect and create missing articles |
| `/wiki status` | Show overall wiki stats and progress |

### Agent Compatibility

This skill is compatible with any agent that supports filesystem-based skills or prompt packs, as long as the folder is installed in the path that agent expects and `SKILL.md` remains the entrypoint.

| Agent / Scope | Expected Path | Notes |
|---------------|---------------|-------|
| Claude Code (personal/global) | `~/.claude/skills/wiki/SKILL.md` | Standard personal skill location |
| Claude Code (project-local) | `.claude/skills/wiki/SKILL.md` | Shared with a specific repository |
| Other agents | Agent-specific skill directory | Install this skill folder wherever that agent loads custom skills |

---

## Repository Structure

```text
.
├── .gitignore                             # Repository-level ignores
├── README.md                              # This file
├── sap-commerce-development/              # Skill package
│   ├── SKILL.md                           # Main skill definition
│   ├── LICENSE.txt                        # Apache 2.0 license
│   ├── agents/
│   │   └── openai.yaml                    # OpenAI agent config
│   └── reference/
│       ├── type-system.md
│       ├── patches.md
│       ├── spring-config.md
│       ├── occ-api.md
│       └── testing.md
├── sap-cloud-cli/                         # Skill package
│   ├── SKILL.md                           # Main skill definition
│   ├── LICENSE.txt                        # Apache 2.0 license
│   ├── .env                               # Local secrets — git-ignored
│   ├── agents/
│   │   └── openai.yaml                    # OpenAI agent config
│   └── reference/
│       ├── commands.md                    # Full command syntax cheat sheet
│       └── service-codes.md              # Service and property codes
└── wiki/                                  # Skill package
    └── SKILL.md                           # Main skill definition
```

## Installation

Each skill can be installed individually. Copy or symlink the skill folder into the directory your agent uses for custom skills.

```bash
# Claude Code (personal/global)
mkdir -p ~/.claude/skills
cp -r /path/to/this-repo/sap-commerce-development ~/.claude/skills/
cp -r /path/to/this-repo/sap-cloud-cli ~/.claude/skills/
cp -r /path/to/this-repo/wiki ~/.claude/skills/
```

```bash
# Claude Code (project-local)
mkdir -p .claude/skills
cp -r /path/to/this-repo/sap-commerce-development .claude/skills/
cp -r /path/to/this-repo/sap-cloud-cli .claude/skills/
cp -r /path/to/this-repo/wiki .claude/skills/
```

> **Note for `sap-cloud-cli`:** After copying, populate `.env` in the skill directory with your credentials. The file is already git-ignored in this repository.

Or add the entire collection as a Git submodule:

```bash
git submodule add <repo-url> agent-skills
ln -s agent-skills/sap-commerce-development .claude/skills/sap-commerce-development
ln -s agent-skills/wiki .claude/skills/wiki
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

Individual skills may have their own licenses and metadata inside each skill directory. The SAP Commerce Cloud Development skill is licensed under the **Apache License 2.0**.
