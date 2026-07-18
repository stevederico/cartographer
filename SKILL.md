---
name: cartographer
description: >
  Generate stunning animated software architecture diagrams as self-contained HTML files.
  Use when the user says: "draw an architecture diagram", "diagram this system",
  "visualize this architecture", "create a system diagram", "architecture diagram",
  "show me the architecture", "map this system", "draw the services".
allowed-tools: Bash, Read, Write, Edit, Glob, Grep, WebFetch
metadata:
  author: cartographer
  version: "1.3.0"
  argument-hint: <description or "from codebase">
---

# cartographer

You generate self-contained HTML files with animated architecture diagrams. Every diagram uses a dark neon glass aesthetic with glassmorphic nodes, glowing edges, animated particles, and smooth entrance animations. The output is always a single `.html` file with zero external dependencies — all CSS and JS are inline.

---

## Workflow

Follow these steps in order:

### Step 1: Gather Information

If the user provides a description, use it directly. If no arguments are provided, or the user says "from codebase", "from this project", or similar, scan the current project to discover architecture:

- Read `package.json`, `docker-compose.yml`, `Dockerfile`, `Makefile`, `Procfile`
- Use `Glob` to find API route files (`**/routes/**`, `**/api/**`, `**/*.controller.*`)
- Use `Grep` to find database connections (`mongoose`, `pg`, `sqlite`, `redis`, `mongo`)
- Use `Grep` to find service calls (`fetch(`, `axios`, `grpc`, `amqp`, `kafka`)
- Use `Grep` to find imports between internal modules
- Read any existing architecture docs (`**/ARCHITECTURE.md`, `**/docs/**`)

### Step 2: Identify Components

List all nodes for the diagram. For each node determine:

- `id` — unique kebab-case identifier (e.g., `api-server`)
- `label` — display name (e.g., "API Server")
- `subtitle` — technology or role (e.g., "Hono / Node.js")
- `icon` — one of the 20 icon names from the Icon System
- `shape` — one of: `rect`, `cylinder`, `hexagon`, `circle`, `stadium`
- `group` — optional group ID this node belongs to

### Step 3: Map Connections

List all edges. For each edge determine:

- `from` — source node ID
- `to` — target node ID
- `label` — protocol or method (e.g., "REST", "gRPC", "SQL", "pub/sub")
- `type` — one of: `sync`, `secondary`, `async`, `error`
- `method` — HTTP method or DB operation (e.g., "POST", "GET", "SELECT", "INSERT", "PUBLISH")
- `endpoint` — route path or table/channel name (e.g., "/api/users", "orders", "order:created")
- `request` — request body/params as a short string (e.g., "{ email, name, password }")
- `response` — response data as a short string (e.g., "{ id, token, createdAt }")

### Step 4: Identify Groups

Cluster related nodes into logical groups:

- `id` — unique group identifier
- `label` — display label (e.g., "Backend Services", "Data Layer", "External")
- `children` — array of node IDs belonging to this group

### Step 5: Choose Diagram Type

Pick the best type based on the system architecture:

| Type | When to Use | Key Elements |
|---|---|---|
| **System Context** | High-level overview: your system + external actors | Central system box, user/actor icons, external system boxes, labeled arrows |
| **Container** | Zoom into a system: services, databases, queues | Service nodes, database cylinders, queue shapes, protocol-labeled edges |
| **Microservices Map** | Service mesh: all services + their dependencies | Service grid, API edges, async (dashed) vs sync (solid) lines |
| **Data Flow** | How data moves through the system | Process nodes, data store cylinders, directional arrows with data labels |
| **Sequence / Request Flow** | Single request traced across services | Vertical lifelines, horizontal arrows, time flows top-to-bottom |
| **Deployment** | Cloud resources and how they connect | Region/VPC/subnet grouping boxes, cloud service icons, network edges |
| **Event-Driven** | Event bus, producers, consumers | Event bus central bar, producer nodes left, consumer nodes right, event arrows |

Default to **Microservices Map** for multi-service apps, **System Context** for simpler apps.

### Step 6: Choose Layout Direction

- **Horizontal** (left-to-right): System Context, Container, Microservices Map, Deployment, Event-Driven
- **Vertical** (top-to-bottom): Sequence / Request Flow, Data Flow

### Step 7: Generate HTML

Write a single `.html` file using the Write tool. Default filename: `architecture-diagram.html` (or user-specified name). The file must follow the design system, include all animations, interactivity, and the layout algorithm defined below.

### Step 8: Open in Browser

Run `open <filename>` via Bash to launch the diagram in the default browser.

---

## Design System — Color Palette

Every diagram must include these CSS custom properties:

```css
:root {
  --bg: #0a0a12;
  --bg-grid: rgba(255, 255, 255, 0.03);
  --node-bg: rgba(255, 255, 255, 0.05);
  --node-border: rgba(255, 255, 255, 0.12);
  --node-text: #e2e8f0;
  --node-sub: #94a3b8;
  --edge-primary: #6366f1;
  --edge-secondary: #06b6d4;
  --edge-async: #f59e0b;
  --edge-error: #ef4444;
  --accent-gradient: linear-gradient(135deg, #6366f1, #06b6d4);
  --glow-primary: drop-shadow(0 0 8px #6366f1) drop-shadow(0 0 24px rgba(99, 102, 241, 0.3));
}
```

---

## Node Shapes

### Base Node (Rounded Rect — default `.node`)

Glassmorphism card with 12px border-radius.

```css
.node {
  position: absolute;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  text-align: center;
  background: var(--node-bg);
  border: 1px solid var(--node-border);
  border-radius: 12px;
  padding: 16px 20px;
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  z-index: 10;
  cursor: default;
  transition: filter 0.2s ease, border-color 0.2s ease;
  min-width: 100px;
}

.node:hover {
  border-color: rgba(255, 255, 255, 0.28);
  filter: var(--glow-primary);
}
```

### Cylinder Node (`.node--cylinder`) — Databases

```css
.node--cylinder {
  border-radius: 8px;
  padding-top: 28px;
  padding-bottom: 28px;
}

.node--cylinder::before,
.node--cylinder::after {
  content: '';
  position: absolute;
  left: 0;
  width: 100%;
  height: 20px;
  background: var(--node-bg);
  border-left: 1px solid var(--node-border);
  border-right: 1px solid var(--node-border);
}

.node--cylinder::before {
  top: -10px;
  border-radius: 50%;
  border: 1px solid var(--node-border);
}

.node--cylinder::after {
  bottom: -10px;
  border-radius: 50%;
  border: 1px solid var(--node-border);
  background: rgba(255, 255, 255, 0.03);
}
```

### Hexagon Node (`.node--hexagon`) — Message Queues

```css
.node--hexagon {
  border-radius: 0;
  border: none;
  background: transparent;
  padding: 0;
  width: 90px;
  height: 80px;
}

.node--hexagon::before {
  content: '';
  position: absolute;
  inset: 0;
  background: var(--node-bg);
  border: 1px solid var(--node-border);
  clip-path: polygon(50% 0%, 100% 25%, 100% 75%, 50% 100%, 0% 75%, 0% 25%);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  z-index: -1;
}
```

### Circle Node (`.node--circle`) — Actors / External

```css
.node--circle {
  border-radius: 50%;
  width: 80px;
  height: 80px;
  padding: 8px;
}
```

### Stadium Node (`.node--stadium`) — Load Balancers / Gateways

```css
.node--stadium {
  border-radius: 999px;
  padding: 12px 28px;
}
```

### Node Inner Elements

Each node contains an icon (28x28 SVG), a label (14px semibold), and a subtitle (11px muted).

```css
.node-icon {
  width: 28px;
  height: 28px;
  margin-bottom: 6px;
  flex-shrink: 0;
}

.node-label {
  font-size: 14px;
  font-weight: 600;
  color: var(--node-text);
  line-height: 1.3;
}

.node-subtitle {
  font-size: 11px;
  color: var(--node-sub);
  margin-top: 2px;
  line-height: 1.4;
}
```

---

## Edge System

Edges are rendered as SVG `<path>` elements on a full-viewport SVG overlay layer.

### Edge Layer CSS

```css
.edge-layer {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: 5;
  overflow: visible;
}
```

### Edge Path CSS

- Solid line = synchronous call
- Dashed line (`stroke-dasharray: 8 4`) = asynchronous / event-driven

```css
.edge {
  fill: none;
  stroke-width: 2;
  stroke-linecap: round;
  stroke-linejoin: round;
}

.edge--sync {
  stroke: var(--edge-primary);
  stroke-dasharray: none;
}

.edge--secondary {
  stroke: var(--edge-secondary);
}

.edge--async {
  stroke: var(--edge-async);
  stroke-dasharray: 8 4;
}

.edge--error {
  stroke: var(--edge-error);
  stroke-dasharray: 6 3;
}
```

### SVG Marker Defs (Arrowheads) and Glow Filters

Place inside the `<svg>` `<defs>` block — one arrow marker per color variant, plus glow filters:

```html
<svg class="edge-layer" xmlns="http://www.w3.org/2000/svg">
  <defs>
    <filter id="glow" x="-50%" y="-50%" width="200%" height="200%">
      <feGaussianBlur stdDeviation="3" result="blur"/>
      <feMerge>
        <feMergeNode in="blur"/>
        <feMergeNode in="SourceGraphic"/>
      </feMerge>
    </filter>

    <marker id="arrow-primary" viewBox="0 0 10 8" refX="10" refY="4"
            markerWidth="8" markerHeight="6" orient="auto-start-reverse">
      <path d="M0 0 L10 4 L0 8 z" fill="var(--edge-primary)"/>
    </marker>

    <marker id="arrow-secondary" viewBox="0 0 10 8" refX="10" refY="4"
            markerWidth="8" markerHeight="6" orient="auto-start-reverse">
      <path d="M0 0 L10 4 L0 8 z" fill="var(--edge-secondary)"/>
    </marker>

    <marker id="arrow-async" viewBox="0 0 10 8" refX="10" refY="4"
            markerWidth="8" markerHeight="6" orient="auto-start-reverse">
      <path d="M0 0 L10 4 L0 8 z" fill="var(--edge-async)"/>
    </marker>

    <marker id="arrow-error" viewBox="0 0 10 8" refX="10" refY="4"
            markerWidth="8" markerHeight="6" orient="auto-start-reverse">
      <path d="M0 0 L10 4 L0 8 z" fill="var(--edge-error)"/>
    </marker>
  </defs>
</svg>
```

### Edge Routing — Cubic Bezier

Edges are routed with cubic bezier curves. Control points are placed at 40% and 60% of the horizontal (or vertical) distance between nodes.

For horizontal (left-to-right) flow:
- Start anchor: center-right of source node
- End anchor: center-left of target node
- `M srcX srcY C cp1X cp1Y, cp2X cp2Y, dstX dstY`

For vertical (top-to-bottom) flow:
- Start anchor: center-bottom of source node
- End anchor: center-top of target node

**Horizontal line fix:** When source and target share the same anchor coordinate on the cross-axis (e.g. `src.y === dst.y` in horizontal flow), the bezier path has a zero-height bounding box. SVG percentage-based filter regions (like the glow filter) collapse to 0, making the stroke invisible. Fix: nudge control points ±0.5px on the cross-axis so the bounding box always has non-zero height.

---

## Animations

### Keyframes

All 4 animation types:

