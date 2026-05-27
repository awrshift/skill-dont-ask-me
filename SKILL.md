---
name: dont-ask-me
description: "Cross-check Claude's answers with a second AI before bothering the user. Use when the user says 'second opinion', 'sanity check', 'cross-check', 'am I missing something', 'dual validation', 'stress-test', 'review this', 'devil's advocate', 'boardroom debate', 'run a full review', 'this is important', 'big decision', 'high-stakes', 'check before I send', 'before publishing', 'brainstorm options', 'help me choose between', 'I have several options', 'I have N angles', 'multiple paths', 'diverge and converge', 'multi-round brainstorm', 'ask Gemini', 'ask Opus', 'critique this', or any Russian equivalents: 'второе мнение', 'не уверен', 'спроси Gemini', 'спроси Опус', 'критикуй', 'найди слепые пятна', 'помоги выбрать', 'обсуди варианты', 'это важное решение', 'прогони полную проверку'. Three review styles — Devil's Advocate (single critique), Boardroom Debate (parallel dual validation, HEADLINE), Round-Table Discussion (3-round brainstorm for multi-option convergence). Two model families (Gemini + isolated Claude Opus subagent) catch different blind spots. NOT for factual lookups — use Claude Code's native WebSearch instead."
allowed-tools: Bash, Read, Glob, Grep, Task, Write
---

# Don't Ask Me — Stop being the bottleneck

Your AI cross-checks itself with another AI before bothering you. Three review styles, picked automatically based on what the user typed. Two independent model families (Gemini + isolated Claude Opus subagent) catch different blind spots than one model alone.

## What this does (in one sentence)

When the user is unsure, asks a high-stakes question, or proposes a decision that affects shipping, Claude quietly asks Gemini and/or an isolated Opus subagent to review the answer first — then shows the user where the reviewers agreed, disagreed, or caught something the user might miss.

## When to invoke

Four trigger conditions. If any fire, pick the matching review style (see § Style Router below).

1. **User has 2+ viable paths to choose between** — architecture options, technology choice, content angle → Round-Table Discussion
2. **High-stakes design decision** — ADR, architecture, launch copy, anything with rollback cost → Boardroom Debate DEFAULT
3. **User stuck on the same problem ≥2 attempts** — failed fix, failed retry, considering adding complexity → Devil's Advocate
4. **Non-trivial proposal awaiting user approval** — new module, refactor plan, multi-step migration → Devil's Advocate or Boardroom Debate depending on stakes

If none fire, do not invoke this skill — single-model answer suffices for routine work.

**Out of scope:** factual lookups ("what's the latest X version?", "is Y still free?") — use Claude Code's native WebSearch tool instead. This skill is about critique and adversarial validation, not fact retrieval.

## The three review styles

### 1. Devil's Advocate (Single Critique)

**Trigger phrases:** "дай второе мнение", "sanity check this", "am I missing something", "stress-test this", "критикуй", "найди слепые пятна"

**Two implementations — Claude picks based on context:**

- **(a) Gemini second-opinion** — different model family entirely. Use when claim is about tech/external world or when you want a perspective trained on different data.
- **(b) idea-validator subagent** — isolated Claude Opus with no parent conversation context. Use when you need to stress-test YOUR reasoning chain without inherited bias from the parent agent's framing.

**Model (a):** `gemini-3.5-flash` with adversarial reviewer system prompt
**Model (b):** `claude-opus`, fixed-format output (Verdict / Top 3 concerns / Hidden assumptions / What to verify / Counterargument)
**Latency:** 10-20 seconds
**Cost:** $$ tier

**Decision rule:**
- Tech/external claim? → Gemini (different training distribution catches different errors)
- Internal reasoning chain? → Opus subagent (different perspective on same family)
- Not sure? → Default to Gemini for breadth, Opus for depth

**Example (Gemini):**
```python
Bash("python3 ~/.claude/skills/dont-ask-me/scripts/gemini.py second-opinion @prompt.md --save out.md")
```

**Example (Opus subagent):**
```python
Agent(
  subagent_type="idea-validator",
  description="Validate <design name>",
  prompt="<self-contained design / decision / plan as artifact>"
)
```

