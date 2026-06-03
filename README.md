# OSM Visual Guide · Bilingual

Visual guide on **Open Sound Meter** — measurement and alignment of audio systems.

Guia visual sobre **Open Sound Meter** — medição e alinhamento de sistemas de áudio.

## Structure / Estrutura

```
osm_site/
├── index.html               ← language picker / seletor de idioma
├── style.css                ← shared styles
├── pt/
│   ├── index.html           ← índice PT
│   ├── 01-fase.html
│   ├── 02-coerencia.html
│   ├── 03-atraso.html
│   ├── 04-transfer.html
│   └── 05-fft.html
└── en/
    ├── index.html           ← EN index
    ├── 01-phase.html
    ├── 02-coherence.html
    ├── 03-delay.html
    ├── 04-transfer.html
    └── 05-fft.html
```

## Deploy on Vercel

### Drag & drop (easiest)
1. Extract the zip into a folder
2. Go to [vercel.com/new](https://vercel.com/new)
3. Drag the folder onto the upload area
4. Vercel detects as "Other" (static), click Deploy
5. Site goes live at `https://your-project.vercel.app/` in ~30 seconds

### Via GitHub (recommended)
1. Create a GitHub repo and push the contents
2. At [vercel.com/new](https://vercel.com/new), import the repo
3. Click Deploy — Vercel auto-detects everything
4. Every `git push` triggers automatic redeploy

### Via CLI
```bash
npm i -g vercel
cd osm_site
vercel
```

## Language toggle / Alternador de idioma

Each page has a PT/EN toggle in the top right corner that takes you directly to the equivalent page in the other language.

Cada página tem um alternador PT/EN no canto superior direito que leva diretamente pra página equivalente no outro idioma.

## Customize / Personalizar

- **Colors / Cores**: edit CSS variables at the top of `style.css` (`--navy`, `--terra`, etc.)
- **Content / Conteúdo**: each `.html` is independent, edit what you need
- **Adding pages**: copy an existing one, adjust content and the prev/next navigation in the footer

## Status

### Block 1 / Bloco 1 — Fundamentals / Fundamentos ✅
- 01 — What is phase / O que é fase
- 02 — Coherence / Coerência
- 03 — Delay and delay finder / Atraso e delay finder
- 04 — Transfer function / Função de transferência
- 05 — FFT window and resolution / Janela FFT e resolução

### Block 2 / Bloco 2 — Practical alignment / Alinhamento prático 🚧
- 06 — Vector summation / Soma vetorial
- 07 — Aligning sub + top / Alinhando sub + top
- 08 — Delay zones
- 09 — Phase trace ideal vs real
- 10 — Polarity inversion / Inversão de polaridade
- 11 — Coupling vs comb filtering

### Block 3 / Bloco 3 — Use cases / Casos práticos 🚧
- 12 — Church · full workflow / Igreja · workflow completo
- 13 — Events · quick measurement / Eventos · medição rápida
- 14 — Studio · room response / Estúdio · resposta de sala
- 15 — Most common mistakes / Erros mais comuns
