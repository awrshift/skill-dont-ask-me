<div align="center">

![Claude Code](https://img.shields.io/badge/Claude%20Code-D97757?style=for-the-badge&logo=claude&logoColor=white)
![Gemini](https://img.shields.io/badge/Gemini-4285F4?style=for-the-badge&logo=google&logoColor=white)
![License MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)

# skill-gemini

**Gemini inside Claude Code. Second opinions, web-grounded facts, diagrams, brainstorms — one skill.**

*Different model family = different blind spots. Use both.*

</div>

---

## Why

Claude is great. But a single model has blind spots. Gemini sees differently — different training, different reasoning patterns, different knowledge cutoffs. This skill gives Claude direct access to Gemini for everything: quick questions, critical reviews, real-time web facts, image generation, code visualization, and structured multi-round brainstorms.

One install. Five capabilities.

<div align="center">
<img src="assets/gemini-capabilities.png" alt="5 capabilities: Ask, Ground, Image, Visualize, Brainstorm" width="800" />
</div>

---

## Quick Install

**Claude Code (recommended):**
```
/plugin marketplace add awrshift/skill-gemini
```

**Manual:**
```bash
mkdir -p .claude/skills/gemini/scripts .claude/skills/gemini/references
curl -sL https://raw.githubusercontent.com/awrshift/skill-gemini/main/SKILL.md \
  -o .claude/skills/gemini/SKILL.md
curl -sL https://raw.githubusercontent.com/awrshift/skill-gemini/main/scripts/gemini.py \
  -o .claude/skills/gemini/scripts/gemini.py
```

## Setup (2 minutes)

1. Get API key at [aistudio.google.com](https://aistudio.google.com) → Create API Key
2. Add to `.env`: `GOOGLE_API_KEY=your_key_here`
3. Install: `pip install google-genai`

That's it. The skill auto-discovers `gemini.py` from its bundle.

## 5 Capabilities

### 1. Ask — Text, Reasoning, Review

| You say | What happens |
|---------|-------------|
| "Ask Gemini about X" | Quick answer via Flash (fast, cheap) |
| "Get a second opinion" | Critical review via Pro (deep reasoning) |
| "Think deeply about X" | Maximum reasoning via Pro (high thinking) |
| "Review this code" | Code review with focus areas |

```bash
python3 gemini.py ask "Quick question"
python3 gemini.py second-opinion "Is this right?" --context "details"
python3 gemini.py think "Design a caching strategy"
python3 gemini.py review "def process(data): ..." --focus "error handling"
```

### 2. Ground — Web-Verified Facts

Gemini searches Google before responding. Real-time facts, not training data.

```bash
python3 gemini.py ask "Latest Next.js version?" --grounded
python3 gemini.py second-opinion "Is Clerk still free?" --grounded
```

### 3. Image — Generate Visuals

Banners, diagrams, illustrations — via Gemini Nano Banana models.

```
"Generate a dark banner for my GitHub repo"
"Create an architecture diagram"
```

### 4. Visualize — Code to Diagrams

Three-phase pipeline: read your codebase → synthesize nodes/edges → render via Gemini Image.

```
"Visualize the architecture of this project"
"Draw the data flow"
"Show me how these services connect"
```

Supports: `system-map`, `flow`, `connections`, `data-model`. Full guide in `references/visualize.md`.

### 5. Brainstorm — 3-Round Adversarial Dialogue

Claude orchestrates. Flash researches facts. Pro reasons on them. 7 ideas → 3 survivors → 1 winner.

```
"Brainstorm how to launch this"
"Let's think through options with Gemini"
```

Full protocol with prompt templates in `references/brainstorm.md`. Cost: ~$0.35 per brainstorm.

## Models (March 2026)

| Model | Price (in/out) | Best for |
|-------|---------------|----------|
| Pro (`gemini-3.1-pro-preview`) | $2 / $12 per 1M | Reasoning, brainstorm, second opinions |
| Flash (`gemini-3-flash-preview`) | $0.50 / $3 per 1M | General tasks, code review (default) |
| Flash-Lite (`gemini-3.1-flash-lite-preview`) | $0.25 / $1.50 per 1M | Research, grounding, fact-check |

## Gotchas

- **`google-genai` not `google-generativeai`** — the new SDK package name (2026)
- **Temperature 1.0 is mandatory** — lower values cause looping in Gemini 3
- **`--grounded` is a tool, not a parameter** — adds Google Search capability
- **Image generation requires Gemini MCP server** — `mcp__gemini-sdk__gemini_image`
- **Second opinion ≠ final answer** — Claude should always evaluate Gemini's output critically

## Works With

| Platform | Install |
|----------|---------|
| Claude Code | `/plugin marketplace add awrshift/skill-gemini` |
| Codex CLI | Copy `skills/gemini/` to `.openai/skills/` |
| Gemini CLI | Copy `skills/gemini/` to `.gemini/skills/` |
| Any Agent Skills tool | Follow [spec](https://agentskills.io) |

## Part of the AWRSHIFT Ecosystem

- [**skill-awrshift**](https://github.com/awrshift/skill-awrshift) — adaptive decision-making framework
- [**skill-brainstorm**](https://github.com/awrshift/skill-brainstorm) — standalone 3-round brainstorm (also bundled here)
- [**skill-telegram**](https://github.com/awrshift/skill-telegram) — Telegram user API integration
- [**ClawClaw Soul**](https://clawclawsoul.com) — persistent identity protocol for AI agents

## Contributing

1. Fork the repository
2. Create a feature branch
3. Submit a pull request

## License

MIT — see [LICENSE](LICENSE) for details.

---

<div align="center">
<em>Different model. Different blind spots. Better decisions.</em>
</div>
