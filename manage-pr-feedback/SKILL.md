---
name: manage-pr-feedback
description: Manages feedback on your PRs by reading, replying to, and resolving inline comments from reviewers (Copilot, humans). Use when addressing PR review feedback as the PR author, not when reviewing others' PRs. Handles inline comments, review threads, and resolution tracking.
---

# Manage PR Feedback Skill

## Overview

This skill addresses recurring failures when responding to PR feedback:
- **Replying to comments but forgetting to resolve threads** (most common)
- Losing track of which feedback has been addressed
- Batch processing feedback (fix all → reply all → resolve all), which causes forgotten resolutions
- Not assessing whether reviewer suggestions are actually correct

**Why this matters**: Unresolved threads block PR merges, frustrate reviewers, and create bottlenecks. Proper per-comment handling ensures nothing falls through the cracks.

---

## Core Principles

### Per-Comment Workflow (Critical)
**Process each comment individually in sequence**:
1. Read comment
2. Assess if suggestion is valid (don't take for granted)
3. Fix if necessary + commit
4. Reply to comment with commit reference
5. **Resolve thread immediately** (if appropriate)
6. Move to next comment

**Why per-comment**: Prevents forgotten resolutions. Resolution happens while context is fresh, not as a separate batch step later.

### Assessment First
- **Don't blindly implement**: Evaluate whether reviewer's suggestion is correct
- **Push back respectfully**: If suggestion is wrong, explain why with reasoning
- **Ask for clarification**: If unclear, ask before implementing
- **Consider trade-offs**: Some suggestions have downsides

### When to Resolve Threads
**Resolve immediately after reply if**:
- ✅ You fixed the issue AND pushed commit
- ✅ You answered question with sufficient detail
- ✅ You declined suggestion with clear reasoning

**Don't resolve if**:
- ❌ Issue not yet fixed
- ❌ Reviewer's question wasn't fully answered
- ❌ Discussion is ongoing (awaiting reviewer response)
- ❌ Reviewer explicitly asks for re-review

### Communication Style
- **Concise**: "Fixed in [sha]. Now validates input before processing."
- **Specific**: Reference exact commits/lines changed
- **Objective**: State facts, avoid defensiveness
- **Actionable**: Clear what was done or will be done

---

## Workflow

### 1. Get All Unresolved Threads
- Fetch PR number and unresolved review threads (GraphQL API)
- Use TodoWrite to track each comment thread
- Prioritize: blocking issues → security → suggestions → questions

### 2. Process Each Comment Individually

**For each unresolved thread (in sequence)**:

#### A. Read and Assess
- Read comment carefully
- **Assess validity**: Is this suggestion correct? Are there trade-offs?
- Decide: implement, decline, or ask for clarification

#### B. Take Action Based on Assessment
**If implementing**:
- Make the code change
- Run relevant tests to verify fix
- Commit with clear message
- Push to remote

**If declining**:
- Prepare respectful explanation with reasoning
- Consider trade-offs or alternative approaches

**If unclear**:
- Prepare clarifying questions for reviewer

#### C. Reply to Comment
- State what you did or decided
- Reference commit SHA if fixed: "Fixed in [sha]"
- If declining: explain reasoning clearly
- If clarifying: ask specific questions

#### D. Resolve Thread (Immediately After Reply)
**Critical step - don't skip**:
- Get thread ID from GraphQL query
- Use GraphQL mutation to resolve thread
- **Only skip resolution if**: awaiting reviewer response to your question

**This is the step that gets forgotten in batch workflows - do it NOW while context is fresh.**

### 3. Post Summary Comment
After processing all threads:
- List what was addressed with commit references
- Request re-review from reviewers
- Mention any items awaiting reviewer response

---

## API Quick Reference

### Get Unresolved Threads
```bash
gh api graphql -f query='
query {
  repository(owner: "owner", name: "repo") {
    pullRequest(number: '$PR') {
      reviewThreads(first: 50) {
        nodes {
          id
          isResolved
          comments(first: 10) {
            nodes {
              id
              author { login }
              body
              path
              line
            }
          }
        }
      }
    }
  }
}' --jq '.data.repository.pullRequest.reviewThreads.nodes[] | select(.isResolved == false)'
```

### Reply to Comment
```bash
# Get comment ID from thread's first comment
gh api --method POST \
  repos/owner/repo/pulls/$PR/comments/$COMMENT_ID/replies \
  --field body="Fixed in commit abc1234. Now validates input before processing."
```

### Resolve Thread (CRITICAL - Don't Forget)
```bash
# Use thread ID from GraphQL query above
gh api graphql -f query='
mutation {
  resolveReviewThread(input: {threadId: "THREAD_ID"}) {
    thread {
      id
      isResolved
    }
  }
}'
```

### Post Summary and Request Re-Review
```bash
gh pr comment $PR -b "All feedback addressed:
- ✅ Fixed SQL injection (commit abc1234)
- ✅ Added error handling (commit def5678)
- ✅ Declined suggestion X (reasoning: performance trade-off)

@reviewer Ready for re-review."
```

---

## Common Mistakes

### ❌ Forgetting to Resolve Threads After Replying
**Problem**: Reply to comment, move to next item, forget to resolve thread
**Fix**: Resolve thread IMMEDIATELY after replying, before moving to next comment

### ❌ Batch Processing (Fix All → Reply All → Resolve All)
**Problem**: By the time you get to resolution step, you've lost context and forget some
**Fix**: Use per-comment workflow - complete one comment fully before moving to next

### ❌ Blindly Implementing All Suggestions
**Problem**: Implementing suggestions without assessing validity or trade-offs
**Fix**: Evaluate each suggestion critically - push back respectfully if incorrect

### ❌ Resolving Without Fixing
**Problem**: Resolving threads before actually addressing the issue
**Fix**: Only resolve after fix is committed and pushed

### ❌ Vague Replies
**Problem**: "Fixed" without specifics
**Fix**: "Fixed in commit abc1234 by adding validation on line 42"

### ❌ Not Tracking Progress
**Problem**: Losing track of which threads are addressed
**Fix**: Use TodoWrite to track each thread systematically

---

## Red Flags (Fail Fast)

- ❌ Resolving thread before pushing fix commit
- ❌ Processing multiple comments before resolving any
- ❌ Not replying to reviewer questions
- ❌ Defensive or dismissive tone in replies
- ❌ Ignoring blocking feedback

---

## Example: Processing One Comment

```markdown
**Thread**: "Add validation for integer overflow in migration"

1. **Read**: Copilot suggests checking INT max range
2. **Assess**: Valid concern - MySQL INT has max value
3. **Fix**: Add overflow check in migration, commit
4. **Reply**: "Fixed in commit 190f0ce. Added check to cap values at INT max."
5. **Resolve**: Run GraphQL mutation to resolve thread IMMEDIATELY
6. **Move to next comment**
```
