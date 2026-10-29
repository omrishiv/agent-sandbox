# Infographic

A one-page summary of the lecture and lab: why agents need a sandbox, containers compared with virtual machines, the lab's breach and containment results, the ladder of isolation runtimes, the four layers of defense, and warm pools.

| File | Use |
| --- | --- |
| `sandboxing-agentic-ai.svg` | The source. Hand-written SVG, so edit it in a text editor. GitHub renders it directly. |
| `sandboxing-agentic-ai.png` | A 2x export (2400 x 3920) for slides, chat, and email. |

The 100-level deck (`slides/slides-100.md`) shows the SVG as its summary slide.

## Style

The colors and type follow the explainer style used in [aarora79.github.io/explainers](https://aarora79.github.io/explainers/self-improving-agent-harnesses/):

| Role | Color |
| --- | --- |
| Background | `#fffdf5` |
| Ink / muted text | `#1d2b2a` / `#52615f` |
| Teal (headings, primary boxes) | `#0f766e`, soft `#e0f2ef` |
| Yellow (rules, the one highlight per section) | `#f5c518`, soft `#fff6cc` |
| Lines / zebra rows | `#e4e0d0` / `#faf8ef` |
| Arrows | `#7c8c8a` |

The only color outside that palette is dark red (`#b42318`) for the BREACH results in the lab table.

## Regenerating the PNG

Edit the SVG, then export it with any headless Chrome or Chromium:

```bash
chrome --headless --hide-scrollbars --force-device-scale-factor=2 \
  --window-size=1200,1960 \
  --screenshot=infographic/sandboxing-agentic-ai.png \
  "file://$PWD/infographic/sandboxing-agentic-ai.svg"
```

If you change the SVG's `height`, change the second number in `--window-size` to match.