```css
/* 1. Nodes entering the scene */
@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* 2. Edge draw-on using stroke-dashoffset */
@keyframes drawLine {
  from {
    stroke-dashoffset: var(--path-length);
  }
  to {
    stroke-dashoffset: 0;
  }
}

/* 3. Pulsing glow for active/highlighted nodes */
@keyframes pulse {
  0%, 100% {
    filter: drop-shadow(0 0 4px rgba(99, 102, 241, 0.4))
            drop-shadow(0 0 12px rgba(99, 102, 241, 0.2));
  }
  50% {
    filter: drop-shadow(0 0 12px rgba(99, 102, 241, 0.9))
            drop-shadow(0 0 32px rgba(99, 102, 241, 0.5))
            drop-shadow(0 0 64px rgba(99, 102, 241, 0.2));
  }
}

/* 4. Group fade-in */
@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}
```

### Applying Animations

```css
/* Nodes — start invisible, JS sets animation-delay per node */
.node {
  opacity: 0;
  animation: fadeInUp 1s ease-out forwards;
  pointer-events: auto;
}

/* Node layer — pass events through to edge layer below */
.node-layer {
  pointer-events: none;
  z-index: 6;
}

/* Edges — stroke-dasharray set to path length in JS */
.edge {
  stroke-dasharray: var(--path-length);
  stroke-dashoffset: var(--path-length);
  animation: drawLine 1.8s ease-in-out forwards;
}

/* Pulsing nodes — must set opacity: 1 so pulse doesn't hide them */
.node.is-pulsing {
  opacity: 1;
  animation: pulse 4s ease-in-out infinite;
}

/* Groups — appear first */
.group {
  opacity: 0;
  animation: fadeIn 0.8s ease-out forwards;
}
```

### Animation Timeline Constants

```js
const GROUP_FADE_DURATION     = 800;   // ms
const NODE_STAGGER            = 150;   // ms per node
const NODE_ENTRANCE_DURATION  = 1000;  // ms
const EDGE_DRAW_DURATION      = 1800;  // ms
const EDGE_STAGGER            = 250;   // ms per edge
const PARTICLE_DURATION       = 3500;  // ms (one full pass)
const PARTICLE_COUNT_PER_EDGE = 3;
const PARTICLE_STAGGER        = 1200;  // ms between particles on same edge
const PULSE_DURATION          = 4000;  // ms
```

### Timeline Orchestration

1. **t=0** — Groups fade in (800ms)
2. **t=800** — Nodes stagger in (node_i at `800 + i * 150` ms)
3. **t=nodes_done** — Edges draw and particles start simultaneously
4. **t=edges_done** — Active nodes begin pulsing

---

## Interactivity

The generated HTML must include these 5 interactive features:

### 1. Hover Highlight (Nodes)

When hovering a node, dim all unconnected nodes and edges to `0.15` opacity. Keep the hovered node and its direct connections at full opacity. Use CSS classes `is-dimmed` and `is-highlighted` instead of inline styles.

**Critical layering**: The `.node-layer` must have `pointer-events: none` with `pointer-events: auto` on individual `.node` elements. This lets mouse events pass through empty space to reach the edge layer below.

```css
.node.is-dimmed { opacity: 0.15 !important; filter: none !important; transition: opacity 0.3s; }
.node.is-highlighted { opacity: 1 !important; transition: opacity 0.3s; }
.edge-layer path.is-dimmed { opacity: 0.1; transition: opacity 0.3s; }
```

```js
let hoveredNodeId = null;

function clearHighlight() {
  hoveredNodeId = null;
  Object.values(nodeElements).forEach(el => {
    el.classList.remove('is-dimmed', 'is-highlighted');
  });
  pathObjs.forEach(obj => obj.path.classList.remove('is-dimmed'));
  tooltip.classList.remove('is-active');
}

function applyHighlight(nodeId) {
  if (hoveredNodeId === nodeId) return;
  hoveredNodeId = nodeId;
  const connectedIds = new Set([nodeId]);
  const connectedPaths = [];
  pathObjs.forEach(obj => {
    if (obj.edge.from === nodeId || obj.edge.to === nodeId) {
      connectedIds.add(obj.edge.from);
      connectedIds.add(obj.edge.to);
      connectedPaths.push(obj.path);
    }
  });
  Object.values(nodeElements).forEach(el => {
    const c = connectedIds.has(el.dataset.nodeId);
    el.classList.toggle('is-dimmed', !c);
    el.classList.toggle('is-highlighted', c);
  });
  pathObjs.forEach(obj => {
    obj.path.classList.toggle('is-dimmed', !connectedPaths.includes(obj.path));
  });
}
```

### 2. Node Tooltip on Hover

Glassmorphism panel showing tech stack, port, and description.

```css
.tooltip {
  position: fixed;
  z-index: 200;
  max-width: 340px;
  padding: 14px 16px;
  background: rgba(15, 15, 30, 0.92);
  border: 1px solid rgba(255, 255, 255, 0.12);
  border-radius: 12px;
  backdrop-filter: blur(16px);
  -webkit-backdrop-filter: blur(16px);
  pointer-events: none;
  opacity: 0;
  transition: opacity 0.2s;
  font-size: 12px;
  color: var(--node-text);
  box-shadow: 0 8px 32px rgba(0,0,0,0.5);
}
.tooltip.is-active { opacity: 1; }
```

### 3. Edge Tooltip on Hover (Request/Response Data)

**This is a critical feature.** When the user hovers near an edge line, show a tooltip with the method, endpoint, request body, and response data. Use proximity-based hit detection since SVG pointer-events on paths are unreliable across browsers.

```css
.tooltip-method {
  display: inline-block;
  padding: 2px 6px;
  border-radius: 4px;
  font-size: 10px;
  font-weight: 700;
  font-family: 'SF Mono', 'Consolas', monospace;
  margin-right: 6px;
}
.tooltip-method--get    { background: rgba(6,182,212,0.2); color: #06b6d4; }
.tooltip-method--post   { background: rgba(99,102,241,0.2); color: #818cf8; }
.tooltip-method--select { background: rgba(6,182,212,0.2); color: #06b6d4; }
.tooltip-method--insert { background: rgba(168,85,247,0.2); color: #c084fc; }
.tooltip-method--publish{ background: rgba(245,158,11,0.2); color: #f59e0b; }

.tooltip-data {
  margin-top: 8px;
  padding: 8px 10px;
  background: rgba(0,0,0,0.4);
  border-radius: 6px;
  font-family: 'SF Mono', 'Consolas', monospace;
  font-size: 11px;
  line-height: 1.6;
}
.tooltip-data-label { font-size: 9px; text-transform: uppercase; letter-spacing: 0.08em; color: var(--node-sub); }
.tooltip-data-value { color: #a5b4fc; }
.tooltip-data-response { color: #6ee7b7; }
```

```js
const EDGE_HIT_DISTANCE = 14;
let hoveredEdgeIndex = null;

function findClosestEdge(mx, my) {
  const vpRect = viewport.getBoundingClientRect();
  const localX = (mx - vpRect.left) / scale;
  const localY = (my - vpRect.top) / scale;
  let closest = null, minDist = Infinity;
  pathObjs.forEach((obj, i) => {
    const path = obj.path;
    const len = path.getTotalLength();
    const steps = Math.max(10, Math.round(len / 20));
    for (let s = 0; s <= steps; s++) {
      const pt = path.getPointAtLength((s / steps) * len);
      const d = Math.hypot(pt.x - localX, pt.y - localY);
      if (d < minDist) { minDist = d; closest = { index: i, dist: d }; }
    }
  });
  return closest && closest.dist < EDGE_HIT_DISTANCE / scale ? closest : null;
}

function buildEdgeTooltip(edge) {
  const from = nodeMap[edge.from], to = nodeMap[edge.to];
  const cls = 'tooltip-method--' + (edge.method || '').toLowerCase();
  return `
    <div style="margin-bottom:8px;">
      <span style="color:var(--node-sub);font-size:11px;">${from.label} → ${to.label}</span>
    </div>
    <div style="margin-bottom:8px;">
      <span class="tooltip-method ${cls}">${edge.method}</span>
      <span style="font-family:monospace;color:#fff;">${edge.endpoint}</span>
    </div>
    <div class="tooltip-data">
      <div class="tooltip-data-label">→ Request</div>
      <div class="tooltip-data-value">${edge.request || '—'}</div>
      <div style="margin-top:6px;" class="tooltip-data-label">← Response</div>
      <div class="tooltip-data-value tooltip-data-response">${edge.response || '—'}</div>
    </div>`;
}

// Listen on document mousemove, skip if over a node
document.addEventListener('mousemove', (e) => {
  const elUnder = document.elementFromPoint(e.clientX, e.clientY);
  if (elUnder && elUnder.closest && elUnder.closest('.node')) return;
  const hit = findClosestEdge(e.clientX, e.clientY);
  if (hit) {
    if (hoveredEdgeIndex === hit.index) { /* update tooltip position */ return; }
    hoveredEdgeIndex = hit.index;
    const edge = edges[hit.index];
    tooltip.innerHTML = buildEdgeTooltip(edge);
    tooltip.classList.add('is-active');
    // highlight connected nodes + this edge, dim rest
  } else if (hoveredEdgeIndex !== null) {
    hoveredEdgeIndex = null;
    clearHighlight();
  }
});
```

### 4. Zoom / Pan

Mouse wheel zooms toward cursor position. Click-drag pans. CSS transform on the diagram container.

```css
.diagram-viewport {
  position: fixed;
  inset: 0;
  overflow: hidden;
  z-index: 5;
  cursor: grab;
}

.diagram-viewport.is-panning {
  cursor: grabbing;
}

.diagram {
  position: relative;
  transform-origin: 0 0;
  will-change: transform;
}
```

```js
const viewport = document.querySelector('.diagram-viewport');
const diagram  = document.querySelector('.diagram');

let scale = 1, originX = 0, originY = 0;
let isPanning = false, startX, startY, startOriginX, startOriginY;
const MIN_SCALE = 0.25, MAX_SCALE = 4, ZOOM_SPEED = 0.001;

function applyTransform() {
  diagram.style.transform = `translate(${originX}px, ${originY}px) scale(${scale})`;
}

/**
 * Scales and centers the diagram to fill the viewport.
 * Computes the true bounding box of all .node elements, then applies
 * a uniform scale with padding so content fills the available space.
 */
function fitToViewport() {
  const FIT_PADDING = 80;
  const vw = viewport.clientWidth;
  const vh = viewport.clientHeight;
  const nodeEls = diagram.querySelectorAll('.node');
  let minX = Infinity, minY = Infinity, maxX = 0, maxY = 0;
  nodeEls.forEach(el => {
    const x = parseFloat(el.style.left);
    const y = parseFloat(el.style.top);
    const w = el.offsetWidth;
    const h = el.offsetHeight;
    if (x < minX) minX = x;
    if (y < minY) minY = y;
    if (x + w > maxX) maxX = x + w;
    if (y + h > maxY) maxY = y + h;
  });
  const contentW = maxX - minX + FIT_PADDING * 2;
  const contentH = maxY - minY + FIT_PADDING * 2;
  scale = Math.min(vw / contentW, vh / contentH);
  scale = Math.max(MIN_SCALE, Math.min(scale, MAX_SCALE));
  originX = (vw - contentW * scale) / 2 - (minX - FIT_PADDING) * scale;
  originY = (vh - contentH * scale) / 2 - (minY - FIT_PADDING) * scale;
  applyTransform();
}

viewport.addEventListener('wheel', (e) => {
  e.preventDefault();
  const delta = -e.deltaY * ZOOM_SPEED;
  const newScale = Math.min(MAX_SCALE, Math.max(MIN_SCALE, scale + delta * scale));
  const rect = viewport.getBoundingClientRect();
  const mouseX = e.clientX - rect.left;
  const mouseY = e.clientY - rect.top;
  originX = mouseX - (mouseX - originX) * (newScale / scale);
  originY = mouseY - (mouseY - originY) * (newScale / scale);
  scale = newScale;
  applyTransform();
}, { passive: false });

viewport.addEventListener('mousedown', (e) => {
  if (e.button !== 0) return;
  isPanning = true;
  startX = e.clientX; startY = e.clientY;
  startOriginX = originX; startOriginY = originY;
  viewport.classList.add('is-panning');
});

window.addEventListener('mousemove', (e) => {
  if (!isPanning) return;
  originX = startOriginX + (e.clientX - startX);
  originY = startOriginY + (e.clientY - startY);
  applyTransform();
});

window.addEventListener('mouseup', () => {
  isPanning = false;
  viewport.classList.remove('is-panning');
});
```

