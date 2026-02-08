# Vision

## The insight

AI code review tools already exist. What doesn't exist is a system that ensures the same engineering standards are applied consistently across both code generation and code review.

Today:
- developers use AI to generate code with loosely defined prompts
- reviewers (human or AI) apply a different, often implicit, set of expectations
- standards live in wikis, onboarding docs, or tribal knowledge
- alignment between generation and review is accidental

This project explores the alternative: **explicit, versioned standards that govern both generation and review**.

## Semantic continuity

The core value is not "better code review." It is **continuity** across the development lifecycle.

| Stage | Traditional approach | This approach |
|-------|---------------------|---------------|
| Define standards | Wiki pages, informal docs | Skills in Git, versioned via PRs |
| Generate code | Ad-hoc prompts, copy-paste guidelines | Claude Code loads skills automatically |
| Review code | Separate tool, separate rules | Same skills enforced in CI |
| Evolve standards | Fragmented, hard to trace | Git history, auditable, one source |

When the same artifacts govern both generation and review, the feedback loop closes. Code generated with skills should pass review against those same skills. When it doesn't, the skill (or the code) needs to change — and that change is explicit and traceable.

## Skills as organizational artifacts

A key distinction: skills live in a **dedicated repository**, not embedded in each project.

This means:
- **One update affects all projects.** Change a standard once, every repo inherits it.
- **Standards are auditable.** Git blame tells you who changed a rule, when, and why.
- **Onboarding is explicit.** "Read the skills repo" replaces "ask someone what the conventions are."
- **Governance is possible.** You can require PR approval for standard changes.

This is conceptually similar to:
- shared ESLint configurations (`eslint-config-company`)
- organization-wide SonarQube quality profiles
- shared CI templates

...but applied to AI behavior control, not just static analysis.

## Positioning

This project is **not** an AI code review tool in the same category as CodeRabbit, Copilot, or PR-Agent. Those are review tools — they analyze code and suggest improvements.

This project is a **standards system** that happens to use AI as an enforcement mechanism. The review step is one output. The generation step is another. The skills are the product.

The relevant comparison is not "which tool gives better review comments" but rather "how do you ensure your AI tools behave according to your engineering standards."

## What this enables

For an individual developer:
- consistent AI behavior across sessions
- explicit rules instead of "the AI just knows"

For a team:
- shared standards that everyone (including AI) follows
- review feedback aligned with generation guidance

For an organization:
- governance over AI-assisted development
- auditable, versionable engineering standards
- scalable onboarding

## Limitations and open questions

- The concept currently depends on the Claude Code ecosystem for the generation side. The review side is more portable (any LLM can evaluate code against skills).
- Token limits constrain how many skills can be loaded simultaneously. Smart skill selection (matching skills to file types/context) is not yet implemented.
- The value proposition is strongest when teams actually use Claude Code for generation. Without the generation side, this becomes "just another configurable AI reviewer."
- It is unclear how well this scales beyond 20-30 skills without skill selection/filtering.
