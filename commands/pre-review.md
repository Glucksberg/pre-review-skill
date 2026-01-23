| allowed-tools | description |
|---|---|
| Bash(git *), Read, Edit, Write, Glob, Grep, Task | Pre-review code and fix issues before publishing PR |

# Pre-Review: Analyze and Fix Code Before PR

Analyze the current branch changes compared to main/master and **fix issues directly** instead of just commenting.

## Instructions

Follow these steps precisely:

### Step 1: Eligibility Check (Haiku Agent)

Use a Haiku agent to check:
- Is there a branch with uncommitted or committed changes compared to main/master?
- Are there actual code changes to review (not just config/docs)?

If no meaningful changes exist, inform the user and stop.

### Step 2: Discover Project Guidelines (Haiku Agent)

Use a Haiku agent to find and read:
- Root CLAUDE.md file (if exists)
- Any CLAUDE.md files in directories with modified files
- .eslintrc, .prettierrc, tsconfig.json, or similar config files that define code standards
- Any CONTRIBUTING.md or code style guides

Return a summary of the coding guidelines found.

### Step 3: Get Change Summary (Haiku Agent)

Use a Haiku agent to:
- Run `git diff main...HEAD` (or master) to see all changes
- Return a structured summary of what files changed and the nature of changes

### Step 4: Parallel Deep Analysis (5 Sonnet Agents)

Launch 5 parallel Sonnet agents to independently analyze the changes. Each agent should return:
- List of specific issues found
- File path and line numbers
- Severity (critical, important, minor)
- Suggested fix

**Agent #1 - CLAUDE.md/Guidelines Compliance:**
Audit the changes to ensure they comply with project guidelines (CLAUDE.md, eslint rules, etc.). Check:
- Naming conventions
- Code patterns required by the project
- Forbidden patterns or anti-patterns
- Documentation requirements

**Agent #2 - Bug Scanner:**
Read the file changes and scan for obvious bugs:
- Logic errors
- Off-by-one errors
- Null/undefined handling issues
- Race conditions
- Resource leaks
- Error handling gaps
Focus on real bugs, not style issues.

**Agent #3 - Git History Context:**
Read git blame and history of modified code to identify:
- Regressions (re-introducing old bugs)
- Breaking changes to stable code
- Patterns that were intentionally changed before
- Context from previous commits that affects current changes

**Agent #4 - Security & Performance:**
Scan for:
- Security vulnerabilities (injection, XSS, auth issues, exposed secrets)
- Performance issues (N+1 queries, unnecessary loops, memory leaks)
- API misuse
- Unsafe operations

**Agent #5 - Code Quality & Tests:**
Check for:
- Missing or inadequate tests for new functionality
- Dead code introduced
- Duplicated logic that should be refactored
- Inconsistent error handling
- Comments that need updating due to code changes

### Step 5: Confidence Scoring (Haiku Agents)

For each issue found in Step 4, launch a parallel Haiku agent to score confidence (0-100):

- **0:** False positive, doesn't stand up to scrutiny, or pre-existing issue
- **25:** Might be real, but could be false positive. Stylistic issue not in guidelines.
- **50:** Real issue but minor/nitpick. Not very important relative to the PR.
- **75:** Verified real issue. Will impact functionality. Directly mentioned in guidelines.
- **100:** Definitely real, will happen frequently. Evidence directly confirms.

Filter out issues scoring below 70.

### Step 6: Fix Issues Directly

For each remaining issue (score >= 70), **fix it directly in the code**:

1. Read the file containing the issue
2. Apply the fix using Edit tool
3. Verify the fix doesn't break surrounding code

Group fixes by file to minimize edits.

### Step 7: Summary Report

After fixing, provide a summary to the user:

```
## Pre-Review Complete

### Issues Found and Fixed: X

1. **[file:line]** - Brief description of issue
   - Severity: critical/important/minor
   - Category: bug/security/performance/style/guidelines
   - Fix applied: Brief description of fix

2. ...

### Issues Skipped (Low Confidence): Y

1. **[file:line]** - Brief description (score: XX)
   - Reason for low confidence

### Files Modified: Z
- path/to/file1.js
- path/to/file2.ts

### Recommendations
- Any manual review suggestions
- Tests to run
- Areas needing human judgment
```

### Step 8: Offer Next Steps

Ask the user:
- "Run tests to verify fixes?"
- "Review the changes with `git diff`?"
- "Commit the fixes?"

## Important Guidelines

- **DO fix issues directly** - Don't just report, actually edit the code
- **DON'T break working code** - If unsure about a fix, skip it and report
- **DON'T fix pre-existing issues** - Only fix issues in the changed code
- **DON'T make style-only changes** unless explicitly required by guidelines
- **DO preserve code intent** - Fixes should maintain the original purpose
- **DO group related fixes** - Multiple issues in same file = single edit session

## False Positives to Avoid

- Pre-existing issues not introduced by current changes
- Issues a linter/typechecker would catch (assume CI runs these)
- Pedantic nitpicks a senior engineer wouldn't flag
- General quality issues not in project guidelines
- Intentional changes that look unusual but are correct
- Code outside the scope of current changes

## Notes

- Use `git diff main...HEAD` to see changes (or master if no main)
- Create a todo list to track progress through the agents
- If no issues found, congratulate the user on clean code
- Always show what was fixed so user can verify
