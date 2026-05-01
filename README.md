# three-brain

Claude Code skill that auto-routes work to Codex (GPT) or Gemini when Claude alone isn't the best tool.

## Install

```bash
claude plugin install three-brain@three-brain
```

## What it does

- **Codex review**: when you ask Claude to check its own work (never self-review)
- **Codex adversarial**: "tear this apart / stress test / find what's wrong"
- **Codex rescue**: Claude failed same operation 2+ times in a row
- **Forced Codex adversarial**: active edit on risky paths (auth, billing, migrations, secrets...)
- **Gemini multimodal**: video/audio/YouTube in message
- **Gemini whole-codebase**: "scan the whole repo / find every place X"
- **All three parallel**: "ask all three / cross-architecture consensus"
