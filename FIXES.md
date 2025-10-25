# Skill Refactoring Tasks

## Problem Summary

**12 of 16 skills have SKILL.md files that are too long** (>650 lines). They need refactoring using progressive disclosure to move detailed content into `references/` and keep core workflows in SKILL.md.

### Skills Requiring Refactoring (by priority)

**Critical (>1,500 lines):**
- brand-guidelines: 4,232 lines → target 300 lines
- mcp-builder: 2,878 lines → target 500 lines
- internal-comms: 1,978 lines → target 250 lines
- pdf: 1,837 lines → target 400 lines
- pptx: 1,550 lines → target 400 lines
- d3js-visualization: 1,504 lines → target 350 lines

**High Priority (900-1,500 lines):**
- webapp-testing: 1,403 lines → target 350 lines
- xlsx: 1,335 lines → target 400 lines
- sql-expert: 907 lines → target 450 lines

**Medium Priority (650-900 lines):**
- env-config: 843 lines → target 450 lines
- api-designer: 667 lines → target 450 lines
- git-advanced: 660 lines → target 450 lines

---

## Agent Spawning Prompt

**Use this prompt to spawn a specialized refactoring agent for each skill:**

```
You are a Claude Skill Refactoring Specialist. Your task is to refactor the [SKILL_NAME] skill to follow Anthropic's progressive disclosure pattern.

## Context
- Current SKILL.md length: [CURRENT_LINES] lines
- Target length: [TARGET_LINES] lines (within 350-650 range)
- Must follow SKILL_CREATION_GUIDE.md standards

## Your Workflow

### Step 1: Analyze Current Structure
Read `/Users/mini/Documents/Projects/ClaudeSkills/[SKILL_NAME]/SKILL.md` and identify:
- Core workflows and instructions (keep in SKILL.md)
- Detailed API references (move to references/)
- Extensive examples (move to examples/)
- Code templates (move to examples/)
- Technical deep-dives (move to references/)

### Step 2: Create Progressive Disclosure Structure
Create new files as needed:
- `references/[topic].md` for detailed documentation
- `examples/[example].md` for extended examples
- Keep only essential quick-reference examples inline

### Step 3: Refactor SKILL.md
Keep in SKILL.md:
- YAML frontmatter (unchanged)
- Overview (2-3 paragraphs)
- Core capabilities list
- Essential workflows (step-by-step)
- Quick reference examples (3-5 lines each)
- Pointers to references/ and examples/

Move out of SKILL.md:
- Full API documentation → `references/api-reference.md`
- Long code examples → `examples/[name].md`
- Deep technical explanations → `references/advanced.md`
- Multiple use-case scenarios → `examples/use-cases.md`

### Step 4: Add Reference Links
In SKILL.md, add clear pointers:
```markdown
For detailed information, see:
- [API Reference](./references/api-reference.md)
- [Advanced Usage](./references/advanced.md)
- [Examples](./examples/)
```

### Step 5: Validate
Ensure:
- SKILL.md is [TARGET_LINES] lines ±50 lines
- All content is preserved (just reorganized)
- YAML frontmatter intact
- Clear navigation to external files
- No broken references

### Step 6: Report
Provide summary:
- Original lines vs new lines
- Files created
- Content moved
- Any issues encountered

## Constraints
- Use Haiku model for cost efficiency
- Preserve ALL content (don't delete, just reorganize)
- Maintain same tone and writing style
- Keep YAML frontmatter exactly as-is
- Follow imperative voice throughout

## Example Refactoring

**Before (in SKILL.md - 200 lines):**
```markdown
## Complex Feature X

[50 lines of detailed explanation]
[30 lines of code examples]
[40 lines of edge cases]
```

**After (in SKILL.md - 10 lines):**
```markdown
## Complex Feature X

Brief overview of Feature X capabilities. Use when [conditions].

Quick example:
```code
[3-line example]
```

For detailed documentation, see [Feature X Reference](./references/feature-x.md).
```

**New file: references/feature-x.md (120 lines):**
```markdown
# Feature X - Detailed Reference

[All the detailed content]
```

## Success Criteria
- SKILL.md is within target range
- Progressive disclosure properly implemented
- All content preserved and accessible
- Clear navigation structure
- Follows SKILL_CREATION_GUIDE.md standards
```

---

## Execution Plan

### Batch 1: Critical Priority (run in parallel)
```bash
# Spawn 6 agents simultaneously
brand-guidelines (4,232 → 300 lines)
mcp-builder (2,878 → 500 lines)
internal-comms (1,978 → 250 lines)
pdf (1,837 → 400 lines)
pptx (1,550 → 400 lines)
d3js-visualization (1,504 → 350 lines)
```

### Batch 2: High Priority (run in parallel)
```bash
# Spawn 3 agents simultaneously
webapp-testing (1,403 → 350 lines)
xlsx (1,335 → 400 lines)
sql-expert (907 → 450 lines)
```

### Batch 3: Medium Priority (run in parallel)
```bash
# Spawn 3 agents simultaneously
env-config (843 → 450 lines)
api-designer (667 → 450 lines)
git-advanced (660 → 450 lines)
```

---

## Recommended Approach

1. **Read SKILL_CREATION_GUIDE.md first** - Understand the standards
2. **Spawn agents in batches** - Run 3-6 in parallel to save time
3. **Review each refactored skill** - Ensure quality
4. **Commit after each batch** - Track progress
5. **Update README if needed** - Reflect any structural changes

---

## Quality Checklist

After refactoring, each skill should:
- ✅ SKILL.md between 350-650 lines (±50 acceptable)
- ✅ Progressive disclosure implemented
- ✅ All content preserved in references/ or examples/
- ✅ Clear navigation pointers
- ✅ YAML frontmatter intact
- ✅ Same quality and completeness
- ✅ Helper scripts unchanged
- ✅ Examples properly organized

---

## Notes

- Use **Haiku agents** for cost efficiency (this is a refactoring task, not complex creation)
- Each agent should work **independently** on one skill
- Total estimated time: ~30-45 minutes for all 12 skills
- Estimated cost: ~$2-4 total (Haiku is cheap)
- Progressive disclosure reduces token usage when skills are activated