### 5. Double-Click Reset

```js
viewport.addEventListener('dblclick', fitToViewport);
fitToViewport(); // Scale to fill viewport on load
window.addEventListener('resize', fitToViewport);
```

---

## Layout Algorithm

The auto-layout JS algorithm positions nodes and routes edges automatically.

### Layout Constants

```js
const LAYOUT = {
  NODE_WIDTH: 180,
  NODE_HEIGHT: 80,
  LAYER_GAP: 200,
  NODE_GAP: 40,
  PADDING: 60,
  GROUP_PADDING: 32
};
```

### Data Structures

```js
// Node: { id, label, subtitle, icon, shape, group? }
// Edge: { from, to, label?, type: 'sync'|'async'|'secondary'|'error' }
// Group: { id, label, children: [nodeIds] }
```

### Algorithm Steps

#### 1. Topological Sort (Kahn's Algorithm)

```js
function topologicalSort(nodes, edges) {
  const ids = nodes.map(n => n.id);
  const inDegree = new Map(ids.map(id => [id, 0]));
  const adj = new Map(ids.map(id => [id, []]));
  const safeEdges = breakCycles(nodes, edges);

  for (const { from, to } of safeEdges) {
    if (!adj.has(from) || !adj.has(to)) continue;
    adj.get(from).push(to);
    inDegree.set(to, inDegree.get(to) + 1);
  }

  const queue = ids.filter(id => inDegree.get(id) === 0);
  const result = [];

  while (queue.length > 0) {
    const node = queue.shift();
    result.push(node);
    for (const neighbor of adj.get(node) ?? []) {
      const deg = inDegree.get(neighbor) - 1;
      inDegree.set(neighbor, deg);
      if (deg === 0) queue.push(neighbor);
    }
  }

  const visited = new Set(result);
  for (const id of ids) {
    if (!visited.has(id)) result.push(id);
  }
  return result;
}

function breakCycles(nodes, edges) {
  const color = new Map(nodes.map(n => [n.id, 0]));
  const adj = new Map(nodes.map(n => [n.id, []]));
  const backEdges = new Set();

  for (const e of edges) {
    if (adj.has(e.from) && adj.has(e.to)) adj.get(e.from).push(e);
  }

  function dfs(id) {
    color.set(id, 1);
    for (const edge of adj.get(id) ?? []) {
      if (color.get(edge.to) === 1) backEdges.add(`${edge.from}->${edge.to}`);
      else if (color.get(edge.to) === 0) dfs(edge.to);
    }
    color.set(id, 2);
  }

  for (const { id } of nodes) {
    if (color.get(id) === 0) dfs(id);
  }
  return edges.filter(e => !backEdges.has(`${e.from}->${e.to}`));
}
```

#### 2. Layer Assignment (Longest Path)

```js
function assignLayers(nodes, edges) {
  const safeEdges = breakCycles(nodes, edges);
  const order = topologicalSort(nodes, safeEdges);
  const layer = new Map(nodes.map(n => [n.id, 0]));

  const predecessors = new Map(nodes.map(n => [n.id, []]));
  for (const { from, to } of safeEdges) {
    if (predecessors.has(to)) predecessors.get(to).push(from);
  }

  for (const id of order) {
    const preds = predecessors.get(id) ?? [];
    if (preds.length === 0) {
      layer.set(id, 0);
    } else {
      layer.set(id, Math.max(...preds.map(p => layer.get(p) ?? 0)) + 1);
    }
  }
  return layer;
}
```

#### 3. Crossing Minimization (Barycenter, 3 Passes)

```js
function minimizeCrossings(nodes, edges, layerMap) {
  const safeEdges = breakCycles(nodes, edges);
  const layerNodes = new Map();
  for (const { id } of nodes) {
    const l = layerMap.get(id) ?? 0;
    if (!layerNodes.has(l)) layerNodes.set(l, []);
    layerNodes.get(l).push(id);
  }

  const maxLayer = Math.max(...layerMap.values());
  const successors   = new Map(nodes.map(n => [n.id, []]));
  const predecessors = new Map(nodes.map(n => [n.id, []]));
  for (const { from, to } of safeEdges) {
    if (successors.has(from))   successors.get(from).push(to);
    if (predecessors.has(to))   predecessors.get(to).push(from);
  }

  function barycenter(id, refOrder, direction) {
    const neighbors = direction === 'pred'
      ? predecessors.get(id) ?? []
      : successors.get(id) ?? [];
    const positions = neighbors.map(n => refOrder.indexOf(n)).filter(i => i >= 0);
    if (positions.length === 0) return Infinity;
    return positions.reduce((a, b) => a + b, 0) / positions.length;
  }

  for (let pass = 0; pass < 3; pass++) {
    for (let l = 1; l <= maxLayer; l++) {
      const current = layerNodes.get(l) ?? [];
      const prev = layerNodes.get(l - 1) ?? [];
      current.sort((a, b) => barycenter(a, prev, 'pred') - barycenter(b, prev, 'pred'));
      layerNodes.set(l, current);
    }
    for (let l = maxLayer - 1; l >= 0; l--) {
      const current = layerNodes.get(l) ?? [];
      const next = layerNodes.get(l + 1) ?? [];
      current.sort((a, b) => barycenter(a, next, 'succ') - barycenter(b, next, 'succ'));
      layerNodes.set(l, current);
    }
  }
  return layerNodes;
}
```

#### 4. Position Assignment

```js
function assignPositions(nodes, layerNodes, layout = LAYOUT) {
  const { NODE_WIDTH, NODE_HEIGHT, LAYER_GAP, NODE_GAP, PADDING } = layout;
  const positions = new Map();

  for (const [layerIndex, ids] of layerNodes) {
    const x = PADDING + layerIndex * (NODE_WIDTH + LAYER_GAP);
    const totalH = ids.length * NODE_HEIGHT + (ids.length - 1) * NODE_GAP;
    const startY = -(totalH / 2);

    ids.forEach((id, rank) => {
      positions.set(id, { x, y: startY + rank * (NODE_HEIGHT + NODE_GAP) });
    });
  }
  return positions;
}
```

#### 5. Edge Routing (Cubic Bezier)

```js
function routeEdge(edge, positions, allEdges, flow = 'horizontal', layout = LAYOUT) {
  const { from, to } = edge;

  // Self-loop
  if (from === to) {
    const pos = positions.get(from);
    const cx = pos.x + layout.NODE_WIDTH / 2;
    const top = pos.y;
    return `M ${cx - 20} ${top} A 30 20 0 1 0 ${cx + 20} ${top}`;
  }

  const srcPos = positions.get(from);
  const dstPos = positions.get(to);
  if (!srcPos || !dstPos) return '';

  // Bidirectional offset
  const isBidi = allEdges.some(e => e.from === to && e.to === from);
  const offset = isBidi ? (from < to ? -4 : 4) : 0;

  if (flow === 'horizontal') {
    const src = { x: srcPos.x + layout.NODE_WIDTH, y: srcPos.y + layout.NODE_HEIGHT / 2 + offset };
    const dst = { x: dstPos.x, y: dstPos.y + layout.NODE_HEIGHT / 2 + offset };
    const dx = dst.x - src.x;
    // Nudge control points ±0.5px when line is horizontal to prevent zero-height
    // bounding box which collapses %-based SVG filter regions (glow becomes invisible)
    const cp1Y = (src.y === dst.y) ? src.y - 0.5 : src.y;
    const cp2Y = (src.y === dst.y) ? dst.y + 0.5 : dst.y;
    return `M ${src.x} ${src.y} C ${src.x + dx * 0.4} ${cp1Y}, ${dst.x - dx * 0.4} ${cp2Y}, ${dst.x} ${dst.y}`;
  } else {
    const src = { x: srcPos.x + layout.NODE_WIDTH / 2 + offset, y: srcPos.y + layout.NODE_HEIGHT };
    const dst = { x: dstPos.x + layout.NODE_WIDTH / 2 + offset, y: dstPos.y };
    const dy = dst.y - src.y;
    // Nudge control points ±0.5px when line is vertical to prevent zero-width
    // bounding box which collapses %-based SVG filter regions (glow becomes invisible)
    const cp1X = (src.x === dst.x) ? src.x - 0.5 : src.x;
    const cp2X = (src.x === dst.x) ? dst.x + 0.5 : dst.x;
    return `M ${src.x} ${src.y} C ${cp1X} ${src.y + dy * 0.4}, ${cp2X} ${dst.y - dy * 0.4}, ${dst.x} ${dst.y}`;
  }
}
```

#### 6. Viewport Fitting

After computing all positions, the `fitToViewport()` function dynamically scales and centers the diagram:
1. Find the true bounding box (minX/minY/maxX/maxY) of all node positions
2. Compute uniform scale so content fills the viewport with 80px padding
3. Translate so the content is centered — accounts for content offset from origin
4. Responds to window resize events

---

## Group / Container Styles

```css
.group {
  position: absolute;
  border: 1px dashed rgba(255, 255, 255, 0.15);
  border-radius: 16px;
  padding: 24px;
  background: rgba(255, 255, 255, 0.02);
  z-index: 2;
}

.group-label {
  position: absolute;
  top: -10px;
  left: 16px;
  font-size: 10px;
  font-weight: 700;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  color: var(--node-sub);
  background: var(--bg);
  padding: 0 6px;
  white-space: nowrap;
}
```

---

## Particle System

Particles are SVG `<circle>` elements animated along edge paths via `getPointAtLength()` + `requestAnimationFrame`. Each edge spawns its particles after its draw animation finishes, creating a staggered effect.

### Particle Layer CSS

```css
.particle-layer {
  position: absolute;
  inset: 0;
  z-index: 4;
  pointer-events: none;
  overflow: visible;
}
```

### Spawning Particles Per Edge (JS)

Particles spawn per-edge, staggered after each edge's draw animation finishes:

```js
/** All active particles — updated by a single rAF loop */
const particles = [];

/**
 * Spawns particle dots for a single edge path.
 * @param {Object} obj - pathObj with path element and edge data
 */
function spawnParticlesForEdge(obj) {
  const particleSvg = document.getElementById('particles');
  const colorVar = obj.edge.color === 'primary' ? '#6366f1' :
                   obj.edge.color === 'secondary' ? '#06b6d4' :
                   obj.edge.color === 'async' ? '#f59e0b' :
                   '#ef4444';
  const totalLength = obj.path.getTotalLength();

  for (let p = 0; p < PARTICLE_COUNT_PER_EDGE; p++) {
    const circle = document.createElementNS('http://www.w3.org/2000/svg', 'circle');
    circle.setAttribute('r', '3');
    circle.setAttribute('fill', colorVar);
    circle.setAttribute('opacity', '0');
    particleSvg.appendChild(circle);

    particles.push({ circle, path: obj.path, totalLength, offset: p * PARTICLE_STAGGER });
  }
}

/**
 * Single rAF loop that updates every particle each frame.
 * Avoids N independent loops which cause jank on large diagrams.
 */
function tickParticles(timestamp) {
  for (const p of particles) {
    const elapsed = (timestamp - p.offset) % PARTICLE_DURATION;
    if (elapsed < 0) continue;            // still in initial stagger
    const progress = ((elapsed % PARTICLE_DURATION) + PARTICLE_DURATION) % PARTICLE_DURATION / PARTICLE_DURATION;
    const point = p.path.getPointAtLength(progress * p.totalLength);
    p.circle.setAttribute('cx', point.x);
    p.circle.setAttribute('cy', point.y);
    p.circle.setAttribute('opacity', '0.85');
  }
  requestAnimationFrame(tickParticles);
}
```

