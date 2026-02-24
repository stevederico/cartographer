# Architecture Diagram Design System

Complete reference for generating animated architecture diagrams in HTML/CSS/JS. Every snippet is copy-paste-ready for use inside an HTML `<style>` block.

---

## 1. CSS Custom Properties (Color Palette)

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

## 2. Base Styles

```css
*,
*::before,
*::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  background-color: var(--bg);
  font-family: 'Inter', system-ui, -apple-system, sans-serif;
  overflow: hidden;
  position: relative;
  width: 100vw;
  height: 100vh;
  color: var(--node-text);
}

/* Dot-grid background */
body::after {
  content: '';
  position: fixed;
  inset: 0;
  background-image: radial-gradient(circle, var(--bg-grid) 1px, transparent 1px);
  background-size: 28px 28px;
  pointer-events: none;
  z-index: 0;
}
```

---

## 3. Node Styles

### Base Node (Glassmorphism)

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

### Cylinder Node (Database)

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

### Hexagon Node (Message Queue)

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

### Circle Node (Actor / External)

```css
.node--circle {
  border-radius: 50%;
  width: 80px;
  height: 80px;
  padding: 8px;
}
```

### Stadium Node (Load Balancer / Proxy)

```css
.node--stadium {
  border-radius: 999px;
  padding: 12px 28px;
}
```

### Node Inner Elements

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

## 4. Edge Styles

### SVG Layer

```css
.edge-layer {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: 1;
  overflow: visible;
}
```

### Edge Paths

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

### SVG Marker Defs (Arrowheads)

Place inside an `<svg>` `<defs>` block — one per color variant:

```html
<svg class="edge-layer" xmlns="http://www.w3.org/2000/svg">
  <defs>
    <!-- Primary (indigo) -->
    <marker id="arrow-primary" markerWidth="10" markerHeight="7"
            refX="9" refY="3.5" orient="auto" markerUnits="strokeWidth">
      <path d="M0,0 L0,7 L10,3.5 z" fill="#6366f1" />
    </marker>

    <!-- Secondary (cyan) -->
    <marker id="arrow-secondary" markerWidth="10" markerHeight="7"
            refX="9" refY="3.5" orient="auto" markerUnits="strokeWidth">
      <path d="M0,0 L0,7 L10,3.5 z" fill="#06b6d4" />
    </marker>

    <!-- Async (amber) -->
    <marker id="arrow-async" markerWidth="10" markerHeight="7"
            refX="9" refY="3.5" orient="auto" markerUnits="strokeWidth">
      <path d="M0,0 L0,7 L10,3.5 z" fill="#f59e0b" />
    </marker>

    <!-- Error (red) -->
    <marker id="arrow-error" markerWidth="10" markerHeight="7"
            refX="9" refY="3.5" orient="auto" markerUnits="strokeWidth">
      <path d="M0,0 L0,7 L10,3.5 z" fill="#ef4444" />
    </marker>

    <!-- Glow filter for edges -->
    <filter id="edge-glow" x="-20%" y="-20%" width="140%" height="140%">
      <feGaussianBlur in="SourceGraphic" stdDeviation="3" result="blur" />
      <feColorMatrix in="blur" type="matrix"
        values="1 0 0 0 0  0 1 0 0 0  0 0 1 0 0  0 0 0 18 -7"
        result="glow" />
      <feMerge>
        <feMergeNode in="glow" />
        <feMergeNode in="SourceGraphic" />
      </feMerge>
    </filter>

    <!-- Stronger glow (active state) -->
    <filter id="edge-glow-strong" x="-30%" y="-30%" width="160%" height="160%">
      <feGaussianBlur in="SourceGraphic" stdDeviation="4" result="blur" />
      <feColorMatrix in="blur" type="matrix"
        values="1 0 0 0 0  0 1 0 0 0  0 0 1 0 0  0 0 0 22 -8"
        result="glow" />
      <feMerge>
        <feMergeNode in="glow" />
        <feMergeNode in="SourceGraphic" />
      </feMerge>
    </filter>
  </defs>
</svg>
```

Usage on a path element:

```html
<path class="edge edge--sync"
      d="M 200 150 C 300 150 300 250 400 250"
      marker-end="url(#arrow-primary)"
      filter="url(#edge-glow)"
      style="--path-length: 320" />
```

---

## 5. Group / Container Styles

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

