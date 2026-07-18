# Auto-Layout Algorithm Reference

Complete JavaScript implementations for positioning nodes and routing edges in animated architecture diagrams. All functions are pure (no side effects) and can be used independently.

---

## 1. Layout Constants

```javascript
const LAYOUT = {
  NODE_WIDTH: 180,
  NODE_HEIGHT: 80,
  LAYER_GAP: 200,
  NODE_GAP: 40,
  PADDING: 60,
  GROUP_PADDING: 32
};
```

---

## 2. Data Structures

```javascript
// Node: { id, label, subtitle, icon, shape, group?, x?, y? }
// Edge: { from, to, label?, type: 'sync'|'async'|'error', color? }
// Group: { id, label, children: [nodeIds] }

// Example:
const nodes = [
  { id: 'client', label: 'Client', subtitle: 'Browser', icon: '🌐', shape: 'rect' },
  { id: 'api',    label: 'API',    subtitle: 'Hono',    icon: '⚡', shape: 'rect', group: 'backend' },
  { id: 'db',     label: 'DB',     subtitle: 'SQLite',  icon: '🗄', shape: 'cylinder', group: 'backend' }
];

const edges = [
  { from: 'client', to: 'api', label: 'HTTP',  type: 'sync' },
  { from: 'api',    to: 'db',  label: 'query', type: 'sync' }
];

const groups = [
  { id: 'backend', label: 'Backend Services', children: ['api', 'db'] }
];
```

---

## 3. Topological Sort (Kahn's Algorithm)

Returns a stable ordering of node IDs with no forward dependencies. Cycles are broken by removing the lowest-degree back-edge.

```javascript
/**
 * Topologically sorts node IDs using Kahn's BFS algorithm.
 * Cycles are detected and broken by removing back edges before sorting.
 *
 * @param {Array<{id: string}>} nodes
 * @param {Array<{from: string, to: string}>} edges
 * @returns {string[]} Ordered array of node IDs, sources first
 */
function topologicalSort(nodes, edges) {
  const ids = nodes.map(n => n.id);
  const inDegree = new Map(ids.map(id => [id, 0]));
  const adj = new Map(ids.map(id => [id, []]));

  // Build adjacency and in-degree from a cycle-free edge set
  const safeEdges = breakCycles(nodes, edges);

  for (const { from, to } of safeEdges) {
    if (!adj.has(from) || !adj.has(to)) continue;
    adj.get(from).push(to);
    inDegree.set(to, inDegree.get(to) + 1);
  }

  // Queue all nodes with no incoming edges
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

  // Any remaining nodes are part of an unbroken cycle — append them
  const visited = new Set(result);
  for (const id of ids) {
    if (!visited.has(id)) result.push(id);
  }

  return result;
}

/**
 * Removes back-edges from a cycle to make the graph a DAG.
 * Uses DFS coloring (white=0, gray=1, black=2).
 * When a back-edge is found, it is excluded from the returned edge list.
 *
 * @param {Array<{id: string}>} nodes
 * @param {Array<{from: string, to: string}>} edges
 * @returns {Array<{from: string, to: string}>} Edge list with back-edges removed
 */
function breakCycles(nodes, edges) {
  const color = new Map(nodes.map(n => [n.id, 0]));
  const adj = new Map(nodes.map(n => [n.id, []]));
  const backEdges = new Set();

  for (const e of edges) {
    if (adj.has(e.from) && adj.has(e.to)) {
      adj.get(e.from).push(e);
    }
  }

  function dfs(id) {
    color.set(id, 1); // gray — currently in stack
    for (const edge of adj.get(id) ?? []) {
      if (color.get(edge.to) === 1) {
        // Back-edge detected: mark for removal
        backEdges.add(`${edge.from}→${edge.to}`);
      } else if (color.get(edge.to) === 0) {
        dfs(edge.to);
      }
    }
    color.set(id, 2); // black — fully processed
  }

  for (const { id } of nodes) {
    if (color.get(id) === 0) dfs(id);
  }

  return edges.filter(e => !backEdges.has(`${e.from}→${e.to}`));
}
```

---

## 4. Layer Assignment (Longest Path)

Assigns each node a column index equal to the longest path from any source node. Guarantees all edges point strictly left-to-right.

