<div align="center">

![Claude Code](https://img.shields.io/badge/Claude%20Code-D97757?style=for-the-badge&logo=claude&logoColor=white)
![Gemini](https://img.shields.io/badge/Gemini-4285F4?style=for-the-badge&logo=google&logoColor=white)
![License MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)

# Don't Ask Me

**Stop being the bottleneck. Your AI now cross-checks itself with other LLMs.**

*Two AIs from different labs argue. You get one decision, not a wall of options.*

</div>

---

## What if Claude double-checked itself before bothering you?

You ask Claude to write your pricing page. Before it answers, Claude quietly asks **Gemini** ("is this right?") and **Opus running in isolated mode** ("what could go wrong?"). Then it shows you where the two reviewers agreed, where they disagreed, and which corrections you actually need to make.

That's the whole skill.

You stop being the validation step. The work shows up already reviewed.

---

## Why this exists

You're the one catching the mistakes Claude makes. Wrong stats. Outdated pricing. Off-brand wording. Logic that sounds confident but doesn't survive ten seconds of scrutiny.

You catch them because **a single AI doesn't know what it doesn't know**. Same training data, same blind spots. You're the only checker in the loop.

That's fine if you have all day to review. It's not fine if you're shipping in 20 minutes and one hallucinated number ends up in front of a customer.

This skill puts a second AI in the loop *before* you. Claude asks Gemini (different lab, different training, different blind spots). For high-stakes calls, it also asks Opus running with no memory of your conversation — a fresh adversarial perspective. You review what they *disagreed* on, not what Claude wrote alone.

---

## Three ways to use it

You don't pick the mode — Claude picks it based on what you type. Here's what each looks like.

> **Not in scope:** factual lookups ("what's the latest X version?", "is Y still free?") — use Claude Code's native WebSearch tool for those. This skill is about critique and adversarial review, not fact retrieval.

### 1. "Give me a second opinion" → Devil's Advocate

You type something like:
> *"Sanity-check this campaign brief."*
> *"Am I missing something on this launch plan?"*
> *"Stress-test this pricing model."*
> *"Дай второе мнение по этому решению."*

**What happens:** Claude sends your work to one independent reviewer — either Gemini (different model family, different blind spots) or an isolated Opus subagent (same family as Claude but with NO memory of your conversation, so it can't be primed by anything you discussed earlier).

**You get:** A focused critique: top concerns, hidden assumptions, what's missing. Claude evaluates the critique and shows you which points apply, which don't, and why.

**Cost:** ~$0.01-0.05 per call.

---

### 2. "This is a big decision — run a full review" → Boardroom Debate

This is the headline. The one people install this for.

You type something like:
> *"Run a full review on this launch copy."*
> *"This is important — dual-check it before I send."*
> *"Прогони dual-validation на этом ADR."*

**What happens:** Claude sends your work to BOTH reviewers at the same time (parallel — they don't see each other's responses, so neither can be biased by the other). One is Gemini, one is isolated Opus. Then Claude reads both critiques and builds a table:

| Concern | Gemini says | Opus says | Claude's take | What to do |
|---|---|---|---|---|

Three patterns to watch for:
- **Both reviewers raise the same concern** → very high signal, almost always worth addressing
- **One raises it, the other doesn't** → medium signal, evaluate context
- **They disagree** → the *most valuable* finding — different reasoning families surfaced different things

**You get:** A single ledger showing what both reviewers caught, where they agree, where they disagree, and Claude's recommendation on which corrections to ship. You make the final call.

**Cost:** ~$0.05-0.08 per call. Worth it when shipping a decision you can't easily reverse.

**Real story:** In one architecture-decision session (ADR sprint, May 2026), we ran this 6 times on what we thought were "easy" decisions. **Every single one of the 6 came back with a load-bearing problem we'd missed.** A misread library setting that would have caused a 2-hour stall in production. A retention policy that would have violated GDPR. A race condition in a payment-counting flow. None of these were caught by a single reviewer — only the disagreements between the two reviewers surfaced them. Total time cost: 4% of the session. Total bugs avoided: at least 6, including ones that would have been measured in customer refunds or compliance fines.

---

### 3. "I'm torn between options — help me converge" → Round-Table Discussion

You type something like:
> *"I have three angles for this campaign — help me pick one."*
> *"Brainstorm options for the homepage hero."*
> *"Обсуди варианты архитектуры со мной."*

**What happens:** Claude runs a structured 3-round debate with Gemini. Round 1: diverge — generate 7+ ideas, challenge your initial framing. Round 2: kill weak ideas, keep the 2-3 survivors. **Mandatory pause for your input** before final round. Round 3: pick ONE winner with concrete next steps and a go/no-go signal.

**You get:** One action, not a list of "options to consider." Plus the full kill list so you see why the losers lost.

**Cost:** ~$0.25 per full brainstorm. 5-10 minutes.

---

## Real stories from using this

These are not hypothetical. They happened.

### The landing page nobody could fix (Visual Design)

We were rebuilding a marketing page against a reference site. First pass: spacing looked off, typography felt inconsistent. We screenshotted both pages, sent them to Gemini with the prompt *"compare these two designs and score each axis 1-10."* Gemini came back with specific CSS fixes per section.

Applied the fixes. Second pass: typography 8/10, spacing 8/10. Found one overcorrection on the H1. Fixed. Done.

Without this loop: probably 6-8 iterations of "looks better? no? try again" with the user manually pointing at things. With this loop: 2 iterations, 30 minutes total.

### The stuck problem at 2am (Stuck Bug)

We were debugging a streaming bug. Three local fixes — none worked. The temptation was to try a fourth.

Instead, we packaged the bug and the three failed fixes, sent it to Gemini. 30 seconds later: *"You're using one timer for both phases. The first phase is eating the budget. Split into two independent timers — soft cutoff for phase one preserving what's done, hard cutoff for phase two."*

That was the architectural reframe. We'd never have found it iterating on the original design. Worth more than 30 seconds.

### The compliance disaster we caught (High-Stakes Review)

We were shipping a product feature for financial advisors. Our copy looked compliant on first read. We ran a Boardroom Debate (both reviewers, parallel).

Gemini caught language that triggered a US strict-liability rule. Opus caught a separate omission that would have failed a FINRA disclosure requirement. Neither caught the other's concern. Both were real.

Single-reviewer pass: ship and find out from a complaint. Dual-reviewer pass: caught at draft time, fixed in 10 minutes.

---

## What does this cost?

Honest numbers, per call:

| Style | Cost | When it's worth it |
|---|---|---|
| Devil's Advocate | ~$0.01-0.05 | Non-trivial decisions, stuck problems |
| Boardroom Debate | ~$0.05-0.08 | High-stakes — anything you can't easily walk back |
| Round-Table Discussion | ~$0.25 | When 2+ paths look viable and you need to converge |

A rough rule: **the cost of validating is almost always less than the cost of being wrong × the chance of being wrong.** Boardroom Debate is the same price as a coffee and catches things that would cost you a customer.

If you trigger validation accidentally on something small, no big deal. The total monthly cost stays in single dollars unless you're running validations on hundreds of decisions per day.

---

## Install in 60 seconds

**The easy way (recommended):**
```
/plugin marketplace add awrshift/skill-dont-ask-me
```
That handles everything.

**The manual way:**
```bash
mkdir -p ~/.claude/skills/dont-ask-me/{scripts,references,agents}
curl -sL https://raw.githubusercontent.com/awrshift/skill-dont-ask-me/main/SKILL.md \
  -o ~/.claude/skills/dont-ask-me/SKILL.md
curl -sL https://raw.githubusercontent.com/awrshift/skill-dont-ask-me/main/scripts/gemini.py \
  -o ~/.claude/skills/dont-ask-me/scripts/gemini.py
curl -sL https://raw.githubusercontent.com/awrshift/skill-dont-ask-me/main/agents/idea-validator.md \
  -o ~/.claude/agents/idea-validator.md
```

## Setup (2 minutes, one-time)

1. **Get a Gemini API key** — go to [aistudio.google.com](https://aistudio.google.com), click "Create API Key", copy the key
2. **Add it to your `.env`:** add the line `GOOGLE_API_KEY=your_key_here`
3. **Install the Python library:** run `pip install google-genai`
4. **Verify the Boardroom Debate dependency** — Boardroom Debate (the headline pattern) needs the `idea-validator` subagent installed at `~/.claude/agents/idea-validator.md`. Plugin marketplace install handles this automatically, but verify with:
   ```bash
   ls ~/.claude/agents/idea-validator.md
   ```
   If missing, run the curl command from the "manual way" section above.

Gemini has a free tier that's enough for most personal use. No credit card needed to start.

---

## Why two model families matter (90 seconds)

If both reviewers come from the same lab, they share training data. They share weaknesses. They get tripped up by the same kinds of edge cases. You get two voices saying the same thing.

**Gemini (Google)** and **Claude (Anthropic)** are trained differently. Where Claude gets confidently wrong about something, Gemini often gets it right — and vice versa. Their disagreements are where the real value lives.

The Opus subagent is from the same family as Claude, but it has **no memory of your conversation**. It can't be primed by your framing. It comes in cold, reads only the artifact, and gives you an adversarial review without the inherited blind spots of the parent conversation.

**Two model families + one isolated context = three independent perspectives** instead of three voices repeating the same answer.

There's hard evidence behind this. In a controlled experiment we ran, the same Gemini model with a basic prompt scored 50% on a classification task. The same model with a prompt that Gemini *itself* had stress-tested for loopholes scored 75-80%. The bottleneck wasn't the model — it was the prompt. A different model family saw the prompt's blind spots that Claude couldn't see from inside.

---

<details>
<summary><strong>For developers — CLI reference, internals, migration</strong></summary>

### CLI

All commands work from any shell once `GOOGLE_API_KEY` is set:

```bash
GEMINI="$(find . ~/.claude/skills -name gemini.py -path '*/dont-ask-me/*' 2>/dev/null | head -1)"

# Quick fact check (Flash + grounded)
python3 $GEMINI ask "What's the latest Next.js version?" --grounded

# Single critique (Flash 3.5 with adversarial system prompt)
python3 $GEMINI second-opinion @prompt.txt --save out.md

# Deep reasoning (Flash 3.5 with high thinking)
python3 $GEMINI think "Design a caching strategy for 10M users"

# Visual comparison (multimodal)
python3 $GEMINI second-opinion @prompt.txt --image ours.png --image ref.png --save review.md

# Structured extraction (JSON mode)
python3 $GEMINI extract "Parse this invoice..." --json-mode

# Parallel batch
python3 $GEMINI second-opinion @aspect1.txt --save r1.md &
python3 $GEMINI second-opinion @aspect2.txt --save r2.md &
python3 $GEMINI second-opinion @aspect3.txt --save r3.md &
wait
```

### Default models

| Command | Model | Why |
|---|---|---|
| `ask` | `gemini-3.5-flash` | Cheap + fast for facts, with optional `--grounded` web search |
| `second-opinion` | `gemini-3.5-flash` | Frontier agentic + coding model with adversarial reviewer system prompt |
| `think` | `gemini-3.5-flash` (high thinking) | Deep reasoning at Flash price point |
| `extract` | `gemini-3.5-flash` (JSON mode) | Cheap structured extraction |

Legacy `gemini-3.1-pro-preview` available via `-m` flag for pure-math reasoning (ARC-AGI-style) where Flash 3.5 falls short. Post-audit May 2026, Flash 3.5 beat Pro by 5 findings, 22% faster, 33% cheaper, 0 factual errors — Pro retained only as fallback.

### Boardroom Debate — how it actually runs

Claude builds a self-contained prompt artifact, then in ONE message sends:

```python
# Tool call 1 (Bash)
python3 ~/.claude/skills/dont-ask-me/scripts/gemini.py second-opinion @prompt.md --save gemini-out.md

# Tool call 2 (Task with idea-validator subagent, SAME message)
Agent(subagent_type="idea-validator", prompt="<artifact pasted inline>")
```

The "same message" part is non-negotiable. If sequential, the second reviewer sees the first's output in tool result → independence destroyed. Claude reads both outputs, builds the acceptance ledger, evaluates critically, presents to user.

### Anti-patterns (do NOT do these)

1. **Sequential parallel calls** — defeats the entire point
2. **Using `@file:` references in Gemini prompts** — Gemini Flash silently hallucinates when file refs fail to load. Always paste content INLINE. Always verify identifiers in response match input (codified Session #199 after 14 KB plan returned plausible review with audit IDs that didn't exist in source)
3. **Asking Gemini BEFORE forming your own opinion** — Gemini becomes a seed instead of a validator
4. **Boardroom Debate for trivial things** — burns tokens for stylistic preferences
5. **Forwarding reviewer output as-is** — same critical evaluation rule applies. Output is INPUT for decisions, not the decision

### Architecture

```
Don't Ask Me (skill)
├── SKILL.md                 — read by Claude on invocation
├── scripts/
│   └── gemini.py            — CLI wrapper for Google Gemini API
├── agents/
│   └── idea-validator.md    — isolated Opus subagent (copy to ~/.claude/agents/)
└── references/
    ├── round-table.md       — full 3-round brainstorm protocol
    ├── dual-validation.md   — parallel call + ledger format
    ├── decision-framework.md — which reviewer to pick when
    ├── critical-evaluation.md — why reviewer output ≠ decision
    ├── models.md            — Gemini model specs + pricing
    └── parameters.md        — all CLI flags
```

### Migrating from `awrshift/skill-gemini` or `awrshift/skill-brainstorm`

This skill **replaces** both. If you had either installed:

1. Uninstall old: `/plugin marketplace remove awrshift/skill-gemini` (and/or `awrshift/skill-brainstorm`)
2. Install new: `/plugin marketplace add awrshift/skill-dont-ask-me`
3. All previous CLI commands work unchanged — `gemini.py ask`, `second-opinion`, `think`, `extract` all behave identically
4. New: Boardroom Debate (dual validation), Round-Table Discussion (was the brainstorm skill), Style Router, idea-validator subagent

`awrshift/skill-gemini` was renamed to `awrshift/skill-dont-ask-me` (GitHub redirect keeps old install URLs working).
`awrshift/skill-brainstorm` is archived with a pointer to this skill (functionality fully merged).

</details>

---

## License

MIT. Use freely, fork freely, contribute freely.

## Contributing

PRs welcome. Particularly interested in:
- New trigger phrases (especially non-English)
- New case studies from real use
- Edge cases where dual-validation didn't work and why
- Marketing-friendly explanations of any developer concept that's still confusing

Open an issue or PR at [github.com/awrshift/skill-dont-ask-me](https://github.com/awrshift/skill-dont-ask-me).

---

<div align="center">

**Don't Ask Me.**

Stop being the bottleneck. Your AI now cross-checks itself with other LLMs before bothering you.

⭐ Star this repo if it saved you from a hallucinated stat in production.

</div>
