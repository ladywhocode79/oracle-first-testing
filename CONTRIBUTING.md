# Contributing to create-test-automation-skills

Thank you for contributing! This guide explains how to add to the repo.

---

## Before You Contribute

1. **Understand the principle** — Read [/concepts/oracle-executor-principle.md](concepts/oracle-executor-principle.md)
2. **Understand the architecture** — Read [NORTH-STAR.md](NORTH-STAR.md)
3. **Follow the pairing model** — We work anchor (Claude) + shadow (you)

---

## What Can You Contribute?

### Tier 1: Low Friction (Start Here)

- ✅ **Bug fixes** (typos, broken links, etc.)
- ✅ **Glossary entries** (new terms, clear definitions)
- ✅ **Examples** (your own password reset example for a different language)
- ✅ **Clarifications** (confusing explanation → clearer version)

**Process:**
1. Fork repo
2. Edit file (one clear change)
3. Submit PR with explanation
4. We review + merge (usually within 1 day)

### Tier 2: Medium Effort (Discuss First)

- ✅ **New prompt** (new lifecycle or domain)
- ✅ **New track** (new testing platform: Cypress, Protractor, etc.)
- ✅ **New pattern** (new flaky test fix, new MCP tool)
- ✅ **New guide** (decision tree, walkthrough for your domain)

**Process:**
1. Open an issue: "I want to add [X]"
2. Discuss scope, approach, examples
3. We agree on structure
4. You implement
5. We review + merge

### Tier 3: High Effort (Long-term Partnership)

- ✅ **Refactor phases** (reorganize sections, restructure)
- ✅ **New skill** (build a Claude Code skill)
- ✅ **Major initiative** (NFR framework, security patterns, etc.)

