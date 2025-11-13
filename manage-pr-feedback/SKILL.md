---
name: manage-pr-feedback
description: Manages feedback on your PRs by reading, replying to, and resolving inline comments from reviewers (Copilot, humans). Use when addressing PR review feedback as the PR author, not when reviewing others' PRs.
---

# Manage PR Feedback Skill

## Overview

This skill helps you systematically address feedback on your PRs from reviewers (Copilot, human reviewers).

**Key distinction**: This is for when YOU are the PR **author** responding to feedback, not when you're reviewing someone else's PR.

**Recurring failures this addresses**:
- Losing track of which feedback has been addressed
- Not replying to reviewer questions/comments
- Forgetting to resolve threads after fixing issues
- Missing feedback scattered across multiple files

**Why this matters**: Proper feedback management helps reviewers see progress, prevents merge delays, and maintains good collaboration.

---

## Core Principles

### Feedback Handling Mindset
- **Acknowledge quickly**: Reply to show you've seen feedback
- **Fix first, resolve second**: Address issue, push fix, then resolve
- **Ask questions**: Clarify unclear feedback before implementing
- **Track systematically**: Use TodoWrite to track what's addressed

### Communication Style
- **Concise**: Brief confirmation or question
- **Specific**: Reference commits/changes when replying
- **Respectful**: Thank reviewers, even for critical feedback
- **Action-oriented**: State what you'll do or did

---

## Workflow

### 1. Read All Feedback
- Get PR number and list all review comments
- Separate inline comments from general review feedback
- Identify blocking vs non-blocking issues
- Use TodoWrite to track items to address

### 2. Prioritize Feedback
**Order**:
1. Blocking issues (REQUEST_CHANGES review state)
2. Security/critical concerns
3. Suggestions and improvements
4. Questions/clarifications

### 3. Address Each Item

**For code issues**:
- Make the fix
- Commit and push changes
- Reply to comment: "Fixed in [commit sha]"
- Resolve thread after confirming fix

**For questions/clarifications**:
- Reply with explanation
- Ask for clarification if needed
- Don't resolve until questioner confirms

**For suggestions (optional)**:
- Reply whether you'll implement or not
- Explain reasoning if declining
- Resolve after decision is clear

### 4. Reply to Comments
- Reply to show you've seen and understood feedback
- Be specific about what you changed or will change
- Reference commits when fixed
- Tag reviewer if you need clarification (@username)

### 5. Resolve Threads
**Only resolve threads when**:
- ✅ Issue is fixed AND pushed
- ✅ Question is answered to reviewer's satisfaction
- ✅ Suggestion is implemented OR declined with explanation

**Don't resolve if**:
- ❌ You haven't fixed the issue yet
- ❌ Reviewer hasn't confirmed understanding
- ❌ Discussion is ongoing

### 6. Request Re-Review
- After addressing all feedback, request re-review
- Mention key changes in comment
- Tag reviewers if needed

---

## API Quick Reference

### List All Inline Comments on Your PR
```bash
# Get all inline comments
gh api repos/{owner}/{repo}/pulls/$PR/comments --jq '.[] | {
  id: .id,
  author: .user.login,
  path: .path,
  line: .line,
  body: .body,
  created: .created_at
}'

# Get unresolved threads (GraphQL)
gh api graphql -f query='
query {
  repository(owner: "{owner}", name: "{repo}") {
    pullRequest(number: '$PR') {
      reviewThreads(first: 100) {
        nodes {
          id
          isResolved
          comments(first: 10) {
            nodes {
              author { login }
              body
            }
          }
        }
      }
    }
  }
}' | jq '.data.repository.pullRequest.reviewThreads.nodes[] | select(.isResolved == false)'
```

### Reply to Inline Comment
```bash
# Reply to specific comment
gh api --method POST \
  repos/{owner}/{repo}/pulls/$PR/comments/$COMMENT_ID/replies \
  --field body="Fixed in commit abc1234. Now validates input before processing."
```

