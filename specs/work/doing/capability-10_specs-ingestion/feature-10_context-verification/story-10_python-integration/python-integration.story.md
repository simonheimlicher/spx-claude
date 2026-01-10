# Story: Python Integration

## Functional Requirements

### FR1: Add context_loading section to Python implementation skills

```gherkin
GIVEN Python implementation skills (coding-python, reviewing-python)
WHEN skill documentation is updated
THEN context_loading section added after essential_principles
AND section instructs invoking understanding-specs before implementation
AND section documents what understanding-specs provides (ADRs, TRD, specs)
```

#### Files created/modified

1. `plugins/python/skills/coding-python/SKILL.md` [modify]: Add context_loading section
2. `plugins/python/skills/reviewing-python/SKILL.md` [modify]: Add context_loading section

### FR2: Context loading section follows standard template

```gherkin
GIVEN context_loading section template from handoff document
WHEN section is added to Python skills
THEN section uses XML tags (<context_loading>)
AND section includes conditional check (if specs-based work item)
AND section includes invocation instruction (invoke /understanding-specs)
AND section includes what skill provides (ADR hierarchy, TRD, specs)
AND section includes abort condition (if context ingestion fails → ABORT)
```

#### Files created/modified

1. Same files as FR1

## Testing Strategy

> Stories require **Level 1** to prove core logic works.
> See testing skill for level definitions.

### Level Assignment

| Component                     | Level | Justification                         |
| ----------------------------- | ----- | ------------------------------------- |
| Documentation structure       | 1     | Text verification, no external deps   |
| Section placement correctness | 1     | Position checking in file, pure logic |

### When to Escalate

This story stays at Level 1 because:

- We're testing documentation correctness, not runtime behavior
- No real file system or skill invocation required for verification
- Can validate structure and content with text fixtures

If verification requires actual skill invocation, that's a feature-level concern (Level 2 or Level 3).

## Unit Tests (Level 1)

```typescript
// tests/unit/skills/python-integration.test.ts
import { describe, expect, it } from "vitest";
import { readFileSync } from "fs";

describe("Python Skills Context Loading", () => {
  /**
   * Level 1: Documentation structure tests
   */

  it("GIVEN coding-python skill WHEN reading file THEN context_loading section exists", () => {
    // Given
    const skillPath = "plugins/python/skills/coding-python/SKILL.md";
    const content = readFileSync(skillPath, "utf-8");

    // When: Search for context_loading section
    const hasContextLoading = content.includes("<context_loading>");

    // Then: Section exists
    expect(hasContextLoading).toBe(true);
  });

  it("GIVEN context_loading section WHEN validating content THEN includes required elements", () => {
    // Given
    const skillPath = "plugins/python/skills/coding-python/SKILL.md";
    const content = readFileSync(skillPath, "utf-8");

    // Extract context_loading section
    const sectionMatch = content.match(/<context_loading>([\s\S]*?)<\/context_loading>/);
    expect(sectionMatch).not.toBeNull();

    const sectionContent = sectionMatch![1];

    // Then: Required elements present
    expect(sectionContent).toContain("/understanding-specs");
    expect(sectionContent).toContain("ABORT");
    expect(sectionContent).toContain("ADR");
    expect(sectionContent).toContain("specs-based work item");
  });

  it("GIVEN reviewing-python skill WHEN reading file THEN context_loading section exists", () => {
    // Given
    const skillPath = "plugins/python/skills/reviewing-python/SKILL.md";
    const content = readFileSync(skillPath, "utf-8");

    // When: Search for context_loading section
    const hasContextLoading = content.includes("<context_loading>");

    // Then: Section exists
    expect(hasContextLoading).toBe(true);
  });
});
```

## Architectural Requirements

### Relevant ADRs

1. `specs/decisions/` - Product-wide ADRs (if any exist)
2. `specs/work/doing/capability-10_specs-ingestion/decisions/` - Capability-specific decisions (if created)

## Quality Requirements

### QR1: Documentation Clarity

**Requirement:** context_loading sections must be clear and actionable
**Target:** User understands when and how to invoke understanding-specs
**Validation:** Manual review + unit tests verify required elements present

### QR2: Consistency Across Skills

**Requirement:** All Python skills follow same pattern for context loading
**Target:** 100% consistency in structure and terminology
**Validation:** Unit tests verify same elements in all skills

### QR3: XML Structure Compliance

**Requirement:** context_loading section must use proper XML tags
**Target:** Valid XML structure (opening and closing tags)
**Validation:** Unit tests verify XML structure

## Completion Criteria

- [ ] All Level 1 unit tests pass
- [ ] context_loading section added to coding-python
- [ ] context_loading section added to reviewing-python
- [ ] Sections follow template from handoff document
- [ ] XML structure valid

## Documentation

No README updates required - internal skill documentation changes only.
