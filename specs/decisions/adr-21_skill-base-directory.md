# ADR: Skill Base Directory Reference Pattern

## Problem

Skills with subdirectories (templates/, references/, workflows/) need to reference their own files, but Claude frequently searches the wrong location (the user's project directory instead of the skill's cache directory).

## Context

- **Business**: Skills that can't find their templates fail silently or produce incorrect output, wasting user time
- **Technical**: Skills are cached at `~/.claude/plugins/cache/{marketplace}/{plugin}/{version}/skills/{skill-name}/`. Claude Code provides this path when loading a skill via the "Base directory for this skill:" message.

## Decision

**Skills shall document that the base directory is provided by Claude Code at load time, and explicitly instruct Claude to use that path for all skill-internal file references.**

## Rationale

Three approaches were considered:

1. **`${SKILL_DIR}` variable convention** - Document paths using a placeholder. Problem: Not actually interpolated; Claude must manually substitute, leading to frequent errors.

2. **Glob pattern discovery** - Search `.claude/plugins/cache/**/{skill-name}/...`. Problem: Slow, may match multiple versions, pattern complexity.

3. **Use the skill loading message** (chosen) - Claude Code already provides the exact path. This is authoritative, fast, and requires no pattern matching. Skills document that Claude should use this path directly.

Option 3 was chosen because it uses an existing, authoritative mechanism that requires no additional infrastructure.

## Trade-offs Accepted

- **Skill must be actively invoked**: The base directory is only known when the skill is loaded. Mitigation: Skills are always invoked before their files are needed.
- **Requires skill documentation update**: Each skill with subdirectories must document this pattern. Mitigation: A standard section template makes this straightforward.

## Validation

### How to Recognize Compliance

You're following this decision if:

- Skills with templates/references/workflows document "use the base directory from the loading message"
- File reads use absolute paths from the loading message, not relative paths or globs

### MUST

- Document the base directory pattern in skills that have subdirectories
- Use the path provided in "Base directory for this skill:" for all skill-internal file access

### NEVER

- Search the project directory for skill files
- Use `${SKILL_DIR}` as if it were interpolated at runtime
- Hardcode version numbers in skill file paths
