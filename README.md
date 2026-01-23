# Pre-Review Skill

A Claude Code skill that performs comprehensive code review using 6 specialized agents and **automatically fixes issues** before you publish your PR.

## Why This Exists

The official `code-review` plugin from Claude Code marketplace posts comments on GitHub PRs **after** publication. This creates an inefficient workflow:

```
PR published → review → issues found → comments posted → you fix → update PR
```

This skill inverts the workflow:

```
Code ready → /pre-review → issues found AND fixed → PR published clean
```

## Features

- **6 Parallel Agents**: 5 Sonnet agents for deep analysis + Haiku agents for coordination
- **Auto-Fix**: Issues are fixed directly in your code, not just reported
- **Confidence Scoring**: 0-100 scale filters out false positives (threshold: 70)
- **Guidelines Aware**: Reads CLAUDE.md, eslint, prettier configs
- **Git Context**: Analyzes git history to understand code evolution

## Agent Specializations

| Agent | Focus |
|-------|-------|
| #1 | CLAUDE.md & project guidelines compliance |
| #2 | Bug scanning (logic errors, null handling, etc.) |
| #3 | Git history context (regressions, breaking changes) |
| #4 | Security & performance issues |
| #5 | Code quality & test coverage |
| Haiku | Coordination, eligibility checks, confidence scoring |

## Installation

### Option 1: Install from Local Path

```bash
claude /plugin install /path/to/pre-review-skill
```

### Option 2: Add to Your Plugin Directory

Copy this folder to your Claude Code plugins directory:
- macOS/Linux: `~/.claude/plugins/pre-review-skill/`
- Windows: `%USERPROFILE%\.claude\plugins\pre-review-skill\`

### Option 3: Register as Local Marketplace

```bash
claude /plugin marketplace add /path/to/PROJETOS/pre-review-skill
```

## Usage

```bash
# In any git repository with changes
/pre-review
```

The skill will:
1. Check for changes compared to main/master
2. Discover project guidelines
3. Launch 5 parallel analysis agents
4. Score each issue for confidence
5. **Fix issues directly in your code**
6. Provide a summary report
7. Offer next steps (run tests, review diff, commit)

## Output Example

```
## Pre-Review Complete

### Issues Found and Fixed: 3

1. **src/api/handler.ts:45** - Missing null check before accessing user.email
   - Severity: critical
   - Category: bug
   - Fix applied: Added optional chaining operator

2. **src/utils/parser.js:112** - SQL injection vulnerability in query builder
   - Severity: critical
   - Category: security
   - Fix applied: Used parameterized query

3. **src/components/List.tsx:78** - useEffect missing dependency
   - Severity: important
   - Category: bug
   - Fix applied: Added 'items' to dependency array

### Files Modified: 3
- src/api/handler.ts
- src/utils/parser.js
- src/components/List.tsx

### Recommendations
- Run `npm test` to verify fixes
- Review changes with `git diff`
```

## Configuration

The skill respects your project's existing configuration:
- `CLAUDE.md` - Project-specific Claude guidelines
- `.eslintrc.*` - ESLint rules
- `.prettierrc` - Prettier configuration
- `tsconfig.json` - TypeScript settings
- `CONTRIBUTING.md` - Contribution guidelines

## Comparison with code-review Plugin

| Aspect | code-review (plugin) | pre-review (this skill) |
|--------|---------------------|------------------------|
| When to run | After PR published | Before PR published |
| Output | GitHub comment | Direct code fixes |
| Workflow | Report issues | Fix issues |
| False positive handling | Threshold 80 | Threshold 70 |
| Use case | Team review bot | Solo dev workflow |

## License

MIT
