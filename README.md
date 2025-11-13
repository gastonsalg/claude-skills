# Claude Code Skills

Personal collection of Claude Code skills for PR workflow, database migrations, and skill authoring.

## Available Skills

### Universal PR Workflow

**create-pr** (165 lines)
- Creating PRs with proper git workflow
- Branch management, tests, code review integration
- Trigger: "create a PR", "commit and push"

**review-pr** (208 lines)
- Reviewing others' PRs with inline comments
- Copilot-style suggestions, thread resolution
- GitHub API for inline comments and replies
- Trigger: "review PR #123"

**manage-pr-feedback** (295 lines)
- Responding to feedback on YOUR PRs (author perspective)
- Reply to comments, resolve threads systematically
- Track and address reviewer feedback
- Trigger: "address PR feedback", "respond to Copilot"

### Database Migrations

**safe-migration** (207 lines)
- Alembic migration safety review
- Checks for dangerous operations, conflicts, table locks
- Production deployment risk assessment
- Trigger: "review migration", "check alembic"

### Meta

**write-skills** (363 lines)
- Creating effective Claude Code skills
- Follows official Anthropic guidelines + community best practices
- WHAT not HOW principle, proper structure
- Trigger: "create a skill", "write a skill"

## Installation

### Clone to Personal Skills Directory

```bash
# Backup existing skills (if any)
mv ~/.claude/skills ~/.claude/skills.backup

# Clone this repo as your skills directory
git clone https://github.com/gastonsalg/claude-skills.git ~/.claude/skills
```

Skills will be available across all your projects immediately.

### Or Install Individual Skills

```bash
# Create skills directory if it doesn't exist
mkdir -p ~/.claude/skills

# Clone and copy specific skills
git clone https://github.com/gastonsalg/claude-skills.git /tmp/claude-skills
cp -r /tmp/claude-skills/create-pr ~/.claude/skills/
cp -r /tmp/claude-skills/review-pr ~/.claude/skills/
# ... copy others as needed
```

## Usage

Skills activate automatically based on your requests:

- "Create a PR for this feature" → `create-pr`
- "Review PR #123" → `review-pr`
- "Address the Copilot feedback on my PR" → `manage-pr-feedback`
- "Check if this Alembic migration is safe" → `safe-migration`
- "Help me write a skill for X" → `write-skills`

## Organization

**Personal Skills** (`~/.claude/skills/`):
- Available across ALL your projects
- Private to you
- This repo

**Project Skills** (`project/.claude/skills/`):
- Specific to one project
- Committed to git, shared with team
- Copy relevant skills from this repo

## Skill Design Principles

All skills follow these principles (from `write-skills`):

✅ **WHAT not HOW**: Describe what to achieve, not exact commands
✅ **Focused scope**: One capability per skill
✅ **Address real failures**: Document recurring problems
✅ **Common Mistakes section**: Learn from actual failure modes
✅ **Concise**: Target 150-220 lines
✅ **Empowering**: Guide Claude, don't hand-hold

## Maintenance

To update skills:

```bash
cd ~/.claude/skills
git pull origin main
```

To contribute improvements:

```bash
cd ~/.claude/skills
# Make changes
git add .
git commit -m "improve: describe change"
git push origin main
```

## Background

These skills were created during development of an ETL project, addressing recurring failures in:
- PR creation workflow (pushing to main, skipping tests)
- PR reviews (blob comments instead of inline, wrong API usage)
- Migration safety (dangerous operations, production risks)

The collection emphasizes practical, battle-tested patterns over theoretical best practices.

## Related

- [Official Claude Code Skills Docs](https://code.claude.com/docs/en/skills)
- [Anthropic Skills Announcement](https://www.anthropic.com/news/skills)
- [Community Skills Repository](https://github.com/anthropics/skills)

## License

Public domain - use freely, modify as needed, no attribution required.
