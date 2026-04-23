# Physics Reference

Exact constants and implementation for the constellation map physics engine.

---

## Constants by Node Type

### Branch Nodes (`type === 'branch'`)

| Parameter | Value | Effect |
|-----------|-------|--------|
| Home anchor spring strength | `0.12` | Pulls node back to its `homeX/homeY` every frame — prevents drift |
| Repulsion scale | `3` | Receives very little push from neighbors; stays put |
| Parent spring stiffness | `0.06` | Tighter connection to root center |
| Velocity damping | `0.55` | Settles ~3× faster than leaves; barely wobbles |

### Sub & Leaf Nodes (`type === 'sub'` or `type === 'leaf'`)

| Parameter | Value | Effect |
|-----------|-------|--------|
| Home anchor spring | none | Floats freely |
| Repulsion scale | `18` | Pushed away from neighbors — creates natural spread |
| Parent spring stiffness | `0.04` | Gentle stretch toward parent |
| Velocity damping | `0.82` | Slow, floaty drift |

### Rest Lengths (parent springs)

| Node type | Rest length |
|-----------|-------------|
| `leaf` | `130px` |
| `sub` | `110px` |
| `branch` | `160px` |

### Soft Boundary

- Margin from canvas edge: `60px`
- Spring strength toward boundary: `0.1`

---

## Physics Function Implementation

```javascript
function physics() {
  const cx = W / 2, cy = H / 2;
  nodes.forEach(n => {
    if (n === dragging || n.type === 'center') return;
    const isBranch = n.type === 'branch';
    let fx = 0, fy = 0;

    // Home anchor — branch nodes only
    if (isBranch && n.homeX != null) {
      fx += (n.homeX - n.x) * 0.12;
      fy += (n.homeY - n.y) * 0.12;
    }

    // Repulsion from all other nodes
    const repScale = isBranch ? 3 : 18;
    nodes.forEach(m => {
      if (m === n) return;
      const dx = n.x - m.x, dy = n.y - m.y, d = Math.hypot(dx, dy) || 1;
      const rep = n.r * m.r * repScale / (d * d);
      fx += dx / d * rep;
      fy += dy / d * rep;
    });

    // Spring to connected parent/child
    edges.forEach(e => {
      if (e.a === n.id || e.b === n.id) {
        const other = nodes.find(x => x.id === (e.a === n.id ? e.b : e.a));
        if (!other) return;
        const dx = other.x - n.x, dy = other.y - n.y, d = Math.hypot(dx, dy) || 1;
        const rest = n.type === 'leaf' ? 130 : n.type === 'sub' ? 110 : 160;
        const stiffness = isBranch ? 0.06 : 0.04;
        const stretch = (d - rest) * stiffness;
        fx += dx / d * stretch;
        fy += dy / d * stretch;
      }
    });

    // Soft boundary
    const margin = 60;
    if (n.x < margin)       fx += (margin - n.x) * 0.1;
    if (n.x > W - margin)   fx -= (n.x - (W - margin)) * 0.1;
    if (n.y < margin)       fy += (margin - n.y) * 0.1;
    if (n.y > H - margin)   fy -= (n.y - (H - margin)) * 0.1;

    // Integrate
    const damping = isBranch ? 0.55 : 0.82;
    n.vx = (n.vx + fx) * damping;
    n.vy = (n.vy + fy) * damping;
    n.x += n.vx;
    n.y += n.vy;
  });
}
```

---

## Branch Node Builder Pattern

When building nodes, always store `homeX` and `homeY` on branch nodes:

```javascript
nodes.push({
  id: b.id,
  label: b.label,
  x: bx, y: by,
  homeX: bx, homeY: by,   // ← required for anchor spring
  vx: 0, vy: 0,
  r: 20,
  type: 'branch',
  branch: b.id,
  pinned: false
});
```

Leaf and sub nodes do **not** need `homeX`/`homeY`.

---

## Tuning Guide

If you need to adjust behavior:

| Goal | Change |
|------|--------|
| Branch nodes even more locked | Increase anchor strength `0.12` → `0.18` |
| Branch nodes allowed to drift slightly | Decrease anchor `0.12` → `0.06` |
| Leaves spread out more | Increase repulsion `18` → `28` |
| Leaves cluster tighter | Decrease repulsion `18` → `10` |
| Everything settles faster | Decrease damping values closer to `0.5` |
| More floaty/bouncy feel | Increase damping values closer to `0.92` |
