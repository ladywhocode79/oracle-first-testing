# Review AI-Suggested Oracle (Critically)

**Version:** 0.1  
**Stage:** Plan  
**Grounded in:** [oracle-executor-principle.md](../../concepts/oracle-executor-principle.md), [human-authored-oracle-guide.md](../../guides/human-authored-oracle-guide.md)

---

## Overview

After you've asked AI to suggest an oracle (using `/prompts/plan/extract-oracle-from-prd.md`), you get a checklist. **Don't just approve it.** This prompt helps you review it critically, spot AI biases, and either refine it or reject it.

**Why this matters:** Anchoring effect is real. If you just approve what AI suggests, you're not exercising your judgment—you're accepting AI's framing. This guide ensures you stay in control.

---

## Prerequisites

- You have a PRD (Product Requirements Document)
- AI has suggested an oracle (output from `/prompts/plan/extract-oracle-from-prd.md`)
- You have 15 minutes to review critically

---

## The Prompt

Use this prompt with any LLM after you've gotten AI's oracle suggestion:

```
I have a PRD and an AI-suggested oracle. Help me review it critically for:

1. Anchoring Bias
   - Are there items I would NOT have thought of on my own?
   - Are there obvious gaps (things I'd care about but AI didn't mention)?
   - Would I write this the same way, or is this AI's framing?

2. Specificity
   - Are items specific and testable?
   - Or vague ("should work," "handle errors")?
   - Flag any items that need clarification.

3. Risk Ranking
   - Did AI correctly identify high-risk vs. low-risk items?
   - Am I comfortable with the priority?
   - What would I rank differently?

4. Security & Privacy
   - Did AI mention sensitive data handling?
   - Did AI consider enumeration attacks, logging, timing attacks?
   - What did it miss (in your opinion)?

5. Completeness
   - What did AI miss entirely?
   - What would you add if AI hadn't suggested anything?
   - Any edge cases or error conditions?

PRD:
[paste your PRD here]

AI-Suggested Oracle:
[paste the oracle AI suggested here]

My concern/question:
[any specific things you want to double-check]

Answer format:
- List anchoring biases (where AI led you astray)
- List genuine gaps (things AI missed)
- List items to clarify or rewrite
- Suggest 3-5 items to add
- Flag anything you'd remove or change priority
```

---

## How to Use This in Practice

### Step 1: Get AI's Suggestion
Run `/prompts/plan/extract-oracle-from-prd.md` and get the oracle.

### Step 2: Read It (Without Editing)
Read the AI oracle once. Note your gut reactions:
- ✓ "Yeah, that's important"
- ? "Huh, I didn't think of that"
- ✗ "Nah, that's not a priority"

### Step 3: Run This Review Prompt
Give the AI your PRD, the suggested oracle, and any specific concerns.

### Step 4: Evaluate the Feedback
The AI will flag:
- **Anchoring biases** — Places where you might be accepting AI framing
- **Gaps** — Things you'd care about that AI didn't mention
- **Items to clarify** — Vague suggestions that need specificity
- **Additions** — Things to add
- **Removals** — Things to deprioritize

### Step 5: Write YOUR Oracle
Take the AI-suggested oracle + this feedback, and write YOUR version.

**Key:** The final oracle should be in YOUR words, reflecting YOUR priorities, not AI's.

---

## Example: Password Reset

**PRD:**
```
Users can reset forgotten passwords via email.
The reset link is valid for 24 hours.
After reset, the old password is deleted.
```

**AI-Suggested Oracle (excerpt):**
```
Happy Path:
- User enters email → system responds with "Check your email"
- User receives email within 30 seconds
- User clicks reset link → form opens
- User enters new password → password changes
- User can log in with new password

Security:
- Reset token is cryptographically random
- Token is not logged
```

**Review Feedback (from this prompt):**
```
Anchoring Biases:
- You might accept "30 seconds" SLA because AI mentioned it, but the PRD says nothing about SLA. Do YOU care? (You probably do, but decide for yourself)
- AI prioritized "cryptographically random" but didn't mention timing attacks or enumeration. This is a gap.

Gaps:
- What happens if user requests password reset twice in 1 minute? (Rate limiting?)
- What about old sessions? Should they be terminated?
- What if email service is down? (Graceful degradation?)
- Special characters in password (you mentioned concern about this)

Items to Rewrite:
- "Token is not logged" → Too vague. Should be: "Token is NOT in access logs, error logs, server logs, or browser history"

Items to Add:
- "User enumeration prevention: Same error message whether email exists or not"
- "Password strength: No minimum length requirement (from PRD)"

Priority Changes:
- AI put "Cryptographically random" as high. It's important, but you care MORE about "user doesn't get locked out" and "old sessions are killed"
```

