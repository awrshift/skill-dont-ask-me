# When to Use Gemini vs Claude Subagents

**Principle:** Claude subagents (Task tool) = DEFAULT. Gemini only where unique value.

## Claude Subagents WIN (use Task tool)

- Code review (can READ files, follow imports)
- Codebase exploration (has Grep, Glob, Read)
- Deep reasoning (`Task(model="opus")` > `gemini think`)
- Any task needing file I/O or project context

## Gemini WINS (use CLI skill)

- **Independent second opinion** — different model family = different biases
- **Quick stateless questions** — `ask` (<3s, cheap)
- **Fact-check verification** — cross-validate Claude's analysis with Gemini 3.1 Pro
- **Parallel batch calls** — 4+ background processes via `&` + `wait`

## Model Selection Quick Reference

| Need | Best option | Fallback |
|------|------------|----------|
| Parallel file-aware work | `Task(model="sonnet")` | — |
| Deep reasoning | `Task(model="opus")` | `gemini.py think` (3.1 Pro, HIGH) |
| **Second opinion** | **`gemini.py second-opinion` (3.1 Pro)** | `gemini.py ask -m gemini-3-flash-preview` |
| Quick question | `gemini.py ask` (3 Flash) | — |
| Research / fact-check | `gemini.py ask -m gemini-3.1-flash-lite-preview --grounded` | — |
| Data extraction | `gemini.py extract` (Flash-Lite, MINIMAL) | — |
| Image generation | `gemini_image` MCP tool (Flash default) | Pro for best quality |

## DO NOT Use Gemini For

- File operations (Claude Code has direct access)
- Neo4j queries (use neo4j driver)
- Tasks requiring conversation history (Gemini calls are stateless)
- Sensitive data that shouldn't leave local machine (Gemini sends to Google API)
- Tasks where Claude subagent can do the same thing WITH file access