### 2. Boardroom Debate (Dual Validation) — THE HEADLINE PATTERN

**Trigger phrases:** "прогони dual-validation", "это важное решение", "high-stakes review", "give me two independent opinions", "boardroom debate"

**Tool:** PARALLEL call to BOTH Gemini second-opinion AND idea-validator Opus subagent in the SAME message.

**Why parallel matters:** if you call sequentially, the second reviewer sees the first's framing in the artifact you sent — independence destroyed, you get two echoes instead of two minds.

**Protocol:**

0. **Pre-flight check (mandatory):** verify `~/.claude/agents/idea-validator.md` exists before attempting Boardroom Debate. If missing, tell user: "Boardroom Debate requires the idea-validator subagent. Run: `curl -sL https://raw.githubusercontent.com/awrshift/skill-dont-ask-me/main/agents/idea-validator.md -o ~/.claude/agents/idea-validator.md`" — then fall back to Devil's Advocate (single Gemini critique) for the current call.

1. Claude prepares one self-contained artifact describing the design / decision / plan. Paste content INLINE, never use `@file:` references in the artifact text itself — Gemini Flash will silently hallucinate if `@file:` doesn't load (Session #199 incident: 14 KB audit plan via `@file:` returned a plausible-looking review citing audit IDs D1/F2/C4 that did not exist in the source).

2. ONE message with TWO tool calls:
   ```python
   # Same message:
   Bash("python3 ~/.claude/skills/dont-ask-me/scripts/gemini.py second-opinion @prompt.md --save gemini-out.md")
   Agent(subagent_type="idea-validator", prompt="<same artifact pasted inline>")
   ```

3. Read both outputs, build acceptance ledger:

   | # | Concern | Gemini says | Opus says | My evaluation | Action |
   |---|---|---|---|---|---|

4. Three outcome classes per row:
   - **Both agree** → high-signal, must address
   - **One raises, other silent** → medium-signal, evaluate whether silent one had context to notice
   - **They disagree** → HIGHEST-value finding (different model perspectives surfaced different things — investigate which is right and why)

5. Present BOTH to user critically (per Critical Evaluation Rule below). Don't forward as-is. Show where you accept, where you push back, where reviewers diverge and which side you take with reasoning.

6. Claude makes final decision — this is not voting between two AIs. Two opinions = input, not majority rule.

**When to use:** ADR drafts, architecture decisions, multi-step plans (≥3 stages), new patterns before codifying, hypothesis falsification, prompt design for core logic, launch copy for public release, anything where rollback cost is meaningful.

