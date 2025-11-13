---
name: review-pr
description: Reviews a Pull Request with inline code comments and structured feedback. Posts comments directly on specific lines of code (like Copilot) instead of monolithic review blobs. Use when asked to review a PR.
---

# Review Pull Request Skill

## Overview

This skill addresses recurring failures in PR reviews:
- Creating blob comments instead of inline comments on specific lines
- Using wrong GitHub API endpoints (reviews endpoint vs comments endpoint)
- Not using suggestion blocks for one-click code fixes
- Verbose, over-praising feedback instead of concise findings

**Why this matters**: Inline comments keep discussions contextual and actionable. Blob comments scatter feedback and make it hard to track what's addressed.

---

## Core Principles

### What to Review
- **Security**: SQL injection, XSS, exposed secrets, auth bypasses
- **Logic**: Off-by-one errors, null handling, edge cases
- **Performance**: N+1 queries, unnecessary loops, memory leaks
- **Architecture**: Violations of project patterns (see CLAUDE.md, ARCHITECTURE.md)
- **Testing**: Missing tests, inadequate coverage

### Review Style
- **Concise**: One finding per comment, no verbosity
- **Objective**: Focus on facts, not validation
- **Specific**: Reference exact lines, provide fixes
- **Non-redundant**: Don't duplicate other reviewers' feedback

### Comment Format
- **Emoji prefix**: ❌ Critical | ⚠️ Warning | 💡 Suggestion | 🔍 Question | ✅ Strength
- **Structure**: Issue (1 sentence) + Fix (code suggestion)
- **Inline placement**: MANDATORY for code-specific issues

---

## Workflow

### 1. Setup and Checkout
- Use TodoWrite to track review progress
- Get PR context (title, body, linked issues, files changed)
- **Get actual branch name**: `gh pr view {pr} --json headRefName`
- **Checkout and pull latest**: `git fetch origin {branch} && git checkout {branch} && git pull`
- Verify you're on correct branch: `git branch --show-current`

### 2. Check Existing Feedback
**CRITICAL: See what's already been reported**
- Fetch all existing inline comments: `gh api repos/{owner}/{repo}/pulls/{pr}/comments`
- Fetch PR conversation reviews: `gh pr view {pr} --json reviews`
- Check recent commits - look for "Fix:" commits addressing previous feedback
- Note which issues have been claimed as fixed

### 3. Review Current State
**Review the ACTUAL code as it exists NOW, not just diffs or commits**
- **Read the actual files**: Use Read tool on changed files to see complete current state
- **Don't rely solely on diffs**: Diffs show what changed, not what exists
- **Don't trust commit messages**: "Fix X" doesn't mean X is actually fixed
- **Verify claimed fixes**: If previous comments say "fixed", check if actually fixed in current code
- Compare current code against existing feedback to see what's truly resolved

### 4. Understand Context
- **Read project docs**: CLAUDE.md, README.md, ARCHITECTURE.md, CONTRIBUTING.md
- **Check commit history**: `git log origin/main..HEAD` to understand evolution
- **Review diff**: `gh pr diff` to see what changed from base
- **Understand intent**: What problem is this PR solving?

### 5. Analyze Code (Adversarial Mindset)
- **Assume bugs exist** - Hunt for them systematically
- **Check security first** - Most critical findings
- **Verify architecture** - Does it follow project patterns?
- **Test coverage** - Are edge cases handled?

### 6. Post NEW Findings Only
**For code-specific issues** - Post inline comments on exact lines:
- Use `/repos/{owner}/{repo}/pulls/{pr}/comments` endpoint
- Requires: `commit_id`, `path`, `line`, `side` ("RIGHT" for new/modified, "LEFT" for deleted)
- Use `suggestion` code fence for one-click fixes

**For architectural/conceptual feedback** - Use review summary

### 7. Resolve Addressed Threads
**CRITICAL: Clean up resolved issues**
- Fetch unresolved review threads (see API reference)
- For each unresolved thread created by you (gastonsalg):
  - Check if issue is fixed in current code
  - Check if author replied explaining the fix
  - If addressed: Resolve the thread using GraphQL mutation
- **Don't leave threads unresolved** if they've been tackled

