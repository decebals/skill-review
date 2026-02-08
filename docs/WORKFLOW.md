# Skill Review - Workflow Documentation

## Purpose
This GitHub Actions workflow performs automated code review on pull requests based on a predefined set of skills. The skills define project or organizational standards, which can be language-specific or general.

The workflow:
- Fetches skills from a configurable skills repository
- Evaluates code changes against those skills
- Posts a single structured comment on the PR with the verdict and findings
- Optionally fails the CI job if changes are required

## Usage in a Project

1. **Workflow Placement**: Copy `skill-review.yml` to `.github/workflows/` in your repository.

2. **Required Secrets**:
   - `ANTHROPIC_API_KEY`: API key for Claude (Anthropic). Set this in your repository's Settings > Secrets.

3. **Environment Variables** (configured in the workflow file):
   - `SKILLS_REPO`: GitHub repository containing skill definitions. Default: `decebals/claude-code-java`.
   - `ENFORCE_BEHAVIOR`: `informative` (default, comment only) or `strict` (fail CI on CHANGES_REQUIRED).

4. **Trigger Events**:
   - Pull request events: opened, synchronize, reopened.

## How it works

The workflow has three stages:

**1. Validate inputs**

Before invoking Claude, the workflow validates that `SKILLS_REPO` matches the expected `owner/repo` format. If not, the job fails immediately with a clear error message.

**2. Run Claude review**

The workflow uses [`anthropics/claude-code-action@v1`](https://github.com/anthropics/claude-code-action) to run Claude with a structured prompt. Claude autonomously:

1. Clones the skills repository
2. Discovers all `SKILL.md` files using `find`
3. Reads each skill to understand the enforced standards
4. Gets the PR diff via `gh pr diff`
5. Evaluates the changes — only flagging violations of specific skills
6. Posts a structured review comment with verdict, summary, findings, and skills evaluated
7. Writes the verdict (`APPROVED` or `CHANGES_REQUIRED`) to `verdict.txt`

**3. Enforce verdict (strict mode only)**

A subsequent workflow step reads `verdict.txt` and fails the CI job if the verdict is `CHANGES_REQUIRED`.

## Review comment format

```
**Verdict**: APPROVED or CHANGES_REQUIRED

**Summary**: 1-2 sentences.

**Findings** (if any):
- **[skill-name]**: description of violation and suggestion

**Skills evaluated**: list of skill names checked
```

## Allowed tools

Claude's tools are restricted to only what the review needs:
- `Read` — read files
- `Write` — write verdict.txt
- `Bash(git clone:*)` — clone the skills repo
- `Bash(find:*)` — discover skill files
- `Bash(gh pr diff:*)` — get the PR diff
- `Bash(gh pr comment:*)` — post the review comment
