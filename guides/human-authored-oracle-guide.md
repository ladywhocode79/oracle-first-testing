# Human-Authored Oracle Guide

**Why This Matters:** Research shows humans paired with AI score 2x higher on defect detection when the human writes the test checklist from scratch. But if AI writes it and humans just approve it, the anchoring effect kicks in—humans stop thinking critically and accept AI's framing.

**This guide teaches you how to author oracles without AI bias.**

---

## What Is a Human-Authored Oracle?

An oracle is a **human-written checklist of what should happen** when a feature works correctly.

```
✅ Human-Authored Oracle (Good)
  - You read the PRD
  - You think about edge cases, risks, security
  - You write your OWN checklist (no AI input)
  - Result: Your critical thinking is preserved

❌ AI-Generated Oracle (Weak)
  - AI reads PRD and writes checklist
  - You approve it ("looks good")
  - Anchoring effect: You don't question AI's framing
  - Result: AI's biases become your tests
```

---

## Step-by-Step: How to Write Your Oracle

### Step 1: Read the PRD Carefully (10 minutes)
- Read it once, without taking notes
- Identify the main user goal
- Identify security/regulatory requirements
- Flag anything unclear or concerning

### Step 2: Brain Dump (5 minutes)
Without looking at the PRD again, write down:
- "What MUST happen for this to work?"
- "What could go wrong?"
- "What am I most worried about?"
- "What edge cases would embarrass us?"

Don't edit. Just dump your thoughts.

### Step 3: Organize Into Sections (10 minutes)

Take your brain dump and organize into these categories:

**Happy Path (Critical)**
What must work when everything goes right?

```
Example: Password Reset
- User enters email → system accepts it
- User receives reset email
- User clicks link → goes to reset form
- User enters new password → password changes
- User can log in with new password
```

**Edge Cases (High)**
What unusual but valid scenarios might break?

```
Example: Password Reset
- User requests reset twice in a row (rate limiting?)
- User enters very long password (255+ chars)
- User resets while already logged in
- User resets from mobile device
- User resets with special characters in password
```

**Error Conditions (High)**
What should happen when users mess up?

```
Example: Password Reset
- User enters non-existent email (don't say "email not found")
- User enters malformed email (missing @)
- User enters expired reset link
- User's token expires mid-process
- User closes browser and comes back later
```

**Security & Privacy (Critical)**
What must be protected?

```
Example: Password Reset
- Reset token is NOT in logs or error messages
- System doesn't reveal if email exists ("prevent enumeration")
- Old password is completely deleted
- Session is terminated after reset
- No timing attacks (same time whether email exists or not)
```

**Non-Functional (Medium)**
What about performance, scale, availability?

```
Example: Password Reset
- Reset email arrives within 30 seconds (SLA)
- Can handle 1000 concurrent reset requests
- Works on slow networks (< 100 Kbps)
- Graceful degradation if email service is down
```

### Step 4: Reality Check (5 minutes)

Ask yourself:
- ❓ Would I explain this to someone using the feature?
- ❓ Does this match what the product team described?
- ❓ Did I miss any obvious scenarios?
- ❓ Is there anything here I'm GUESSING about? (Flag it)

If you answer "no" to any question, revise.

### Step 5: (Optional) AI Sanity Check

Now you can use AI to:
- ✅ Check if you missed anything obvious
- ✅ Suggest edge cases you didn't think of
- ✅ Review for consistency

**But:** Only add items if you genuinely think "oh yeah, that should be tested." If you're just copying AI suggestions, stop.

---

## Real Example: Password Reset Oracle

**PRD (condensed):**
```
Users can reset forgotten passwords via email.
They receive a reset link valid for 24 hours.
After reset, old password no longer works.
```

**Human-Authored Oracle (what you should write):**

```markdown
# Password Reset Oracle

## Happy Path (Critical)
- User enters email → system returns "Check your email"
- User receives email within 2 minutes
- Email contains reset link with token
- User clicks link → goes to reset form (no 404)
- User enters password + confirm → both must match
- User submits → system returns "Password reset"
- User is logged out (old session killed)
- User logs in with new password

## Edge Cases (High)
- Multiple requests in 1 minute (should they work? Or rate-limit?)
- Password with 100+ characters
- Password with spaces, emojis, special chars
- User requests reset while already logged in (should they be logged out first?)
- User opens reset link on mobile (form should be responsive)
- User has two accounts with same email variant (test@example vs test+tag@example)

## Error Conditions (High)
- Email field empty → form shows "Email required"
- Email with no @ → form shows "Invalid email"
- Email not in system → "Check your email" (same as success, don't leak info)
- Reset token expired → "Link expired, request new one"
- Password fields don't match → "Passwords don't match"
- Very short password (< 8 chars) → "Password too short"

## Security & Privacy (Critical)
- Token is NOT printed in any log file
- Token is NOT in email headers or metadata
- System doesn't reveal whether email exists
- Old password doesn't work after reset (completely deleted)
- User's old sessions are terminated
- No database backups contain old passwords

## Non-Functional (Medium)
- Email arrives within 2 minutes (SLA)
- Works on 3G networks (slow)
- If email service is down, system gracefully fails (queue for retry)
```

