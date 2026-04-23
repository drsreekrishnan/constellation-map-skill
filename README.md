# 🌌 Constellation Map Skill

A Claude skill that transforms any **mind map image** into a stunning, physics-based interactive constellation visualization — rendered as a single self-contained HTML file.

---

## ✨ Demo

Upload a mind map image and ask Claude to *"make this interactive"* or *"turn this into a constellation map"* — Claude will parse the structure, assign branch colors, and generate a fully interactive HTML file you can open in any browser.

**Features:**
- 🌠 Dark-space aesthetic with 200 twinkling background stars
- 🪐 Physics simulation — nodes float, bounce, and settle naturally
- 🎨 Auto color-coded branches (up to 6 colors, cycling if more)
- 🖱️ Drag any node, click a branch to focus it, click root to reset
- 📱 Touch-friendly — works on mobile too
- 💾 Zero dependencies — one `.html` file, open anywhere

---

## 🚀 Installation

### Claude.ai (Web / Desktop / Mobile)

1. Go to **Settings → Capabilities → Skills**
2. Click **Add Skill**
3. Upload or paste the contents of `SKILL.md`

### Claude Code (CLI)

```bash
# Clone this repo and copy the skill folder
git clone https://github.com/drsreekrishnan/constellation-map-skill
cp -r constellation-map-skill/.claude/skills/constellation-map ~/.claude/skills/
```

Or install directly:

```bash
npx skills add drsreekrishnan/constellation-map-skill
```

---

## 🧠 How to Use

Once installed, just upload a mind map image (PNG or JPG) and say any of:

- *"Turn this into a constellation map"*
- *"Make this interactive"*
- *"Visualize this as a star map"*
- *"Animate this mind map"*

Claude will:
1. Parse all nodes and connections from your image
2. Show you a JSON summary to confirm the structure
3. Generate a complete `.html` file with the constellation visualization

---

## 📁 Repo Structure

```
constellation-map-skill/
├── skills/
│   └── constellation-map/
│       ├── SKILL.md                  # Main skill instructions (Claude reads this)
│       └── references/
│           └── physics.md            # Physics engine constants & implementation
├── README.md                         # This file
└── LICENSE                           # MIT License
```

---

## 🎛️ What the Output Looks Like

| Element | Behavior |
|---|---|
| **Root node** | White, centered, always on top |
| **Branch nodes** | Color-coded, anchored near home position |
| **Sub nodes** | Lighter tint of parent branch color |
| **Leaf nodes** | Lightest tint, float freely with physics |
| **Edges** | Solid for branches, dashed for leaves |
| **Focus mode** | Click a branch to dim all others |
| **Tooltip** | Hover any node for label + type info |

---

## 🛠️ Compatibility

This skill follows the [Agent Skills open standard](https://code.claude.com/docs/en/skills) and works with:

- ✅ Claude.ai (Web, Desktop, Mobile)
- ✅ Claude Code (CLI)

---

## 📄 License

MIT — see [LICENSE](./LICENSE) for details. Free to use, modify, and redistribute with attribution.

---

## 🙏 Credits

Built for [Claude](https://claude.ai) by Anthropic. Skill authored by Sreekrishnan Nair.
