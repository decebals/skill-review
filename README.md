# skill-review

> Skill-based code review using versioned engineering standards.

### Purpose

This project is not another AI code reviewer.

It is a reference implementation for a different idea:

> Engineering standards should be explicit, versioned, and enforced consistently - across AI assisted code generation and human-written code.

This repository demonstrates how a set of well-defined skills (engineering rules, standards, constraints) can be reused:
- during AI-driven code generation
- and during automated code review in CI

using the **same source of truth**.

---

### The problem this explores

AI tools are increasingly used to generate code.
At the same time, teams still rely on code review and CI to enforce standards.

These two worlds are usually disconnected:
- code is generated using loosely defined prompts
- code is reviewed using a different set of rules
- standards live in docs, wikis, or tribal knowledge
- consistency is accidental

This project explores a different approach.

---

### The core idea

> Same mental model. Same rules. Same artifacts. From generation to review.

Instead of treating standards as informal guidelines, we treat them as:
- skills stored in Git
- reviewed via pull requests
- versioned and auditable
- shared across projects

Those skills are then:
- consumed by Claude Code during code generation
- enforced by an AI-driven review step during pull requests

This closes the loop between how code is written and how code is validated.

---

### Skills as first-class engineering artifacts

Skills are simple Markdown files stored in a dedicated repository.

They may describe:
- language-specific standards (e.g. Java)
- architectural constraints
- API design rules
- performance or security guidelines
- readability and maintainability expectations

Because skills live in Git:
- they evolve via pull requests
- changes are reviewable and traceable
- one update can affect multiple repositories
- onboarding becomes explicit and repeatable

Conceptually, this is similar to:
- shared ESLint configurations
- organization-wide SonarQube profiles

…but applied to AI behavior, not just static analysis.

### Relationship with skills repositories

`skill-review` is the **mechanism** — it knows how to fetch skills and review code against them.

The skills themselves live in separate repositories. For example, [`claude-code-java`](https://github.com/decebals/claude-code-java) is a concrete set of Java engineering standards.

The analogy:

| Role | Example |
|------|---------|
| Engine | `eslint` / `skill-review` |
| Rules | `eslint-config-airbnb` / `claude-code-java` |

This separation means:
- the review mechanism is language-agnostic and reusable
- teams can define, version, and share their own skill sets
- one skill set can be used across multiple projects

---

### What this repository provides

This repository contains:

- a **GitHub Actions workflow** that:
  - fetches skills from a skills repository
  - combines them with the pull request diff
  - asks an LLM to review the changes against those skills
  - posts a structured review comment on the PR
  - can optionally fail CI when standards are violated

- documentation explaining:
  - the execution model
  - design choices and trade-offs 
  - known limitations

The implementation is intentionally **simple and transparent**.

---

### What this is NOT

This project is **not**:
- a production-ready GitHub Action
- a commercial AI code review tool
- a competitor to CodeRabbit, Copilot, or PR-Agent
- a fully hardened CI solution

It is a concept demonstrator and a learning artifact.

---

### Why this exists

This project exists to explore:
- how AI can be integrated into real engineering workflows
- how to constrain and control LLM behavior
- how to make standards explicit instead of implicit
- how to reason about AI as part of a system, not a replacement for judgment

This is not a commercial product and does not compete with existing AI review tools.
It operates at a different level: standards governance, not code analysis.

---

### How to use it

Add this workflow to your repository (`.github/workflows/skill-review.yml`):

```yaml
name: Skill Review
on:
  pull_request:
    types: [opened, synchronize, reopened]
permissions:
  contents: read
  pull-requests: write
jobs:
  skill-review:
    uses: decebals/skill-review/.github/workflows/skill-review.yml@main
    secrets:
      anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

That's it. Every PR will be reviewed against the skills in [`claude-code-java`](https://github.com/decebals/claude-code-java). You can point to your own skills repo and choose between `informative` (advisory) or `strict` (fail CI) mode.

See [docs/WORKFLOW.md](docs/WORKFLOW.md) for full configuration options and [docs/TECHNICAL_DETAILS.md](docs/TECHNICAL_DETAILS.md) for design decisions.

For a working example, see [`skill-review-sandbox`](https://github.com/decebals/skill-review-sandbox) — a minimal Java project with skill-review configured and a [real review comment](https://github.com/decebals/skill-review-sandbox/pull/1) posted by the workflow.

---

### Who this is for

This project may be interesting if you are:
- a senior engineer or tech lead
- experimenting with AI-assisted development
- interested in engineering standards at scale
- skeptical of AI hype, but curious about disciplined usage

---

### Status

This project is in early development.

The concept is validated. The implementation is being hardened.

See [docs/ROADMAP.md](docs/ROADMAP.md) for planned next steps.
