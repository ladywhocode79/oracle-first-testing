# Development Workflow

How to work on this repo with local-first development and published releases.

---

## Branch Strategy

| Branch | Purpose | Status | Deploy |
|--------|---------|--------|--------|
| **main** | Published, stable version | ✅ Production | GitHub (public) |
| **develop** | Active development, QA feedback | 🔄 WIP | Local testing |

---

## Workflow: From Feedback to Published

### 1. Start with Develop Branch (Local Iteration)

```bash
# Switch to develop (active development)
git checkout develop

# Verify you're on develop
git branch
# Output: * develop
#         main
```

### 2. Make Changes & Test Locally

```bash
# Make changes to documentation, prompts, skills
# For example: update guides, add new prompts, etc.

# Sync to ~/.claude/skills/ for testing in Claude Code
./sync.sh to-global

# Test the changes locally in Claude Code or with sample PRDs
# [test, iterate, refine]

# Sync changes back to repo
./sync.sh to-repo

# Commit changes
git add .
git commit -m "Work in progress: [what you changed and why]"
```

### 3. Gather QA Community Feedback (Stay on Develop)

- Share changes with QA community
- Collect feedback
- Iterate on develop branch
- Make refinements

```bash
# More changes based on feedback
git add .
git commit -m "Refine based on feedback: [changes]"
```

### 4. Ready to Publish? Merge to Main

```bash
# Switch to main
git checkout main

# Merge develop into main
git merge develop

# Push to GitHub (this is what users see)
git push origin main

# Switch back to develop for next iteration
git checkout develop
```

### 5. Continue Development on Develop

```bash
# Pull latest main back into develop
git pull origin develop  # Usually automatic, but good to verify

# Continue iterating for next release
```

---

## Quick Reference Commands

```bash
# Check current branch
git branch

# Switch branches
git checkout main      # Switch to main (published)
git checkout develop   # Switch to develop (active work)

# Sync skills locally
./sync.sh to-global   # Copy from repo to ~/.claude/skills/
./sync.sh to-repo     # Copy from ~/.claude/skills/ back to repo

# Commit and push
git add .
git commit -m "your message"
git push origin develop  # Push to develop branch
git push origin main     # Push to main branch (publish)

# Merge when ready
git checkout main
git merge develop
git push origin main
```

---

## Example: QA Feedback Cycle

**Day 1: QA Community gives feedback**
```
"The anchoring effect section is great! Can you add more examples?"
```

**Day 1-2: Local iteration on develop**
```bash
git checkout develop
# Edit /concepts/oracle-executor-principle.md (add examples)
./sync.sh to-global
# Test in Claude Code, refine
./sync.sh to-repo
git add .
git commit -m "Add examples to anchoring effect section"
git push origin develop
```

**Day 2: Share updated develop with QA**
```
"Thanks for the feedback! Check out the develop branch: 
 https://github.com/ladywhocode79/oracle-first-testing/tree/develop"
```

**Day 3: More feedback, more iteration**
```bash
# Repeat: make changes, test, iterate, push to develop
```

**Day 5: Ready to publish**
```bash
git checkout main
git merge develop
git push origin main  # Now it's public
```

---

## Protecting Main

**Main branch should only receive:**
- Merges from develop (never direct commits)
- Tested, reviewed changes
- Ready-to-publish quality

**This means:**
✅ Always work on `develop`  
✅ Test locally before merging to `main`  
✅ Push to `main` only when confident  
❌ Never directly commit to `main`

---

## Syncing Skills Locally

Use `sync.sh` to keep the repo and your installed skills in sync:

```bash
# After making changes to skill files:
./sync.sh to-global   # Copy latest from repo to ~/.claude/skills/

# After testing changes in Claude Code:
./sync.sh to-repo     # Copy back to repo

# Then:
git add .
git commit -m "your message"
```

---

## FAQ

**Q: Can I commit directly to main?**  
A: No. Always use develop → main workflow. This keeps history clean.

**Q: What if I make a mistake on develop?**  
A: No problem! Develop is for iteration. Reset and try again:
```bash
git reset --soft HEAD~1  # Undo last commit, keep changes
git reset --hard HEAD~1  # Undo and discard last commit
```

**Q: How do I update develop after publishing?**  
A: Usually automatic, but to be safe:
```bash
git checkout develop
git pull origin develop
```

**Q: Can others contribute?**  
A: Yes! They can fork, make changes, and submit PRs to develop.

**Q: When do I tag releases?**  
A: After merging to main:
```bash
git tag v1.0.0
git push origin v1.0.0
```

---

## Status

- **Main branch:** Published and stable ✅
- **Develop branch:** Active development 🔄
- **Sync script:** Ready to use (`./sync.sh`)
- **Workflow:** Established

Ready to iterate! 🚀

---

*Last updated: 2026-06-10*
