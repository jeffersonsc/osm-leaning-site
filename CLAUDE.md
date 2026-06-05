# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

**Church Audio Tech** — static bilingual-ish site for church audio operators. Four parallel content tracks, all deployed from the same Vercel project:

- **Trilha 1 / Curso iniciante (`/curso/`)** — beginner course based on `apostila_curso_audio.pdf`. 6 blocos: como o som funciona, níveis de sinal, cabos, impedância, mesa de som, equipamentos além. PT-only.
- **Trilha 2 / Curso equipamentos (`/equipamentos/`)** — practical equipment + installation course. 6 blocos: caixas ativas/passivas, amplificadores, crossover + sub, cabos e soldagem (com tutoriais passo-a-passo XLR/P10/Speakon/RCA), multímetro, sistema completo (rack/snake/multi-zona/aterramento). Reusa `.page.narrow` + `.lesson-prose` do `style.css` raiz, com componentes próprios `.diagrama-wrap`, `.solda-step`, `.problema-checklist`, `.posicao-block`, `.ref-table-lg`. PT-only.
- **Trilha 3 / OSM medição (`/pt/`, `/en/`)** — visual guide on Open Sound Meter (measurement and alignment). 15 infographics + posicionamento físico (16). PT complete, EN only first 5.
- **Trilha 4 / DSP avançado (`/dsp/`)** — "Áudio em Rust — BR": 11 módulos pra construir plugins/DSPs em Rust, com trilhas paralelas Rust + FAUST a partir do módulo 4. Plus `eq-cookbook-pt.html` (RBJ Cookbook em PT) e `referencias.html`. **Estilo visual próprio** (warm-dark `#0a0908`/`#d4704a`, Fraunces + Inter Tight + JetBrains Mono) — não usa `style.css` raiz. Cada HTML é autocontido com `<style>` inline; o `styles.css` da pasta existe como referência mas não é carregado.

Plus `/materiais/` for PDF downloads (apostila + checklist for the in-person consult visits).

Pure HTML + CSS + vanilla JS, no build step. Deployed on Vercel.

## Running locally

There is no build, package manager, or test suite. Serve the directory:

```bash
python3 -m http.server 8000      # then visit localhost:8000
# or
npx serve .
```

Deploy: `vercel` (the `.vercel/` dir is already linked; Vercel treats this as a static "Other" project).

## Architecture

The site is a flat collection of standalone HTML pages — there is no router, no shared layout template, no JS framework. The root `index.html` is the hub showing both tracks. Each page is self-contained and must hand-maintain its own:

1. **Crumb** at the top (or `.lang-toggle` on OSM trilha pages) — manual back-link.
2. **Footer prev/next nav** (`.nav-footer`) — hardcoded links to neighbouring pages within the same track. Disabled links use class `nav-disabled`.
3. **Bloco/Infográfico counter** in the header label (e.g. `Bloco 1 / 6`, `Infográfico 01 / 15`).

All visual styling lives in the single root `style.css`. Pages reference it as `../style.css` (or `style.css` from root).

### Tracks differ in tone and structure

- **Curso pages** use `.page.narrow` (820px max) and `.lesson-prose` (font 17px, line-height 1.7) — long-form reading. Components: `.tip-box` (amber, 💡), `.warn-box` (red, ⚠), `.stat-grid` (3 colored stat cards), `.ref-table` (signal levels, connector reference). Inspired by the apostila PDF layout.
- **OSM pages** use full-width `.page` (1200px) with `.grid-2`/`.grid-2-bottom` and dense infographic components: `.callout`, `.chart-wrap`, `.coh-scale`, `.steps`. Designed for visual density, not reading.

When adding a new curso page, copy `curso/01-como-som-funciona.html` as template. When adding a new OSM page, copy `pt/01-fase.html` (or the EN counterpart).

### OSM bilingual filename convention

PT files use Portuguese terms (`01-fase.html`, `02-coerencia.html`, `03-atraso.html`) and EN uses English (`01-phase.html`, `02-coherence.html`, `03-delay.html`). When adding/renaming a page, update the `.lang-toggle` on **both** language counterparts.

The curso track is PT-only — no EN counterparts to maintain.

### SVG charts

Inline `<svg>` with an `id`, populated by a vanilla-JS IIFE at the bottom of the same page. The script generates `<path>` elements with CSS classes from `style.css` (`.curve.navy`, `.curve.terra`, `.zero-line`, `.axis`, `.grid-line`, `.axis-text`). For more illustrative diagrams (like `curso/01`'s "caminho do som"), use emoji icons + arrow lines in plain SVG markup, not heavy procedural drawing.

### Theme tokens

`style.css` defines a dark theme via CSS variables at `:root` (`--bg`, `--navy`, `--terra`, `--teal`, `--amber`, `--green`, `--red`, plus tints). Reuse these tokens; do not hardcode colors in page-level styles or SVG.

### Responsive

Breakpoints in `style.css`: 800px (grids collapse to 1 column) and 640px (typography scales down, paddings tighten, `.nav-footer .center` hides, `.info-header` stacks). Test changes at 375px (mobile), 768px (tablet), 1280px (desktop) before shipping.

## Content status

- **Curso (Trilha 1)**: complete. All 6 blocos plus `e-agora.html` (encerramento). Sourced from `apostila_curso_audio.pdf` in `~/Documents/Consultoria/Curso de Audio/`.
- **OSM (Trilha 2)**: Block 1 (pages 01–05, Fundamentals) complete in both languages. Blocks 2 (06–11) and 3 (12–15) are placeholders on the index pages — no HTML files exist for them yet. Source images for these are in `~/Documents/Consultoria/Materiais/PT/OSM-img/Images novas/` (75 slides) and `OSM-v1.5/` (57 screenshots).
- **Materiais**: apostila + checklist PDFs in `/materiais/`. The page is prepared to receive more PDFs later (a future OSM training PDF, mentioned by user).