## 6. Animation Keyframes

```css
/* Nodes entering the scene */
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

/* Edge draw-on using stroke-dashoffset */
@keyframes drawLine {
  from {
    stroke-dashoffset: var(--path-length);
  }
  to {
    stroke-dashoffset: 0;
  }
}

/* Particle traveling along an edge path */
@keyframes flowDot {
  from {
    offset-distance: 0%;
  }
  to {
    offset-distance: 100%;
  }
}

/* Pulsing glow for active/highlighted nodes */
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

/* Group fade-in */
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
/* Nodes */
.node {
  opacity: 0;
  animation: fadeInUp 0.6s ease-out forwards;
}

/* Edges — requires stroke-dasharray set to path length in JS */
.edge {
  stroke-dasharray: var(--path-length);
  stroke-dashoffset: var(--path-length);
  animation: drawLine 1s ease-in-out forwards;
}

/* Active/highlighted node */
.node--active {
  animation: fadeInUp 0.6s ease-out forwards,
             pulse 3s ease-in-out infinite;
}

/* Groups */
.group {
  opacity: 0;
  animation: fadeIn 0.4s ease-out forwards;
}
```

---

## 7. Animation Timeline Constants

| Constant | Value | Notes |
|---|---|---|
| `GROUP_FADE_DURATION` | `400ms` | Groups animate in first |
| `NODE_STAGGER` | `80ms` | Delay increment per node |
| `NODE_ENTRANCE_DURATION` | `600ms` | fadeInUp duration |
| `EDGE_DRAW_DURATION` | `1000ms` | drawLine duration |
| `EDGE_STAGGER` | `150ms` | Delay increment per edge |
| `PARTICLE_DURATION` | `2000ms` | One full pass along an edge |
| `PARTICLE_COUNT_PER_EDGE` | `3` | Particles per active edge |
| `PARTICLE_STAGGER` | `700ms` | Delay between particles on same edge |
| `PULSE_DURATION` | `3000ms` | Glow pulse cycle for active nodes |

### JS Sequencing Reference

```js
const GROUP_FADE_DURATION     = 400;   // ms
const NODE_STAGGER            = 80;    // ms
const NODE_ENTRANCE_DURATION  = 600;   // ms
const EDGE_DRAW_DURATION      = 1000;  // ms
const EDGE_STAGGER            = 150;   // ms
const PARTICLE_DURATION       = 2000;  // ms
const PARTICLE_COUNT_PER_EDGE = 3;
const PARTICLE_STAGGER        = 700;   // ms
const PULSE_DURATION          = 3000;  // ms

// Typical sequencing:
// t=0          Groups fade in
// t=400        Nodes stagger in (node_i at 400 + i * NODE_STAGGER)
// t=400+n*80   Edges draw (edge_i at groups+nodes_done + i * EDGE_STAGGER)
// t=edges_done Particles start flowing; active nodes begin pulsing
```

---

## 8. Particle Styles

Particles ride along a CSS `offset-path` that mirrors each SVG edge path.

```css
.particle {
  position: absolute;
  width: 6px;
  height: 6px;
  border-radius: 50%;
  pointer-events: none;
  z-index: 20;
  animation: flowDot var(--particle-duration, 2000ms) linear infinite;
  animation-delay: var(--particle-delay, 0ms);
  offset-rotate: 0deg; /* keep dot circular, not rotated to path */
}

/* Color variants matching edge types */
.particle--primary {
  background: #6366f1;
  filter: drop-shadow(0 0 4px #6366f1) drop-shadow(0 0 10px rgba(99, 102, 241, 0.6));
}

.particle--secondary {
  background: #06b6d4;
  filter: drop-shadow(0 0 4px #06b6d4) drop-shadow(0 0 10px rgba(6, 182, 212, 0.6));
}

.particle--async {
  background: #f59e0b;
  filter: drop-shadow(0 0 4px #f59e0b) drop-shadow(0 0 10px rgba(245, 158, 11, 0.6));
}

.particle--error {
  background: #ef4444;
  filter: drop-shadow(0 0 4px #ef4444) drop-shadow(0 0 10px rgba(239, 68, 68, 0.6));
}
```

### JS: Spawning Particles