### 8. Create Review Summary
- Determine event: APPROVE (no blockers) | REQUEST_CHANGES (critical issues) | COMMENT (suggestions only)
- List NEW findings by severity with file:line references
- State approval rationale clearly
- Keep concise - no PR overview, no file lists

### 9. Return to Main Branch
- Always return to main after review

---

## API Quick Reference

### Create Inline Comment (with Suggestion)
```bash
# Get commit SHA
COMMIT_SHA=$(gh pr view $PR --json headRefOid --jq '.headRefOid')

# Post inline comment
gh api repos/{owner}/{repo}/pulls/$PR/comments --method POST \
  --field body="⚠️ **Warning**: Missing error handling.

\`\`\`suggestion
try:
    result = operation()
except SpecificError as e:
    logger.error(f\"Failed: {e}\")
    return None
\`\`\`

Handle exceptions to prevent silent failures." \
  --field commit_id="$COMMIT_SHA" \
  --field path="file.py" \
  --field line=174 \
  --field side="RIGHT"
```

### Reply to Comment
```bash
gh api --method POST repos/{owner}/{repo}/pulls/$PR/comments/$COMMENT_ID/replies \
  --field body="Fixed in latest commit. Now validates input before processing."
```

### Resolve Thread
```bash
# List unresolved threads
gh api graphql -f query='
query {
  repository(owner: "{owner}", name: "{repo}") {
    pullRequest(number: '$PR') {
      reviewThreads(first: 100) {
        nodes {
          id
          isResolved
          comments(first: 1) { nodes { body } }
        }
      }
    }
  }
}'

# Resolve thread
gh api graphql -f query='
mutation($threadId: ID!) {
  resolveReviewThread(input: {threadId: $threadId}) {
    thread { isResolved }
  }
}' -f threadId="THREAD_ID"
```

### Create Review Summary
```bash
gh pr review $PR --approve -b "## Review Summary

**Status**: Ready to merge

**Reviewed**: 3/3 files

---

**⚠️ Warnings**:
1. Missing error handling at file.py:174
2. SQL injection risk at db.py:89

**💡 Suggestions**:
1. Extract constant at calc.py:42

---

**Rationale**: Warnings are minor, not blocking."
```

---

## Common Mistakes

### ❌ Creating Blob Comments
**Problem**: Using regular PR comments instead of inline comments
**Fix**: Use `/pulls/{pr}/comments` endpoint with `line` parameter

### ❌ Wrong API Endpoint
**Problem**: Using `/reviews` endpoint with `line` parameter (doesn't work)
**Fix**: Use `/comments` endpoint for inline comments, `/reviews` for summary

### ❌ Not Using Suggestion Blocks
**Problem**: Describing fixes in prose instead of showing code
**Fix**: Use `suggestion` code fence - GitHub creates one-click apply button

### ❌ Leaving Threads Unresolved
**Problem**: Not resolving threads after issues are fixed or answered
**Fix**: Check unresolved threads at end of review, resolve those that have been addressed

### ❌ Verbose Feedback
**Problem**: Over-explaining, excessive praise, repeating context
**Fix**: One finding per comment, state issue + fix only

### ❌ Duplicating Feedback
**Problem**: Repeating what other reviewers already said
**Fix**: Check existing reviews first, only add new findings

### ❌ Ignoring Project Standards
**Problem**: Reviewing against generic best practices
**Fix**: Read CLAUDE.md and ARCHITECTURE.md first

### ❌ Re-Reporting Fixed Issues
**Problem**: Reporting issues that were already fixed in recent commits
**Fix**: Checkout PR branch, read actual current files, verify issue exists in current code before reporting

### ❌ Reviewing Diffs Instead of Actual Code
**Problem**: Only looking at diffs/commits without reading complete current files
**Fix**: Always read the full current files to see actual state, not just what changed

---

## Red Flags (Fail Fast)

- ❌ Running commands before TodoWrite
- ❌ Operating on wrong branch (not PR branch)
- ❌ Not checking out and pulling latest PR branch code
- ❌ Reviewing diffs/commits instead of reading actual current files
- ❌ Skipping existing feedback check (inline comments + reviews)
- ❌ Posting blob comments instead of inline
- ❌ Assuming code is correct without adversarial analysis
- ❌ Reporting issues without verifying they exist in current code
- ❌ Leaving your unresolved threads when issues have been fixed
