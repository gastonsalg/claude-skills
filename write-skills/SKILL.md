---
name: write-skills
description: Creates effective Claude Code skills following official Anthropic guidelines and community best practices. Use when creating or refactoring skills. Focuses on WHAT not HOW, proper structure, and addressing real failure modes.
---

# Write Skills Skill

## Overview

This skill encodes best practices for creating Claude Code skills, synthesized from official Anthropic documentation and the community "writing-skills" TDD methodology.

**Key principle**: Skills should tell Claude WHAT to do (principles), not HOW (exact commands). Skills empower rather than hand-hold.

---

## When Skills Are Appropriate

### ✅ Good Use Cases (Create a Skill)
- **Enforcing critical workflows** (e.g., git branch rules, production safety)
- **Teaching specialized domain knowledge** (e.g., Alembic safety patterns)
- **Encoding project-specific conventions** (e.g., PR review style)
- **Addressing recurring failures** (documented baseline problems)

### ❌ Bad Use Cases (Use Documentation Instead)
- **Reference material** (templates, checklists, examples) → docs/
- **Extensive how-to guides** (step-by-step tutorials) → docs/
- **General best practices** (already in training data)
- **One-time tasks** (no recurring pattern)

### The Test
Ask: "Does this address a recurring failure or encode unique knowledge?"
- **Yes** → Skill
- **No** → Documentation

---

## Official Anthropic Requirements

### File Structure (CRITICAL)

**The skill file MUST be named `SKILL.md` (uppercase)**, not `.claude.md`, `skill.md`, or any other variation.

**Directory structure:**
- Personal skills: `~/.claude/skills/skill-name/SKILL.md`
- Project skills: `.claude/skills/skill-name/SKILL.md`

**Common naming mistakes:**
- ❌ `.claude.md` (incorrect)
- ❌ `skill.md` (lowercase - incorrect)
- ❌ `Skill.md` (mixed case - incorrect)
- ✅ `SKILL.md` (correct)

### YAML Frontmatter
```yaml
---
name: skill-name           # lowercase, hyphens, max 64 chars
description: Brief description of what this skill does and when to use it. Max 1024 chars. Include trigger terms that help Claude identify when to activate this skill.
---
```

### Key Guidelines
- **Focused scope**: One capability per skill
- **Specific descriptions**: Include trigger terms and use cases
- **Clear naming**: Descriptive, lowercase, hyphens only (for directory and name field)
- **No duplication**: Don't overlap with other skills

---

## Recommended Structure

### 1. Overview (REQUIRED)
**Purpose**: Explain WHY this skill exists

**Include**:
- What recurring failures this addresses
- Why these failures matter (consequences)
- Context (2-3 sentences)

**Example**:
```markdown
## Overview

This skill addresses recurring failures in PR reviews:
- Creating blob comments instead of inline comments
- Using wrong GitHub API endpoints
- Not using suggestion blocks for fixes

**Why this matters**: Inline comments keep discussions contextual and actionable.
```

### 2. Core Principles (WHAT to do)
**Purpose**: High-level guidance, not commands

**Structure**:
- Bullet lists
- Principles, not procedures
- "What" not "how"

**Example**:
```markdown
## Core Principles

### What to Review
- **Security**: SQL injection, XSS, exposed secrets
- **Logic**: Off-by-one errors, null handling
- **Performance**: N+1 queries, unnecessary loops

### Review Style
- **Concise**: One finding per comment
- **Objective**: Focus on facts, not validation
```

### 3. Workflow (High-Level)
**Purpose**: Major steps without verbose commands

**Keep brief**:
- Step names only (1-2 sentences each)
- Reference what to do, not exact syntax
- Link to docs for details if needed

**Example**:
```markdown
## Workflow

### 1. Setup
- Use TodoWrite to track progress
- Check existing reviews to avoid duplicates
- Get PR context and checkout branch

### 2. Analyze Code
- Assume bugs exist - hunt systematically
- Check security, logic, performance, architecture
```

### 4. API Quick Reference (Optional)
**Purpose**: Essential syntax for APIs/commands

**When to include**:
- ✅ If skill requires specific API/command syntax
- ✅ If syntax is non-obvious or error-prone
- ❌ Skip if commands are standard (git, basic tools)

**Keep minimal**:
- One example per concept
- Essential parameters only
- Brief comments

**Example**:
```markdown
## API Quick Reference

### Create Inline Comment
\`\`\`bash
COMMIT_SHA=$(gh pr view $PR --json headRefOid --jq '.headRefOid')

gh api repos/{owner}/{repo}/pulls/$PR/comments --method POST \\
  --field body="Issue + fix" \\
  --field commit_id="$COMMIT_SHA" \\
  --field path="file.py" \\
  --field line=174 \\
  --field side="RIGHT"
\`\`\`
```

### 5. Common Mistakes (REQUIRED)
**Purpose**: Document real failure modes

**Structure**:
```markdown
### ❌ Mistake Title
**Problem**: What goes wrong
**Fix**: How to avoid it
```

**Source failures from**:
- User-reported recurring issues
- TDD RED phase (observed baseline failures)
- Your own testing/experience

**Example**:
```markdown
### ❌ Creating Blob Comments
**Problem**: Using regular PR comments instead of inline
**Fix**: Use /pulls/{pr}/comments endpoint with line parameter

### ❌ Wrong API Endpoint
**Problem**: Using /reviews endpoint with line parameter (doesn't work)
**Fix**: Use /comments endpoint for inline, /reviews for summary
```

### 6. Red Flags (Optional but Recommended)
**Purpose**: Fast-fail conditions

