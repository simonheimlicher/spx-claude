# SPX-Claude Plugin Marketplace

Simon Heimlicher's Claude Code plugin marketplace, based on the spec-driven development framework [SPX](https://spx.sh).

## Marketplace Is a Product

We develop the “features” of this market place like a software product. We are currently starting out from scratch, so there is not yet much to be found, but as we progress, everything will be in `specs` and `docs`.

## Always use `AskUserQuestion` Tool

**Always use the `AskUserQuestion` tool to obtain guidance from the user, such as: discover context, obtain rationale, as well as to support the user in makking the right call by asking critical questions before blindly following the user's requests**

**NEVER ask the user any questions without using the `AskUserQuestion` tool**

## Markdown Formatting Rules

**IMPORTANT: Pseudo-XML in Markdown Code Fences**

When documenting XML-like syntax that isn't valid XML (pseudo-XML with text content, no proper elements), **ALWAYS use `text` as the language identifier**, not `xml`:

```text
<!-- ✅ CORRECT: Use "text" for pseudo-XML -->
<metadata>
  timestamp: [UTC timestamp]
  project: [Project name]
</metadata>
```

**Why:** The markup formatter (`markup_fmt`) in dprint will attempt to format XML code fences and can mangle pseudo-XML syntax. Using `text` prevents this issue while maintaining syntax highlighting compatibility with most linters.

**Never use:**

- `` ```xml `` for pseudo-XML (causes formatting issues)
- `` ``` `` with no language identifier (rejected by some markdown linters)

## Documentation

### Official Anthropic Resources

**Core Documentation:**