**Process:**
1. Open an issue: "I want to build [X]"
2. Schedule sync with maintainers (async discussion OK too)
3. Plan together (what's the scope? dependencies? timeline?)
4. You implement in phases (small PRs, frequent feedback)
5. We merge pieces incrementally

---

## How to Contribute: Specific Types

### Adding a New Prompt

1. **Decide which lifecycle stage:** plan, author, execute, heal, or analyze?
2. **Create file:** `/prompts/[stage]/[name].md`
3. **Follow template:**

```markdown
# [Prompt Title]

**Version:** 0.1  
**Stage:** [Plan/Author/Execute/Heal/Analyze]  
**Grounded in:** [link to concept]

## Overview
[One paragraph: what does this prompt do?]

## Prerequisites
[What input does it need?]

## The Prompt
[The actual prompt text here]

## Example Input
[Sample input]

## Example Output
[Sample output]

## Tradeoffs
[Cost, accuracy, effort, latency table]

## When to Use / When NOT to Use
[Guidance on applicability]

## Links
[Cross-references to concepts, other prompts, examples]
```

4. **Add to /prompts/[stage]/README.md** — link to your new prompt
5. **Submit PR** with explanation of why this prompt was needed

### Adding a New Track

1. **Create directory:** `/tracks/[track-name]/`
2. **Create files:**
   - `README.md` (overview)
   - `oracle-adaptation.md` (domain-specific oracle claims)
   - `examples/password-reset/` (worked example)
3. **Follow Appium/Contract patterns** (see existing tracks)
4. **Submit PR**

### Adding a Glossary Entry

1. **Edit:** `/guides/glossary.md`
2. **Add entry:**

```markdown
## [Term]

**Definition:** One sentence, clear and concise.

**Why it matters:** One paragraph explaining why this term is important.

**See also:** [Related terms], [relevant docs]

**Example:**
[Brief example of how the term is used]
```

### Fixing a Bug / Clarifying Docs

1. **Edit the file** — be clear about what changed
2. **Submit PR** with "Fixes #[issue]" in description
3. **Be specific** — "Typo" is OK; "Confusing explanation" needs detail

---

## Code Style & Standards

### Markdown

- **Headers:** Use `#`, `##`, `###` (not underlines)
- **Code blocks:** Use triple backticks with language
- **Links:** Use relative paths `[text](path/to/file.md)`
- **Emphasis:** Use `**bold**` for important, `_italic_` for de-emphasis
- **Lists:** Use `-` for unordered, numbers for ordered

### Python Examples

- Use type hints: `def func(x: str) -> bool:`
- Follow PEP 8
- Include docstrings: `"""One-line summary."""`
- Real example > abstract example

### Terminology

- Use "oracle" (human-authored checklist)
- Use "executor" (AI running tests)
- Use "MCP" (not "tool" or "extension")
- Use "flaky" (not "intermittent" or "unreliable")
- Use "track" (not "framework" or "automation type")

---

## PR Checklist

Before submitting a PR:

- [ ] I've read [oracle-executor-principle.md](concepts/oracle-executor-principle.md)
- [ ] My change follows the existing structure (see `/prompts/`, `/guides/`, etc.)
- [ ] I've tested my example (if applicable)
- [ ] I've checked for broken links
- [ ] I've added cross-references (linked to related docs)
- [ ] My PR description explains WHY this change was needed (not just WHAT)
- [ ] I've kept changes focused (one idea per PR)

---

## PR Description Template

```markdown
## What

[One sentence: what does this PR add/fix?]

## Why

[Why is this needed? What problem does it solve?]
Research/evidence/example that motivated this change.

## How

[What changed? Key files modified, new files added.]

## Testing (if applicable)

[Did you test this? How?]
[For prompts: did you run it against Claude?]
[For examples: did you run the test code?]

## Closes

[Closes #123] or [Relates to #456]
```

---

## Review Process

1. **You submit PR**
2. **We review:** Check structure, accuracy, clarity
3. **Discussion:** We might ask for clarifications or changes
4. **Revision:** You update PR if needed
5. **Approval:** We approve and merge
6. **Credit:** Your contribution is acknowledged

---

## Question: Can I Add a New Phase?

Phases 0-7 are planned (see [NORTH-STAR.md](NORTH-STAR.md)). We're not planning Phase 8+ yet.

**Instead:**
- Contribute a **new track** (domain-specific testing)
- Contribute a **new pattern** (flaky fix, healing approach)
- Contribute a **new guide** (decision tree, glossary, learning path)

---

## Question: Can I Fork and Build My Own?

Absolutely! This repo is MIT licensed. You can:
- Fork it and customize for your team
- Use prompts/patterns in your own project
- Build skills/tools on top of this foundation
- Share improvements back if you'd like

Just credit the original repo. 🙏

---

## Question: How Do I Get Help?

- **Confused about structure?** → Read [NORTH-STAR.md](NORTH-STAR.md)
- **Don't understand oracle-executor?** → Read [/concepts/oracle-executor-principle.md](concepts/oracle-executor-principle.md)
- **Not sure if your idea fits?** → Open an issue and ask
- **Found a bug?** → File an issue with details

---

## Maintenance & Release Cadence

- **Phases:** New phase every 2-4 weeks (as implemented)
- **Prompts:** New prompts added as needed (usually monthly)
- **Patterns:** Self-healing patterns added when discovered
- **Docs:** Clarifications + examples ongoing

No strict schedule—we move as fast as quality allows.

---

## Code of Conduct

- Be respectful
- Assume good intent
- Cite sources (research, examples)
- Acknowledge mistakes and learn
- Welcome all experience levels (beginner → expert)

---

## Contact

- **Issues:** GitHub issues (preferred)
- **Discussions:** GitHub discussions
- **Email:** [maintainer email if applicable]
- **Slack:** [if applicable]

---

## Thank You

Contributing makes this better for everyone. We appreciate your time and ideas! 🙏

---

*Pairing model: Anchor (Claude) proposes, Shadow (you) refines and merges.*