```javascript
/**
 * Assigns each node a layer index using the longest-path algorithm.
 * Layer 0 contains all source nodes (in-degree 0 after cycle breaking).
 * Each node's layer is max(predecessor layers) + 1.
 *
 * @param {Array<{id: string}>} nodes
 * @param {Array<{from: string, to: string}>} edges
 * @returns {Map<string, number>} nodeId → layerIndex
 */
function assignLayers(nodes, edges) {
  const safeEdges = breakCycles(nodes, edges);
  const order = topologicalSort(nodes, safeEdges);
  const layer = new Map(nodes.map(n => [n.id, 0]));

  // Build a fast lookup: for each node, which nodes point to it
  const predecessors = new Map(nodes.map(n => [n.id, []]));
  for (const { from, to } of safeEdges) {
    if (predecessors.has(to)) predecessors.get(to).push(from);
  }

  // Process in topological order; each node gets max(pred layers) + 1
  for (const id of order) {
    const preds = predecessors.get(id) ?? [];
    if (preds.length === 0) {
      layer.set(id, 0);
    } else {
      const maxPredLayer = Math.max(...preds.map(p => layer.get(p) ?? 0));
      layer.set(id, maxPredLayer + 1);
    }
  }

  return layer;
}
```

---

## 5. Crossing Minimization (Barycenter Heuristic)

Reorders nodes within each layer to reduce edge crossings. Runs 3 forward + 3 backward passes.

```javascript
/**
 * Groups nodes by layer and sorts them within each layer using the
 * barycenter heuristic to reduce edge crossings.
 * Alternates forward (layer 0→N) and backward (layer N→0) passes.
 *
 * @param {Array<{id: string}>} nodes
 * @param {Array<{from: string, to: string}>} edges
 * @param {Map<string, number>} layerMap - nodeId → layerIndex from assignLayers()
 * @returns {Map<number, string[]>} layerIndex → ordered array of nodeIds
 */
function minimizeCrossings(nodes, edges, layerMap) {
  const safeEdges = breakCycles(nodes, edges);

  // Initialize: group node IDs by layer
  const layerNodes = new Map();
  for (const { id } of nodes) {
    const l = layerMap.get(id) ?? 0;
    if (!layerNodes.has(l)) layerNodes.set(l, []);
    layerNodes.get(l).push(id);
  }

  const maxLayer = Math.max(...layerMap.values());

  // Build neighbor lookups (predecessors and successors per layer boundary)
  const successors   = new Map(nodes.map(n => [n.id, []]));
  const predecessors = new Map(nodes.map(n => [n.id, []]));
  for (const { from, to } of safeEdges) {
    if (successors.has(from))   successors.get(from).push(to);
    if (predecessors.has(to))   predecessors.get(to).push(from);
  }

  /**
   * Computes the barycenter for a node given an ordered reference layer.
   * Barycenter = mean position index of connected nodes in the reference layer.
   * Returns Infinity if the node has no connections (places it last).
   *
   * @param {string} id - Node to score
   * @param {string[]} refOrder - Ordered IDs of the reference layer
   * @param {'pred'|'succ'} direction - Which neighbors to look at
   * @returns {number}
   */
  function barycenter(id, refOrder, direction) {
    const neighbors = direction === 'pred'
      ? predecessors.get(id) ?? []
      : successors.get(id) ?? [];

    const positions = neighbors
      .map(n => refOrder.indexOf(n))
      .filter(i => i >= 0);

    if (positions.length === 0) return Infinity;
    return positions.reduce((a, b) => a + b, 0) / positions.length;
  }

  const PASSES = 3;

  // Forward passes: sort each layer by barycenter relative to previous layer
  for (let pass = 0; pass < PASSES; pass++) {
    for (let l = 1; l <= maxLayer; l++) {
      const current = layerNodes.get(l) ?? [];
      const prev    = layerNodes.get(l - 1) ?? [];
      current.sort((a, b) => barycenter(a, prev, 'pred') - barycenter(b, prev, 'pred'));
      layerNodes.set(l, current);
    }

    // Backward pass
    for (let l = maxLayer - 1; l >= 0; l--) {
      const current = layerNodes.get(l) ?? [];
      const next    = layerNodes.get(l + 1) ?? [];
      current.sort((a, b) => barycenter(a, next, 'succ') - barycenter(b, next, 'succ'));
      layerNodes.set(l, current);
    }
  }

  return layerNodes;
}
```

