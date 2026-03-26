---
name: gemini
description: "Gemini inside Claude Code. Second opinions, web-grounded facts, image generation, code visualization, and multi-round brainstorms — all via one skill. Use when user says 'ask Gemini', 'second opinion', 'cross-validate', 'check with Gemini', 'ground this', 'verify this claim', 'generate image', 'visualize', 'draw architecture', 'diagram', 'brainstorm with Gemini', or needs an independent opinion from a different model family. Also trigger when evaluating hypotheses, fact-checking, generating diagrams from code, or wanting adversarial multi-model ideation."
allowed-tools: Bash, Read, Glob, Grep, mcp__gemini-sdk__gemini_image
---

# Gemini Skill — Full Toolkit for Claude Code

Different model family = different blind spots. This skill gives Claude direct access to Gemini for text, search, images, and multi-round dialogue.

## Setup (first run)

1. Get API key at https://aistudio.google.com → create key
2. Add to `.env`: `GOOGLE_API_KEY=your_key`
3. Install SDK: `pip install google-genai`
4. Find `gemini.py`: bundled at `scripts/gemini.py` relative to this SKILL.md

```bash
GEMINI="$(find . ~/.claude/skills -name gemini.py -path '*/gemini/*' 2>/dev/null | head -1)"
```

## 1. Text — Ask, Think, Review

```bash
# Quick question (Flash — fast, cheap)
python3 $GEMINI ask "What's the difference between REST and GraphQL?"

# Second opinion (Pro — deep reasoning, critical)
python3 $GEMINI second-opinion "Is this the right architecture?" --context "context here"

# Deep reasoning (Pro — maximum thinking)
python3 $GEMINI think "Design a caching strategy for 10M users"

# Code review
python3 $GEMINI review "def process(data): ..." --focus "error handling"

# Data analysis
python3 $GEMINI analyze "revenue data shows..." --context "Q1 report"

# Structured extraction
python3 $GEMINI extract "Parse this invoice..." --json-mode
```

| Command | Model | Best for |
|---------|-------|----------|
| `ask` | Flash | Quick questions, general tasks |
| `second-opinion` | Pro | Decisions, validation, critical review |
| `think` | Pro (high thinking) | Complex reasoning, architecture |
| `review` | Flash | Code quality, bugs |
| `analyze` | Flash | Data patterns, CSV analysis |
| `extract` | Flash | Structured JSON from text |

## 2. Web-Grounded Answers

Add `--grounded` to any command. Gemini searches Google before responding — real-time facts, not training data.

```bash
# Current version of a library
python3 $GEMINI ask "What's the latest Next.js version?" --grounded

# Verify a claim with sources
python3 $GEMINI second-opinion "Is Clerk still free for 10K MAU?" --grounded

# Research mode (Flash-Lite — cheapest, fastest for facts)
python3 $GEMINI ask "Compare Drizzle vs Prisma pricing 2026" \
  -m gemini-3.1-flash-lite-preview --grounded --save research.md
```

## 3. Image Generation

Generate images via Gemini Nano Banana models. Requires `mcp__gemini-sdk__gemini_image` MCP tool.

```
# From Claude Code — just describe what you want:
"Generate an architecture diagram for my project"
"Create a banner image with dark background and gold accents"
"Draw a flow chart showing the auth process"
```

The skill uses Gemini Flash Image (fast) or Pro Image (quality). Supports 14 aspect ratios, 512-4K resolution.

For **code-to-diagram** generation, see the Visualize workflow below.

## 4. Visualize — Code to Diagrams

Turn any codebase into professional diagrams. Three phases: Gather → Synthesize → Render.

**Types:** `system-map` | `flow` | `connections` | `data-model`

**Quick usage:**
```
"Visualize the architecture of this project"
"Draw how the API pipeline works"
"Show me the data model"
"How are these services connected?"
```

**Process:**
1. **Gather** — read interfaces, configs, entry points (max 10 files, never .env)
2. **Synthesize** — convert to Nodes + Edges in plain English (anti-hallucination: Gemini never sees raw code)
3. **Render** — call Gemini Image with 1500+ char prompt built from synthesis

For detailed gathering strategies and prompt templates, read `references/visualize.md`.

## 5. Brainstorm — 3-Round Claude x Gemini

Structured adversarial dialogue. Flash researches facts, Pro reasons on them, Claude orchestrates.

```
"Brainstorm how to launch this product"
"Let's think through options for the architecture"
"Diverge and converge on the best approach"
```

**Pipeline:** Ground (dual web-check) → R1: Diverge (7+ ideas) → R2: Deepen (kill to 3) → User check-in → R3: Converge (1 winner) → Fact-check → One decision.

For the full protocol with prompt templates, read `references/brainstorm.md`.

## Common Arguments

| Arg | Short | Description |
|-----|-------|-------------|
| `--model` | `-m` | Override model (e.g., `gemini-3.1-flash-lite-preview`) |
| `--context` | `-c` | Additional context |
| `--grounded` | `-g` | Enable Google Search grounding |
| `--save` | | Save response to file |
| `--thinking` | `-t` | Level: minimal/low/medium/high |
| `--json-mode` | | Force structured JSON output |
| `--temp` | | Temperature (default 1.0 — keep default for Gemini 3) |
| `@file` | | Read prompt from file |

## Models (March 2026)

| Model | Price (in/out per 1M) | Best for |
|-------|----------------------|----------|
| `gemini-3.1-pro-preview` | $2 / $12 | Reasoning, brainstorm, second opinions |
| `gemini-3-flash-preview` | $0.50 / $3 | General tasks, code review (default) |
| `gemini-3.1-flash-lite-preview` | $0.25 / $1.50 | Research, grounding, fact-check |

For full model specs, read `references/models.md`.

## Critical Evaluation Rule

Gemini's output is INPUT for decisions, not the decision itself. After every call:
1. Challenge each recommendation — fact or speculation?
2. Check for missing context — Gemini doesn't see your codebase
3. Verify numbers — predictions are estimates, not measurements
4. Present both views to user — "Gemini said / My evaluation / Reason"

## Parallel Execution

```bash
python3 $GEMINI second-opinion @aspect1.txt --save r1.md &
python3 $GEMINI second-opinion @aspect2.txt --save r2.md &
python3 $GEMINI second-opinion @aspect3.txt --save r3.md &
wait
```

## References

Read these only when needed:
- `references/models.md` — all models, pricing, thinking levels
- `references/parameters.md` — complete API parameter reference
- `references/visualize.md` — diagram generation guide (types, gathering, synthesis, rendering)
- `references/brainstorm.md` — 3-round protocol with prompt templates
- `references/decision-framework.md` — when to use Gemini vs Claude subagents