**What makes this good:**
- ✅ Written from YOUR critical thinking, not AI
- ✅ Specific (actionable test cases, not vague)
- ✅ Security-focused (token handling, enumeration)
- ✅ Realistic (edge cases you'd actually encounter)
- ✅ Honest (you'd feel confident explaining this to the product team)

---

## Common Mistakes to Avoid

### ❌ Mistake 1: Copying AI Oracle Wholesale
```
BAD:
You: "Let me ask Claude to suggest an oracle"
Claude: [generates 50-item oracle]
You: "Looks good!" → Copy it as-is

PROBLEM: Anchoring effect. You didn't think critically.
You just approved AI's framing.

BETTER:
You: Write your own oracle first
Then: "Let me check if Claude thinks I missed anything"
Then: Evaluate Claude's suggestions critically
```

### ❌ Mistake 2: Writing Vague Checklists
```
BAD:
- Feature should work
- Errors should be handled
- Security should be good

PROBLEM: Not testable. What does "work" mean?

BETTER:
- User enters valid email → system accepts, sends email
- User enters non-existent email → system returns generic message
- User's password completely replaced, old one rejected
```

### ❌ Mistake 3: Forgetting Edge Cases
```
BAD:
You only test the happy path (user enters email, gets reset link, resets).

PROBLEM: You miss 90% of bugs (edge cases, errors, security).

BETTER:
Include: Edge cases (special chars, rate limits), Error conditions (expired link, 
network errors), Security (token leaks, enumeration), Performance (SLA, slow networks)
```

### ❌ Mistake 4: Guessing at Requirements
```
BAD:
You: "I think password reset should work on slow networks"
But the PRD says nothing about this.
You test something nobody asked for.

PROBLEM: Wasted effort. Not clear on priorities.

BETTER:
Only test what's explicitly required.
Flag uncertainties and ask the product team.
```

---

## Checklist: Is Your Oracle Ready?

Before you generate tests from your oracle, check:

- [ ] **Completeness** — Would I feel confident showing this to the product team?
- [ ] **Specificity** — Could a QA engineer write tests from this without asking questions?
- [ ] **Security** — Did I explicitly call out sensitive data handling?
- [ ] **Edge Cases** — Did I cover unusual but valid scenarios?
- [ ] **Errors** — Did I cover failure modes?
- [ ] **Performance** — Did I call out any SLAs or scaling requirements?
- [ ] **No Guessing** — Did I write only what I actually understand from the PRD?
- [ ] **No AI Bias** — Did I think this through myself, or did I just copy AI suggestions?

If you check all these boxes, **your oracle is ready.**

---

## When to Use Human-Authored Oracle

**Always use human-authored oracle for:**
- ✅ Security-critical features (authentication, payment, data access)
- ✅ Regulatory requirements (compliance, audit trails)
- ✅ Customer-facing flows (authentication, checkout, profile)
- ✅ High-risk changes (database schema, API contracts)

**Can use AI-suggested oracle for:**
- 🟡 Internal tools (admin panels, dashboards)
- 🟡 Low-risk changes (copy text, styling)
- 🟡 Well-defined features (standard CRUD)

**But:** Even for low-risk features, if you have time, human-authored is better.

---

## Next Steps

1. **Write your oracle** using this guide (30 minutes)
2. **Review it** against the checklist above
3. **Move to Phase 2:** Use `/prompts/author/generate-tests-from-oracle.md` to generate tests
4. **Run tests:** Use `/prompts/execute/run-tests-and-report.md`

---

## FAQ

**Q: Do I need to write oracles this detailed?**
A: For security-critical or high-risk features, yes. For simple features, you can be more concise. But the checklist above is the quality bar.

**Q: Can I use AI to help write my oracle?**
A: Yes, but read the warning in Step 5. Only add AI suggestions if you genuinely think they're valuable. Don't just copy.

**Q: How long does it take?**
A: 30 minutes for a complex feature (password reset). 5-10 minutes for a simple feature (change profile name).

**Q: What if I don't understand the requirement?**
A: Flag it and ask the product team. Don't guess. An uncertain oracle is worse than no oracle.

**Q: Should I write oracles for every feature?**
A: For security-critical and high-risk features, yes. For routine features, you can move faster with AI-suggested oracles (with critical review).

---

## Related

- [Oracle-Executor Principle](/concepts/oracle-executor-principle.md) — Why this matters
- [Critically Review AI-Suggested Oracles](/prompts/plan/review-ai-suggested-oracle.md) — When you do use AI
- [Extract Oracle from PRD](/prompts/plan/extract-oracle-from-prd.md) — AI-suggested oracle prompt (use carefully)

---

*Your critical thinking is the most valuable part of testing. Protect it by authoring your own oracle.*