---

## 6. Position Assignment

Converts layer/rank indices into pixel coordinates. Nodes are vertically centered within each layer.

```javascript
/**
 * Computes the {x, y} pixel position for every node.
 * x is determined by layer index (left-to-right flow).
 * y is determined by rank within the layer (vertically centered).
 *
 * @param {Array<{id: string}>} nodes
 * @param {Map<number, string[]>} layerNodes - layerIndex → ordered nodeIds
 * @param {object} [layout] - Override LAYOUT constants
 * @returns {Map<string, {x: number, y: number}>} nodeId → pixel position (top-left corner)
 */
function assignPositions(nodes, layerNodes, layout = LAYOUT) {
  const {
    NODE_WIDTH, NODE_HEIGHT, LAYER_GAP, NODE_GAP, PADDING
  } = layout;

  const positions = new Map();

  for (const [layerIndex, ids] of layerNodes) {
    const x = PADDING + layerIndex * (NODE_WIDTH + LAYER_GAP);

    // Total height of this column including gaps
    const totalH = ids.length * NODE_HEIGHT + (ids.length - 1) * NODE_GAP;

    // Offset so the column is vertically centered at y=0 (caller can translate)
    const startY = -(totalH / 2);

    ids.forEach((id, rank) => {
      const y = startY + rank * (NODE_HEIGHT + NODE_GAP);
      positions.set(id, { x, y });
    });
  }

  return positions;
}
```

---

## 7. Group Bounds Calculation

Computes a tight bounding rectangle around each group's children, padded on all sides.

```javascript
/**
 * Calculates bounding box for each group based on its member nodes' positions.
 * Adds GROUP_PADDING on all four sides.
 *
 * @param {Array<{id: string, children: string[]}>} groups
 * @param {Map<string, {x: number, y: number}>} positions - from assignPositions()
 * @param {object} [layout] - Override LAYOUT constants
 * @returns {Map<string, {x: number, y: number, width: number, height: number}>}
 */
function calcGroupBounds(groups, positions, layout = LAYOUT) {
  const { NODE_WIDTH, NODE_HEIGHT, GROUP_PADDING } = layout;
  const bounds = new Map();

  for (const group of groups) {
    let minX = Infinity, minY = Infinity;
    let maxX = -Infinity, maxY = -Infinity;

    for (const childId of group.children) {
      const pos = positions.get(childId);
      if (!pos) continue;

      minX = Math.min(minX, pos.x);
      minY = Math.min(minY, pos.y);
      maxX = Math.max(maxX, pos.x + NODE_WIDTH);
      maxY = Math.max(maxY, pos.y + NODE_HEIGHT);
    }

    if (!isFinite(minX)) {
      // Group has no positioned children — skip
      continue;
    }

    bounds.set(group.id, {
      x:      minX - GROUP_PADDING,
      y:      minY - GROUP_PADDING,
      width:  (maxX - minX) + GROUP_PADDING * 2,
      height: (maxY - minY) + GROUP_PADDING * 2
    });
  }

  return bounds;
}
```

---

## 8. Edge Routing (Cubic Bezier)

Generates SVG `d` attribute strings for all edge types. Handles straight, bidirectional, and self-loop cases.

