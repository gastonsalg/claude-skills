# Personal Claude Code Skills

This directory contains skills available across all your projects.

## Available Skills

### Universal PR Workflow
1. **create-pr** (165 lines)
   - Creating PRs with proper workflow
   - Git branch management, tests, code review
   - Trigger: "create a PR", "commit and push"

2. **review-pr** (208 lines)
   - Reviewing others' PRs with inline comments
   - Copilot-style suggestions, thread resolution
   - Trigger: "review PR #123"

3. **manage-pr-feedback** (295 lines)
   - Responding to feedback on YOUR PRs
   - Reply to comments, resolve threads systematically
   - Trigger: "address PR feedback", "respond to Copilot"

### Database Migrations
4. **safe-migration** (207 lines)
   - Alembic migration safety review
   - Checks for dangerous operations, conflicts
   - Trigger: "review migration", "check alembic"

### Meta
5. **write-skills** (363 lines)
   - Creating effective Claude Code skills
   - Follows official Anthropic guidelines
   - Trigger: "create a skill", "write a skill"

## Organization

**Personal Skills** (~/.claude/skills/):
- Available across ALL projects
- Private to you

**Project Skills** (project/.claude/skills/):
- Specific to one project
- Shared with team via git

## Maintenance

These skills are also in ~/Code/insights-etl/.claude/skills/ for team testing.

To update both:
1. Edit in ~/.claude/skills/ (master copy)
2. Copy to project: `cp -r ~/.claude/skills/SKILL ~/Code/project/.claude/skills/`
3. Commit project copy for team collaboration

## Skill Principles

- Focus on WHAT to do, not HOW
- Target 150-220 lines
- Address recurring failures
- Include "Common Mistakes" section
- One capability per skill