**Evidence from real history (Session #201 ADR sprint, 2026-05-25):** 6 dual-validations covering 8 architecture decisions (some validations spanned multi-decision batches). EVERY validation found at least one load-bearing problem the proposing agent had missed — 6+ distinct issues total. Examples: BullMQ `limiter` semantics misread (caps execution not enqueue, silent 2-hour SSE stall), LLM "deterministic at temp=0.2" myth (both validators independently flagged non-determinism), indefinite retention plan = GDPR storage limitation violation, pre-call `SELECT SUM` plan = TOCTOU race + 500ms × 50 calls burst stall. Total overhead: ~4% of session time. Issues caught would have cost an entire ADR rewrite or production incident.

**Latency:** 15-30 seconds (parallel)
**Cost:** $$$ tier

### 3. Round-Table Discussion (3-Round Brainstorm)

**Trigger phrases:** "помоги выбрать", "brainstorm options", "we have several paths", "diverge then converge", "multi-round brainstorm", "обсуди варианты"

**Tool:** 3-round structured protocol between Claude and Gemini, with mandatory user check-in mid-flow.

**Two-layer model assignment (Flash researches, Pro reasons):**
- **Flash (grounded):** Phase 0.5 ground + Phase 3.5 fact-check — cheap web verification
- **Pro (ungrounded):** R1 / R2 / R3 — all thinking tokens spent on critical analysis, not search

**Process:**

```
Phase 0:   Gather context (what exists, constraints, goal)
Phase 0.5: GROUND — dual web-check (Claude WebSearch + Flash --grounded in parallel)
R1: DIVERGE — Pro proposes 7+ ideas, challenges Claude's framing
R1.5: VERIFY (if R1 introduced new tech) — Flash checks new claims
R2: DEEPEN — kill to 3 with concrete arguments
   ★ MANDATORY USER CHECK-IN here — present 2-3 survivors, ask "do you have a preference?"
R3: CONVERGE — Pro picks ONE primary action, secondary, kill list
Phase 3.5: FACT-CHECK — verify all R3 claims (dual: Claude + Flash)
Phase 4: Synthesize — summary table + final recommendation + kill list
```

**Use when:** 2+ viable paths, technology choice, strategy decision, before committing to architecture with multiple defensible options.

**For full protocol with prompt templates per phase**, read `references/round-table.md`.

**Latency:** 5-10 minutes (multi-turn, includes user check-in)
**Cost:** $$$$ tier

## Style Router — how Claude picks

Decision tree, evaluated top-down:

1. Is this a high-stakes design decision (ADR, architecture, launch copy, anything with meaningful rollback cost)? → **Boardroom Debate DEFAULT**
2. Are there 2+ viable paths to choose between with no clear winner? → **Round-Table Discussion**
3. Otherwise — stuck on a problem, need critique on a non-trivial proposal, want sanity check? → **Devil's Advocate**
   - Pick **Gemini second-opinion** if claim is about tech / external world / has different training distribution
   - Pick **idea-validator Opus** if need to stress-test YOUR reasoning chain without inherited bias

**Out of router:** factual lookups (current version, price, statistic, recent event) → not this skill. Use Claude Code's native WebSearch tool.

## Critical Evaluation Rule (applies to ALL styles)

Reviewer output is INPUT for decision-making, NOT the decision itself.

After every reviewer call:

1. **Challenge each recommendation** — is this fact or speculation?
2. **Check for missing context** — reviewer doesn't see codebase / prior decisions / project constraints
3. **Verify numbers** — predictions like "90% speedup" are estimates, not measurements
4. **Look for blind spots** — implementation complexity, side effects, existing code reviewer can't see
5. **Present critically to user** — table "Reviewer said / My evaluation / Reason / Action"

**Workflow:** Reviewer recommends → Claude evaluates critically → present BOTH to user → user decides.

**Anti-patterns:**
- Forwarding reviewer output as-is without analysis
- Accepting all recommendations without questioning
- Using reviewer confidence as proxy for correctness
- Skipping critical evaluation "because the reviewer is from a different model family / is Opus / is smart"

For why this rule exists with full anti-pattern examples, read `references/critical-evaluation.md`.

## Own thinking FIRST, reviewer SECOND

Before invoking ANY review style, Claude must have:

1. Done its own deep research (read relevant files, prior decisions, source material)
2. Formed its own proposal with rationale + trade-offs + open questions
3. Shown that proposal to the user

ONLY THEN invoke the reviewer to critique that proposal.

**Why:** if Claude goes to Gemini without a thought-through position, it's easy to accept Gemini's recommendation uncritically — Gemini lacks codebase context and can confidently suggest something wrong. Reviewer = validator of YOUR thinking, not the seed of new ideas.

**Trigger sequence:**
- User asks complex question → Claude researches, synthesizes own answer
- Claude shows user the answer with reasoning
- IF stakes are high (per § When to invoke) → Claude offers to cross-check via review style
- User approves → Claude invokes the chosen style
- Reviewer returns → Claude evaluates critically, presents ledger
- User decides

Skipping step 1-3 and asking Gemini first = anti-pattern. Codified Session #142 after L4.1 topic clusterer where this exact mistake was caught.

## Cost rules — when expensive styles are justified

| Style | Cost per call | When justified |
|---|---|---|
| Devil's Advocate (Gemini) | ~$0.01 | Non-trivial decisions, prompt before finalizing, stuck after ≥2 attempts |
| Devil's Advocate (Opus subagent) | ~$0.03-0.05 | Reasoning chain stress-test, fresh adversarial perspective |
| Boardroom Debate (parallel both) | ~$0.05-0.08 | High-stakes: ADR, architecture, launch copy, anything with rollback cost |
| Round-Table Discussion (3-round) | ~$0.25 | Multi-path strategy decisions, technology choice, before committing to architecture |

**Rule of thumb:** cost of validating < cost of being wrong × probability of wrong.

**Real cost reality (Session #201 ADR sprint):** 8 decisions, 6 dual-validations (some batched), ~60s overhead each = ~6 minutes total. ADR took ~3 hours. Validation overhead = ~4% of session time. Caught 6+ load-bearing issues. Obvious ROI for ADR-class work.

## Anti-patterns (do NOT do these)

1. **Sequential parallel calls** — running Gemini THEN Opus destroys independence. Same message, two tool calls. (Per `references/dual-validation.md`.)

2. **`@file:` references in Gemini prompts** — Gemini Flash silently hallucinates content when `@file:` fails to load. Returns plausible-looking review with citations that don't exist in source. ALWAYS paste content INLINE. ALWAYS verify identifiers in response match input (scan for 2-3 IDs, throw out if even one doesn't match). Codified Session #199 (2026-05-24) after 14 KB audit plan incident.

3. **Asking Gemini BEFORE forming own opinion** — turns Gemini into a seed for the decision, not a validator. Do your own deep thinking first. Codified Session #142 (2026-05-16) on L4.1 topic clusterer.

4. **Boardroom Debate for trivial things** — burns tokens for stylistic preferences (file naming, format choices, code style with project rules). Save dual validation for genuinely high-stakes design moments.

5. **Accepting reviewer output as-is** — same Critical Evaluation Rule applies. Output is INPUT for decisions, not the decision. Two opinions ≠ majority rule.

6. **Using `gemini think` for code review** — Claude subagents (Task tool) can READ files and follow imports. Gemini cannot. Use Claude subagent for file-aware code review, Gemini for independent reasoning without codebase access.

## CLI Reference (for direct invocation when Claude needs custom params)

```bash
GEMINI="$(find . ~/.claude/skills -name gemini.py -path '*/dont-ask-me/*' 2>/dev/null | head -1)"

# Web-grounded ask (used internally by Round-Table Phase 0.5 + 3.5 for tech verification —
# NOT a user-facing review style. For factual lookups, use Claude Code's native WebSearch.)
python3 $GEMINI ask "Verify these technologies: A, B, C" --grounded

# Devil's Advocate — single critique
python3 $GEMINI second-opinion @prompt.txt --save out.md

# Boardroom Debate — paired with Agent call to idea-validator in same Claude message
python3 $GEMINI second-opinion @prompt.txt --save gemini-out.md

# Deep reasoning
python3 $GEMINI think "Design a caching strategy for 10M users"

# Visual review (screenshots)
python3 $GEMINI second-opinion @prompt.txt --image our.png --image ref.png --save review.md

# Structured extraction
python3 $GEMINI extract "Parse this invoice..." --json-mode
```

| Command | Default Model | Use When |
|---------|---------------|----------|
| `ask` | `gemini-3.5-flash` | Quick questions, research with `--grounded` |
| `second-opinion` | `gemini-3.5-flash` (adversarial system prompt) | Devil's Advocate, Boardroom Debate (paired with Opus) |
| `think` | `gemini-3.5-flash` (high thinking) | Complex reasoning, architecture sketch |
| `extract` | `gemini-3.5-flash` (JSON mode) | Structured extraction from unstructured text |

Legacy: `gemini-3.1-pro-preview` available via `-m` flag for pure-math reasoning (ARC-AGI-style) where Flash 3.5 falls short. Post-audit 2026-05-20, Flash 3.5 beat Pro by 5 findings, 22% faster, 33% cheaper, 0 factual errors — Pro retained only as legacy fallback.

## Setup (4 steps, one-time)

1. **Get Gemini API key** at [aistudio.google.com](https://aistudio.google.com) → Create API Key
2. **Add to `.env`:** `GOOGLE_API_KEY=your_key_here`
3. **Install SDK:** `pip install google-genai`
4. **Install idea-validator subagent** (required for Boardroom Debate):
   ```bash
   mkdir -p ~/.claude/agents
   curl -sL https://raw.githubusercontent.com/awrshift/skill-dont-ask-me/main/agents/idea-validator.md \
     -o ~/.claude/agents/idea-validator.md
   ```

(Or one-line install via plugin marketplace: `/plugin marketplace add awrshift/skill-dont-ask-me` — handles all four steps.)

## Multimodal — images and video

Pass `--image` (repeatable) or `--video` (URL or local file) to any text command:

```bash
# Compare two designs visually
python3 $GEMINI second-opinion "compare designs, score 1-10" --image ours.png --image ref.png --save review.md

# YouTube video analysis
python3 $GEMINI ask "summarize this video" --video https://youtube.com/watch?v=ID
```

Supported images: PNG, JPEG, WebP, GIF (max ~20 MB total). Gemini 3.x natively multimodal. Use case examples: design review, visual QA, screenshot diff, accessibility audit.

## Parallel execution (advanced)

When you have multiple independent aspects to validate:

```bash
python3 $GEMINI second-opinion @aspect1.txt --save r1.md &
python3 $GEMINI second-opinion @aspect2.txt --save r2.md &
python3 $GEMINI second-opinion @aspect3.txt --save r3.md &
wait
```

All run concurrently. Useful for multi-axis review (security + performance + readability in parallel).

## References (read on-demand)

- **`references/round-table.md`** — Full 3-round brainstorm protocol with phase-by-phase prompt templates
- **`references/dual-validation.md`** — Parallel call protocol, acceptance ledger format, three outcome classes
- **`references/decision-framework.md`** — When to use Gemini vs Claude subagents vs Opus subagent
- **`references/critical-evaluation.md`** — Why reviewer output ≠ decision, with anti-pattern examples
- **`references/models.md`** — All Gemini models, pricing, thinking levels, deprecation table
- **`references/parameters.md`** — Complete CLI argument reference

## Migrating from skill-gemini or skill-brainstorm

If you previously installed `awrshift/skill-gemini` or `awrshift/skill-brainstorm`:

1. **Uninstall old:** `/plugin marketplace remove awrshift/skill-gemini` (and/or `awrshift/skill-brainstorm`)
2. **Install new:** `/plugin marketplace add awrshift/skill-dont-ask-me`
3. **All previous CLI commands work unchanged** — `gemini.py ask`, `second-opinion`, `think`, `extract`, `--grounded`, `--image` all behave identically. Just point to `~/.claude/skills/dont-ask-me/scripts/gemini.py` instead of `~/.claude/skills/gemini/gemini.py`.
4. **New capabilities added:**
   - Boardroom Debate (parallel dual validation with idea-validator subagent)
   - Round-Table Discussion (3-round brainstorm — was `skill-brainstorm`)
   - Style Router (Claude auto-picks based on user phrasing)
   - Critical Evaluation Rule (explicit anti-patterns)

`awrshift/skill-brainstorm` is archived with pointer to this skill (functionality fully merged).
`awrshift/skill-gemini` is renamed to `awrshift/skill-dont-ask-me` (GitHub redirect handles old install URLs).

---

## Why two model families matter (in 30 seconds)

If both reviewers are from the same lab (e.g., two Claude instances), they share training distribution, same blind spots, similar reasoning failure modes. You get two echoes, not two minds.

**Gemini (Google)** has different training data, different reasoning patterns, different post-training feedback loops than Claude (Anthropic). Where Claude misses, Gemini sometimes catches. Where Gemini misses, Claude catches.

**Isolated Opus subagent** is same family as parent Claude — but without inherited context. It cannot anchor on the parent's framing, can't be primed by prior conversation. Fresh adversarial perspective on the artifact.

**Boardroom Debate uses both** because:
- Gemini catches what Anthropic-family models miss (external optics, different training distribution)
- Opus subagent catches what's wrong with YOUR reasoning chain (internal logic without parent bias)

Single reviewer = one angle. Two reviewers from different families = two angles, not two voices.

Evidence: Phase D experiment A433. Same Gemini model with basic prompt = 50% accuracy. Same model + prompt stress-tested by Gemini itself = 75-80%. The model wasn't the bottleneck — the prompt was. Different model family caught the prompt's blind spots that Claude couldn't see from inside.
