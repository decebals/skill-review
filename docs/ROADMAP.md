# Roadmap (directions, not promises)

## Done

- [x] Test with a real PR on a real repository — see [skill-review-sandbox PR #1](https://github.com/decebals/skill-review-sandbox/pull/1)
- [x] Add input validation: `SKILLS_REPO` format is validated before invoking Claude
- [x] Convert from copy-paste workflow to a **reusable workflow** (via `workflow_call`)
- [x] Support configurable inputs: skills repo and enforcement mode as `workflow_call` inputs
- [x] Structured review output format (verdict, summary, findings, skills evaluated)
- [x] Prompt injection defense: diff treated as untrusted data

## Next (when real usage demands it)

- [ ] **Support private skills repositories**: authenticated clone for organization-internal skills
- [ ] **Smart skill selection**: match skills to changed file types instead of loading all skills for every review
- [ ] **Token budget management**: estimate prompt size, truncate or summarize large diffs
- [ ] **Multiple skills repos**: compose standards from base + project-specific skills (e.g., `org/base-skills` + `org/java-skills`)

## Future (if the project grows)

- [ ] **Skills inheritance**: project skills can extend organization skills (override, add, disable)
- [ ] **Skills testing**: validate that skills produce expected review results on known code samples
- [ ] **Support additional LLM providers**: the concept is provider-agnostic even if the current implementation uses Claude

## What's deliberately out of scope

- Building a general-purpose AI code review tool (CodeRabbit already exists)
- Supporting non-Git workflows
- Real-time / IDE integration (the generation side is handled by Claude Code)
- Auto-fixing code based on review results (the review should inform humans, not bypass them)
