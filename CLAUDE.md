# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-page marketing site for "Record Express" (a Brussels courier company). There is no build system, package manager, linter, or test suite — just three files:

- `index.html` — the entire site: markup, styles, content, and component logic
- `support.js` — generated runtime (the "dc-runtime"). **Do not edit**; it was built elsewhere (`cd dc-runtime && bun run build`) and the source is not in this repo
- `readme.md` — the project goal: create a git repo, deploy via a Cloudflare Worker, and serve the site at `test.webhelpagency.com` (Cloudflare configuration to be done through the Chrome extension)

## Running locally

Serve statically and open in a browser, e.g.:

```
python3 -m http.server 8000
```

Note: the runtime loads React 18 from unpkg at runtime, so it needs network access to render.

## How index.html works

`index.html` is not plain HTML — it's a "DC document" interpreted client-side by `support.js`:

- All page markup lives inside a single `<x-dc>` element. A `<helmet>` block at the top holds `<link>`/`<style>` tags for the document head.
- `{{ name }}` interpolations are resolved from the object returned by `renderVals()` in the component script. This includes attribute values (styles, event handlers like `onClick="{{ onQuote }}"`, input `value`/`onChange`).
- `<sc-for list="{{ items }}" as="item">` repeats its child for each array element (`hint-placeholder-count` controls skeleton placeholders while streaming).
- `style-hover="..."` declares hover styles inline (handled by the runtime; there are no CSS classes — the whole page is styled with inline `style` attributes).
- `data-reveal` elements get a scroll-triggered fade-in via an IntersectionObserver set up in `componentDidMount`.
- The `<script type="text/x-dc" data-dc-script>` at the bottom must define `class Component extends DCLogic`. It is React-like: `state`/`setState`, `componentDidMount`/`componentWillUnmount`, plus `renderVals()` which supplies every template value (all page content — services, sectors, steps, testimonials — is data in this method, not in the markup).
- The script tag's `data-props` attribute is JSON declaring editor-configurable props (`accentColor`, `animatedRoutes`), read in `renderVals()` via `this.props`.

When editing the page: content changes usually go in the `renderVals()` data arrays; layout/style changes go in the inline-styled markup; interactive behavior goes in the Component class.