### Particle Spawning

Particles spawn immediately and a single rAF loop drives all of them:

```js
pathObjs.forEach((obj) => { spawnParticlesForEdge(obj); });
requestAnimationFrame(tickParticles);
```

---

## Legend Styles

```css
.legend {
  position: fixed;
  bottom: 24px;
  right: 24px;
  z-index: 50;
  padding: 14px 18px;
  background: rgba(10, 10, 18, 0.85);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 12px;
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  font-size: 11px;
  color: var(--node-sub);
  min-width: 160px;
}

.legend-title {
  font-size: 10px;
  font-weight: 700;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: var(--node-text);
  margin-bottom: 10px;
}

.legend-item {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 7px;
}

.legend-item:last-child {
  margin-bottom: 0;
}

.legend-line {
  flex-shrink: 0;
  width: 28px;
  height: 2px;
}

.legend-line--sync { background: var(--edge-primary); }
.legend-line--secondary { background: var(--edge-secondary); }
.legend-line--async {
  background: repeating-linear-gradient(90deg,
    var(--edge-async) 0px, var(--edge-async) 8px,
    transparent 8px, transparent 12px);
}
.legend-line--error {
  background: repeating-linear-gradient(90deg,
    var(--edge-error) 0px, var(--edge-error) 6px,
    transparent 6px, transparent 9px);
}
```

### Legend HTML

```html
<aside class="legend">
  <div class="legend-title">Edge Types</div>
  <div class="legend-item">
    <div class="legend-line legend-line--sync"></div>
    <span>Sync / RPC</span>
  </div>
  <div class="legend-item">
    <div class="legend-line legend-line--secondary"></div>
    <span>Data / Stream</span>
  </div>
  <div class="legend-item">
    <div class="legend-line legend-line--async"></div>
    <span>Async / Event</span>
  </div>
  <div class="legend-item">
    <div class="legend-line legend-line--error"></div>
    <span>Error / DLQ</span>
  </div>
</aside>
```

---

## Icon System

A JavaScript ICONS object mapping 20 icon names to their Lucide-style SVG inner content (24x24 viewBox, 2px stroke, round caps/joins, no fill).

```js
const ICONS = {
  'server':     '<rect x="2" y="2" width="20" height="8" rx="2" ry="2"/><rect x="2" y="14" width="20" height="8" rx="2" ry="2"/><line x1="6" y1="6" x2="6.01" y2="6"/><line x1="6" y1="18" x2="6.01" y2="18"/>',
  'database':   '<ellipse cx="12" cy="5" rx="9" ry="3"/><path d="M21 12c0 1.66-4 3-9 3s-9-1.34-9-3"/><path d="M3 5v14c0 1.66 4 3 9 3s9-1.34 9-3V5"/>',
  'globe':      '<circle cx="12" cy="12" r="10"/><line x1="2" y1="12" x2="22" y2="12"/><path d="M12 2a15.3 15.3 0 0 1 4 10 15.3 15.3 0 0 1-4 10 15.3 15.3 0 0 1-4-10 15.3 15.3 0 0 1 4-10z"/>',
  'user':       '<path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"/><circle cx="12" cy="7" r="4"/>',
  'cloud':      '<path d="M18 10h-1.26A8 8 0 1 0 9 20h9a5 5 0 0 0 0-10z"/>',
  'lock':       '<rect x="3" y="11" width="18" height="11" rx="2" ry="2"/><path d="M7 11V7a5 5 0 0 1 10 0v4"/>',
  'mail':       '<path d="M4 4h16c1.1 0 2 .9 2 2v12c0 1.1-.9 2-2 2H4c-1.1 0-2-.9-2-2V6c0-1.1.9-2 2-2z"/><polyline points="22,6 12,13 2,6"/>',
  'zap':        '<polygon points="13 2 3 14 12 14 11 22 21 10 12 10 13 2"/>',
  'cpu':        '<rect x="9" y="9" width="6" height="6"/><rect x="5" y="5" width="14" height="14" rx="2" ry="2"/><line x1="9" y1="1" x2="9" y2="5"/><line x1="15" y1="1" x2="15" y2="5"/><line x1="9" y1="19" x2="9" y2="23"/><line x1="15" y1="19" x2="15" y2="23"/><line x1="1" y1="9" x2="5" y2="9"/><line x1="1" y1="15" x2="5" y2="15"/><line x1="19" y1="9" x2="23" y2="9"/><line x1="19" y1="15" x2="23" y2="15"/>',
  'hard-drive': '<line x1="22" y1="12" x2="2" y2="12"/><path d="M5.45 5.11L2 12v6a2 2 0 0 0 2 2h16a2 2 0 0 0 2-2v-6l-3.45-6.89A2 2 0 0 0 16.76 4H7.24a2 2 0 0 0-1.79 1.11z"/><line x1="6" y1="16" x2="6.01" y2="16"/><line x1="10" y1="16" x2="10.01" y2="16"/>',
  'git-branch': '<line x1="6" y1="3" x2="6" y2="15"/><circle cx="18" cy="6" r="3"/><circle cx="6" cy="18" r="3"/><path d="M18 9a9 9 0 0 1-9 9"/>',
  'shield':     '<path d="M12 22s8-4 8-10V5l-8-3-8 3v7c0 6 8 10 8 10z"/>',
  'activity':   '<polyline points="22 12 18 12 15 21 9 3 6 12 2 12"/>',
  'layers':     '<polygon points="12 2 2 7 12 12 22 7 12 2"/><polyline points="2 17 12 22 22 17"/><polyline points="2 12 12 17 22 12"/>',
  'smartphone': '<rect x="5" y="2" width="14" height="20" rx="2" ry="2"/><line x1="12" y1="18" x2="12.01" y2="18"/>',
  'terminal':   '<polyline points="4 17 10 11 4 5"/><line x1="12" y1="19" x2="20" y2="19"/>',
  'box':        '<path d="M21 16V8a2 2 0 0 0-1-1.73l-7-4a2 2 0 0 0-2 0l-7 4A2 2 0 0 0 3 8v8a2 2 0 0 0 1 1.73l7 4a2 2 0 0 0 2 0l7-4A2 2 0 0 0 21 16z"/><polyline points="3.27 6.96 12 12.01 20.73 6.96"/><line x1="12" y1="22.08" x2="12" y2="12"/>',
  'wifi':       '<path d="M5 12.55a11 11 0 0 1 14.08 0"/><path d="M1.42 9a16 16 0 0 1 21.16 0"/><path d="M8.53 16.11a6 6 0 0 1 6.95 0"/><line x1="12" y1="20" x2="12.01" y2="20"/>',
  'key':        '<path d="M21 2l-2 2m-7.61 7.61a5.5 5.5 0 1 1-7.778 7.778 5.5 5.5 0 0 1 7.777-7.777zm0 0L15.5 7.5m0 0l3 3L22 7l-3-3m-3.5 3.5L19 4"/>',
  'refresh-cw': '<polyline points="23 4 23 10 17 10"/><polyline points="1 20 1 14 7 14"/><path d="M3.51 9a9 9 0 0 1 14.85-3.36L23 10M1 14l4.64 4.36A9 9 0 0 0 20.49 15"/>',
};

function getIcon(name, size = 28) {
  return `<svg width="${size}" height="${size}" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">${ICONS[name] ?? ''}</svg>`;
}
```

---

## HTML Template

The complete HTML structure skeleton:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{Diagram Title}</title>
  <style>
    /* === 1. CSS Custom Properties === */
    /* === 2. Reset + Base + Grid Background === */
    /* === 3. Node Shapes (all 5 variants) === */
    /* === 4. Node Inner Elements === */
    /* === 5. Edge Layer + Edge Paths === */
    /* === 6. Group / Container === */
    /* === 7. Animation Keyframes (all 5) === */
    /* === 8. Animation Application (nodes, edges, groups) === */
    /* === 9. Particle Layer === */
    /* === 10. Tooltip === */
    /* === 11. Zoom/Pan Viewport === */
    /* === 12. Legend === */
    /* === 13. Print Stylesheet === */
    /* === 14. Responsive === */
  </style>
</head>
<body>
  <div class="diagram-viewport">
    <div class="diagram">
      <!-- Groups (rendered first, behind nodes) -->
      <!-- Nodes (absolute positioned) -->

      <!-- SVG edge layer -->
      <svg class="edge-layer" xmlns="http://www.w3.org/2000/svg">
        <defs>
          <!-- Arrow markers + glow filters -->
        </defs>
        <!-- Edge <path> elements -->
      </svg>

      <!-- Particle layer (SVG circles animated via JS) -->
      <svg id="particles" class="particle-layer"></svg>
    </div>
  </div>

  <!-- Tooltip (fixed, outside viewport for correct positioning) -->
  <div class="tooltip"></div>

  <!-- Legend -->
  <aside class="legend">
    <!-- Legend items -->
  </aside>

  <script>
    // 1. ICONS object + getIcon()
    // 2. Layout constants + algorithm (topoSort, layers, crossings, positions)
    // 3. Define nodes, edges, groups arrays
    // 4. Compute layout
    // 5. Render nodes, groups, edges to DOM
    // 6. Compute edge path lengths for animations
    // 7. Set animation delays (orchestration timeline)
    // 8. Spawn particles per-edge (SVG circles + requestAnimationFrame)
    // 9. Setup hover highlight + tooltips
    // 10. Setup zoom/pan + double-click reset
  </script>
</body>
</html>
```

---

## Print Stylesheet

```css
@media print {
  body {
    background: #ffffff;
    overflow: visible;
    color: #000000;
  }
  body::after { display: none; }
  .diagram-viewport { position: static; overflow: visible; }
  .diagram { transform: none !important; }
  .node {
    background: #f8f8f8;
    border: 1px solid #cccccc;
    backdrop-filter: none;
    -webkit-backdrop-filter: none;
    animation: none;
    opacity: 1;
    filter: none;
    color: #000000;
  }
  .node-label { color: #111111; }
  .node-subtitle { color: #555555; }
  .group {
    border-color: #aaaaaa;
    background: transparent;
    animation: none;
    opacity: 1;
  }
  .group-label { background: #ffffff; color: #555555; }
  .edge { stroke: #333333 !important; }
  .edge--error { stroke: #cc0000 !important; }
  .particle, .tooltip, .legend { display: none; }
}
```

---

## Responsive Styles

```css
@media (max-width: 768px) {
  .node {
    padding: 10px 14px;
    border-radius: 10px;
    min-width: 72px;
  }
  .node-icon { width: 22px; height: 22px; margin-bottom: 4px; }
  .node-label { font-size: 12px; }
  .node-subtitle { font-size: 10px; }
  .node--circle { width: 64px; height: 64px; }
  .node--hexagon { width: 72px; height: 64px; }
  .group { padding: 16px; border-radius: 12px; }
  .group-label { font-size: 9px; letter-spacing: 0.1em; }
  .legend {
    bottom: 12px; right: 12px;
    padding: 10px 14px;
    font-size: 10px; min-width: 130px;
  }
  .legend-title { font-size: 9px; margin-bottom: 8px; }
  .legend-line { width: 20px; }
  .tooltip { max-width: 180px; font-size: 11px; }
}
```

---

## Constraints