- [Create plugins](https://code.claude.com/docs/en/plugins) - How to create and structure plugins
- [Plugin Marketplaces](https://code.claude.com/docs/en/plugin-marketplaces) - How to create and distribute marketplaces
- [Plugins Reference](https://code.claude.com/docs/en/plugins-reference) - Complete technical specifications, schemas, and CLI commands
- [Discover Plugins](https://code.claude.com/docs/en/discover-plugins) - How users find and install plugins
- [Agent Skills](https://code.claude.com/docs/en/skills) - Creating and using Skills

**Announcements:**

- [Claude Code Plugins Announcement](https://www.anthropic.com/news/claude-code-plugins) - Official plugin system launch
- [Agent Skills Introduction](https://www.anthropic.com/news/skills) - Skills feature announcement

**Best Practices:**

- [Claude Code Best Practices](https://www.anthropic.com/engineering/claude-code-best-practices) - Agentic coding patterns

## Version Management

### Versioning Rules (Conservative Approach)

All plugins follow semantic versioning: `MAJOR.MINOR.PATCH`

**MAJOR version (0.x.x → 1.x.x):**

- ⛔ **NEVER bump unless user explicitly requests it**
- All plugins remain at major version `0` until stable release
- Reserved for future stable release when all features are production-ready

**MINOR version (0.3.x → 0.4.x):**

- ✅ Adding new commands (e.g., new `/pickup` command)
- ✅ Adding new skills (e.g., new `/designing-frontend` skill)
- ✅ Major functional changes (e.g., atomic claim mechanism in `/pickup`)
- ✅ Significant user experience improvements
- 🎯 **Use sparingly** - only for substantial additions or changes

**PATCH version (0.3.1 → 0.3.2):**

- ✅ **Most common** - default for most changes
- ✅ Bug fixes
- ✅ Refactoring existing code
- ✅ Documentation improvements
- ✅ Small enhancements to existing features
- ✅ Performance optimizations
- ✅ Internal implementation changes
- 🎯 **Use liberally** - when in doubt, use PATCH

### Files to Update When Bumping Version

**Plugin version** (always update):

```bash
plugins/{plugin-name}/.claude-plugin/plugin.json
```

```json
{
  "name": "claude",
  "version": "0.4.0" // ← Update this
}
```

**Marketplace catalog** (optional, only if description changes):

```bash
.claude-plugin/marketplace.json
```

```json
{
  "plugins": [
    {
      "name": "claude",
      "source": "./plugins/claude",
      "description": "..." // ← Only update if description changes
    }
  ]
}
```

### Version Bump Workflow

**CRITICAL: Version bumps must be in the SAME commit as the changes that warrant them.**

❌ **WRONG** - Separate commits:

```bash
git commit -m "refactor(skills): simplify descriptions"
# ... then later ...
git commit -m "chore: bump versions"
```

✅ **CORRECT** - Single atomic commit:

```bash
# 1. Make your changes to skills/commands/etc
# 2. Update version numbers in plugin.json files
# 3. Stage everything together
git add plugins/*/skills/ plugins/*/.claude-plugin/plugin.json
# 4. Create ONE commit with both the changes and version bumps
git commit -m "refactor(skills): simplify descriptions

- Simplified skill descriptions from formal jargon to natural language
- All plugins: patch version bump (descriptions improved)"
```

**Rationale:** The version number is metadata about the changes, not a separate logical change. Splitting them creates awkward history where commits have changes but outdated version numbers.

**Exception:** Only create a separate version bump commit if you're bumping versions WITHOUT any code/doc changes (rare).

### Version Bump Examples

| Change                      | Old   | New   | Reason                          |
| --------------------------- | ----- | ----- | ------------------------------- |
| Add `/handoff` command      | 0.2.0 | 0.3.0 | New command = MINOR             |
| Add self-organizing handoff | 0.3.0 | 0.4.0 | Major functional change = MINOR |
| Fix typo in handoff.md      | 0.4.0 | 0.4.1 | Documentation fix = PATCH       |
| Refactor pickup logic       | 0.4.1 | 0.4.2 | Refactoring = PATCH             |
| Improve error messages      | 0.4.2 | 0.4.3 | Small enhancement = PATCH       |
| Add `/designing-frontend`   | 0.4.3 | 0.5.0 | New skill = MINOR               |

## Skill Organization Principles

### Foundational + Language-Specific Pattern

Skills follow a **reference pattern** to avoid duplication:

1. **Foundational skill** (`/testing`) - Contains core principles, methodology, and language-agnostic patterns
2. **Language-specific skills** (`/testing-python`, `/testing-typescript`) - Reference the foundational skill, provide only language-specific implementations

**Usage:** Always read the foundational skill first, then the language-specific skill for concrete patterns.

### Why This Pattern?

- **Single source of truth** - Core principles live in one place
- **No drift** - Changes to methodology propagate automatically
- **Focused content** - Each skill contains only what's unique to it
- **Maintainability** - Less duplication means less divergence over time

### Skill Invocation

Claude Code skills cannot automatically invoke other skills. However, skills can:

1. Instruct the AI to read another skill file first
2. Reference foundational concepts by skill name
3. Be invoked sequentially by the user/AI

## Test Plugin (`/testing`)

The test plugin provides BDD testing methodology with three-tier testing:

| Level          | Question                               | Requirements                     |
| -------------- | -------------------------------------- | -------------------------------- |
| 1: Unit        | "Is our logic correct?"                | Dependency injection, NO mocking |
| 2: Integration | "Does it work with real dependencies?" | Documented test harnesses        |
| 3: E2E         | "Does it work for users?"              | Documented credentials           |

**Core rules:**

- No mocking - use dependency injection at Level 1, real dependencies at Level 2+
- Progress tests (may fail) go in `specs/.../tests/`
- Regression tests (must pass) go in `test/` or `tests/`

## TypeScript Plugin

Complete TypeScript development workflow with testing, implementation, and review.

### Skills

| Skill                                | Purpose                                                    |
| ------------------------------------ | ---------------------------------------------------------- |
| `/testing-typescript`                | TypeScript-specific testing patterns (requires `/testing`) |
| `/coding-typescript`                 | Implementation workhorse with remediation loop             |
| `/reviewing-typescript`              | Strict code review with zero-tolerance                     |
| `/architecting-typescript`           | ADR producer with testing strategy                         |
| `/reviewing-typescript-architecture` | ADR validator against testing principles                   |

### Core Principles

- No mocking - dependency injection only
- Reality is the oracle
- Behavior testing, not implementation testing
- Tests at appropriate levels (Unit/Integration/E2E)
- Pure XML structure in all skills
- Self-contained skills (no inter-skill invocations)

## Python Plugin

Complete Python development workflow with testing, implementation, and review.

### Skills

| Skill                            | Purpose                                                |
| -------------------------------- | ------------------------------------------------------ |
| `/testing-python`                | Python-specific testing patterns (requires `/testing`) |
| `/coding-python`                 | Implementation workhorse with remediation loop         |
| `/reviewing-python`              | Strict code review with zero-tolerance                 |
| `/architecting-python`           | ADR producer with testing strategy                     |
| `/reviewing-python-architecture` | ADR validator against testing principles               |

### Commands

| Command       | Purpose                                |
| ------------- | -------------------------------------- |
| `/autopython` | Autonomous implementation orchestrator |

### Core Principles

- No mocking - dependency injection only
- Reality is the oracle
- Behavior testing, not implementation testing
- Tests at appropriate levels (Unit/Integration/E2E)

## Claude Plugin

Productivity skills and commands for Claude Code.

### Skills

| Skill                 | Purpose                                   |
| --------------------- | ----------------------------------------- |
| `/creating-skills`    | Create production-grade, reusable skills  |
| `/committing-changes` | Comprehensive git commit message guidance |

### Commands

| Command    | Purpose                                             |
| ---------- | --------------------------------------------------- |
| `/ci`      | Git commit with Conventional Commits (auto-context) |
| `/handoff` | Create timestamped context handoff                  |
| `/pickup`  | Load and continue from previous handoff             |

## Specs Plugin

Requirements documentation and specification skills.

### Skills

| Skill                  | Purpose                                                                                                 |
| ---------------------- | ------------------------------------------------------------------------------------------------------- |
| `/writing-prd`         | Systematic PRD creation with user value proposition, measurable outcomes, and acceptance criteria       |
| `/writing-trd`         | Systematic TRD creation with testing methodology, validation strategy, and infrastructure documentation |
| `/managing-specs`      | Manage spec-driven development structure with templates for PRDs, TRDs, ADRs, and work items            |
| `/understanding-specs` | Hierarchical context ingestion protocol that verifies all specification documents before implementation |

### Core Principles

- Testing is first-class: Validation strategy documented before implementation
- Three-tier testing: Level 1 (Unit) → Level 2 (Integration) → Level 3 (E2E)
- No mocking: Dependency injection + real infrastructure
- Infrastructure explicit: Test harnesses and credentials documented or tracked as gaps
- User confirmation required: Problem understanding and measurable outcomes
- Quantified value: Measurable outcomes with X% improvement targets

## Discovering Other Installed Skills

Search for `SKILL.md` in `.claude/plugins/cache/{marketplace-name}/{plugin-name}/`

## Naming Skills

The `name` field in SKILL.md YAML frontmatter is how users invoke your skill (`/skill-name`).

**Match user speech patterns:**

- Use domain acronyms: `writing-prd` not `writing-prd-document`
- Use terms users actually say: `testing-python` not `python-unit-test-framework`
- Think "CD-ROM" not "Compact Disc Read Only Memory"

**Directory name must match:**

- Directory: `skills/writing-prd/`
- YAML name: `name: writing-prd`
- User invokes: `/writing-prd`

**Examples from this marketplace:**

```yaml
# ✅ Good: matches user speech
name: writing-prd
# Users say "write a PRD"

name: testing-typescript
# Users say "test TypeScript code"

name: autopython
# Users say "autopython" (command name)
```

```yaml
# ❌ Bad: nobody says these
name: writing-prd-document
# Too verbose, doesn't match speech

name: typescript-testing-framework
# Wrong order, unnatural phrasing
```

## Writing effective descriptions

The description field enables Skill discovery and should include both what the Skill does and when to use it. The description is critical for skill selection: Claude uses it to choose the right Skill from potentially 100+ available Skills. The description must provide enough detail for Claude to know when to select this Skill, while the rest of SKILL.md provides the implementation details.

**Keep descriptions concise** - Claude has a character budget for all skill metadata (name, args, description). When the budget is exceeded, Claude sees only a subset of available skills, making some skills invisible.

Always write in third person. The description is injected into the system prompt, and inconsistent point-of-view can cause discovery problems.

**Match actual user speech, not formal jargon.** Use the exact words and phrases users say, avoiding technical or formal language. Use abbreviations if the user would (ADR not Architecture Decision Record). Avoid corporate speak ("hierarchical context ingestion protocol" → "read all specs").

### Include both what the Skill does and specific triggers/contexts for when to use it

**A good description answers two questions:**

1. What does this Skill do? List the specific capabilities.
2. When should Claude use it? Include trigger terms users would mention.

```yaml
description: Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.
```

This description works because it names specific **actions** (extract, fill, merge) and includes **keywords** users would say (PDF, forms, document extraction).

**Multiple Skills conflict:**

If Claude uses the wrong Skill or seems confused between similar Skills, the descriptions are probably too similar. Make each description distinct by using specific trigger terms.

For example, instead of two Skills with "data analysis" in both descriptions, differentiate them: one for "sales data in Excel files and CRM exports" and another for "log files and system metrics". The more specific your trigger terms, the easier it is for Claude to match the right Skill to your request.

### Effective examples

**PDF Processing skill:**

```yaml
description: Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.
```

**Excel Analysis skill:**

```yaml
description: Analyze Excel spreadsheets, create pivot tables, generate charts. Use when analyzing Excel files, spreadsheets, tabular data, or .xlsx files.
```

**Git Commit Helper skill:**

```yaml
description: Generate descriptive commit messages by analyzing git diffs. Use when the user asks for help writing commit messages or reviewing staged changes.
```

### Examples from this marketplace

**Specs Plugin skills (differentiating similar skills):**

```yaml
# ✅ Good: natural language, clear triggers
name: writing-prd
description: Write PRDs documenting what users need and why. Use when writing PRDs or product requirements.

name: writing-trd
description: Write TRDs documenting how to build and test it. Use when writing TRDs or technical requirements.

name: managing-specs
description: Set up specs directory with templates for PRDs, TRDs, and ADRs. Use when creating or organizing spec structure.

name: understanding-specs
description: Read all specs for a story, feature, or capability before starting work. Use when starting implementation to load requirements and context.
```

**Why these work:**

- Natural language users actually say ("what users need" not "user value propositions")
- Short and direct ("build and test" not "testing methodology and validation strategy")
- Clear action verbs (write, set up, read)
- No jargon or corporate speak

```yaml
# ❌ Bad: formal jargon instead of user speech
name: understanding-specs
description: Hierarchical context ingestion protocol that verifies all specification documents before implementation.
# Problem: Nobody says "hierarchical context ingestion protocol"

name: writing-prd
description: Systematic PRD creation with user value propositions and measurable outcomes.
# Problem: Users say "what users need" not "user value propositions"
```

**Testing Plugin skills (language-specific differentiation):**

```yaml
# ✅ Good: clear language distinction
name: testing-python
description: Python-specific testing patterns with dependency injection and real infrastructure. Use when testing Python code or writing Python tests.

name: testing-typescript
description: TypeScript-specific testing patterns with type-safe test design. Use when testing TypeScript code or writing TypeScript tests.
```

**Why these work:**

- Language is in the name AND description
- Specific to each ecosystem (dependency injection for Python, type-safe for TypeScript)
- Clear trigger: "test Python code" → testing-python

---

## Writing Skills with Templates

### Template Access Pattern (MANDATORY)

**When a skill includes templates in subdirectories, always use the `${SKILL_DIR}` variable pattern to make template locations unambiguous.**

Skills are loaded from the skill's base directory (`.claude/plugins/cache/{marketplace}/{plugin}/{version}/skills/{skill-name}/`), NOT from the user's project directory. Agents frequently guess wrong and search in the project directory instead.

#### Required Pattern

```markdown
<accessing_templates>

## How to Access Templates

**All templates are stored within this skill's base directory.**

### Understanding Skill Directory Structure

When you invoke `/{skill-name}`, Claude loads this skill from the skill's base directory. Throughout this documentation, we refer to this as `${SKILL_DIR}`.

**The skill's base directory path pattern:**

\`\`\`
.claude/plugins/cache/{marketplace-name}/{plugin-name}/{version}/skills/{skill-name}/
\`\`\`

### Template Organization

All templates are under `${SKILL_DIR}/templates/`:

\`\`\`
${SKILL_DIR}/
├── SKILL.md # This file
└── templates/
└── {template-file}.md
\`\`\`

### How to Read Templates

**Always use the skill's base directory, not the user's project directory.**

\`\`\`bash

# Pattern

Read: ${SKILL_DIR}/templates/{template-name}

# Example: Read specific template

Read: ${SKILL_DIR}/templates/example.md
\`\`\`

### Troubleshooting

If you cannot find a template:

1. ✅ Verify you're using the skill's base directory, NOT the project directory
2. ✅ Ensure path starts with `${SKILL_DIR}/templates/...`
3. ✅ Use Glob to discover: `Glob: .claude/plugins/cache/**/{skill-name}/templates/**/*.md`
4. ❌ Do NOT look for templates in the user's project

</accessing_templates>
```

#### Update All Template References

Replace all relative paths with `${SKILL_DIR}` prefix:

```bash
# ❌ WRONG - Ambiguous
Read: templates/example.md

# ✅ CORRECT - Unambiguous
Read: ${SKILL_DIR}/templates/example.md
```

### XML Tag Formatting (MANDATORY)

**Always add a blank line before closing pseudo-XML tags that follow unordered lists.**

Without the blank line, markdown parsers interpret the closing tag as part of the list item, causing incorrect indentation.

```markdown
# ❌ WRONG - Tag gets indented as part of list

- Item 1
- Item 2
- Item 3

</section>

# ✅ CORRECT - Blank line prevents indentation

- Item 1
- Item 2
- Item 3

</section>
```

This applies to all pseudo-XML tags used in skills:

- `</objective>`
- `</quick_start>`
- `</structure_definition>`
- `</accessing_templates>`
- `</adr_templates>`
- `</requirement_templates>`
- `</work_item_templates>`
- `</success_criteria>`

**Automated Enforcement:**

This formatting rule is automatically enforced by the `fix-xml-spacing` pre-commit hook:

- **Script**: `scripts/fix-xml-spacing.py`
- **When**: Runs on every commit before other formatters (priority 0)
- **What**: Detects list items followed by closing XML tags, adds blank line, removes indentation
- **Scope**: All `*.md` files in staged changes
- **Behavior**: Automatically stages fixed files

The script respects code fences and won't modify content inside `` ``` `` blocks.

---

## Restrictions on Using `!` Expansion in Commands

```zsh
# Avoid shell operators such as `(N)` (nullglob in zsh)
Error: Bash command permission check failed for pattern "!ls .spx/sessions/TODO_*.md(N) | wc -l | xargs printf "TODO: %s\n" && ls .spx/sessions/DOING_*.md(N) | wc -l | xargs printf "DOING: %s\n"": This command uses shell operators that require approval for safety

# Avoid parameter substitution
Error: Bash command permission check failed for pattern "!for f (.spx/sessions/DOING_*) print mv $f ${f/DOING/TODO}": Command contains ${} parameter substitution

# Avoid loops
Error: Bash command permission check failed for pattern "!find .spx/sessions -maxdepth 1 -name 'DOING_*' | while read f; do echo mv "$f" "$(echo "$f" | sed 's/DOING/TODO/')"; done": This Bash command contains multiple operations. The following part requires approval: while read f ;
     do echo mv "$f" "$(echo "$f" | sed ''s/DOING/TODO/'')" ; done

Error: Bash command permission check failed for pattern "!find .spx/sessions -maxdepth 1 -name 'DOING_*' | awk '{new=$0; sub(/DOING/,"TODO",new); print "mv "$0" "new}'": This Bash command contains multiple operations. The following part requires approval: awk '{new=$0;
     sub(/DOING/,""TODO"",new); print ""mv ""$0"" ""new}'
```

---

## For AI Agents Modifying This Marketplace

### ⛔ Path Restrictions

**NEVER write to these locations:**

- `.claude/` - Requires user permission for every operation, breaks workflow
- `~/.claude/` - User home directory, not project-specific
- Any path containing `.claude` in user home

**ALWAYS write to project directories:**

- `plugins/` - Plugin code, skills, commands, templates
- `specs/` - Work items, requirements, decisions (see [specs/CLAUDE.md](specs/CLAUDE.md))
- `.spx/` - Tool operational files (sessions, cache) - gitignored
- Project root - Package files, config files

**Rationale:** Claude Code requires user permission for every file operation in `.claude/` directories. This creates friction and breaks the development flow. All project artifacts belong in the project directory structure.

### Before Making Changes

1. **Read the context**: Check [CLAUDE.md](CLAUDE.md:1) (this file) for current structure and versioning rules
2. **Check existing commands**: Use Glob to find existing `.md` files in `plugins/*/commands/`
3. **Review plugin structure**: Each plugin has its own `plugin.json` in `.claude-plugin/`

### After Adding/Modifying Commands or Skills

**⚠️ CRITICAL: Version bumps must be in the SAME commit as your changes.** See [Version Bump Workflow](#version-bump-workflow) above.

**Workflow:**

1. **Make your changes** to skills, commands, templates, etc.

2. **Determine version bump type** (see [Version Management](#version-management) above):
   - **MAJOR** (0.x.x → 1.x.x): ⛔ NEVER unless user explicitly requests
   - **MINOR** (0.3.x → 0.4.x): New command/skill OR major functional change
   - **PATCH** (0.3.x → 0.3.1): Bug fixes, refactoring, small changes (MOST COMMON)

3. **Update plugin.json version** in the same working session:

   ```bash
   # Location: plugins/{plugin-name}/.claude-plugin/plugin.json
   # Update "version" field according to rules above
   ```

4. **Update marketplace description** (only if needed):

   ```bash
   # Location: .claude-plugin/marketplace.json
   # Update description for the modified plugin (only if description changes)
   ```

5. **Document changes**: Update this [CLAUDE.md](CLAUDE.md:1) file if adding new commands/skills to the plugin tables

6. **Stage and commit EVERYTHING together** in ONE commit:

   ```bash
   git add plugins/{plugin-name}/ plugins/{plugin-name}/.claude-plugin/plugin.json
   git commit -m "type(scope): your changes including version bump"
   ```

**Validation**: The pre-commit hook automatically validates marketplace and plugins. If validation fails, the commit is blocked until errors are fixed.

### Quick Reference: File Locations

```
spx-claude/
├── .claude-plugin/
│   └── marketplace.json          # Marketplace catalog
├── .spx/                          # Tool operational (gitignored)
│   └── sessions/                  # Session handoffs
│       ├── TODO_*.md             # Available for /pickup
│       └── DOING_*.md            # Currently claimed
├── plugins/
│   ├── claude/
│   │   ├── .claude-plugin/
│   │   │   └── plugin.json       # Version: 0.4.0
│   │   ├── commands/
│   │   │   ├── ci.md
│   │   │   ├── handoff.md
│   │   │   └── pickup.md
│   │   └── skills/
│   │       ├── committing-changes/
│   │       └── creating-skills/
│   ├── python/
│   │   ├── .claude-plugin/
│   │   │   └── plugin.json       # Version: 0.x.x
│   │   └── commands/
│   │       └── autopython.md
│   ├── specs/
│   │   ├── .claude-plugin/
│   │   │   └── plugin.json       # Version: 0.3.0
│   │   └── skills/
│   │       ├── managing-specs/
│   │       ├── understanding-specs/
│   │       ├── writing-prd/
│   │       └── writing-trd/
│   ├── typescript/
│   │   └── .claude-plugin/
│   │       └── plugin.json       # Version: 0.x.x
│   └── test/
│       └── .claude-plugin/
│           └── plugin.json       # Version: 0.x.x
├── specs/                         # Spec-driven development
│   ├── CLAUDE.md                 # Specs directory guide
│   ├── decisions/                 # Product-wide ADRs
│   └── work/
│       ├── backlog/              # Future work
│       ├── doing/                 # Active work
│       └── done/                  # Completed work
└── CLAUDE.md                      # This file
```

### Versioning Reminder

**When in doubt:**

- Most changes = PATCH version bump
- New items or major changes = MINOR version bump
- Major version stays at 0.x.x unless user requests otherwise