```js
/**
 * Spawns animated particles along a given SVG path.
 *
 * @param {SVGPathElement} pathEl - The SVG path to follow.
 * @param {string} colorClass     - CSS class for particle color variant.
 * @param {HTMLElement} container - Positioned container for the particles.
 */
function spawnParticles(pathEl, colorClass, container) {
  const pathData = pathEl.getAttribute('d');
  for (let i = 0; i < PARTICLE_COUNT_PER_EDGE; i++) {
    const dot = document.createElement('div');
    dot.className = `particle ${colorClass}`;
    dot.style.setProperty('--particle-delay', `${i * PARTICLE_STAGGER}ms`);
    dot.style.setProperty('--particle-duration', `${PARTICLE_DURATION}ms`);
    dot.style.offsetPath = `path('${pathData}')`;
    container.appendChild(dot);
  }
}
```

---

## 9. Tooltip Styles

```css
.tooltip {
  position: absolute;
  z-index: 100;
  max-width: 240px;
  padding: 10px 14px;
  background: rgba(15, 15, 30, 0.92);
  border: 1px solid rgba(255, 255, 255, 0.14);
  border-radius: 10px;
  backdrop-filter: blur(16px);
  -webkit-backdrop-filter: blur(16px);
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.48);
  pointer-events: none;
  opacity: 0;
  transform: translateY(4px);
  transition: opacity 0.15s ease, transform 0.15s ease;
  font-size: 12px;
  color: var(--node-text);
  line-height: 1.5;
}

.tooltip.is-visible {
  opacity: 1;
  transform: translateY(0);
}

.tooltip-title {
  font-weight: 600;
  font-size: 13px;
  margin-bottom: 4px;
}

.tooltip-body {
  color: var(--node-sub);
}
```

### JS: Tooltip Positioning

```js
const tooltip = document.querySelector('.tooltip');

/**
 * Shows the tooltip near the target element.
 *
 * @param {HTMLElement} target  - Element that triggered the tooltip.
 * @param {string}      title   - Tooltip heading.
 * @param {string}      body    - Tooltip body text.
 */
function showTooltip(target, title, body) {
  tooltip.querySelector('.tooltip-title').textContent = title;
  tooltip.querySelector('.tooltip-body').textContent = body;

  const rect = target.getBoundingClientRect();
  tooltip.style.left = `${rect.left + rect.width / 2}px`;
  tooltip.style.top  = `${rect.top - 8}px`;
  tooltip.style.transform = 'translate(-50%, -100%)';
  tooltip.classList.add('is-visible');
}

function hideTooltip() {
  tooltip.classList.remove('is-visible');
}
```

---

## 10. Zoom / Pan Container

### CSS

```css
/* Outer viewport — fills the window */
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

/* Inner canvas — nodes, edges, and groups live here */
.diagram {
  position: relative;
  transform-origin: 0 0;
  will-change: transform;
  /* width/height set in JS to match the diagram bounds */
}
```

### JS: Wheel Zoom + Drag Pan + Double-Click Reset

```js
const viewport = document.querySelector('.diagram-viewport');
const diagram  = document.querySelector('.diagram');

let scale   = 1;
let originX = 0;
let originY = 0;
let isPanning = false;
let startX, startY, startOriginX, startOriginY;

const MIN_SCALE = 0.25;
const MAX_SCALE = 4;
const ZOOM_SPEED = 0.001;

/** Applies the current scale + pan to the diagram canvas. */
function applyTransform() {
  diagram.style.transform = `translate(${originX}px, ${originY}px) scale(${scale})`;
}

/** Resets zoom and pan to fit the diagram centered in the viewport. */
function resetTransform() {
  scale   = 1;
  originX = (viewport.clientWidth  - diagram.offsetWidth)  / 2;
  originY = (viewport.clientHeight - diagram.offsetHeight) / 2;
  applyTransform();
}

// Wheel zoom — zooms toward cursor position
viewport.addEventListener('wheel', (e) => {
  e.preventDefault();
  const delta = -e.deltaY * ZOOM_SPEED;
  const newScale = Math.min(MAX_SCALE, Math.max(MIN_SCALE, scale + delta * scale));

  // Adjust origin so zoom anchors to the cursor
  const rect = viewport.getBoundingClientRect();
  const mouseX = e.clientX - rect.left;
  const mouseY = e.clientY - rect.top;

  originX = mouseX - (mouseX - originX) * (newScale / scale);
  originY = mouseY - (mouseY - originY) * (newScale / scale);
  scale   = newScale;

  applyTransform();
}, { passive: false });

// Drag pan
viewport.addEventListener('mousedown', (e) => {
  if (e.button !== 0) return;
  isPanning   = true;
  startX      = e.clientX;
  startY      = e.clientY;
  startOriginX = originX;
  startOriginY = originY;
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

// Double-click resets to default view
viewport.addEventListener('dblclick', resetTransform);

// Initialize centered
resetTransform();
```

