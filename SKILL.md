---
name: pr-reviewer
description: Comprehensive GitHub Pull Request code review skill. This skill should be used when users provide a GitHub PR URL and request a code review. Automatically fetches PR metadata, diff, comments, commits, and related issues using gh CLI. Creates organized review workspace, analyzes code against industry-standard criteria (functionality, security, testing, maintainability), and optionally adds inline comments to the PR. Trigger phrases include review this PR, code review, review pull request, check this PR, or when a GitHub PR URL is provided.
version: 1.0.0
category: code-review
triggers:
  - review pr
  - code review
  - review pull request
  - check pr
  - pr review
  - github.com/*/pull/*
author: Claude Code
license: MIT
tags:
  - github
  - code-review
  - pull-request
  - quality-assurance
---

# PR Reviewer Skill

Conduct comprehensive, professional code reviews for GitHub Pull Requests using industry-standard criteria and automated tooling.

## Purpose

This skill enables thorough, consistent code reviews by:

1. **Automating data collection** - Fetches all PR-related information (metadata, diff, comments, commits, issues)
2. **Organizing review workspace** - Creates structured directory with all artifacts
3. **Applying systematic criteria** - Reviews against comprehensive quality checklist
4. **Facilitating inline feedback** - Optionally adds comments directly to PR code
5. **Ensuring completeness** - Checks functionality, security, testing, maintainability, and more

## When to Use This Skill

Use this skill when:
- User provides a GitHub PR URL and requests a review
- User asks to "review this PR" or "code review"
- User wants to check PR quality before merging
- User needs systematic feedback on proposed changes
- User mentions GitHub PR review in any context

## Review Process Workflow

**IMPORTANT**: This skill uses a **two-stage approval process**. Nothing is posted to GitHub until you explicitly approve with `/send` or `/send-decline`.

### Overview

1. **Fetch PR data** - Collect all information
2. **Generate review files** - Create detailed, human, and inline comment files
3. **Review and edit** - Examine files, make changes as needed (use `/show`)
4. **Approve and post** - Use `/send` (approve) or `/send-decline` (request changes)

### Step 1: Fetch PR Data

Use the `fetch_pr_data.py` script to automatically collect all PR information:

```bash
python scripts/fetch_pr_data.py <pr_url> [--output-dir <dir>] [--no-clone]
```

**What it does:**
- Parses PR URL to extract owner, repo, and PR number
- Creates directory structure: `<output-dir>/PRs/<repo-name>/<PR-NUMBER>/`
- Fetches PR metadata (title, author, state, branches, labels, etc.)
- Downloads PR diff and commit history
- Retrieves all PR comments and reviews
- Extracts ticket references (JIRA, GitHub issues)
- Optionally clones source branch and generates git diff
- Saves all data to organized JSON and text files

**Example:**
```bash
python scripts/fetch_pr_data.py https://github.com/facebook/react/pull/28476

# Custom output directory
python scripts/fetch_pr_data.py https://github.com/owner/repo/pull/123 --output-dir /tmp/reviews

# Skip cloning (faster, but no git diff)
python scripts/fetch_pr_data.py https://github.com/owner/repo/pull/123 --no-clone
```

**Output structure:**
```
/tmp/PRs/<repo-name>/<PR-NUMBER>/
├── metadata.json           # PR metadata (title, author, branches, etc.)
├── diff.patch             # PR diff from gh CLI
├── git_diff.patch         # Git diff (if cloned)
├── comments.json          # Review comments on code
├── commits.json           # Commit history
├── related_issues.json    # Linked GitHub issues
├── ticket_numbers.json    # Extracted ticket references
├── SUMMARY.txt            # Human-readable summary
└── source/                # Cloned repository (if not --no-clone)
```

### Step 2: Analyze PR Data

After fetching, analyze the collected data against review criteria:

1. **Read SUMMARY.txt** - Get high-level overview
2. **Review metadata.json** - Understand PR context, labels, assignees
3. **Examine diff.patch** - Analyze code changes
4. **Check comments.json** - See existing feedback
5. **Review commits.json** - Assess commit quality and messages
6. **Check related_issues.json** - Understand linked tickets/issues
7. **Apply review criteria** - Evaluate against comprehensive checklist

Use the Read tool to examine files:
```
Read /tmp/PRs/<repo-name>/<PR-NUMBER>/SUMMARY.txt
Read /tmp/PRs/<repo-name>/<PR-NUMBER>/metadata.json
Read /tmp/PRs/<repo-name>/<PR-NUMBER>/diff.patch
```

### Step 3: Generate Review Files

**CRITICAL**: After analysis, use `generate_review_files.py` to create structured review documents:

```bash
python scripts/generate_review_files.py <pr_review_dir> --findings <findings_json> [--metadata <metadata_json>]
```

This creates three files in `pr_review_dir/pr/`:

1. **`pr/review.md`** - Detailed internal review
   - Complete analysis with all findings
   - Includes emojis for quick scanning (🔴 🟡 🟢 💡 ❓ ✅)
   - Code snippets with line numbers
   - Full context for your evaluation

2. **`pr/human.md`** - Clean review for posting
   - Short, professional format
   - NO emojis
   - NO em dashes (uses regular hyphens)
   - NO code line numbers
   - Concise and respectful

3. **`pr/inline.md`** - Proposed inline comments
   - List of specific code comments
   - Code snippets with line number headers
   - Commands to post each comment
   - Edit before using

**Also creates slash commands** in `.claude/commands/`:
- `/send` - Post human.md and approve PR
- `/send-decline` - Post human.md and request changes
- `/show` - Open review directory in VS Code

**Example findings JSON structure**:
```json
{
  "summary": "Overall assessment of the PR...",
  "metadata": {
    "repository": "owner/repo",
    "number": 123,
    "title": "PR title",
    "author": "username",
    "head_branch": "feature",
    "base_branch": "main"
  },
  "blockers": [
    {
      "category": "Security",
      "issue": "SQL injection vulnerability",
      "file": "src/db/queries.py",
      "line": 45,
      "details": "Using string concatenation for SQL query",
      "fix": "Use parameterized queries",
      "code_snippet": "result = db.execute('SELECT * FROM users WHERE id = ' + user_id)"
    }
  ],
  "important": [...],
  "nits": [...],
  "suggestions": ["Consider adding...", "Future enhancement..."],
  "questions": ["Is this intended to...", "Should we..."],
  "praise": ["Excellent test coverage", "Clear documentation"],
  "inline_comments": [
    {
      "file": "src/app.py",
      "line": 42,
      "comment": "Consider edge case handling for empty input",
      "code_snippet": "def process(data):\n    return data.strip()",
      "start_line": 41,
      "end_line": 43,
      "owner": "owner",
      "repo": "repo",
      "pr_number": 123
    }
  ]
}
```

### Step 4: Review and Edit Files

**Use `/show` to open the review directory in VS Code:**

```
/show
```

This opens the pr_review_dir where you can:
- Read `pr/review.md` - Detailed analysis
- Edit `pr/human.md` - Modify before posting
- Review `pr/inline.md` - Check proposed comments
- Adjust any content as needed

**NOTHING is posted until you explicitly approve in Step 5.**

### Step 5: Approve and Post

When ready to post the review:

**Option A: Approve the PR**
```
/send
```
- Posts `pr/human.md` as a comment
- Approves the PR
- Confirms action

**Option B: Request Changes**
```
/send-decline
```
- Posts `pr/human.md` as a comment
- Requests changes on the PR
- Confirms action

**Posting inline comments** (optional, after /send or /send-decline):
Review `pr/inline.md` and run the provided commands for specific code comments.

### Step 6: Apply Review Criteria (During Analysis)

Reference `references/review_criteria.md` for comprehensive checklist. Review against these categories:

**1. Functionality and Correctness**
- Does code solve the intended problem?
- Are there bugs or logical errors?
- Edge cases and error handling?
- Compatibility across environments?

**2. Readability and Maintainability**
- Code clarity and meaningful names?
- Functions/classes follow Single Responsibility?
- Code duplication avoided (DRY)?
- Future-proofing and extensibility?

**3. Style and Conventions**
- Follows project linter rules?
- Consistent with existing codebase?
- Appropriate comments and documentation?

**4. Performance and Efficiency**
- Algorithm efficiency (avoid O(n²) where possible)?
- Scalability under load?
- Optimizations balanced with readability?

**5. Security and Best Practices**
- Common vulnerabilities addressed (SQL injection, XSS, etc.)?
- Sensitive data protected?
- Secrets not hardcoded?
- Dependencies justified and secure?

**6. Testing and Quality Assurance**
- Tests exist for new code?
- Tests cover happy paths, errors, edge cases?
- Tests are meaningful, not just for coverage?
- CI/CD checks pass?

**7. Overall PR Quality**
- PR scope focused (single feature/fix)?
- Clean commit history with descriptive messages?
- Clear PR description with context?
- Impact on downstream code considered?

**For detailed criteria and examples, always reference:**
```
Read references/review_criteria.md
```

**Priority markers for findings:**

- 🔴 **Blocker**: Must be fixed before merge
- 🟡 **Important**: Should be addressed
- 🟢 **Nit**: Nice to have, optional
- 💡 **Suggestion**: Consider for future
- ❓ **Question**: Clarification needed
- ✅ **Praise**: Good work!

## Reference Documentation

This skill includes comprehensive reference guides:

### `references/review_criteria.md`
Complete checklist for code reviews covering:
- Functionality and correctness
- Readability and maintainability
- Style and conventions
- Performance and efficiency
- Security and best practices
- Testing and quality assurance
- Overall PR quality
- Language-specific considerations

**Always reference this for thorough reviews.**

### `references/gh_cli_guide.md`
Quick reference for GitHub CLI commands:
- Fetching PR data
- Accessing comments and reviews
- Getting commit information
- Working with branches
- Adding inline comments
- Understanding diff line numbers
- JQ patterns for filtering
- Error handling

**Use this when working directly with gh CLI.**

## Scripts Reference

### `scripts/fetch_pr_data.py`

Automated PR data fetching and organization.

**Features:**
- Parses GitHub PR URLs
- Fetches comprehensive PR data via gh CLI
- Extracts ticket references from title, body, commits
- Optionally clones repository
- Generates git diff between branches
- Saves organized data structure

**Usage:**
```bash
python scripts/fetch_pr_data.py <pr_url> [options]

Options:
  --output-dir DIR    Base output directory (default: /tmp)
  --no-clone         Skip cloning repository
```

**Output:**
Creates `/tmp/PRs/<repo>/<number>/` with all PR artifacts.

### `scripts/generate_review_files.py`

**NEW**: Generate structured review files from analysis findings.

**Features:**
- Creates three review files (review.md, human.md, inline.md)
- Generates contextual slash commands in pr_review_dir
- Formats detailed review with emojis and line numbers
- Formats human review without emojis or line numbers
- Lists inline comments with code snippets

**Usage:**
```bash
python scripts/generate_review_files.py <pr_review_dir> --findings <findings_json> [--metadata <metadata_json>]
```

**Creates:**
- `pr/review.md` - Detailed internal review
- `pr/human.md` - Clean review for posting (no emojis, em-dashes, line numbers)
- `pr/inline.md` - Proposed inline comments with commands
- `.claude/commands/send.md` - Slash command to approve and post
- `.claude/commands/send-decline.md` - Slash command to request changes
- `.claude/commands/show.md` - Slash command to open in VS Code
- `REVIEW_READY.txt` - Summary of next steps

**Note**: Slash commands are local to the pr_review_dir. Navigate to that directory to use them.

### `scripts/add_inline_comment.py`

Add inline code review comments to specific lines in PR.

**Features:**
- Posts inline comments via GitHub API
- Auto-fetches latest commit if requested
- Supports multi-line comments
- Handles both old (LEFT) and new (RIGHT) versions

**Usage:**
```bash
python scripts/add_inline_comment.py <owner> <repo> <pr_number> <commit_id> <file_path> <line> "<comment>" [options]

Options:
  --side RIGHT|LEFT       Side of diff (default: RIGHT)
  --start-line N         Starting line for multi-line comment
  --start-side RIGHT|LEFT Starting side for multi-line comment
```

**Example:**
```bash
python scripts/add_inline_comment.py owner repo 123 latest "src/app.py" 42 "Consider edge case handling here"
```

## Best Practices

### Communication
- **Be constructive**: Frame as suggestions, not criticism
- **Explain why**: Don't just say what's wrong, explain why it matters
- **Acknowledge good work**: Call out excellent practices
- **Prioritize**: Focus on blockers first, style issues last

### Review Efficiency
- **Use scripts**: Automate data fetching and comment posting
- **Reference criteria**: Use review_criteria.md as checklist
- **Focus review**: Critical issues > Important > Nice-to-have
- **Be timely**: Review promptly (within 24 hours if possible)

### Inline Comments
- **Be specific**: Reference exact lines and files
- **Provide examples**: Show better alternatives
- **Test first**: Try inline comments on test PRs
- **Use sparingly**: Too many inline comments can overwhelm

### PR Size Handling
- **Large PRs (>400 lines)**: Suggest splitting into smaller PRs
- **Review in chunks**: Break large reviews into logical sections
- **Focus on architecture**: For large changes, review design first

## Common Scenarios

### Scenario 1: Quick Review Request
```
User: "Can you review this PR? https://github.com/org/repo/pull/456"

Response:
1. Run fetch_pr_data.py to collect data
2. Read SUMMARY.txt and metadata.json
3. Scan diff.patch for obvious issues
4. Apply critical criteria (security, bugs, tests)
5. Create findings JSON with analysis
6. Run generate_review_files.py to create review files
7. Tell user to review pr/review.md and pr/human.md
8. Remind user to use /show to edit, then /send or /send-decline when ready
```

### Scenario 2: Thorough Review with Inline Comments
```
User: "Do a comprehensive review and add inline comments where needed"

Response:
1. Run fetch_pr_data.py with cloning
2. Read all collected files (metadata, diff, comments, commits)
3. Apply full review_criteria.md checklist
4. Identify critical issues, important issues, and nits
5. Create findings JSON with inline_comments array
6. Run generate_review_files.py to create all files
7. Tell user:
   - Review pr/review.md for detailed analysis
   - Edit pr/human.md if needed
   - Check pr/inline.md for proposed comments
   - Use /show to open in VS Code
   - Use /send or /send-decline when ready
   - Optionally post inline comments from pr/inline.md
```

### Scenario 3: Security-Focused Review
```
User: "Check this PR for security issues"

Response:
1. Fetch PR data
2. Focus on review_criteria.md Section 5 (Security)
3. Check for: SQL injection, XSS, CSRF, secrets, etc.
4. Examine dependencies in metadata
5. Review authentication/authorization changes
6. Report security findings with severity
```

### Scenario 4: Review with Related Tickets
```
User: "Review this PR and check if it addresses the linked JIRA ticket"

Response:
1. Fetch PR data (captures ticket references)
2. Read related_issues.json
3. Compare PR changes against ticket requirements
4. Verify all acceptance criteria met
5. Note any missing functionality
6. Suggest additional tests if needed
```

## Troubleshooting

### "gh CLI not found"
Install GitHub CLI: https://cli.github.com/
```bash
# macOS
brew install gh

# Linux
sudo apt install gh  # or yum, dnf, etc.

# Authenticate
gh auth login
```

### "Permission denied" errors
Check authentication:
```bash
gh auth status
gh auth refresh -s repo
```

### "Invalid PR URL"
Ensure URL format: `https://github.com/owner/repo/pull/NUMBER`

### "Line number in diff doesn't match"
Remember: inline comment line numbers are **relative to the diff**, not absolute file positions.
Use `gh pr diff <number>` to see diff line numbers.

### Rate limit errors
```bash
# Check rate limit
gh api /rate_limit

# Authenticated users get higher limits
gh auth login
```

## Quick Reference Commands

```bash
# Fetch PR data
python scripts/fetch_pr_data.py https://github.com/owner/repo/pull/123

# Add inline comment
python scripts/add_inline_comment.py owner repo 123 latest "src/app.py" 42 "Your comment"

# View PR in browser
gh pr view 123 --repo owner/repo --web

# Check PR status
gh pr checks 123 --repo owner/repo

# View existing comments
gh api /repos/owner/repo/pulls/123/comments --jq '.[] | {path, line, body}'
```

## Tips for Effective Reviews

1. **Start with context**: Read PR description, linked issues, commit messages
2. **Understand intent**: What problem is being solved?
3. **Check tests first**: Do tests demonstrate the fix/feature?
4. **Look for patterns**: Repeated issues suggest architecture problems
5. **Consider alternatives**: Could this be done more simply?
6. **Think about maintenance**: Will this be easy to modify later?
7. **Remember humans**: Be kind, respectful, and constructive

## Resources

- **Review Criteria**: `references/review_criteria.md`
- **gh CLI Guide**: `references/gh_cli_guide.md`
- **GitHub PR Review Docs**: https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-changes-in-pull-requests
- **Google Engineering Practices**: https://google.github.io/eng-practices/review/
- **OWASP Top 10**: https://owasp.org/www-project-top-ten/
