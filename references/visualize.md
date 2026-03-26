# Visualize — Code to Diagrams Reference

## Diagram Types

| Type | Layout | What to Read | Color Strategy |
|------|--------|-------------|---------------|
| `system-map` | Horizontal layers, top to bottom | Module indexes, configs, entry points | Distinct color per layer |
| `flow` | Left-to-right sequence | Stage/step definitions, function signatures | Gradient warm→cool |
| `connections` | Hub-and-spoke or bipartite | Service imports, API clients, configs | Internal=blue, External=green, User=purple |
| `data-model` | ER-style boxes | Schemas, types, interfaces, migrations | One color per domain |

## Gathering Strategy

Max 10 files. Never read function bodies or .env files.

1. **Overview** — README.md or CLAUDE.md (1 file)
2. **Structure** — `ls` root and `src/` dirs (1-2 calls)
3. **Targeted reads** based on type:
   - system-map: `__init__.py`, docker-compose, entry points
   - flow: stage files (first 50 lines), grep for `execute|process|run`
   - connections: `.env.example`, `package.json`, grep for `import.*client|API`
   - data-model: `**/types.*`, `**/schema*`, `**/models/*`

## Synthesis Format

**Nodes:**
```
- [node_id] "Human-Readable Name" | type: service/database/api/user/tool | group: Layer Name
  What it does in one sentence.
```

**Edges:**
```
- [source] → [target] "Action verb" (e.g., "Sends data", "Authenticates")
```

Rules:
- Translate technical names: `s4_generate.py` → "Article Generator (S4)"
- Group related nodes into layers
- If uncertain about a connection, omit it
- Count nodes and groups for the subtitle

## Render Prompt Structure

Must be **1500+ characters**. Richer prompts = better diagrams.

```
Professional {layout_style} diagram: "{title}"
Subtitle: "{N} {things}, {M} {other things}"

{GROUP_NAME} ({color} zone):
- {Node labels with 1-sentence descriptions}

Connections:
- {Source} → {Target}: "{action}" (solid/dashed arrow)

Include a color-coded legend.
Style: clean modern infographic, high-contrast labels, professional palette.
```

Default: `aspect_ratio: "4:3"`, `image_size: "2K"`, `filename: "viz-{type}-{target}.png"`

## After Rendering

1. Show the image path
2. Describe what the diagram shows (2-3 sentences)
3. Ask if user wants adjustments
