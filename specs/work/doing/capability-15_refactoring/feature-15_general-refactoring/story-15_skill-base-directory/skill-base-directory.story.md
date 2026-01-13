# Story: Skill Base Directory Documentation

## Purpose

Update all skills that have subdirectories (templates/, references/, workflows/) to document the base directory pattern per [ADR: Skill Base Directory](../../../../../decisions/adr-21_skill-base-directory.md).

## Skills to Update

The following 10 skills have subdirectories and need documentation updates:

| Plugin     | Skill                   | Subdirectories                   |
| ---------- | ----------------------- | -------------------------------- |
| claude     | creating-skills         | templates, references, workflows |
| specs      | managing-specs          | templates                        |
| specs      | understanding-specs     | references, workflows            |
| specs      | writing-prd             | references, workflows            |
| specs      | writing-trd             | references, workflows            |
| python     | reviewing-python        | templates                        |
| python     | architecting-python     | references                       |
| typescript | reviewing-typescript    | templates, references, workflows |
| typescript | coding-typescript       | references, workflows            |
| typescript | architecting-typescript | references                       |

## Implementation Plan

### Step 1: Define Standard Section Template

Create an `<accessing_skill_files>` section for XML-structured skills:

```text
<accessing_skill_files>
When this skill is invoked, Claude Code provides the base directory in the loading message:

    Base directory for this skill: {skill_dir}

Use this path to access skill files:

- Templates: `{skill_dir}/templates/`
- References: `{skill_dir}/references/`
- Workflows: `{skill_dir}/workflows/`

**IMPORTANT**: Do NOT search the project directory for skill files.
</accessing_skill_files>
```

For markdown-structured skills (Python), use a heading instead:

```text
## Accessing Skill Files

When this skill is invoked, Claude Code provides the base directory in the loading message:

    Base directory for this skill: {skill_dir}

Use this path to access skill files:

- References: `{skill_dir}/references/`

**IMPORTANT**: Do NOT search the project directory for skill files.
```

### Step 2: Update Each Skill's SKILL.md

For each skill in the table above:

1. Read the existing SKILL.md
2. Add the `<accessing_skill_files>` section after `<quick_start>` or at an appropriate location
3. Customize the section based on which subdirectories exist (templates, references, workflows)
4. Remove any outdated `${SKILL_DIR}` documentation that implies variable interpolation

### Step 3: Verify Updates

For each updated skill:

1. Confirm the section is present and correctly customized
2. Confirm file paths in the skill use absolute paths from the loading message
3. Confirm no misleading `${SKILL_DIR}` variable references remain

## Files Modified

1. `plugins/claude/skills/creating-skills/SKILL.md`
2. `plugins/specs/skills/managing-specs/SKILL.md`
3. `plugins/specs/skills/understanding-specs/SKILL.md`
4. `plugins/specs/skills/writing-prd/SKILL.md`
5. `plugins/specs/skills/writing-trd/SKILL.md`
6. `plugins/python/skills/reviewing-python/SKILL.md`
7. `plugins/python/skills/architecting-python/SKILL.md`
8. `plugins/typescript/skills/reviewing-typescript/SKILL.md`
9. `plugins/typescript/skills/coding-typescript/SKILL.md`
10. `plugins/typescript/skills/architecting-typescript/SKILL.md`

## Completion Criteria

- [ ] All 10 skills have `<accessing_skill_files>` section
- [ ] Section is customized per skill's actual subdirectories
- [ ] No misleading `${SKILL_DIR}` interpolation references remain
