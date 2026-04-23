---
name: constellation-map
description: >
  Transforms a mind map image (PNG, JPG) into a fully interactive dark-space
  constellation map visualization — a single self-contained HTML file with
  physics-based floating nodes, color-coded branches, drag/focus interactions,
  and twinkling background stars. Use this skill whenever the user uploads or
  references a mind map, concept map, hierarchy diagram, or any tree-structured
  image and wants it turned into an interactive visual, constellation, star map,
  or node graph. Also trigger when the user asks to "visualize", "animate", or
  "make interactive" any hierarchical PNG or diagram. Even if the user just says
  "make this interactive" alongside an image with a tree structure — use this skill.
---

# Constellation Map Skill

Converts a mind map PNG into a physics-based interactive HTML constellation visualization.

---

## Workflow

Follow these four steps in order. Do not skip the JSON confirmation step.

---

### Step 1 — Parse the Image

Carefully read every node and connector in the uploaded image. Extract:

- **Root node** — the central/leftmost topic
- **Branch nodes** — direct children of root (level 1)
- **Sub nodes** — children of branch nodes (level 2), if present
- **Leaf nodes** — outermost nodes with no further children

Preserve all label text exactly as written in the image.

Output a JSON summary before writing any code:

```json
{
  "root": "Root Label",
  "branches": [
    {
      "id": "b1",
      "label": "Branch Label",
      "kids": ["Leaf 1", "Leaf 2"]
    },
    {
      "id": "b2",
      "label": "Branch Label",
      "subs": [
        { "id": "b2_s0", "label": "Sub Label", "kids": ["Leaf A", "Leaf B"] }
      ]
    }
  ]
}
```

Ask the user to confirm the structure is correct before proceeding.

---

### Step 2 — Assign Branch Colors

Assign one color slot per branch, cycling through the palette if there are more than 6:

| Slot | Fill | Glow | Label text |
|------|------|------|------------|
| 1 | `#c084fc` | `#a855f7` | `#1a0030` |
| 2 | `#34d399` | `#10b981` | `#002918` |
| 3 | `#fb923c` | `#f97316` | `#2d0a00` |
| 4 | `#60a5fa` | `#3b82f6` | `#001638` |
| 5 | `#fbbf24` | `#f59e0b` | `#2a1400` |
| 6 | `#f87171` | `#ef4444` | `#2d0000` |

- Sub nodes: lighter tint of the branch fill
- Leaf nodes: even lighter tint
- Root node: fill `#ffffff`, glow `#a0c8ff`, label `#03060f`

---

### Step 3 — Build the HTML File

Produce a **single self-contained HTML file**. No external dependencies.

#### Canvas & Container

```html
<div style="background:#03060f;border-radius:16px;overflow:hidden;
            position:relative;user-select:none;">
  <canvas id="c" style="display:block;width:100%;touch-action:none;"></canvas>
  <div id="tip" style="position:absolute;pointer-events:none;display:none;
       background:rgba(5,10,28,0.95);border:1px solid rgba(255,255,255,0.18);
       border-radius:10px;padding:9px 14px;z-index:20;"></div>
  <div style="position:absolute;bottom:10px;left:50%;transform:translateX(-50%);
       color:rgba(255,255,255,0.35);font-size:11px;font-family:sans-serif;
       pointer-events:none;">drag any star · click branch to focus</div>
</div>
```

#### Canvas Setup

- Size: full container width, height = `max(600px, width * 0.8)`
- HiDPI support via `devicePixelRatio`
- Background: `#03060f`
- 200 background stars with sine-wave twinkle animation

#### Node Sizes

| Type | Radius |
|------|--------|
| root | 30px |
| branch | 20px |
| sub | 13px |
| leaf | 8–9px |

All non-leaf nodes get an inner bright core: `rgba(255,255,255,0.35)` circle at `r * 0.4`.

#### Labels

- Branch nodes: short label inside node (truncate >12 chars with `…`), full label floating outside
- All non-root nodes: floating label in a pill `rgba(3,6,15,0.7)` background, `borderRadius:4`, positioned radially outward from canvas center
- Hover: full opacity; non-hovered branches: 0.7 opacity; focused-out: 0.1 opacity

#### Edges

- Branch→root: `lineWidth 2`, solid, `shadowBlur 6`
- Sub→branch: `lineWidth 1`, solid
- Leaf→parent: `lineWidth 1`, dashed `[4,5]`
- Color: branch glow color, `globalAlpha 0.55` (active) or `0.06` (dimmed)

#### Node Layout

Place nodes using polar coordinates from canvas center `(W/2, H/2)`:

- Branch nodes: evenly spaced angles `(i / numBranches) * 2π`, radius `~0.28 * min(W,H)`. Use `ax/ay` offsets scaled by `W/900` and `H/700`.
- Sub nodes: fan from parent at `~0.26 * min(W,H)` from center
- Leaf nodes: fan further at `0.28–0.43 * min(W,H)` depending on depth
- **Store `homeX` and `homeY` on every branch node at build time**

> ⚠️ **Critical — root node JavaScript `type` must be `'center'`**, not `'root'`.
> The Step 1 JSON schema uses the key `"root"` for readability, but in the generated
> JavaScript the root node object **must** be created with `type: 'center'`.
> The physics function guards with `n.type === 'center'` to exempt the root from all forces.
> Using any other string will cause the root to drift under repulsion and break the entire layout.

#### Physics

Read `references/physics.md` for all exact constants and the full physics function implementation.

#### Interactions

| Action | Behavior |
|--------|----------|
| `mousedown` / `touchstart` on node | Begin drag, store offset |
| `mousemove` / `touchmove` | Move dragged node, zero its velocity |
| `mouseup` with movement < 5px | Toggle branch focus |
| Click root | Clear focus |
| `mouseleave` | Hide tooltip, stop drag |
| `touchend` | Stop drag |

> ⚠️ **Critical — physics loop must keep running during drag.**
> The animation loop (`requestAnimationFrame`) must never pause or skip physics while a node
> is being dragged. The physics function already skips `n === dragging`, so the dragged node
> is positioned directly by the mouse. All other nodes — especially leaves and subs — must
> continue receiving spring forces every frame so they follow the moved branch naturally.
> If physics stops during drag, connected leaf/sub nodes will freeze completely and not follow.

**Tooltip on hover:** Node label (bold 12px white) + type and parent branch name (10px, 45% white). Hidden while dragging.

#### Draw Order Per Frame

1. Clear + fill `#03060f`
2. Twinkling background stars
3. All edges (with glow)
4. Nodes sorted: `leaf → sub → branch → root` (root always on top)

---

### Step 4 — Output & Confirm

- Deliver a single `.html` file
- Confirm all branch labels from the parsed JSON are present in the output
- Mention that branch nodes are anchored (won't drift) and leaves float freely

---

## Quality Checklist

Before finalising, verify:
- [ ] All node labels from Step 1 JSON are present
- [ ] Branch nodes stay near their home positions (anchor spring active)
- [ ] Leaves and subs float gently without flying off screen
- [ ] Focus mode dims other branches correctly
- [ ] Tooltip shows on hover
- [ ] Drag works on mouse and touch
- [ ] Canvas resizes correctly on window resize
