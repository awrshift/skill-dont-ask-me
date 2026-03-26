# Brainstorm — 3-Round Claude x Gemini Protocol

## Architecture

```
              Claude (orchestrator)
                 |         |
    Phase 0.5    |         |    R1 / R2 / R3
    Phase 3.5    |         |
                 v         v
          Flash-Lite     Pro (no grounding)
       (research layer)  (reasoning layer)
                 |              ^
                 +--Verified----+
                    Context
```

Flash ($0.25/1M) searches the web. Pro ($2/1M) reasons on verified facts. Claude challenges between rounds.

## Pipeline

| Phase | What | Model |
|-------|------|-------|
| 0: Context | Gather constraints, goals, existing work | Claude |
| 0.5: Ground | Web-verify all tech/prices/versions | Flash-Lite --grounded + Claude WebSearch |
| R1: Diverge | Gemini challenges framing, proposes 5+ angles | Pro (ungrounded) |
| R1.5: Verify | Check new tech Gemini introduced (if any) | Flash-Lite --grounded |
| R2: Deepen | Claude kills weak ideas, Gemini stress-tests | Pro (ungrounded) |
| Check-in | User picks preference | Human |
| R3: Converge | One winner, timeline, go/no-go | Pro (ungrounded) |
| 3.5: Fact-check | Dual-verify final decision claims | Flash-Lite + Claude WebSearch |
| Synthesize | Summary table, kill list, recommendation | Claude |

## R1 Prompt Template

```
You are an adversarial brainstorming partner. Round 1 of 3.
NOT agreeable — challenge, flip assumptions, add angles I haven't considered.

IMPORTANT: "Verified Context" below is web-checked. Treat as ground truth.
Focus on STRATEGY, not verifying facts.

## Verified Context (web-checked)
[Phase 0.5 findings]

## Context
[What exists, constraints, goals]

## My Initial Assessment
[Strengths, weaknesses, open questions]

## What I want (Round 1)
1. Challenge my framing — what am I NOT seeing?
2. Propose 3-5 alternative angles
3. Kill at least 1 idea (with reasoning)
4. Add 1 wildcard idea
5. Flag any NEW technology not in Verified Context
```

## R2 Prompt Template

```
Round 2. STRESS-TEST my pushback, pick 2 strongest threads, kill everything else.

## Your Round 1 Key Ideas (summary)
[Synthesize, don't copy-paste]

## My Critical Evaluation
[KILL or KEEP each idea with reasoning]

## Constraints
[Time, resources, regulatory, skill gaps]

## What I want (Round 2)
1. Defend killed ideas — ONE sentence each
2. Pick 2 strongest survivors
3. Concrete 2-week plan for each
4. Propose 1 combined idea from survivors
```

**After R2 — USER CHECK-IN (mandatory):** Present survivors, ask preference.

## R3 Prompt Template

```
Round 3 (FINAL). Time to converge.

## What survived
[2-3 artifacts with descriptions]

## My objection
[One remaining concern]

## What I want (Round 3 — FINAL)
1. Address my objection
2. Pick ONE action (not 2, not 3)
3. Sequence: primary → secondary → kill list
4. 1-sentence pitch for the winner
5. Concrete timeline (hours/days)
6. Go/no-go signal
```

## Execution Commands

```bash
GEMINI="path/to/gemini.py"
AGENT="myproject"

# Phase 0.5
python3 $GEMINI ask @/tmp/brainstorm-${AGENT}-ground.txt \
  -m gemini-3.1-flash-lite-preview --grounded \
  --save /tmp/brainstorm-${AGENT}-ground-response.md

# R1-R3 (Pro, ungrounded)
python3 $GEMINI second-opinion @/tmp/brainstorm-${AGENT}-r1.txt \
  --save /tmp/brainstorm-${AGENT}-r1-response.md

# Phase 3.5 (fact-check)
python3 $GEMINI ask @/tmp/brainstorm-${AGENT}-factcheck.txt \
  -m gemini-3.1-flash-lite-preview --grounded \
  --save /tmp/brainstorm-${AGENT}-factcheck-response.md
```

## Key Principles

1. Flash researches, Pro reasons — never waste Pro on factual lookups
2. Web-ground before reasoning — stale facts poison all rounds
3. Kill early, kill often — R1: 7+ ideas, R2: cut to 3, R3: converge to 1
4. One action, not a strategy — R3 ends with single deliverable + timeline
5. Go/no-go criteria — every brainstorm ends with abandonment signal
6. Show the funnel (7 → 3 → 1) so user sees why the winner won

Cost per brainstorm: ~$0.35 (3 Pro calls + 2-3 Flash-Lite calls).
