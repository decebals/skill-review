# Technical Details

## How the workflow executes

```
PR opened/updated
    |
    v
Validate SKILLS_REPO format (fail early if invalid)
    |
    v
claude-code-action runs Claude with prompt
    |
    v
Claude clones skills repo --> find SKILL.md files --> Read each skill
    |
    v
Claude runs `gh pr diff` to get code changes
    |
    v
Evaluate changes against loaded skills
    |
    v
Post structured PR comment via `gh pr comment`
    |
    v
Write verdict to verdict.txt
    |
    v
(strict mode only) Next step reads verdict.txt --> fail CI if CHANGES_REQUIRED
```

## Implementation

The workflow uses [`anthropics/claude-code-action@v1`](https://github.com/anthropics/claude-code-action), the official Anthropic GitHub Action. This eliminates the need for manual CLI installation, prompt file construction, or output parsing.

Claude is given a structured prompt with 7 steps:
1. Clone the skills repository
2. Discover all `SKILL.md` files
3. Read each skill to understand the standards
4. Get the PR diff via `gh pr diff`
5. Evaluate changes against skills (flag only skill violations, distinguish severity)
6. Post a single structured review comment
7. Write the verdict (`APPROVED` or `CHANGES_REQUIRED`) to `verdict.txt`

The allowed tools are restricted to: `Read`, `Write`, `Bash(git clone:*)`, `Bash(find:*)`, `Bash(gh pr diff:*)`, `Bash(gh pr comment:*)`.

## Design decisions

**Why `claude-code-action` instead of direct API calls?**

`claude-code-action` handles Claude CLI installation, execution, and permissions. It lets Claude use tools (Read, Write, Bash) within the action, which means Claude can autonomously clone repos, discover files, read diffs, and post comments — all within a single action step.

**Why a skills repository instead of inline config?**

Skills live in a separate Git repository so they can be shared across projects and versioned independently. A change to a standard is a PR in the skills repo, not a change in every project.

**Why Markdown for skills?**

Markdown is human-readable, Git-friendly, and directly consumable by LLMs. No parsing layer needed. The same file that a developer reads for onboarding is the file the AI reads for enforcement.

**Why a single PR comment instead of inline annotations?**

Inline annotations require mapping LLM output to specific lines, which is fragile and error-prone. A single structured comment is simpler, more reliable, and easier to parse for CI decisions.

**Why `verdict.txt` for CI decisions?**

The strict mode step needs to know the review result. Having Claude write the verdict to a file is simple and reliable — the next workflow step reads the file and decides whether to fail. This avoids parsing free-form LLM output with shell commands.

**Why optional CI failure?**

Two modes serve different needs:
- `informative` (default): the review is advisory. Teams adopting the tool can see what it flags without blocking their workflow.
- `strict`: the review is a gate. Standards violations block the PR. This is for teams that trust their skills and want hard enforcement.

## Known limitations

**Skill loading is naive**: The workflow discovers all skill files and Claude reads them all. There is no filtering by relevance (e.g., loading Java skills for a Python PR). This wastes tokens and may exceed model context limits for large skill sets.

**No token budget management**: Large diffs combined with many skills can exceed model context limits. There is no estimation or truncation logic.

**Public repos only**: The skills repo is cloned via `git clone https://github.com/...` without authentication. Private skills repos are not supported.

**Not yet tested end-to-end**: The workflow has been designed and implemented but not yet validated against a real PR with real skills.

## Security considerations

- `SKILLS_REPO` is validated against an `owner/repo` pattern before the workflow reaches Claude. The value is interpolated into the prompt (not directly into a shell command), and Claude executes the clone.
- The PR diff is passed as part of the prompt. Malicious code in a PR could attempt prompt injection. The prompt includes instructions to only evaluate against skills and not follow meta-instructions in the diff.
- `ANTHROPIC_API_KEY` is passed as a secret and should never appear in logs.