- **Max 30 nodes** — if more than 30 nodes are identified, warn the user and suggest splitting into multiple diagrams by domain/bounded context.
- **Zero external dependencies** — no CDN links, no `<script src>`, no `<link rel="stylesheet" href>`. Everything is inline in the single HTML file.
- **Self-referencing edges** — a node connecting to itself draws a loop arc above the node using SVG arc command.
- **Bidirectional edges** — two arrows between the same pair of nodes are offset +/-4px vertically to avoid overlap.
- **Print stylesheet** — `@media print` block removes animations, particles, tooltips, legend and switches to white background with dark text.
- **Font stack** — `system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif` (no external font loading to keep zero dependencies).
- **Responsive** — `@media (max-width: 768px)` adjustments for smaller viewports.

---

## Example — Complete Working Reference

Below is a COMPLETE, working HTML file for a 3-node diagram: **Web App -> API Server -> Database**. Save this as `.html` and open it in a browser to see a fully animated architecture diagram with all features.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Architecture Diagram</title>
  <style>
    /* === 1. CSS Custom Properties === */
    :root {
      --bg: #0a0a12;
      --bg-grid: rgba(255, 255, 255, 0.03);
      --node-bg: rgba(255, 255, 255, 0.05);
      --node-border: rgba(255, 255, 255, 0.12);
      --node-text: #e2e8f0;
      --node-sub: #94a3b8;
      --edge-primary: #6366f1;
      --edge-secondary: #06b6d4;
      --edge-async: #f59e0b;
      --edge-error: #ef4444;
      --accent-gradient: linear-gradient(135deg, #6366f1, #06b6d4);
      --glow-primary: drop-shadow(0 0 8px #6366f1) drop-shadow(0 0 24px rgba(99, 102, 241, 0.3));
    }

    /* === 2. Reset + Base === */
    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

    body {
      background-color: var(--bg);
      font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
      overflow: hidden;
      position: relative;
      width: 100vw;
      height: 100vh;
      color: var(--node-text);
    }

    body::after {
      content: '';
      position: fixed;
      inset: 0;
      background-image: radial-gradient(circle, var(--bg-grid) 1px, transparent 1px);
      background-size: 28px 28px;
      pointer-events: none;
      z-index: 0;
    }

    /* === 3. Node Layer === */
    .node-layer {
      position: absolute;
      inset: 0;
      z-index: 6;
      pointer-events: none;
    }

    .node {
      position: absolute;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      text-align: center;
      background: var(--node-bg);
      border: 1px solid var(--node-border);
      border-radius: 12px;
      padding: 16px 20px;
      backdrop-filter: blur(12px);
      -webkit-backdrop-filter: blur(12px);
      cursor: default;
      transition: filter 0.2s ease, border-color 0.2s ease, opacity 0.3s ease;
      min-width: 100px;
      opacity: 0;
      animation: fadeInUp 1s ease-out forwards;
      pointer-events: auto;
    }

    .node:hover {
      border-color: rgba(255, 255, 255, 0.28);
      filter: var(--glow-primary);
    }

    .node.is-pulsing {
      opacity: 1;
      animation: pulse 4s ease-in-out infinite;
    }

    .node.is-dimmed { opacity: 0.15 !important; filter: none !important; transition: opacity 0.3s; }
    .node.is-highlighted { opacity: 1 !important; transition: opacity 0.3s; }
    .edge-layer path.is-dimmed { opacity: 0.1; transition: opacity 0.3s; }

    .node--cylinder {
      border-radius: 8px;
      padding-top: 28px;
      padding-bottom: 28px;
    }
    .node--cylinder::before,
    .node--cylinder::after {
      content: '';
      position: absolute;
      left: 0;
      width: 100%;
      height: 20px;
      background: var(--node-bg);
      border-left: 1px solid var(--node-border);
      border-right: 1px solid var(--node-border);
    }
    .node--cylinder::before {
      top: -10px;
      border-radius: 50%;
      border: 1px solid var(--node-border);
    }
    .node--cylinder::after {
      bottom: -10px;
      border-radius: 50%;
      border: 1px solid var(--node-border);
      background: rgba(255, 255, 255, 0.03);
    }

    .node--hexagon {
      border-radius: 0;
      border: none;
      background: transparent;
      padding: 0;
      width: 90px;
      height: 80px;
    }
    .node--hexagon::before {
      content: '';
      position: absolute;
      inset: 0;
      background: var(--node-bg);
      border: 1px solid var(--node-border);
      clip-path: polygon(50% 0%, 100% 25%, 100% 75%, 50% 100%, 0% 75%, 0% 25%);
      backdrop-filter: blur(12px);
      -webkit-backdrop-filter: blur(12px);
      z-index: -1;
    }

    .node--circle {
      border-radius: 50%;
      width: 80px;
      height: 80px;
      padding: 8px;
    }

    .node--stadium {
      border-radius: 999px;
      padding: 12px 28px;
    }

    .node-icon {
      width: 28px;
      height: 28px;
      margin-bottom: 6px;
      flex-shrink: 0;
    }

    .node-label {
      font-size: 14px;
      font-weight: 600;
      color: var(--node-text);
      line-height: 1.3;
    }

    .node-subtitle {
      font-size: 11px;
      color: var(--node-sub);
      margin-top: 2px;
      line-height: 1.4;
    }

    /* === 4. Edge Styles === */
    .edge-layer {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      pointer-events: none;
      z-index: 5;
      overflow: visible;
    }

    .edge {
      fill: none;
      stroke-width: 2;
      stroke-linecap: round;
      stroke-linejoin: round;
      transition: opacity 0.2s ease;
    }

    .edge--sync { stroke: var(--edge-primary); }
    .edge--secondary { stroke: var(--edge-secondary); }
    .edge--async { stroke: var(--edge-async); stroke-dasharray: 8 4; }
    .edge--error { stroke: var(--edge-error); stroke-dasharray: 6 3; }

    .edge-label {
      font-size: 10px;
      fill: var(--node-sub);
      text-anchor: middle;
      dominant-baseline: middle;
      pointer-events: none;
    }

    /* === 5. Group Styles === */
    .group {
      position: absolute;
      border: 1px dashed rgba(255, 255, 255, 0.15);
      border-radius: 16px;
      padding: 24px;
      background: rgba(255, 255, 255, 0.02);
      z-index: 2;
      opacity: 0;
      animation: fadeIn 0.8s ease-out forwards;
    }

    .group-label {
      position: absolute;
      top: -10px;
      left: 16px;
      font-size: 10px;
      font-weight: 700;
      letter-spacing: 0.12em;
      text-transform: uppercase;
      color: var(--node-sub);
      background: var(--bg);
      padding: 0 6px;
      white-space: nowrap;
    }

    /* === 6. Animation Keyframes === */
    @keyframes fadeInUp {
      from { opacity: 0; transform: translateY(20px); }
      to { opacity: 1; transform: translateY(0); }
    }

    @keyframes drawLine {
      from { stroke-dashoffset: var(--path-length); }
      to { stroke-dashoffset: 0; }
    }

    @keyframes pulse {
      0%, 100% {
        filter: drop-shadow(0 0 4px rgba(99, 102, 241, 0.4))
                drop-shadow(0 0 12px rgba(99, 102, 241, 0.2));
      }
      50% {
        filter: drop-shadow(0 0 12px rgba(99, 102, 241, 0.9))
                drop-shadow(0 0 32px rgba(99, 102, 241, 0.5))
                drop-shadow(0 0 64px rgba(99, 102, 241, 0.2));
      }
    }

    @keyframes fadeIn {
      from { opacity: 0; }
      to { opacity: 1; }
    }

    /* === 7. Particle Layer === */
    .particle-layer {
      position: absolute;
      inset: 0;
      z-index: 4;
      pointer-events: none;
      overflow: visible;
    }

    /* === 8. Tooltip === */
    .tooltip {
      position: fixed;
      z-index: 200;
      max-width: 340px;
      padding: 14px 16px;
      background: rgba(15, 15, 30, 0.92);
      border: 1px solid rgba(255, 255, 255, 0.12);
      border-radius: 12px;
      backdrop-filter: blur(16px);
      -webkit-backdrop-filter: blur(16px);
      box-shadow: 0 8px 32px rgba(0, 0, 0, 0.5);
      pointer-events: none;
      opacity: 0;
      transition: opacity 0.2s;
      font-size: 12px;
      color: var(--node-text);
      line-height: 1.5;
    }
    .tooltip.is-active { opacity: 1; }

    .tooltip-method {
      display: inline-block;
      padding: 2px 6px;
      border-radius: 4px;
      font-size: 10px;
      font-weight: 700;
      font-family: 'SF Mono', 'Consolas', monospace;
      margin-right: 6px;
    }
    .tooltip-method--get    { background: rgba(6,182,212,0.2); color: #06b6d4; }
    .tooltip-method--post   { background: rgba(99,102,241,0.2); color: #818cf8; }
    .tooltip-method--select { background: rgba(6,182,212,0.2); color: #06b6d4; }
    .tooltip-method--insert { background: rgba(168,85,247,0.2); color: #c084fc; }
    .tooltip-method--publish{ background: rgba(245,158,11,0.2); color: #f59e0b; }

    .tooltip-data {
      margin-top: 8px;
      padding: 8px 10px;
      background: rgba(0,0,0,0.4);
      border-radius: 6px;
      font-family: 'SF Mono', 'Consolas', monospace;
      font-size: 11px;
      line-height: 1.6;
    }
    .tooltip-data-label { font-size: 9px; text-transform: uppercase; letter-spacing: 0.08em; color: var(--node-sub); }
    .tooltip-data-value { color: #a5b4fc; }
    .tooltip-data-response { color: #6ee7b7; }

    /* === 9. Zoom/Pan === */
    .diagram-viewport {
      position: fixed;
      inset: 0;
      overflow: hidden;
      z-index: 5;
      cursor: grab;
    }
    .diagram-viewport.is-panning { cursor: grabbing; }

    .diagram {
      position: relative;
      transform-origin: 0 0;
      will-change: transform;
    }

    /* === 10. Legend === */
    .legend {
      position: fixed;
      bottom: 24px;
      right: 24px;
      z-index: 50;
      padding: 14px 18px;
      background: rgba(10, 10, 18, 0.85);
      border: 1px solid rgba(255, 255, 255, 0.1);
      border-radius: 12px;
      backdrop-filter: blur(12px);
      -webkit-backdrop-filter: blur(12px);
      font-size: 11px;
      color: var(--node-sub);
      min-width: 160px;
    }
    .legend-title {
      font-size: 10px;
      font-weight: 700;
      letter-spacing: 0.1em;
      text-transform: uppercase;
      color: var(--node-text);
      margin-bottom: 10px;
    }
    .legend-item {
      display: flex;
      align-items: center;
      gap: 10px;
      margin-bottom: 7px;
    }
    .legend-item:last-child { margin-bottom: 0; }
    .legend-line { flex-shrink: 0; width: 28px; height: 2px; }
    .legend-line--sync { background: var(--edge-primary); }
    .legend-line--secondary { background: var(--edge-secondary); }
    .legend-line--async {
      background: repeating-linear-gradient(90deg,
        var(--edge-async) 0px, var(--edge-async) 8px,
        transparent 8px, transparent 12px);
    }
    .legend-line--error {
      background: repeating-linear-gradient(90deg,
        var(--edge-error) 0px, var(--edge-error) 6px,
        transparent 6px, transparent 9px);
    }

    /* === 11. Print === */
    @media print {
      body { background: #ffffff; overflow: visible; color: #000000; }
      body::after { display: none; }
      .diagram-viewport { position: static; overflow: visible; }
      .diagram { transform: none !important; }
      .node {
        background: #f8f8f8; border: 1px solid #cccccc;
        backdrop-filter: none; -webkit-backdrop-filter: none;
        animation: none; opacity: 1; filter: none; color: #000000;
      }
      .node-label { color: #111111; }
      .node-subtitle { color: #555555; }
      .group { border-color: #aaaaaa; background: transparent; animation: none; opacity: 1; }
      .group-label { background: #ffffff; color: #555555; }
      .edge { stroke: #333333 !important; }
      .edge--error { stroke: #cc0000 !important; }
      .particle, .tooltip, .legend { display: none; }
    }

    /* === 12. Responsive === */
    @media (max-width: 768px) {
      .node { padding: 10px 14px; border-radius: 10px; min-width: 72px; }
      .node-icon { width: 22px; height: 22px; margin-bottom: 4px; }
      .node-label { font-size: 12px; }
      .node-subtitle { font-size: 10px; }
      .node--circle { width: 64px; height: 64px; }
      .node--hexagon { width: 72px; height: 64px; }
      .group { padding: 16px; border-radius: 12px; }
      .group-label { font-size: 9px; letter-spacing: 0.1em; }
      .legend { bottom: 12px; right: 12px; padding: 10px 14px; font-size: 10px; min-width: 130px; }
      .legend-title { font-size: 9px; margin-bottom: 8px; }
      .legend-line { width: 20px; }
      .tooltip { max-width: 180px; font-size: 11px; }
    }
  </style>
</head>
<body>

  <div class="diagram-viewport">
    <div class="diagram" id="diagram"></div>
  </div>

  <div class="tooltip"></div>

  <aside class="legend">
    <div class="legend-title">Edge Types</div>
    <div class="legend-item">
      <div class="legend-line legend-line--sync"></div>
      <span>Sync / RPC</span>
    </div>
    <div class="legend-item">
      <div class="legend-line legend-line--secondary"></div>
      <span>Data / Stream</span>
    </div>
    <div class="legend-item">
      <div class="legend-line legend-line--async"></div>
      <span>Async / Event</span>
    </div>
    <div class="legend-item">
      <div class="legend-line legend-line--error"></div>
      <span>Error / DLQ</span>
    </div>
  </aside>

  <script>
    // =============================================
    // ICON SYSTEM
    // =============================================
    const ICONS = {
      'server':     '<rect x="2" y="2" width="20" height="8" rx="2" ry="2"/><rect x="2" y="14" width="20" height="8" rx="2" ry="2"/><line x1="6" y1="6" x2="6.01" y2="6"/><line x1="6" y1="18" x2="6.01" y2="18"/>',
      'database':   '<ellipse cx="12" cy="5" rx="9" ry="3"/><path d="M21 12c0 1.66-4 3-9 3s-9-1.34-9-3"/><path d="M3 5v14c0 1.66 4 3 9 3s9-1.34 9-3V5"/>',
      'globe':      '<circle cx="12" cy="12" r="10"/><line x1="2" y1="12" x2="22" y2="12"/><path d="M12 2a15.3 15.3 0 0 1 4 10 15.3 15.3 0 0 1-4 10 15.3 15.3 0 0 1-4-10 15.3 15.3 0 0 1 4-10z"/>',
      'user':       '<path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"/><circle cx="12" cy="7" r="4"/>',
      'cloud':      '<path d="M18 10h-1.26A8 8 0 1 0 9 20h9a5 5 0 0 0 0-10z"/>',
      'lock':       '<rect x="3" y="11" width="18" height="11" rx="2" ry="2"/><path d="M7 11V7a5 5 0 0 1 10 0v4"/>',
      'mail':       '<path d="M4 4h16c1.1 0 2 .9 2 2v12c0 1.1-.9 2-2 2H4c-1.1 0-2-.9-2-2V6c0-1.1.9-2 2-2z"/><polyline points="22,6 12,13 2,6"/>',
      'zap':        '<polygon points="13 2 3 14 12 14 11 22 21 10 12 10 13 2"/>',
      'cpu':        '<rect x="9" y="9" width="6" height="6"/><rect x="5" y="5" width="14" height="14" rx="2" ry="2"/><line x1="9" y1="1" x2="9" y2="5"/><line x1="15" y1="1" x2="15" y2="5"/><line x1="9" y1="19" x2="9" y2="23"/><line x1="15" y1="19" x2="15" y2="23"/><line x1="1" y1="9" x2="5" y2="9"/><line x1="1" y1="15" x2="5" y2="15"/><line x1="19" y1="9" x2="23" y2="9"/><line x1="19" y1="15" x2="23" y2="15"/>',
      'hard-drive': '<line x1="22" y1="12" x2="2" y2="12"/><path d="M5.45 5.11L2 12v6a2 2 0 0 0 2 2h16a2 2 0 0 0 2-2v-6l-3.45-6.89A2 2 0 0 0 16.76 4H7.24a2 2 0 0 0-1.79 1.11z"/><line x1="6" y1="16" x2="6.01" y2="16"/><line x1="10" y1="16" x2="10.01" y2="16"/>',
      'git-branch': '<line x1="6" y1="3" x2="6" y2="15"/><circle cx="18" cy="6" r="3"/><circle cx="6" cy="18" r="3"/><path d="M18 9a9 9 0 0 1-9 9"/>',
      'shield':     '<path d="M12 22s8-4 8-10V5l-8-3-8 3v7c0 6 8 10 8 10z"/>',
      'activity':   '<polyline points="22 12 18 12 15 21 9 3 6 12 2 12"/>',
      'layers':     '<polygon points="12 2 2 7 12 12 22 7 12 2"/><polyline points="2 17 12 22 22 17"/><polyline points="2 12 12 17 22 12"/>',
      'smartphone': '<rect x="5" y="2" width="14" height="20" rx="2" ry="2"/><line x1="12" y1="18" x2="12.01" y2="18"/>',
      'terminal':   '<polyline points="4 17 10 11 4 5"/><line x1="12" y1="19" x2="20" y2="19"/>',
      'box':        '<path d="M21 16V8a2 2 0 0 0-1-1.73l-7-4a2 2 0 0 0-2 0l-7 4A2 2 0 0 0 3 8v8a2 2 0 0 0 1 1.73l7 4a2 2 0 0 0 2 0l7-4A2 2 0 0 0 21 16z"/><polyline points="3.27 6.96 12 12.01 20.73 6.96"/><line x1="12" y1="22.08" x2="12" y2="12"/>',
      'wifi':       '<path d="M5 12.55a11 11 0 0 1 14.08 0"/><path d="M1.42 9a16 16 0 0 1 21.16 0"/><path d="M8.53 16.11a6 6 0 0 1 6.95 0"/><line x1="12" y1="20" x2="12.01" y2="20"/>',
      'key':        '<path d="M21 2l-2 2m-7.61 7.61a5.5 5.5 0 1 1-7.778 7.778 5.5 5.5 0 0 1 7.777-7.777zm0 0L15.5 7.5m0 0l3 3L22 7l-3-3m-3.5 3.5L19 4"/>',
      'refresh-cw': '<polyline points="23 4 23 10 17 10"/><polyline points="1 20 1 14 7 14"/><path d="M3.51 9a9 9 0 0 1 14.85-3.36L23 10M1 14l4.64 4.36A9 9 0 0 0 20.49 15"/>',
    };

    function getIcon(name, size = 28) {
      return `<svg width="${size}" height="${size}" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">${ICONS[name] ?? ''}</svg>`;
    }

    // =============================================
    // LAYOUT CONSTANTS
    // =============================================
    const LAYOUT = {
      NODE_WIDTH: 180,
      NODE_HEIGHT: 80,
      LAYER_GAP: 200,
      NODE_GAP: 40,
      PADDING: 60,
      GROUP_PADDING: 32
    };

    // =============================================
    // ANIMATION CONSTANTS
    // =============================================
    const GROUP_FADE_DURATION     = 800;
    const NODE_STAGGER            = 150;
    const NODE_ENTRANCE_DURATION  = 1000;
    const EDGE_DRAW_DURATION      = 1800;
    const EDGE_STAGGER            = 250;
    const PARTICLE_DURATION       = 3500;
    const PARTICLE_COUNT_PER_EDGE = 3;
    const PARTICLE_STAGGER        = 1200;

    // =============================================
    // DIAGRAM DATA
    // =============================================
    const nodes = [
      { id: 'web-app',    label: 'Web App',    subtitle: 'React / Vite', icon: 'globe',    shape: 'rect' },
      { id: 'api-server', label: 'API Server', subtitle: 'Hono / Node',  icon: 'server',   shape: 'rect', group: 'backend' },
      { id: 'database',   label: 'Database',   subtitle: 'SQLite',       icon: 'database', shape: 'cylinder', group: 'backend' },
    ];

    const edges = [
      { from: 'web-app',    to: 'api-server', label: 'REST API', type: 'sync',
        method: 'POST', endpoint: '/api/data', request: '{ query, filters }', response: '{ results[], total }' },
      { from: 'api-server', to: 'database',   label: 'SQL',      type: 'secondary',
        method: 'SELECT', endpoint: 'records', request: 'WHERE filters', response: '{ rows[], count }' },
    ];

    const groups = [
      { id: 'backend', label: 'Backend Services', children: ['api-server', 'database'] },
    ];

    // =============================================
    // LAYOUT ALGORITHM
    // =============================================
    function breakCycles(nodes, edges) {
      const color = new Map(nodes.map(n => [n.id, 0]));
      const adj = new Map(nodes.map(n => [n.id, []]));
      const backEdges = new Set();
      for (const e of edges) {
        if (adj.has(e.from) && adj.has(e.to)) adj.get(e.from).push(e);
      }
      function dfs(id) {
        color.set(id, 1);
        for (const edge of adj.get(id) ?? []) {
          if (color.get(edge.to) === 1) backEdges.add(`${edge.from}->${edge.to}`);
          else if (color.get(edge.to) === 0) dfs(edge.to);
        }
        color.set(id, 2);
      }
      for (const { id } of nodes) {
        if (color.get(id) === 0) dfs(id);
      }
      return edges.filter(e => !backEdges.has(`${e.from}->${e.to}`));
    }

    function topologicalSort(nodes, edges) {
      const ids = nodes.map(n => n.id);
      const inDegree = new Map(ids.map(id => [id, 0]));
      const adj = new Map(ids.map(id => [id, []]));
      const safeEdges = breakCycles(nodes, edges);
      for (const { from, to } of safeEdges) {
        if (!adj.has(from) || !adj.has(to)) continue;
        adj.get(from).push(to);
        inDegree.set(to, inDegree.get(to) + 1);
      }
      const queue = ids.filter(id => inDegree.get(id) === 0);
      const result = [];
      while (queue.length > 0) {
        const node = queue.shift();
        result.push(node);
        for (const neighbor of adj.get(node) ?? []) {
          const deg = inDegree.get(neighbor) - 1;
          inDegree.set(neighbor, deg);
          if (deg === 0) queue.push(neighbor);
        }
      }
      const visited = new Set(result);
      for (const id of ids) { if (!visited.has(id)) result.push(id); }
      return result;
    }

    function assignLayers(nodes, edges) {
      const safeEdges = breakCycles(nodes, edges);
      const order = topologicalSort(nodes, safeEdges);
      const layer = new Map(nodes.map(n => [n.id, 0]));
      const predecessors = new Map(nodes.map(n => [n.id, []]));
      for (const { from, to } of safeEdges) {
        if (predecessors.has(to)) predecessors.get(to).push(from);
      }
      for (const id of order) {
        const preds = predecessors.get(id) ?? [];
        if (preds.length > 0) {
          layer.set(id, Math.max(...preds.map(p => layer.get(p) ?? 0)) + 1);
        }
      }
      return layer;
    }

    function minimizeCrossings(nodes, edges, layerMap) {
      const safeEdges = breakCycles(nodes, edges);
      const layerNodes = new Map();
      for (const { id } of nodes) {
        const l = layerMap.get(id) ?? 0;
        if (!layerNodes.has(l)) layerNodes.set(l, []);
        layerNodes.get(l).push(id);
      }
      const maxLayer = Math.max(...layerMap.values());
      const successors = new Map(nodes.map(n => [n.id, []]));
      const predecessors = new Map(nodes.map(n => [n.id, []]));
      for (const { from, to } of safeEdges) {
        if (successors.has(from)) successors.get(from).push(to);
        if (predecessors.has(to)) predecessors.get(to).push(from);
      }
      function barycenter(id, refOrder, direction) {
        const neighbors = direction === 'pred' ? predecessors.get(id) ?? [] : successors.get(id) ?? [];
        const positions = neighbors.map(n => refOrder.indexOf(n)).filter(i => i >= 0);
        if (positions.length === 0) return Infinity;
        return positions.reduce((a, b) => a + b, 0) / positions.length;
      }
      for (let pass = 0; pass < 3; pass++) {
        for (let l = 1; l <= maxLayer; l++) {
          const current = layerNodes.get(l) ?? [];
          const prev = layerNodes.get(l - 1) ?? [];
          current.sort((a, b) => barycenter(a, prev, 'pred') - barycenter(b, prev, 'pred'));
          layerNodes.set(l, current);
        }
        for (let l = maxLayer - 1; l >= 0; l--) {
          const current = layerNodes.get(l) ?? [];
          const next = layerNodes.get(l + 1) ?? [];
          current.sort((a, b) => barycenter(a, next, 'succ') - barycenter(b, next, 'succ'));
          layerNodes.set(l, current);
        }
      }
      return layerNodes;
    }

    function assignPositions(nodes, layerNodes) {
      const { NODE_WIDTH, NODE_HEIGHT, LAYER_GAP, NODE_GAP, PADDING } = LAYOUT;
      const positions = new Map();
      for (const [layerIndex, ids] of layerNodes) {
        const x = PADDING + layerIndex * (NODE_WIDTH + LAYER_GAP);
        const totalH = ids.length * NODE_HEIGHT + (ids.length - 1) * NODE_GAP;
        const startY = PADDING + (400 - totalH) / 2;
        ids.forEach((id, rank) => {
          positions.set(id, { x, y: startY + rank * (NODE_HEIGHT + NODE_GAP) });
        });
      }
      return positions;
    }

    function calcGroupBounds(groups, positions) {
      const { NODE_WIDTH, NODE_HEIGHT, GROUP_PADDING } = LAYOUT;
      const bounds = new Map();
      for (const group of groups) {
        let minX = Infinity, minY = Infinity, maxX = -Infinity, maxY = -Infinity;
        for (const childId of group.children) {
          const pos = positions.get(childId);
          if (!pos) continue;
          minX = Math.min(minX, pos.x);
          minY = Math.min(minY, pos.y);
          maxX = Math.max(maxX, pos.x + NODE_WIDTH);
          maxY = Math.max(maxY, pos.y + NODE_HEIGHT);
        }
        if (!isFinite(minX)) continue;
        bounds.set(group.id, {
          x: minX - GROUP_PADDING,
          y: minY - GROUP_PADDING,
          width: (maxX - minX) + GROUP_PADDING * 2,
          height: (maxY - minY) + GROUP_PADDING * 2
        });
      }
      return bounds;
    }

    function routeEdge(edge, positions) {
      const { from, to } = edge;
      const { NODE_WIDTH, NODE_HEIGHT } = LAYOUT;
      if (from === to) {
        const pos = positions.get(from);
        const cx = pos.x + NODE_WIDTH / 2;
        return `M ${cx - 20} ${pos.y} A 30 20 0 1 0 ${cx + 20} ${pos.y}`;
      }
      const srcPos = positions.get(from);
      const dstPos = positions.get(to);
      if (!srcPos || !dstPos) return '';
      const isBidi = edges.some(e => e.from === to && e.to === from);
      const offset = isBidi ? (from < to ? -4 : 4) : 0;
      const src = { x: srcPos.x + NODE_WIDTH, y: srcPos.y + NODE_HEIGHT / 2 + offset };
      const dst = { x: dstPos.x, y: dstPos.y + NODE_HEIGHT / 2 + offset };
      const dx = dst.x - src.x;
      // Nudge control points ±0.5px when horizontal to prevent zero-height bbox
      const cp1Y = (src.y === dst.y) ? src.y - 0.5 : src.y;
      const cp2Y = (src.y === dst.y) ? dst.y + 0.5 : dst.y;
      return `M ${src.x} ${src.y} C ${src.x + dx * 0.4} ${cp1Y}, ${dst.x - dx * 0.4} ${cp2Y}, ${dst.x} ${dst.y}`;
    }

    // =============================================
    // COMPUTE LAYOUT
    // =============================================
    const layerMap = assignLayers(nodes, edges);
    const layerNodes = minimizeCrossings(nodes, edges, layerMap);
    const positions = assignPositions(nodes, layerNodes);
    const groupBounds = calcGroupBounds(groups, positions);

    // =============================================
    // RENDER
    // =============================================
    const diagram = document.getElementById('diagram');

    // Compute diagram size
    let maxX = 0, maxY = 0;
    for (const pos of positions.values()) {
      maxX = Math.max(maxX, pos.x + LAYOUT.NODE_WIDTH + LAYOUT.PADDING);
      maxY = Math.max(maxY, pos.y + LAYOUT.NODE_HEIGHT + LAYOUT.PADDING);
    }
    for (const b of groupBounds.values()) {
      maxX = Math.max(maxX, b.x + b.width + LAYOUT.PADDING);
      maxY = Math.max(maxY, b.y + b.height + LAYOUT.PADDING);
    }
    diagram.style.width = `${maxX}px`;
    diagram.style.height = `${maxY}px`;

    // Render groups
    groups.forEach((group, gi) => {
      const b = groupBounds.get(group.id);
      if (!b) return;
      const el = document.createElement('div');
      el.className = 'group';
      el.style.left = `${b.x}px`;
      el.style.top = `${b.y}px`;
      el.style.width = `${b.width}px`;
      el.style.height = `${b.height}px`;
      el.style.animationDelay = `0ms`;
      el.innerHTML = `<span class="group-label">${group.label}</span>`;
      diagram.appendChild(el);
    });

    // Render nodes
    const nodeLayer = document.createElement('div');
    nodeLayer.className = 'node-layer';
    diagram.appendChild(nodeLayer);

    const nodeElements = {};
    const nodeMap = {};
    nodes.forEach(n => { nodeMap[n.id] = n; });
    const topoOrder = topologicalSort(nodes, edges);

    nodes.forEach(node => {
      const pos = positions.get(node.id);
      if (!pos) return;
      const topoIndex = topoOrder.indexOf(node.id);
      const shapeClass = node.shape && node.shape !== 'rect' ? ` node--${node.shape}` : '';
      const el = document.createElement('div');
      el.className = `node${shapeClass}`;
      el.dataset.nodeId = node.id;
      el.dataset.label = node.label;
      el.dataset.subtitle = node.subtitle;
      el.style.left = `${pos.x}px`;
      el.style.top = `${pos.y}px`;
      el.style.width = `${LAYOUT.NODE_WIDTH}px`;
      el.style.height = `${LAYOUT.NODE_HEIGHT}px`;
      el.style.animationDelay = `${GROUP_FADE_DURATION + topoIndex * NODE_STAGGER}ms`;
      el.innerHTML = `
        <div class="node-icon">${getIcon(node.icon)}</div>
        <div class="node-label">${node.label}</div>
        <div class="node-subtitle">${node.subtitle}</div>
      `;
      nodeLayer.appendChild(el);
      nodeElements[node.id] = el;
    });

    // Render edges (SVG)
    const svgNS = 'http://www.w3.org/2000/svg';
    const svg = document.createElementNS(svgNS, 'svg');
    svg.setAttribute('class', 'edge-layer');
    svg.setAttribute('xmlns', svgNS);
    svg.style.width = `${maxX}px`;
    svg.style.height = `${maxY}px`;

    // Defs: markers + filters
    const defs = document.createElementNS(svgNS, 'defs');
    const markerTypes = ['primary', 'secondary', 'async', 'error'];
    const markerCssVars = {
      'primary': 'var(--edge-primary)',
      'secondary': 'var(--edge-secondary)',
      'async': 'var(--edge-async)',
      'error': 'var(--edge-error)'
    };
    for (const name of markerTypes) {
      const marker = document.createElementNS(svgNS, 'marker');
      marker.setAttribute('id', `arrow-${name}`);
      marker.setAttribute('viewBox', '0 0 10 8');
      marker.setAttribute('refX', '10');
      marker.setAttribute('refY', '4');
      marker.setAttribute('markerWidth', '8');
      marker.setAttribute('markerHeight', '6');
      marker.setAttribute('orient', 'auto-start-reverse');
      const arrowPath = document.createElementNS(svgNS, 'path');
      arrowPath.setAttribute('d', 'M0 0 L10 4 L0 8 z');
      arrowPath.setAttribute('fill', markerCssVars[name]);
      marker.appendChild(arrowPath);
      defs.appendChild(marker);
    }

    // Glow filter (2-stage: blur → merge)
    const filter = document.createElementNS(svgNS, 'filter');
    filter.setAttribute('id', 'glow');
    filter.setAttribute('x', '-50%');
    filter.setAttribute('y', '-50%');
    filter.setAttribute('width', '200%');
    filter.setAttribute('height', '200%');
    const blur = document.createElementNS(svgNS, 'feGaussianBlur');
    blur.setAttribute('stdDeviation', '3');
    blur.setAttribute('result', 'blur');
    filter.appendChild(blur);
    const merge = document.createElementNS(svgNS, 'feMerge');
    const mn1 = document.createElementNS(svgNS, 'feMergeNode');
    mn1.setAttribute('in', 'blur');
    merge.appendChild(mn1);
    const mn2 = document.createElementNS(svgNS, 'feMergeNode');
    mn2.setAttribute('in', 'SourceGraphic');
    merge.appendChild(mn2);
    filter.appendChild(merge);
    defs.appendChild(filter);
    svg.appendChild(defs);

    const pathObjs = [];
    const nodesFinishTime = GROUP_FADE_DURATION + nodes.length * NODE_STAGGER + NODE_ENTRANCE_DURATION;

    edges.forEach((edge, i) => {
      const d = routeEdge(edge, positions);
      if (!d) return;

      const typeClass = edge.type === 'async' ? 'edge--async'
        : edge.type === 'error' ? 'edge--error'
        : edge.type === 'secondary' ? 'edge--secondary'
        : 'edge--sync';

      const markerName = edge.type === 'async' ? 'arrow-async'
        : edge.type === 'error' ? 'arrow-error'
        : edge.type === 'secondary' ? 'arrow-secondary'
        : 'arrow-primary';

      const path = document.createElementNS(svgNS, 'path');
      path.setAttribute('class', `edge ${typeClass}`);
      path.setAttribute('d', d);
      path.setAttribute('marker-end', `url(#${markerName})`);
      path.setAttribute('filter', 'url(#glow)');
      svg.appendChild(path);
      pathObjs.push({ path, edge });

      // Set path length for draw animation after appending to DOM
      requestAnimationFrame(() => {
        const pathLength = path.getTotalLength();
        path.style.setProperty('--path-length', pathLength);
        path.style.strokeDasharray = pathLength;
        path.style.strokeDashoffset = pathLength;
        path.style.animation = `drawLine ${EDGE_DRAW_DURATION}ms ease-in-out forwards`;
        path.style.animationDelay = `${nodesFinishTime + i * EDGE_STAGGER}ms`;
      });

      // Edge label
      if (edge.label) {
        const srcPos = positions.get(edge.from);
        const dstPos = positions.get(edge.to);
        if (srcPos && dstPos) {
          const midX = (srcPos.x + LAYOUT.NODE_WIDTH + dstPos.x) / 2;
          const midY = (srcPos.y + LAYOUT.NODE_HEIGHT / 2 + dstPos.y + LAYOUT.NODE_HEIGHT / 2) / 2 - 10;
          const text = document.createElementNS(svgNS, 'text');
          text.setAttribute('class', 'edge-label');
          text.setAttribute('x', midX);
          text.setAttribute('y', midY);
          text.textContent = edge.label;
          text.style.opacity = '0';
          text.style.animation = `fadeIn 0.3s ease-out forwards`;
          text.style.animationDelay = `${nodesFinishTime + i * EDGE_STAGGER + EDGE_DRAW_DURATION / 2}ms`;
          svg.appendChild(text);
        }
      }
    });

    diagram.appendChild(svg);

    // Particle layer (SVG circles animated via requestAnimationFrame)
    const particleSvg = document.createElementNS(svgNS, 'svg');
    particleSvg.setAttribute('class', 'particle-layer');
    particleSvg.setAttribute('id', 'particles');
    particleSvg.style.width = `${maxX}px`;
    particleSvg.style.height = `${maxY}px`;
    diagram.appendChild(particleSvg);

    /** All active particles — updated by a single rAF loop */
    const particles = [];

    /**
     * Spawns particle dots for a single edge path.
     * @param {Object} obj - pathObj with path element and edge data
     */
    function spawnParticlesForEdge(obj) {
      const colorMap = { sync: '#6366f1', secondary: '#06b6d4', async: '#f59e0b', error: '#ef4444' };
      const colorVar = colorMap[obj.edge.type] || '#6366f1';
      const totalLength = obj.path.getTotalLength();

      for (let p = 0; p < PARTICLE_COUNT_PER_EDGE; p++) {
        const circle = document.createElementNS(svgNS, 'circle');
        circle.setAttribute('r', '3');
        circle.setAttribute('fill', colorVar);
        circle.setAttribute('opacity', '0');
        circle.style.opacity = '0';
        particleSvg.appendChild(circle);

        particles.push({ circle, path: obj.path, totalLength, offset: p * PARTICLE_STAGGER });
      }
    }

    /**
     * Single rAF loop that updates every particle each frame.
     */
    function tickParticles(timestamp) {
      for (const p of particles) {
        const elapsed = (timestamp - p.offset) % PARTICLE_DURATION;
        if (elapsed < 0) continue;
        const progress = ((elapsed % PARTICLE_DURATION) + PARTICLE_DURATION) % PARTICLE_DURATION / PARTICLE_DURATION;
        const point = p.path.getPointAtLength(progress * p.totalLength);
        p.circle.setAttribute('cx', point.x);
        p.circle.setAttribute('cy', point.y);
        p.circle.setAttribute('opacity', '0.85');
      }
      requestAnimationFrame(tickParticles);
    }

    // Spawn particles immediately, single loop drives all
    pathObjs.forEach((obj) => { spawnParticlesForEdge(obj); });
    requestAnimationFrame(tickParticles);

    // =============================================
    // INTERACTIVITY: Setup
    // =============================================
    const tooltip = document.querySelector('.tooltip');
    const viewport = document.querySelector('.diagram-viewport');

    let scale = 1, oX = 0, oY = 0;

    // =============================================
    // INTERACTIVITY: Hover Highlight + Node Tooltip
    // =============================================
    let hoveredNodeId = null;
    let hoveredEdgeIndex = null;

    function clearHighlight() {
      hoveredNodeId = null;
      hoveredEdgeIndex = null;
      Object.values(nodeElements).forEach(el => {
        el.classList.remove('is-dimmed', 'is-highlighted');
      });
      pathObjs.forEach(obj => obj.path.classList.remove('is-dimmed'));
      tooltip.classList.remove('is-active');
    }

    function applyHighlight(nodeId) {
      if (hoveredNodeId === nodeId) return;
      hoveredNodeId = nodeId;
      hoveredEdgeIndex = null;
      const connectedIds = new Set([nodeId]);
      const connectedPaths = [];
      pathObjs.forEach(obj => {
        if (obj.edge.from === nodeId || obj.edge.to === nodeId) {
          connectedIds.add(obj.edge.from);
          connectedIds.add(obj.edge.to);
          connectedPaths.push(obj.path);
        }
      });
      Object.values(nodeElements).forEach(el => {
        const c = connectedIds.has(el.dataset.nodeId);
        el.classList.toggle('is-dimmed', !c);
        el.classList.toggle('is-highlighted', c);
      });
      pathObjs.forEach(obj => {
        obj.path.classList.toggle('is-dimmed', !connectedPaths.includes(obj.path));
      });
    }

    // Node hover: highlight + tooltip
    Object.values(nodeElements).forEach(nodeEl => {
      nodeEl.addEventListener('mouseenter', () => {
        applyHighlight(nodeEl.dataset.nodeId);
        tooltip.innerHTML = `<div style="font-weight:600;font-size:13px;margin-bottom:4px;">${nodeEl.dataset.label}</div><div style="color:var(--node-sub);">${nodeEl.dataset.subtitle}</div>`;
        const rect = nodeEl.getBoundingClientRect();
        tooltip.style.left = `${rect.left + rect.width / 2}px`;
        tooltip.style.top = `${rect.top - 8}px`;
        tooltip.style.transform = 'translate(-50%, -100%)';
        tooltip.classList.add('is-active');
      });
      nodeEl.addEventListener('mouseleave', (e) => {
        if (e.relatedTarget && e.relatedTarget.closest && e.relatedTarget.closest('.node')) return;
        clearHighlight();
      });
    });

    // =============================================
    // INTERACTIVITY: Edge Tooltip (Proximity Detection)
    // =============================================
    const EDGE_HIT_DISTANCE = 14;

    function findClosestEdge(mx, my) {
      const vpRect = viewport.getBoundingClientRect();
      const localX = (mx - vpRect.left - oX) / scale;
      const localY = (my - vpRect.top - oY) / scale;
      let closest = null, minDist = Infinity;
      pathObjs.forEach((obj, i) => {
        const path = obj.path;
        const len = path.getTotalLength();
        const steps = Math.max(10, Math.round(len / 20));
        for (let s = 0; s <= steps; s++) {
          const pt = path.getPointAtLength((s / steps) * len);
          const d = Math.hypot(pt.x - localX, pt.y - localY);
          if (d < minDist) { minDist = d; closest = { index: i, dist: d }; }
        }
      });
      return closest && closest.dist < EDGE_HIT_DISTANCE / scale ? closest : null;
    }

    function buildEdgeTooltip(edge) {
      const from = nodeMap[edge.from], to = nodeMap[edge.to];
      const cls = 'tooltip-method--' + (edge.method || '').toLowerCase();
      return `
        <div style="margin-bottom:8px;">
          <span style="color:var(--node-sub);font-size:11px;">${from.label} → ${to.label}</span>
        </div>
        <div style="margin-bottom:8px;">
          <span class="tooltip-method ${cls}">${edge.method || edge.type}</span>
          <span style="font-family:monospace;color:#fff;">${edge.endpoint || edge.label}</span>
        </div>
        <div class="tooltip-data">
          <div class="tooltip-data-label">→ Request</div>
          <div class="tooltip-data-value">${edge.request || '—'}</div>
          <div style="margin-top:6px;" class="tooltip-data-label">← Response</div>
          <div class="tooltip-data-value tooltip-data-response">${edge.response || '—'}</div>
        </div>`;
    }

    document.addEventListener('mousemove', (e) => {
      const elUnder = document.elementFromPoint(e.clientX, e.clientY);
      if (elUnder && elUnder.closest && elUnder.closest('.node')) return;
      const hit = findClosestEdge(e.clientX, e.clientY);
      if (hit) {
        if (hoveredEdgeIndex === hit.index) {
          tooltip.style.left = `${e.clientX + 12}px`;
          tooltip.style.top = `${e.clientY - 12}px`;
          tooltip.style.transform = 'translateY(-100%)';
          return;
        }
        hoveredEdgeIndex = hit.index;
        hoveredNodeId = null;
        const edge = edges[hit.index];
        tooltip.innerHTML = buildEdgeTooltip(edge);
        tooltip.style.left = `${e.clientX + 12}px`;
        tooltip.style.top = `${e.clientY - 12}px`;
        tooltip.style.transform = 'translateY(-100%)';
        tooltip.classList.add('is-active');
      } else if (hoveredEdgeIndex !== null) {
        hoveredEdgeIndex = null;
        clearHighlight();
      }
    });

    // =============================================
    // INTERACTIVITY: Zoom / Pan
    // =============================================
    let isPanning = false, panStartX, panStartY, panStartOX, panStartOY;
    const MIN_SCALE = 0.25, MAX_SCALE = 4, ZOOM_SPEED = 0.001;

    function applyTransform() {
      diagram.style.transform = `translate(${oX}px, ${oY}px) scale(${scale})`;
    }

    function fitToViewport() {
      const FIT_PADDING = 80;
      const vw = viewport.clientWidth;
      const vh = viewport.clientHeight;
      const nodeEls = diagram.querySelectorAll('.node');
      let minX = Infinity, minY = Infinity, maxX = 0, maxY = 0;
      nodeEls.forEach(el => {
        const x = parseFloat(el.style.left);
        const y = parseFloat(el.style.top);
        const w = el.offsetWidth;
        const h = el.offsetHeight;
        if (x < minX) minX = x;
        if (y < minY) minY = y;
        if (x + w > maxX) maxX = x + w;
        if (y + h > maxY) maxY = y + h;
      });
      const contentW = maxX - minX + FIT_PADDING * 2;
      const contentH = maxY - minY + FIT_PADDING * 2;
      scale = Math.min(vw / contentW, vh / contentH);
      scale = Math.max(MIN_SCALE, Math.min(scale, MAX_SCALE));
      oX = (vw - contentW * scale) / 2 - (minX - FIT_PADDING) * scale;
      oY = (vh - contentH * scale) / 2 - (minY - FIT_PADDING) * scale;
      applyTransform();
    }

    viewport.addEventListener('wheel', (e) => {
      e.preventDefault();
      const delta = -e.deltaY * ZOOM_SPEED;
      const newScale = Math.min(MAX_SCALE, Math.max(MIN_SCALE, scale + delta * scale));
      const rect = viewport.getBoundingClientRect();
      const mouseX = e.clientX - rect.left;
      const mouseY = e.clientY - rect.top;
      oX = mouseX - (mouseX - oX) * (newScale / scale);
      oY = mouseY - (mouseY - oY) * (newScale / scale);
      scale = newScale;
      applyTransform();
    }, { passive: false });

    viewport.addEventListener('mousedown', (e) => {
      if (e.button !== 0) return;
      isPanning = true;
      panStartX = e.clientX;
      panStartY = e.clientY;
      panStartOX = oX;
      panStartOY = oY;
      viewport.classList.add('is-panning');
    });

    window.addEventListener('mousemove', (e) => {
      if (!isPanning) return;
      oX = panStartOX + (e.clientX - panStartX);
      oY = panStartOY + (e.clientY - panStartY);
      applyTransform();
    });

    window.addEventListener('mouseup', () => {
      isPanning = false;
      viewport.classList.remove('is-panning');
    });

    viewport.addEventListener('dblclick', fitToViewport);
    window.addEventListener('resize', fitToViewport);

    // Scale to fill viewport on load
    fitToViewport();
  </script>
</body>
</html>
```

Use this example as the reference implementation. Every diagram you generate must follow the same structure, design system, and feature set. Adapt the `nodes`, `edges`, and `groups` arrays to match the user's architecture, choose the appropriate diagram type and layout direction, and produce a complete working HTML file.
