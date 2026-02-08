# Roadmap (directions, not promises)

## Phase 1: Validate end-to-end

- [ ] Test with a real PR on a real repository
- [x] Add input validation: `SKILLS_REPO` format is validated before invoking Claude
- [ ] Add error handling: skills repo unreachable, empty diff, Claude failure

**Exit criteria**: A PR on a test repository gets a meaningful AI review comment based on skills.

## Phase 2: Make it usable

Turn the prototype into something others can adopt without reading the source.

- [x] Convert from copy-paste workflow to a **reusable workflow** (via `workflow_call`)
- [ ] Publish on GitHub Marketplace
- [ ] Support configurable inputs: skills repo, enforcement mode, LLM model, skill filter
- [ ] Add a `setup` guide: "from zero to working AI review in 10 minutes"
- [ ] Create a minimal starter skills repo template (3-5 simple skills, not 18)
- [ ] Support private skills repositories (authentication)
- [ ] Add structured review output format (consistent markdown with verdict, findings, summary)

**Exit criteria**: Someone can add the Action to their repo in under 10 minutes and get a working review.

## Phase 3: Make it scalable

Handle the real-world constraints that appear at scale.

- [ ] **Smart skill selection**: don't load all skills for every review. Match skills to changed file types (Java skills for `.java` files, API skills for controller changes)
- [ ] **Token budget management**: estimate prompt size before sending, truncate or summarize diff if needed, warn when skills + diff exceed model limits
- [ ] **Skills caching**: don't clone the entire skills repo on every run. Cache with TTL or use GitHub API to fetch specific files
- [ ] **Multiple skills repos**: compose standards from base + project-specific skills (e.g., `org/base-skills` + `org/java-skills`)
- [ ] **Review result persistence**: store review results for trend analysis ("which skills trigger most violations?")
- [ ] **Support additional LLM providers**: not just Claude. The concept is provider-agnostic even if the generation side uses Claude Code

**Exit criteria**: Works reliably on a repository with 10+ active contributors and 20+ skills.

## Phase 4: Organizational value

Features that matter when multiple teams adopt this across an organization.

- [ ] **Skills inheritance**: project skills can extend organization skills (override, add, disable specific rules)
- [ ] **Skills testing**: a way to validate that skills produce expected review results on known code samples
- [ ] **Metrics dashboard**: which standards are violated most, trend over time, per-team compliance
- [ ] **Integration with existing tools**: feed SonarQube findings, ESLint results, or test coverage as additional context to the review
- [ ] **Diff-aware skill loading**: analyze the diff first to determine which skills are relevant, then load only those
- [ ] **Review history**: track how a PR's review results change across pushes

**Exit criteria**: An organization with 5+ teams and 20+ repositories uses this as their standards governance system.

## What's deliberately out of scope

- Building a general-purpose AI code review tool (CodeRabbit already exists)
- Supporting non-Git workflows
- Real-time / IDE integration (the generation side is handled by Claude Code)
- Auto-fixing code based on review results (the review should inform humans, not bypass them)
