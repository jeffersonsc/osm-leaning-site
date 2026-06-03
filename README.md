# Church Audio Tech

Site educacional sobre áudio para igrejas, em português, com duas trilhas de aprendizado e um manual traduzido do Open Sound Meter.

🌐 **Produção:** [igreja-audio-tech.vercel.app](https://igreja-audio-tech.vercel.app/)

## Para quem é

- **Operadores de igreja** que querem entender de verdade o que fazem na mesa de som — do "som é ar se mexendo" até a mesa digital.
- **Técnicos intermediários** que querem aprender medição e alinhamento de sistemas com o Open Sound Meter (OSM).

## As três trilhas

### 1. Curso para operadores de igreja `/curso/`
Conteúdo iniciante, 6 blocos baseados na apostila do curso presencial:

1. Como o som funciona (frequência, dB, caminho do som)
2. Níveis de sinal (mic, linha, alto-falante, ganho, clip)
3. Cabos e conectores (XLR, P10, balanceado vs desbalanceado)
4. Impedância e potência (Ω, série/paralelo, microfonia)
5. A mesa de som por dentro (ordem dos controles, sequência prática)
6. O que existe além (mesa digital, GEQ, crossover, DSP)

### 2. Open Sound Meter — Conceitos `/pt/`
15 infográficos visuais sobre medição e alinhamento, em PT (5 originais também em EN):

- **Fundamentos** (01–05): fase, coerência, atraso, função de transferência, FFT.
- **Alinhamento prático** (06–11): soma vetorial, sub+top, delay zones, phase trace, polaridade, coupling vs comb.
- **Casos práticos** (12–15): workflow de igreja, eventos ao vivo, estúdio, erros mais comuns.

### 3. Open Sound Meter — Manual em português `/pt/manual/`
Tradução guiada do manual oficial do OSM v1.5, 8 páginas com screenshots originais e legendas em PT:

1. Instalação e primeiros passos
2. A tela principal e os modos (Single/Double/Three)
3. Os 9 tipos de gráfico (RTA, Spectrum, Magnitude, Phase, Coherence, Impulse, Step, Group Delay, Spectrogram)
4. Barra direita: Generator e Measurements
5. Barra direita: Groups e Stores
6. Barra inferior de controles
7. Equalizer, SPL e Numeric
8. Math, Remote Control e menus

## Materiais para download `/materiais/`

- `apostila-curso-audio.pdf` — apostila completa do curso (17 páginas).
- `checklist-visita-igreja.pdf` — formulário de inventário da igreja para visita técnica.

## Estrutura de arquivos

```
osm_site_v2/
├── index.html               ← hub raiz com 2 trilhas
├── style.css                ← estilos compartilhados (tema escuro + responsivo)
├── CLAUDE.md                ← notas técnicas para futuras edições
├── curso/                   ← trilha iniciante (PT)
│   ├── index.html
│   ├── 01-como-som-funciona.html
│   ├── 02-niveis-de-sinal.html
│   ├── 03-cabos-e-conectores.html
│   ├── 04-impedancia-e-potencia.html
│   ├── 05-mesa-de-som.html
│   ├── 06-alem-da-mesa.html
│   └── e-agora.html
├── pt/                      ← OSM conceitos (PT)
│   ├── index.html
│   ├── 01-fase.html ... 15-erros-comuns.html
│   └── manual/              ← manual OSM v1.5 traduzido
│       ├── index.html
│       ├── 01-instalacao.html ... 08-math-remote-menus.html
│       └── img/             ← screenshots originais do manual
├── en/                      ← OSM conceitos (EN, só blocos 01-05)
└── materiais/               ← PDFs para download
```

## Desenvolvimento

Site 100% estático, sem build, sem dependências, sem framework. Para rodar localmente:

```bash
python3 -m http.server 8000
# ou
npx serve .
```

Abra `http://localhost:8000`.

### Como editar

- **Cores e tipografia:** variáveis CSS no topo de `style.css` (`--bg`, `--terra`, `--navy`, etc).
- **Conteúdo de uma página:** edita o `.html` direto. Cada página é independente.
- **Nova página em uma trilha:** copia uma existente da mesma trilha, ajusta conteúdo + `nav-footer` prev/next + cards na landing.
- **Responsivo:** breakpoints em 640px e 800px no `style.css`. Testa em mobile/tablet/desktop antes de fazer deploy.

## Deploy

Configurado para Vercel via `.vercel/`. Cada `git push` na master gera deploy automático em produção.

Para deploy manual via CLI:

```bash
npm i -g vercel
vercel
```

## Licença e atribuições

Código e conteúdo originais sob licença [MIT](./LICENSE) — veja o arquivo.

**Atribuições terceiros:**

- **Open Sound Meter** é desenvolvido por Pavel Smokotnin sob licença GPL v3. Os screenshots da interface usados em `pt/manual/img/` vêm do manual oficial v1.5, reproduzidos para fins de documentação e tradução para português. Veja [opensoundmeter.com](https://opensoundmeter.com/).
- A apostila e o checklist em `/materiais/` foram produzidos como material de apoio do curso presencial.

## Autor

**Jefferson Silva** — Consultor de áudio com foco em sistemas de igreja.

Para sugestões, correções ou contribuições, abra uma issue ou PR no [repositório do GitHub](https://github.com/jeffersonsc/osm-leaning-site).
