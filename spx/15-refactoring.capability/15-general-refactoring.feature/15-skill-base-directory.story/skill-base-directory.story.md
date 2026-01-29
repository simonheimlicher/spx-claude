# Story: Skill Base Directory Documentation

## Purpose

Update all skills that have subdirectories (templates/, references/, workflows/) to document the base directory pattern per [ADR-21](../../../../adr-21_skill-base-directory.md).

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

## Functional Requirements

### FR1: Add accessing_skill_files section to each skill

```gherkin
GIVEN skill has subdirectories (templates, references, or workflows)
WHEN skill SKILL.md is updated
THEN <accessing_skill_files> section is added
AND section documents base directory from loading message
AND section lists available subdirectories
AND section warns against searching project directory
```

## Tests

- [Unit: creating-skills has section](tests/creating-skills.unit.test.ts)
- [Unit: managing-specs has section](tests/managing-specs.unit.test.ts)
- [Unit: All 10 skills have section](tests/all-skills.unit.test.ts)

## Completion Criteria

- [ ] All 10 skills have `<accessing_skill_files>` section
- [ ] Section is customized per skill's actual subdirectories
- [ ] No misleading `${SKILL_DIR}` interpolation references remain