**Format**: Bullet list of situations to abort/warn

**Example**:
```markdown
## Red Flags (Fail Fast)

- ❌ Current branch is main/master
- ❌ Tests failing
- ❌ Critical security issues unresolved
```

---

## Length Guidelines

**Target**: 150-220 lines per skill

**Breakdown**:
- Overview: ~15 lines
- Core Principles: ~40 lines
- Workflow: ~30 lines
- API Reference: ~40 lines (if needed)
- Common Mistakes: ~50 lines
- Red Flags: ~10 lines

**If skill exceeds 250 lines**:
- Consider splitting into multiple skills
- Move reference material to docs/
- Consolidate duplicated content

---

## WHAT vs HOW - The Key Distinction

### ❌ TOO MUCH HOW (Hand-Holding)
```markdown
## Workflow

1. **Check branch**:
   \`\`\`bash
   CURRENT_BRANCH=$(git branch --show-current)
   if [[ "$CURRENT_BRANCH" == "main" ]]; then
       echo "❌ On main"
       git checkout -b feature/name
   fi
   \`\`\`

2. **Stage files**:
   \`\`\`bash
   git add file1.py file2.py
   git diff --staged
   \`\`\`
```

**Problems**:
- Verbose bash commands throughout
- Hand-holding exact syntax
- Assumes Claude doesn't know basic commands
- Makes skill fragile (syntax changes)

### ✅ RIGHT BALANCE (Empowering)
```markdown
## Workflow

### 1. Verify Git State
- Check current branch - if on main/master, create feature branch
- Verify changes exist before proceeding
- Review staged files (avoid secrets)

### 2. Run Tests
**Test strategy based on changes:**
- **Migrations changed**: Run alembic safety checks
- **ETL code changed**: Run targeted regression tests
- **Refactoring**: Run pytest for golden master comparison
```

**Strengths**:
- Describes WHAT to do
- Lets Claude choose HOW
- Concise, scannable
- Resilient to command changes

---

## Testing Your Skill (Optional - Community Practice)

### RED Phase (Document Baseline Failures)
1. Test Claude WITHOUT the skill
2. Document exact failures (what it says, what it does wrong)
3. Note rationalizations it uses to avoid correct behavior

### GREEN Phase (Verify Skill Works)
1. Add the skill
2. Test same scenarios
3. Verify failures are now prevented

### REFACTOR Phase (Close Loopholes)
1. Try to find ways around the skill
2. Add "Common Mistakes" or "Red Flags" for workarounds discovered
3. Test again

**Note**: This TDD methodology is community best practice, not Anthropic requirement.

---

## Common Mistakes in Writing Skills

### ❌ Too Much HOW, Not Enough WHAT
**Problem**: Verbose command examples instead of principles
**Fix**: Describe what to achieve, let Claude choose commands

### ❌ Reference Material Disguised as Skill
**Problem**: 300+ lines of templates and checklists
**Fix**: Move to docs/, keep skill as principles + links

### ❌ No "Recurring Failures" Context
**Problem**: Unclear why skill exists
**Fix**: Add Overview explaining real problems this solves

### ❌ Missing "Common Mistakes" Section
**Problem**: No documentation of failure modes
**Fix**: Document real failures from user experience

### ❌ Duplicating Built-In Knowledge
**Problem**: Teaching Claude things it already knows
**Fix**: Only encode unique project/domain knowledge

### ❌ Unclear Trigger Description
**Problem**: Claude doesn't know when to use skill
**Fix**: Add specific trigger terms and use cases to description

### ❌ Scope Too Broad
**Problem**: Skill tries to cover multiple distinct capabilities
**Fix**: Split into focused skills (one capability each)

### ❌ Wrong Filename
**Problem**: Looking for `.claude.md` or `skill.md` (lowercase) instead of `SKILL.md`
**Fix**: The file MUST be named `SKILL.md` (uppercase). Located at `~/.claude/skills/skill-name/SKILL.md` or `.claude/skills/skill-name/SKILL.md`

**Example of this mistake:**
- Attempting to read `/path/to/skill/.claude.md` (doesn't exist)
- Attempting to read `/path/to/skill/skill.md` (lowercase - doesn't exist)
- Correct path: `/path/to/skill/SKILL.md` (uppercase)

---

## Skill Locations

**Personal Skills** (`~/.claude/skills/`):
- Available across all your projects
- Private to you
- Good for: Personal workflow preferences

**Project Skills** (`.claude/skills/`):
- Specific to one project
- Committed to git, shared with team
- Good for: Project-specific patterns, team conventions

**Which to use?**:
- Universal workflows (PR management) → Personal
- Project-specific patterns (migration safety) → Project
- Team testing → Both (personal + project copies)

---

## Refactoring Existing Skills

**When to refactor**:
- Skill exceeds 250 lines
- Lots of verbose command examples
- Duplicated content
- Missing "Common Mistakes" section
- No clear overview

**Refactoring process**:
1. Identify WHAT vs HOW sections
2. Reduce HOW (consolidate commands to API reference)
3. Expand WHAT (principles, workflow)
4. Add "Common Mistakes" from real failures
5. Add Overview with recurring failures
6. Aim for 20-40% reduction while keeping knowledge

---

## Red Flags When Writing Skills

- ❌ Skill is mostly bash commands
- ❌ Over 300 lines without clear sections
- ❌ No mention of recurring failures
- ❌ Duplicates general knowledge
- ❌ Could be a docs page instead
- ❌ Overlaps significantly with another skill
- ❌ No "Common Mistakes" section
- ❌ File not named `SKILL.md` (uppercase)
