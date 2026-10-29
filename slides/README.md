# Sandboxing Agentic AI — slides

Two Marp decks accompany the `lab/`: `slides.md` (the main, 200-level deck) and `slides-100.md` (an introductory deck, described below). Live-preview with `npm run preview` (opens a watch server at http://localhost:8080), or build artifacts with `npm run build:html` (produces `slides.html`) and `npm run build:pdf` (produces `slides.pdf`). All commands run Marp through `npx -y @marp-team/marp-cli@^4`, so no global install is needed; `npm install` is optional.

## Introductory deck (`slides-100.md`)

`slides-100.md` covers the same arc for an audience with no Linux or Kubernetes background. Each core idea (agents, sandboxes, kernels, Kubernetes, hardening, isolation runtimes, warm pools) gets a plain-language slide with a picture, then a "100 to 300" slide with three columns that take the idea from the basics to the production details. Terms the 200 and 300 columns rely on, such as cgroup and CNI, get a blue definition box on the slide where they first appear. The deck ends with the infographic from `../infographic/`, the lab commands, and a four-slide glossary.

```bash
npm run build:html:100   # -> slides-100.html
npm run build:pdf:100    # -> slides-100.pdf
```

It uses its own light theme, [`themes/sandbox-light.css`](themes/sandbox-light.css), which adds three highlight boxes and the three-column layout. Use them as HTML blocks, with a blank line after each opening tag so Marp renders the Markdown inside:

```html
<div class="idea">

A question for the room, or the key idea (amber).

</div>
```

`def` (blue) marks a definition or fact to remember, and `hot` (red) marks a warning. For the three-column slides, wrap three `<div>` blocks in `<div class="levels">`.

The pictures are hand-written SVGs in `assets/intro-*.svg`, so edit them in a text editor. They have no Mermaid source.

## Diagrams

Marp core does not render ```mermaid``` fenced blocks (and client-side scripts are stripped by its bespoke template), so the diagrams are **pre-rendered to SVG** and embedded as images:

- Sources live in `diagrams/*.mmd`.
- Rendered output in `assets/*.svg` (referenced from `slides.md` as `![h:…](assets/…​.svg)`), so they appear in `preview`, `slides.html`, and `slides.pdf`.

Edit a diagram by changing its `.mmd`, then regenerate the SVGs:

```bash
npm run build:diagrams   # uses @mermaid-js/mermaid-cli (downloads a headless Chromium on first run)
```

## Theme & avoiding overflow

The main deck (`slides.md`) uses a custom **`sandbox`** theme ([`themes/sandbox.css`](themes/sandbox.css)):
gaia with a smaller base font (30px) and tighter padding, because gaia's defaults
(35px / 70px) overflow dense slides (tables, code, long bullet lists). Authoring
against this theme means new slides fit by default.

- **marp-cli** loads it via `--theme-set themes/sandbox.css` (already in every
  `slides.md` script above; the `:100` scripts load `themes/sandbox-light.css`, and `preview` loads both).
- **VS Code "Marp for VS Code"** loads it via [`.vscode/settings.json`](../.vscode/settings.json)
  (`markdown.marp.themes`). Open the **repo root** in VS Code, trust the workspace,
  and reload the window once so the preview picks up the theme (otherwise the
  preview falls back to default and looks cramped). To preview `slides-100.md` there, add `slides/themes/sandbox-light.css` to the same list.

A handful of the densest slides add a one-line `<style scoped>section{font-size:…}</style>`
nudge on top of the theme. If you add content and a slide overflows, either split
it or drop in the same scoped style. Quick check: `npm run build:html` then eyeball,
or just watch the live preview — if content touches the bottom edge, trim it.