```javascript
/**
 * Returns the center-right anchor point of a node (start of outgoing edge).
 * @param {{x: number, y: number}} pos
 * @param {object} layout
 * @returns {{x: number, y: number}}
 */
function anchorRight(pos, layout = LAYOUT) {
  return {
    x: pos.x + layout.NODE_WIDTH,
    y: pos.y + layout.NODE_HEIGHT / 2
  };
}

/**
 * Returns the center-left anchor point of a node (end of incoming edge).
 * @param {{x: number, y: number}} pos
 * @param {object} layout
 * @returns {{x: number, y: number}}
 */
function anchorLeft(pos, layout = LAYOUT) {
  return {
    x: pos.x,
    y: pos.y + layout.NODE_HEIGHT / 2
  };
}

/**
 * Returns the center-bottom anchor point of a node (used in vertical layout).
 * @param {{x: number, y: number}} pos
 * @param {object} layout
 * @returns {{x: number, y: number}}
 */
function anchorBottom(pos, layout = LAYOUT) {
  return {
    x: pos.x + layout.NODE_WIDTH / 2,
    y: pos.y + layout.NODE_HEIGHT
  };
}

/**
 * Returns the center-top anchor point of a node (used in vertical layout).
 * @param {{x: number, y: number}} pos
 * @param {object} layout
 * @returns {{x: number, y: number}}
 */
function anchorTop(pos, layout = LAYOUT) {
  return {
    x: pos.x + layout.NODE_WIDTH / 2,
    y: pos.y
  };
}

/**
 * Generates the SVG cubic Bezier path `d` string for a single edge.
 *
 * - Horizontal flow (default): right-anchor → left-anchor with horizontal control points
 * - Vertical flow: bottom-anchor → top-anchor with vertical control points
 * - Self-loop: arc above the source node
 * - Bidirectional (both A→B and B→A exist): offset ±4px to avoid overlap
 *
 * @param {{from: string, to: string, type?: string}} edge
 * @param {Map<string, {x: number, y: number}>} positions
 * @param {Array<{from: string, to: string}>} allEdges - Full edge list (for bidirectional detection)
 * @param {'horizontal'|'vertical'} [flow='horizontal']
 * @param {object} [layout]
 * @returns {string} SVG path `d` attribute value
 */
function routeEdge(edge, positions, allEdges, flow = 'horizontal', layout = LAYOUT) {
  const { from, to } = edge;

  // Self-loop: arc above the node
  if (from === to) {
    return routeSelfLoop(positions.get(from), layout);
  }

  const srcPos = positions.get(from);
  const dstPos = positions.get(to);
  if (!srcPos || !dstPos) return '';

  // Bidirectional offset: if the reverse edge also exists, shift ±4px
  const isBidirectional = allEdges.some(e => e.from === to && e.to === from);
  const offset = isBidirectional
    ? (from < to ? -4 : 4) // deterministic direction based on ID sort
    : 0;

  if (flow === 'vertical') {
    return routeVertical(srcPos, dstPos, offset, layout);
  }

  return routeHorizontal(srcPos, dstPos, offset, layout);
}

/**
 * Cubic Bezier for left-to-right (horizontal) flow.
 * Control points are placed at 40% and 60% of the horizontal distance.
 *
 * @param {{x: number, y: number}} srcPos
 * @param {{x: number, y: number}} dstPos
 * @param {number} offset - Vertical pixel offset for bidirectional separation
 * @param {object} layout
 * @returns {string}
 */
function routeHorizontal(srcPos, dstPos, offset, layout = LAYOUT) {
  const src = anchorRight(srcPos, layout);
  const dst = anchorLeft(dstPos, layout);

  src.y += offset;
  dst.y += offset;

  const dx = dst.x - src.x;
  const cp1 = { x: src.x + dx * 0.4, y: src.y };
  const cp2 = { x: dst.x - dx * 0.4, y: dst.y };

  return `M ${src.x} ${src.y} C ${cp1.x} ${cp1.y}, ${cp2.x} ${cp2.y}, ${dst.x} ${dst.y}`;
}

/**
 * Cubic Bezier for top-to-bottom (vertical) flow.
 * Control points are placed at 40% and 60% of the vertical distance.
 *
 * @param {{x: number, y: number}} srcPos
 * @param {{x: number, y: number}} dstPos
 * @param {number} offset - Horizontal pixel offset for bidirectional separation
 * @param {object} layout
 * @returns {string}
 */
function routeVertical(srcPos, dstPos, offset, layout = LAYOUT) {
  const src = anchorBottom(srcPos, layout);
  const dst = anchorTop(dstPos, layout);

  src.x += offset;
  dst.x += offset;

  const dy = dst.y - src.y;
  const cp1 = { x: src.x, y: src.y + dy * 0.4 };
  const cp2 = { x: dst.x, y: dst.y - dy * 0.4 };

  return `M ${src.x} ${src.y} C ${cp1.x} ${cp1.y}, ${cp2.x} ${cp2.y}, ${dst.x} ${dst.y}`;
}

/**
 * Elliptical arc self-loop drawn above the node.
 * Uses SVG arc command (rx=30, ry=20) sweeping 270° counterclockwise.
 *
 * @param {{x: number, y: number}} pos
 * @param {object} layout
 * @returns {string}
 */
function routeSelfLoop(pos, layout = LAYOUT) {
  const cx = pos.x + layout.NODE_WIDTH / 2;
  const top = pos.y;
  const startX = cx - 20;
  const endX   = cx + 20;
  const rx = 30;
  const ry = 20;
  // large-arc-flag=1, sweep-flag=0 (counterclockwise)
  return `M ${startX} ${top} A ${rx} ${ry} 0 1 0 ${endX} ${top}`;
}

/**
 * Generates SVG path `d` strings for all edges in the graph.
 *
 * @param {Array<{from: string, to: string, type?: string}>} edges
 * @param {Map<string, {x: number, y: number}>} positions
 * @param {'horizontal'|'vertical'} [flow='horizontal']
 * @param {object} [layout]
 * @returns {Map<string, string>} `${edge.from}→${edge.to}` → SVG path string
 */
function routeAllEdges(edges, positions, flow = 'horizontal', layout = LAYOUT) {
  const paths = new Map();
  for (const edge of edges) {
    const key = `${edge.from}→${edge.to}`;
    paths.set(key, routeEdge(edge, positions, edges, flow, layout));
  }
  return paths;
}
```

