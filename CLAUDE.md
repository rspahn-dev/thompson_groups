# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development

No build step. Open `index.html` directly in a browser or serve with any static file server:

```
python -m http.server 8080
```

## Architecture

The entire application is a single `index.html` file with inline `<style>` and `<script>` blocks — no external dependencies, no modules, no build tools.

### Key layout constants (top of `<script>`)

`SS` (strand spacing), `LH` (level height), `NR` (node radius), `PAD` (canvas padding), `BRAID_BASE/STEP`, `ARROW_GAP` — changing these rescales all SVG rendering globally.

### State model

Each of the two element panels (`E₁`, `E₂`) has its own `ST[0]` / `ST[1]` state object (`makeState()`). A third `resultST` holds the computed product. State contains:
- `reg`: `Map<id, node>` — the binary tree node registry
- `roots1` / `roots2`: root IDs for the top and bottom trees
- `perm`: permutation array (leaf reordering for V/nV)
- `braid`: list of crossing objects `{i, sign}` (for bV)

### Rendering pipeline

`render(st, svgId)` is the main entry point — it recomputes layout from state and redraws the SVG from scratch on every interaction. Layout math lives in `layoutTree()` (places tree nodes on a pixel grid), and braid rendering is handled separately with its own zone height (`braidH`).

### Interaction

Two global drag states (`_drag` for strand crossings, `_resizeDrag` for the braid-zone resize handle) are tracked at module scope. All SVG pointer events flow through handlers registered inside `render()` — they are re-bound on every redraw.

### Multiplication

`compute()` runs the Thompson group product: it merges the bottom tree of `E₁` with the top tree of `E₂` via repeated carets, composes permutations, and concatenates braid words. The result is rendered into `#svg-result`.
