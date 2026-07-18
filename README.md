<p align="center">
  <img src="headline.jpg" alt="Cartographer">
</p>

# Cartographer

[![skills.sh](https://skills.sh/b/stevederico/cartographer)](https://skills.sh/stevederico/cartographer)

One agent skill for architecture: **diagram** mode (animated HTML) and **docs** mode (text deep-dives). Install once; mode is explicit (`diagram` / `docs`) or **chosen automatically** from your wording.

| | skills.sh | Install |
|---|---|---|
| `/cartographer` | [stevederico/cartographer/cartographer](https://skills.sh/stevederico/cartographer/cartographer) | `npx skills add stevederico/cartographer` |

## Example

<p align="center">
  <img src="example.png" alt="E-Commerce Microservices Architecture">
</p>

## Quick Start

```bash
npx skills add stevederico/cartographer

# Diagram mode (or auto when you say draw/diagram/map)
"draw an architecture diagram for a web app with an API, database, and cache"
"/cartographer diagram from codebase"

# Docs mode (or auto when you say explain/how does/deep dive)
"explain the architecture"
"/cartographer docs audio"
```

### Modes

| Mode | Output | Auto when… |
|---|---|---|
| **diagram** | Self-contained animated HTML | draw, diagram, visualize, map the services |
| **docs** | Markdown + file:line | explain, how does, deep dive, walk me through |

Ambiguous prompts default to **docs** (cheaper); say “diagram” if you want a visual.

## Features

- **Two modes** — visual diagrams + text architecture deep-dives in one skill
- **Self-contained HTML** — single file, zero CDN links, zero dependencies
- **Dark neon glass aesthetic** — glassmorphic nodes, glowing edges, dot-grid background
- **Animated particles** — flowing dots along edges show data direction
- **Staggered entrance animations** — nodes and edges fade in sequentially
- **Interactive** — hover highlight, node tooltips, edge tooltips (request/response), zoom/pan, double-click reset
- **Auto-layout** — topological sort, layer assignment, crossing minimization
- **20 built-in icons** — Lucide-style SVGs (server, database, globe, cloud, lock, etc.)
- **5 node shapes** — rect, cylinder, hexagon, circle, stadium
- **4 edge types** — sync, secondary, async, error (each with distinct color and style)
- **Print stylesheet** — white background, no animations for clean printing

## Diagram Types

| Type | When to Use |
|---|---|
| **System Context** | High-level overview with external actors |
| **Container** | Services, databases, and queues within a system |
| **Microservices Map** | Service mesh with sync/async dependencies |
| **Data Flow** | How data moves through the system |
| **Sequence / Request Flow** | Single request traced across services |
| **Deployment** | Cloud resources, regions, and networking |
| **Event-Driven** | Event bus with producers and consumers |

## How It Works

```
  User prompt
       |
       v
  +-----------+
  |  Gather   |  Scan codebase or parse description
  +-----------+
       |
       v
  +-----------+
  | Identify  |  Nodes, edges, groups
  +-----------+
       |
       v
  +-----------+
  |  Layout   |  Topological sort → layer assignment
  |           |  → crossing minimization → positions
  +-----------+
       |
       v
  +-----------+
  | Generate  |  Single .html with inline CSS + JS + SVG
  +-----------+
       |
       v
  +-----------+
  |   Open    |  Launch in default browser
  +-----------+
```

1. **Gather** — Reads codebase files (`package.json`, `docker-compose.yml`, route files, DB connections) or parses a text description.
2. **Identify** — Extracts nodes (services, databases, queues), edges (REST, gRPC, SQL, pub/sub), and groups (logical clusters).
3. **Layout** — Auto-positions nodes using a layered graph algorithm with crossing minimization.
4. **Generate** — Writes a single HTML file with all styles, animations, icons, and interactivity inline.
5. **Open** — Launches the diagram in the default browser.

## Interactivity

Every generated diagram includes:

| Feature | Behavior |
|---|---|
| **Hover highlight** | Dims unconnected nodes/edges to 15% opacity |
| **Node tooltips** | Shows label, subtitle, tech stack on hover |
| **Edge tooltips** | Shows method, endpoint, request/response data on hover |
| **Zoom / Pan** | Mouse wheel zooms toward cursor, click-drag pans |
| **Double-click reset** | Resets zoom and centers the diagram |

## Skills

| Slash command | What |
|---|---|
| `/cartographer` | Animated HTML architecture diagrams |
| `/cartographer-docs` | Text architecture deep-dives (markdown + file:line) |

## File Structure

```
cartographer/
├── skills/
│   ├── cartographer/
│   │   ├── SKILL.md                      # /cartographer (diagrams)
│   │   ├── references/                   # design-system, icons, layout-algorithm
│   │   └── examples/                     # example-microservices.html
│   └── cartographer-docs/
│       └── SKILL.md                      # /cartographer-docs (text deep-dives)
├── README.md
└── CHANGELOG.md
```

## Icon Library

20 built-in icons in Lucide style (24x24, 2px stroke, round caps):

`server` · `database` · `globe` · `user` · `cloud` · `lock` · `mail` · `zap` · `cpu` · `hard-drive` · `git-branch` · `shield` · `activity` · `layers` · `smartphone` · `terminal` · `box` · `wifi` · `key` · `refresh-cw`

## Constraints

- **Max 30 nodes** per diagram — split larger systems by domain
- **Zero external dependencies** — no CDN, no `<script src>`, no `<link href>`
- **System font stack** — `system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif`

## License

MIT
