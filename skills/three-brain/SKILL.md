---
name: three-brain
description: |
  Auto-routes work to Codex (GPT-5.5) or Gemini 2.5 Pro when Claude alone is not the best tool.

  FIRE when:
  - User asks to review/check/verify/audit/sanity-check/double-check ANY work Claude just produced → Codex review. Hard rule: never self-review Claude output.
  - "tear apart / stress test / find what is wrong / break this" → Codex adversarial
  - Claude failed same operation 2+ times, or user says "stuck / try GPT / hand it off" → Codex rescue
  - Active edit on risky paths (src/auth/**, src/billing/**, migrations/**, deploy/**, .env*, secrets/**, infra/**, Stripe/Plaid/jwt/oauth) → forced Codex adversarial
  - Video, audio, or YouTube URL in message → Gemini multimodal
  - "scan the whole repo / find every place X / map the architecture" → Gemini whole-codebase
  - "ask all three / consensus / before I commit" → all three in parallel

  DO NOT FIRE for: explain/write/edit/plan on non-risky paths, casual chat, user reviewing their own content.
  Ambiguous on Claude output: FIRE IT.
---

# Three-Brain Auto-Router

A single skill that routes work invisibly. Three brains, one terminal:

- **Claude** = builder, driver, IDE harness — stays in front of the user the whole time
- **Codex (GPT-5.5)** = reviewer, second brain, rescue
- **Gemini 2.5 Pro** = eyes, ears, long-context — handles anything Claude literally cannot see or hear

## NO-SELF-REVIEW LAW (HARD RULE)

When the user asks Claude to "check / review / look over / proof / verify / audit / sanity-check / second-opinion" ANY work Claude just produced — code, writing, plan, design, edit, anything — MUST route to Codex. Same architecture = same blind spots.

Phrases that MUST trigger Codex review:
- "check over your work" / "check your work" / "is this right?" / "is the code right?"
- "review what you just did" / "review your code" / "look over this"
- "second opinion" / "sanity check" / "double-check" / "proof this"
- "audit this" / "verify this" / "make sure this works"

Do NOT silently self-review. Route to Codex:
```bash
git diff | codex exec --skip-git-repo-check "Review this. Find bugs, risks, missing tests."
```
Or for un-tracked code, pipe the file content directly.

After Codex returns, integrate findings into your reply. State at the end: "(Routed via three-brain → Codex review.)"

## Startup self-check (run once per session, before first route)

```bash
codex --version 2>&1 | head -1   # expect: codex-cli 0.125+
gemini --version 2>&1 | head -1  # expect: 0.39+
```

If `codex` is missing → announce once: "Codex CLI not found — review/rescue routes off until installed." Continue without those routes.
If `gemini` is missing → announce once the same way.

## Announcement protocol (REQUIRED for forced routes)

When the skill fires WITHOUT the user explicitly asking — risk-path detection or failure-counter rescue — announce in one line BEFORE running:

```
[three-brain] routing to Codex (adversarial-review) — risk path: src/auth/
[three-brain] handing off to Codex rescue — Claude failed same test 2× in a row
```

For routes the user explicitly asked for: no announcement needed.

## Failure-detection rule (HARD)

Deterministic counter, not a vibe:
- **2× same test failure on same code path** → MUST invoke Codex rescue
- **2× same error on same shell command** → MUST invoke Codex rescue
- **2× same edit re-tried with no progress** → MUST invoke Codex rescue

Reset counter when: (a) test/build passes, (b) user changes the goal, (c) user says "keep trying."

```bash
cat <context-bundle> | codex exec --skip-git-repo-check "rescue: [task]. Claude has tried 2x and failed. Full context attached."
```

## Gemini preprocessing contract

**Video pipeline:**
```bash
yt-dlp -f "best[ext=mp4][height<=720]" "<url>" -o /tmp/three-brain/in.mp4
ffmpeg -t 120 -i /tmp/three-brain/in.mp4 /tmp/three-brain/clip.mp4 -y
ffmpeg -i /tmp/three-brain/clip.mp4 -vn -ac 1 -ar 16000 /tmp/three-brain/audio.wav -y
gemini -p "Analyze frame-by-frame. Return findings as timestamped list: [MM:SS] event. Cap at 800 words." @/tmp/three-brain/clip.mp4
```

**PDF pipeline:**
```bash
qpdf --pages input.pdf 1-100 -- /tmp/three-brain/doc.pdf 2>/dev/null || cp input.pdf /tmp/three-brain/doc.pdf
gemini -p "Extract: key claims, data tables, chart findings, page-numbered. Cap at 1000 words." @/tmp/three-brain/doc.pdf
```

**Whole-codebase scan:**
```bash
/cc-gemini-plugin:gemini --dirs <comma-separated-paths> "Find every place X. Return file:line list."
```

Always demand timestamps, page numbers, or file:line citations in output.

## Risk-path detection (path-based, not keyword-based)

Codex adversarial review forced when an active edit touches:
```
src/auth/**          src/billing/**       **/migrations/**
**/deploy/**         **/.env*             **/secrets/**
**/policy/**         infra/**             **/Stripe*  **/Plaid*
**/jwt*              **/oauth*
```

Keywords alone in casual chat do NOT trigger — must be an active file edit.

## Risk-by-reversibility rule

Verb is irrelevant if the target is risky:
- "Refactor the auth middleware" → fires Codex review
- "Plan the Stripe migration" → fires Codex review
- "Design the deployment pipeline" → fires Codex review

## Parallel consensus mode

Only when explicitly invoked: "ask all three / before I commit to this / cross-architecture consensus."

Each model returns:
```
Recommendation: <one line>
Blocking risks: <bullet list>
Assumptions: <bullet list>
Confidence: low / medium / high
Tests required to verify: <bullet list>
```

Claude diffs them by evidence, not by averaging.

## Output filing

```
./three-brain-out/<YYYY-MM-DD>-<slug>/
  ├── input.txt
  ├── gemini-analysis.md
  ├── claude-build.html (or .md, .py, etc.)
  ├── codex-review.md
  └── log.md
```

Append to `./three-brain-out/log.md` every run:
```
[2026-04-27 16:24] route=video-build target=alex-clip duration=187s files=4
```

## Calling pattern

```bash
# Codex review
git diff | codex exec --skip-git-repo-check "Review this diff. Flag bugs, risks, missing tests. Be specific."

# Codex adversarial
git diff | codex exec --skip-git-repo-check "Adversarial review. Challenge the design. Find what's wrong. Prove it's broken."

# Codex rescue
cat <bundle> | codex exec --skip-git-repo-check "Rescue mode. Claude tried 2x and failed. Solve it from scratch."

# Gemini multimodal
gemini -p "<focused ask with output format>" @./file

# Gemini whole-codebase
/cc-gemini-plugin:gemini --dirs <paths> "<focused question>"
```

## Stay-asleep rules

Do not fire on:
- Casual conversation, greetings, status checks
- Explain/what-is/how-does questions Claude can answer directly
- "Recall / what did we decide / look up" → recall skill
- "Save this / wrap up / log this" → wrap-up skill
- User reviewing their own content (email, notes, docs they wrote)
- Any task already in another skill's territory

When uncertain: **stay asleep**. Under-firing is fine. Over-firing breaks trust.