### Resolve Thread (After Fix)
```bash
# Resolve thread
gh api graphql -f query='
mutation($threadId: ID!) {
  resolveReviewThread(input: {threadId: $threadId}) {
    thread { isResolved }
  }
}' -f threadId="THREAD_ID"
```

### Request Re-Review
```bash
# Request review from specific users
gh api --method POST \
  repos/{owner}/{repo}/pulls/$PR/requested_reviewers \
  --field reviewers[]="username1" \
  --field reviewers[]="username2"

# Or comment to notify
gh pr comment $PR -b "@reviewer Ready for re-review. Addressed all feedback:
- Fixed validation issue (commit abc1234)
- Added error handling (commit def5678)
- Clarified naming per suggestion"
```

---

## Common Mistakes

### ❌ Auto-Resolving Without Fixing
**Problem**: Resolving threads before actually addressing the issue
**Fix**: Fix first, push commit, then resolve with commit reference

### ❌ Not Replying to Comments
**Problem**: Fixing issues silently, reviewers don't know status
**Fix**: Always reply to acknowledge and reference fix commit

### ❌ Defensive Responses
**Problem**: Getting defensive about critical feedback
**Fix**: Thank reviewer, implement fix or explain reasoning calmly

### ❌ Resolving Ongoing Discussions
**Problem**: Resolving threads while discussion continues
**Fix**: Wait for mutual understanding/agreement before resolving

### ❌ Ignoring Non-Blocking Feedback
**Problem**: Only addressing blocking issues
**Fix**: Address suggestions too, or explain why you're not implementing

### ❌ Not Tracking Progress
**Problem**: Losing track of which items are addressed
**Fix**: Use TodoWrite to systematically track feedback items

### ❌ Vague Replies
**Problem**: "Fixed" without specifics
**Fix**: "Fixed in commit abc1234 by adding validation on line 42"

---

## Example Workflow

### Scenario: Copilot left 5 inline comments on your PR

**Step 1: List and track**
```bash
# Get comments
gh api repos/owner/repo/pulls/123/comments

# Create todos
TodoWrite:
1. Fix SQL injection in db.py:89
2. Add error handling in process.py:174
3. Clarify variable name per suggestion
4. Answer question about caching strategy
5. Implement suggested optimization
```

**Step 2: Address blocking issues first**
```bash
# Fix SQL injection
# Commit: "fix: use parameterized query to prevent SQL injection"
git commit -m "fix: use parameterized query to prevent SQL injection"
git push

# Reply to comment
gh api --method POST repos/owner/repo/pulls/123/comments/456/replies \
  --field body="Fixed in commit abc1234. Now using parameterized queries."
```

**Step 3: Resolve after fixing**
```bash
# Get thread ID for that comment
gh api graphql -f query='...' # (from API reference above)

# Resolve thread
gh api graphql -f query='mutation($threadId: ID!) { ... }' \
  -f threadId="PRRT_xyz"
```

**Step 4: Continue for each item**
- Address
- Reply
- Resolve

**Step 5: Request re-review**
```bash
gh pr comment 123 -b "@copilot Ready for re-review. All feedback addressed:
- ✅ Fixed SQL injection (commit abc1234)
- ✅ Added error handling (commit def5678)
- ✅ Clarified variable naming (commit ghi9012)
- ✅ Implemented caching optimization (commit jkl3456)"
```

---

## Reply Templates

### For Fixes
```markdown
Fixed in commit [sha]. Now [specific change made].
```

### For Questions
```markdown
Good question! [Answer]. Does this address your concern?
```

### For Declined Suggestions
```markdown
Thanks for the suggestion! I'm not implementing this because [specific reason].
Open to discussion if you feel strongly about it.
```

### For Clarification Needed
```markdown
Could you clarify what you mean by [specific part]?
I want to make sure I understand before implementing.
```

### For Acknowledgment
```markdown
Great catch! Fixing now.
```

---

## Red Flags (Fail Fast)

- ❌ Resolving threads before fixing issues
- ❌ Not replying to reviewer questions
- ❌ Defensive or dismissive tone in replies
- ❌ Ignoring blocking feedback
- ❌ Resolving threads while discussion ongoing
- ❌ Vague replies without commit references