---

## 9. Vertical Layout Variant

Same pipeline as sections 3–8 but with axes swapped: layers run top-to-bottom and nodes fan left-to-right within each layer.

```javascript
/**
 * Assigns pixel positions for a top-to-bottom (vertical) layout.
 * Layer index maps to the Y axis; rank within layer maps to the X axis.
 *
 * @param {Array<{id: string}>} nodes
 * @param {Map<number, string[]>} layerNodes - layerIndex → ordered nodeIds
 * @param {object} [layout]
 * @returns {Map<string, {x: number, y: number}>}
 */
function assignPositionsVertical(nodes, layerNodes, layout = LAYOUT) {
  const {
    NODE_WIDTH, NODE_HEIGHT, LAYER_GAP, NODE_GAP, PADDING
  } = layout;

  const positions = new Map();

  for (const [layerIndex, ids] of layerNodes) {
    // In vertical layout, layer index drives Y; rank drives X
    const y = PADDING + layerIndex * (NODE_HEIGHT + LAYER_GAP);

    const totalW = ids.length * NODE_WIDTH + (ids.length - 1) * NODE_GAP;
    const startX = -(totalW / 2);

    ids.forEach((id, rank) => {
      const x = startX + rank * (NODE_WIDTH + NODE_GAP);
      positions.set(id, { x, y });
    });
  }

  return positions;
}

/**
 * Full vertical layout pipeline.
 * Runs topological sort → layer assignment → crossing minimization →
 * vertical position assignment → group bounds.
 *
 * @param {Array<{id: string}>} nodes
 * @param {Array<{from: string, to: string}>} edges
 * @param {Array<{id: string, children: string[]}>} groups
 * @param {object} [layout]
 * @returns {{ positions: Map, groupBounds: Map, edgePaths: Map }}
 */
function computeVerticalLayout(nodes, edges, groups, layout = LAYOUT) {
  const layerMap   = assignLayers(nodes, edges);
  const layerNodes = minimizeCrossings(nodes, edges, layerMap);
  const positions  = assignPositionsVertical(nodes, layerNodes, layout);
  const groupBounds = calcGroupBounds(groups, positions, layout);
  const edgePaths   = routeAllEdges(edges, positions, 'vertical', layout);

  return { positions, groupBounds, edgePaths };
}
```

---

## 10. Viewport Fitting

Computes the SVG `viewBox` transform needed to fit all content inside a given viewport with uniform padding.