---

## 11. Legend Styles

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

.legend-line--sync {
  background: var(--edge-primary);
}

.legend-line--async {
  /* dashed via repeating-linear-gradient */
  background: repeating-linear-gradient(
    90deg,
    var(--edge-async) 0px,
    var(--edge-async) 8px,
    transparent 8px,
    transparent 12px
  );
}

.legend-line--secondary {
  background: var(--edge-secondary);
}

.legend-line--error {
  background: repeating-linear-gradient(
    90deg,
    var(--edge-error) 0px,
    var(--edge-error) 6px,
    transparent 6px,
    transparent 9px
  );
}
```

### Legend HTML Template

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

## 12. Print Styles

```css
@media print {
  body {
    background: #ffffff;
    overflow: visible;
    color: #000000;
  }

  body::after {
    display: none;
  }

  .diagram-viewport {
    position: static;
    overflow: visible;
  }

  .diagram {
    transform: none !important;
  }

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

  .node-label {
    color: #111111;
  }

  .node-subtitle {
    color: #555555;
  }

  .group {
    border-color: #aaaaaa;
    background: transparent;
    animation: none;
    opacity: 1;
  }

  .group-label {
    background: #ffffff;
    color: #555555;
  }

  .edge {
    /* Print-safe colors */
    stroke: #333333 !important;
  }

  .edge--error {
    stroke: #cc0000 !important;
  }

  .particle,
  .tooltip,
  .legend {
    display: none;
  }
}
```

---

## 13. Responsive Styles

```css
@media (max-width: 768px) {
  .node {
    padding: 10px 14px;
    border-radius: 10px;
    min-width: 72px;
  }

  .node-icon {
    width: 22px;
    height: 22px;
    margin-bottom: 4px;
  }

  .node-label {
    font-size: 12px;
  }

  .node-subtitle {
    font-size: 10px;
  }

  .node--circle {
    width: 64px;
    height: 64px;
  }

  .node--hexagon {
    width: 72px;
    height: 64px;
  }

  .group {
    padding: 16px;
    border-radius: 12px;
  }

  .group-label {
    font-size: 9px;
    letter-spacing: 0.1em;
  }

  .legend {
    bottom: 12px;
    right: 12px;
    padding: 10px 14px;
    font-size: 10px;
    min-width: 130px;
  }

  .legend-title {
    font-size: 9px;
    margin-bottom: 8px;
  }

  .legend-line {
    width: 20px;
  }

  .tooltip {
    max-width: 180px;
    font-size: 11px;
  }
}
```

---

## Full Style Block Template

Paste this into the `<head>` of any diagram HTML file and fill in the sections you need:

```html
<style>
  /* === 1. Tokens === */
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
  body { background: var(--bg); font-family: 'Inter', system-ui, -apple-system, sans-serif; overflow: hidden; width: 100vw; height: 100vh; }
  body::after { content: ''; position: fixed; inset: 0; background-image: radial-gradient(circle, var(--bg-grid) 1px, transparent 1px); background-size: 28px 28px; pointer-events: none; z-index: 0; }

  /* === 3. Nodes === */
  /* ... paste Section 3 ... */

  /* === 4. Edges === */
  /* ... paste Section 4 CSS + SVG defs inline ... */

  /* === 5. Groups === */
  /* ... paste Section 5 ... */

  /* === 6. Keyframes === */
  /* ... paste Section 6 ... */

  /* === 8. Particles === */
  /* ... paste Section 8 ... */

  /* === 9. Tooltips === */
  /* ... paste Section 9 ... */

  /* === 10. Zoom/Pan === */
  /* ... paste Section 10 CSS ... */

  /* === 11. Legend === */
  /* ... paste Section 11 ... */

  /* === 12. Print === */
  /* ... paste Section 12 ... */

  /* === 13. Responsive === */
  /* ... paste Section 13 ... */
</style>
```
