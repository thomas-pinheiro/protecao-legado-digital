# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Academic presentation for a Digital Law course (Bacharelado em Sistemas de Informação, 4º período). The topic is **Proteção e Legado Digital** — what happens to digital assets (accounts, game libraries, image/likeness) after death under Brazilian law.

## Running the presentation

The engine loads slides via `fetch()`, so a static server is required (no `file://`):

```
npx serve .
```

Or use the VS Code Live Server extension. Then open `http://localhost:3000` (or whichever port `serve` picks).

**Keyboard shortcuts (built into the presentation):**
- `← →` / `Space` / `PageUp PageDown` — navigate slides
- `T` — toggle table of contents
- `N` — toggle presenter notes
- `F` — toggle fullscreen
- `Home` / `End` — jump to first/last slide
- Touch swipe (50px threshold) also works
- `← →` also navigate images inside an open lightbox

## Architecture

Everything lives in `index.html` — a zero-dependency, self-contained presentation engine:

**Slide markup:** Each slide is a `<section class="slide">` with `data-title` (used in the TOC) and `data-notes` (presenter notes) attributes. The active slide gets the `.active` class; the JS engine adds `slideIn` CSS animation on each transition.

**Critical parsing rule:** The engine uses `tmp.firstElementChild` to extract each slide fragment. The `<section>` **must** be the first element in the file — never put a `<style>` or any other tag before it. Inline `<style>` blocks must go inside the `<section>`.

**Viewport scaling:** The presentation renders at a fixed 1280×720 canvas (`#scaler`) that is CSS-scaled to fill any screen via `Math.min(innerWidth/1280, innerHeight/720)`. This means layout is pixel-perfect regardless of browser window size — edit styles at 1280×720 coordinates.

**Block color themes:** Slides are grouped into thematic blocks via CSS classes on the `<section>`:
- `.blk-dir` — indigo (Conceito/Direito block)
- `.blk-game` — cyan (Games block)
- `.blk-ia` — violet (IA e imagem pós-morte block)
- `.blk-end` — green (Conclusão block)

Each block class overrides `.eyebrow`, `.casebox`, and `.tl` colors. The default (no block class) uses the global indigo accent.

**CSS design system:** All colors and radii are CSS custom properties on `:root` (`--indigo`, `--cyan`, `--violet`, `--amber`, etc.). Reusable layout components: `.grid.c2/.c3/.c4` (CSS grid), `.card`, `.cols`/`.col.good`/`.col.warn`, `.casebox`, `.tl` (timeline), `.stats`/`.stat`, `.crit`, `.topics`.

**Step reveals:** Any element with class `.step` inside a slide is hidden until the user advances within that slide. The JS tracks `step` count per slide and toggles `.show`.

**Slide files:** each slide lives in `slides/NN-nome.html` as a standalone `<section class="slide">` fragment. The engine fetches them in order at startup via `Promise.all`, parses each fragment, and appends it to `#slides-host` inside `#scaler`. To edit a slide, open only its file — nothing else is affected. To add a slide, create the file and add its path to the `SLIDE_FILES` array in `index.html`.

**Supporting files:**
- `roteiro/Roteiro_Slides_Protecao_e_Legado_Digital.md` — full speaker script with "Tela / Fala / Fonte" notes per slide
- `Protecao_e_Legado_Digital_Texto_Completo.md` — complete essay version of the content (source of truth for facts and legal citations)
- `roteiro/v1.json` — legacy JSON draft, not loaded at runtime

## Interactive components

### Tooltips (`.tip`)

Wrap any term in a `.tip` span with `data-tip` to show a definition on hover or click:

```html
<span class="tip" data-tip="Definição do termo aqui.">termo</span>
```

- Hover → tooltip appears above; click → toggle (touch support)
- Clicking outside closes all open tooltips
- **Flex container rule (critical):** `.topics li` and `.note` are `display:flex`. Any `.tip` span that is a direct child of a flex container becomes a flex item and receives `gap:15px` spacing, breaking layout. Always wrap the surrounding text in a `<span>` so the entire text block is one flex item:

```html
<!-- correct -->
<li><span class="mk">🧬</span><span>Texto com <span class="tip" data-tip="...">termo</span> aqui.</span></li>

<!-- wrong — gap breaks layout -->
<li><span class="mk">🧬</span> Texto com <span class="tip" data-tip="...">termo</span> aqui.</li>
```

The same rule applies to `<br>` tags inside `.topics li` or `.note`.

### Image lightbox (`.img-trigger`)

Use `.img-gallery` + `.img-thumb` to show thumbnails that expand on click:

```html
<div class="img-gallery">
  <figure class="img-thumb">
    <img class="img-trigger"
         src="assets/minha-imagem.png"
         data-src="assets/minha-imagem.png"
         data-alt="Descrição da imagem"
         data-caption="Legenda exibida abaixo da imagem no lightbox.">
    <figcaption>Título<br>abaixo da thumbnail</figcaption>
  </figure>
</div>
```

- Thumbnails render at 160×100px with label below
- Click opens lightbox fullscreen with caption and close button
- Multiple images in the same `.img-gallery` get ‹ › navigation arrows automatically
- More than 3 images in the same gallery also shows dot indicators
- `Esc` or click outside closes; `← →` keys navigate between images
- Image files go in `assets/` at the project root

### `.note` with mixed content

`.note` is `display:flex`. If the text after the `.b` badge contains any inline element (`.tip`, `<strong>`, etc.) that could become a flex item, wrap everything after the badge in a `<span>`:

```html
<div class="note"><span class="b">Label</span><span>Texto com <span class="tip" data-tip="...">termo</span> aqui.</span></div>
```