```javascript
/**
 * Calculates the scale and translation needed to fit all nodes and groups
 * within the target viewport dimensions.
 *
 * @param {Map<string, {x: number, y: number}>} positions - From assignPositions()
 * @param {Map<string, {x: number, y: number, width: number, height: number}>} groupBounds
 * @param {number} viewportWidth  - Available width in px
 * @param {number} viewportHeight - Available height in px
 * @param {number} [viewportPadding=40] - Minimum margin inside viewport
 * @param {object} [layout]
 * @returns {{ scale: number, translateX: number, translateY: number, viewBox: string }}
 */
function fitViewport(
  positions,
  groupBounds,
  viewportWidth,
  viewportHeight,
  viewportPadding = 40,
  layout = LAYOUT
) {
  const { NODE_WIDTH, NODE_HEIGHT } = layout;

  let minX = Infinity, minY = Infinity;
  let maxX = -Infinity, maxY = -Infinity;

  // Expand bounding box to include every node
  for (const { x, y } of positions.values()) {
    minX = Math.min(minX, x);
    minY = Math.min(minY, y);
    maxX = Math.max(maxX, x + NODE_WIDTH);
    maxY = Math.max(maxY, y + NODE_HEIGHT);
  }

  // Expand to include group backgrounds
  for (const { x, y, width, height } of groupBounds.values()) {
    minX = Math.min(minX, x);
    minY = Math.min(minY, y);
    maxX = Math.max(maxX, x + width);
    maxY = Math.max(maxY, y + height);
  }

  if (!isFinite(minX)) {
    // Nothing to fit — return identity
    return { scale: 1, translateX: 0, translateY: 0, viewBox: `0 0 ${viewportWidth} ${viewportHeight}` };
  }

  const contentWidth  = maxX - minX;
  const contentHeight = maxY - minY;

  const availableW = viewportWidth  - viewportPadding * 2;
  const availableH = viewportHeight - viewportPadding * 2;

  // Uniform scale — never exceed 1:1 (don't zoom in past native size)
  const scale = Math.min(
    availableW / contentWidth,
    availableH / contentHeight,
    1
  );

  // Center the scaled content in the viewport
  const scaledW = contentWidth  * scale;
  const scaledH = contentHeight * scale;

  const translateX = viewportPadding + (availableW - scaledW) / 2 - minX * scale;
  const translateY = viewportPadding + (availableH - scaledH) / 2 - minY * scale;

  // SVG viewBox that achieves the same result without a CSS transform
  const vbX = minX - viewportPadding / scale;
  const vbY = minY - viewportPadding / scale;
  const vbW = contentWidth  + (viewportPadding * 2) / scale;
  const vbH = contentHeight + (viewportPadding * 2) / scale;

  const viewBox = `${vbX.toFixed(2)} ${vbY.toFixed(2)} ${vbW.toFixed(2)} ${vbH.toFixed(2)}`;

  return { scale, translateX, translateY, viewBox };
}
```

---

## Full Pipeline (Horizontal Layout)

```javascript
/**
 * Runs the complete horizontal auto-layout pipeline.
 * Steps: topological sort → layer assignment → crossing minimization →
 * position assignment → group bounds → edge routing → viewport fit.
 *
 * @param {Array<{id: string}>} nodes
 * @param {Array<{from: string, to: string}>} edges
 * @param {Array<{id: string, children: string[]}>} groups
 * @param {number} viewportWidth
 * @param {number} viewportHeight
 * @param {object} [layout]
 * @returns {{
 *   positions: Map<string, {x: number, y: number}>,
 *   groupBounds: Map<string, {x: number, y: number, width: number, height: number}>,
 *   edgePaths: Map<string, string>,
 *   viewport: { scale: number, translateX: number, translateY: number, viewBox: string }
 * }}
 */
function computeLayout(nodes, edges, groups, viewportWidth, viewportHeight, layout = LAYOUT) {
  const layerMap    = assignLayers(nodes, edges);
  const layerNodes  = minimizeCrossings(nodes, edges, layerMap);
  const positions   = assignPositions(nodes, layerNodes, layout);
  const groupBounds = calcGroupBounds(groups, positions, layout);
  const edgePaths   = routeAllEdges(edges, positions, 'horizontal', layout);
  const viewport    = fitViewport(positions, groupBounds, viewportWidth, viewportHeight, 40, layout);

  return { positions, groupBounds, edgePaths, viewport };
}
```