**Your Revised Oracle (after this review):**
```
Happy Path:
- User enters email → system returns "Check your email" (no mention of which emails exist)
- User receives reset email within 2 minutes
- User clicks link → form opens (or 401 if expired)
- User enters password → password must match confirmation
- User clicks submit → password is changed
- User's old sessions are terminated (logged out everywhere)
- User can log in with new password (old password rejected)

Security & Privacy:
- Reset token: Cryptographically random, NOT in any logs, NOT in browser history
- Enumeration prevention: Same generic message whether email exists or not
- Password: No minimum length enforced (user decides)
- Old sessions: All terminated after reset (not just current session)

Edge Cases:
- Multiple reset requests in 1 minute: [You decide: allowed or rate-limited?]
- Password with special characters: Accepted as-is
- If email service is down: Queue email for retry, user gets confirmation message

Error Conditions:
- Invalid email format: "Check your email" (don't reveal format is wrong)
- Email not in system: "Check your email" (don't say it doesn't exist)
- Expired link: "Your link expired. Request a new reset link"
- Password mismatch: "Passwords don't match"
```

**What changed:**
- You removed AI's specific 30-second SLA (you didn't care)
- You added enumeration prevention (you cared, AI missed it)
- You clarified what "token not logged" really means
- You made edge cases explicit (your decision, not AI's)

**Result:** Oracle is now YOUR critical thinking + AI's helpful suggestions.

---

## Red Flags: When to Reject AI Oracle Entirely

If the AI oracle has these problems, **start over with human-authored oracle**:

- ❌ AI made assumptions not in the PRD (e.g., "should be real-time" when PRD says nothing about latency)
- ❌ AI missed obvious security requirements for your domain (e.g., PCI compliance for payment flows)
- ❌ AI's risk ranking is way off (e.g., treats nice-to-have as critical)
- ❌ AI defaulted to happy path only (no edge cases or errors)
- ❌ You read it and think "this doesn't match how we actually build features"
- ❌ You can't understand why AI included certain items

In these cases, **write your own oracle** using `/guides/human-authored-oracle-guide.md`.

---

## Questions to Ask Yourself

As you review the AI oracle, ask:

1. **Would I have written this?** For each section, ask: "Would I think of this on my own?" If yes ✓. If no, decide if it's valuable or anchoring bias.

2. **Am I just approving this?** If you're reading it and thinking "fine," you might be anchoring. Stop and actually evaluate.

3. **What would I add?** If you had written this oracle from scratch (without AI), what would you include that's missing?

4. **What would I remove?** Any items that feel less important than others? Any assumptions you don't agree with?

5. **What's my gut telling me?** Your instinct as a QA person is valuable. If something feels off, investigate it.

6. **Could I defend this to the product team?** If not, revise it until you can.

---

## Tradeoffs

| Approach | Time | Quality | Risk |
|----------|------|---------|------|
| **Accept AI oracle as-is** | 2 min | Low (anchoring bias) | High (miss gaps) |
| **Review + refine (this guide)** | 15 min | Medium (hybrid) | Medium (balanced) |
| **Human-authored (from scratch)** | 30 min | High (critical thinking) | Low (your judgment) |

**Recommendation:** 
- Use this guide (review + refine) for **routine features**
- Use human-authored for **security-critical, high-risk features**

---

## When to Use This Prompt

- ✅ You ran `/prompts/plan/extract-oracle-from-prd.md` and got a suggestion
- ✅ You want to verify AI didn't bias your thinking
- ✅ You want to keep AI's good suggestions but filter out bad ones
- ✅ You're short on time but don't want to sacrifice quality

---

## Next Steps

1. Use this prompt to review AI's oracle
2. Decide: Accept, refine, or rewrite
3. If refining: Combine AI's suggestions + your critical feedback → your oracle
4. If rewriting: Use `/guides/human-authored-oracle-guide.md` instead
5. Move to Phase 2: `/prompts/author/generate-tests-from-oracle.md`

---

## Related

- [Oracle-Executor Principle](/concepts/oracle-executor-principle.md) — Why human + AI matters
- [Human-Authored Oracle Guide](/guides/human-authored-oracle-guide.md) — How to write oracles from scratch
- [Extract Oracle from PRD](/prompts/plan/extract-oracle-from-prd.md) — AI-suggested oracle (reviewed via this prompt)

---

*Use AI to amplify your thinking, not replace it.*
