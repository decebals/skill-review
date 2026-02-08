# Skill Review - Workflow Documentation

## Purpose
This GitHub Actions workflow performs automated code review on pull requests based on a predefined set of skills. The skills define project or organizational standards, which can be language-specific or general.

The workflow:
- Fetches skills from a configurable skills repository
- Evaluates code changes against those skills
- Posts a single structured comment on the PR with the verdict and findings
- Optionally fails the CI job if changes are required

## Usage as a reusable workflow

Add a thin caller workflow to your repository (`.github/workflows/skill-review.yml`):

```yaml
name: Skill Review
on:
  pull_request:
    types: [opened, synchronize, reopened]
jobs:
  skill-review:
    uses: decebals/skill-review/.github/workflows/skill-review.yml@main
    secrets:
      anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

To customize:

```yaml
    uses: decebals/skill-review/.github/workflows/skill-review.yml@main
    with:
      skills_repo: 'your-org/your-skills'
      enforce_behavior: 'strict'
    secrets:
      anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

### Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `skills_repo` | `decebals/claude-code-java` | Skills repository (`owner/repo`) |
| `enforce_behavior` | `informative` | `informative` (comment only) or `strict` (fail CI) |

### Required secrets

- `ANTHROPIC_API_KEY`: API key for Claude (Anthropic). Set in Settings > Secrets.

### Permissions

The workflow needs write access to pull requests (to post review comments). Make sure your repository has "Read and write permissions" enabled under Settings > Actions > General > Workflow permissions.

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
