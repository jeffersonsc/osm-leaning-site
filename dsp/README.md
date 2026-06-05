# Áudio em Rust — BR

Material de estudo brasileiro sobre áudio digital e Rust, focado em quem quer construir plugins, DSPs e consoles digitais — com linguagem direta, experimentos práticos e foco em quem aprende fazendo.

## Estrutura

Cada módulo é um HTML estático, autocontido, página única. Sem build step, sem framework. CSS compartilhado em arquivo externo pra reduzir duplicação.

```
.
├── styles.css          # CSS compartilhado entre todos os módulos
├── modulo-01.html      # Áudio digital: do som à amostra
├── modulo-02.html      # O bloco e o callback (cpal, latência)
├── modulo-03.html      # Arquiteturas de áudio do SO (ALSA/JACK/PipeWire)
├── modulo-04.html      # Filtros: o biquad (Rust + FAUST, duas trilhas)
├── modulo-05.html      # Fader, smoothing e pan
├── modulo-06.html      # (próximo) Dinâmicas: gate e compressor
...
└── README.md
```

## Como usar

### Local
Abra qualquer arquivo `modulo-*.html` no navegador. O `styles.css` é carregado automaticamente do mesmo diretório.

### Deploy no Vercel
```bash
# Na pasta do repositório
npx vercel
```
Vercel detecta automaticamente que é um site estático.

### Deploy no GitHub Pages
Settings → Pages → Source: Deploy from branch → main → /root.
Acessível em `https://jeffersonsc.github.io/audio-rust-br/modulo-01.html`.

### Deploy no Netlify
Arraste a pasta inteira em app.netlify.com/drop. Pronto.

## Roadmap de módulos

- [x] **01** — Áudio digital: do som à amostra
- [x] **02** — O bloco e o callback (cpal, latência)
- [x] **03** — Arquiteturas de áudio do SO (ALSA/JACK/PipeWire, foco Linux)
- [x] **04** — Filtros: o biquad e seus filhos (trilhas Rust + FAUST)
- [x] **05** — Fader, smoothing e pan (trilhas Rust + FAUST)
- [~] **06** — Dinâmicas: gate e compressor (conceitual)
- [~] **07** — Somando sinais: buses e headroom (conceitual)
- [~] **08** — Threading, lock-free e tempo real (conceitual)
- [~] **09** — Roteamento: o grafo do mixer (conceitual)
- [~] **10** — RTP e AES67: áudio pela rede (conceitual)
- [~] **11** — PTP: o relógio que importa (conceitual)

> **Legenda:** `[x]` implementado com código rodável · `[~]` modelo conceitual completo (esqueleto Rust + decisões arquiteturais), pendente implementação

## Trilhas paralelas

A partir do Módulo 04, conteúdos de DSP aparecem em duas trilhas simultâneas:

- **Trilha Rust** — implementação manual, controle total, fundamentos
- **Trilha FAUST** — DSL pra DSP, prototipagem rápida, geração de Rust otimizado

Ambas são complementares. A filosofia: Rust é o motor, FAUST é o acelerador.

## Filosofia

- **Português brasileiro, direto.** Sem jargão desnecessário, sem pose acadêmica.
- **Prático sobre teórico.** Todo módulo termina em código que roda.
- **Ritmo TDAH-friendly.** Blocos curtos, objetivos claros, experimentos tangíveis.
- **Aberto e evolutivo.** Críticas, correções e contribuições bem-vindas.

## Licença

Material: CC-BY-SA 4.0
Código: MIT

---

Mantido por Jefferson · construindo pra resolver a dor de áudio digital acessível em igrejas brasileiras.
