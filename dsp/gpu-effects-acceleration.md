# Design Doc — Aceleração GPU de Efeitos de Áudio

> **Status**: Experimentação futura. Não compromete arquitetura do motor V1.
> **Criado**: 2026-04
> **Contexto**: Após análise da interface do Violet dMix 128 (8 efeitos room/hall + 2 reverbs distintos), surgiu a ideia de usar GPU consumidora (como a RTX 4050 já disponível no ambiente de desenvolvimento) pra efeitos computacionalmente caros que concorrentes não conseguem oferecer sem hardware DSP dedicado.

---

## Por que isso importa

### A oportunidade competitiva

Nenhum mixer digital pro-audio brasileiro ou internacional de mesma faixa de preço usa GPU consumidora pra efeitos. O padrão da indústria é:

- **YAMAHA/Allen & Heath/Digico**: DSP chips SHARC/TigerSHARC (custo alto, limitado a features que couberam no chip)
- **Violet dMix 128**: FPGA Zynq com HLS (custo muito alto, flexível mas trabalhoso)
- **Fourier Audio transform.go**: CPU x86 + VST plugins (flexível, limitado pela CPU)
- **Waves SoundGrid**: CPU x86 numa rede proprietária (caro, scalability por adicionar servidores)

GPU consumidora moderna (RTX 3050, RTX 4060, até GPUs integradas AMD) oferece **teraflops de processamento paralelo** que em áudio seriam aproveitados pra algoritmos que hoje são considerados caros demais pra produção ao vivo.

### O que GPU faz bem pra áudio

**Embaraçosamente paralelo**:
- Convolução longa (reverbs de catedral com IRs de 5+ segundos)
- Upsampling/downsampling de alta qualidade (filtros FIR longos)
- Síntese aditiva com centenas de parciais
- Banks de osciladores pra sintetizadores
- Análise espectral em tempo real (STFT, wavelets)

**Redes neurais aplicadas a áudio**:
- Denoise neural (RNNoise, NSNet, DeepFilterNet)
- Dereverb neural
- Separação de fontes (Demucs, Spleeter)
- Pitch correction com ML (não-autotune, mais natural)
- Super-resolution (reconstruir agudos perdidos)
- Voice conversion (útil em dublagem/tradução live)

### O que GPU faz mal pra áudio

**Latência de transferência**:
- PCIe round-trip (CPU → GPU → CPU) adiciona ~0.5-2ms mesmo sem processamento
- Inaceitável pra inserts no canal crítico (vocalista monitorando)
- Aceitável pra sends/retornos (reverb num aux, usado pelo operador)

**Serialização de jobs curtos**:
- GPU foi desenhada pra batches grandes, não pra blocos de 128 amostras
- Overhead de launch de kernel pode rivalizar com o processamento
- Precisa agrupar múltiplos blocos ou usar persistent threads

**Contenção com outras aplicações**:
- Se OBS tá fazendo encoding NVENC na mesma GPU, throughput cai
- Driver pode preempt o kernel pra renderizar UI do Windows
- Pior em GPU integrada (compartilha com display)

---

## Os candidatos de tecnologia

### CUDA (NVIDIA)

**A favor**:
- Ecossistema mais maduro pra DSP: cuFFT (FFT otimizada), cuDNN (redes neurais), Thrust (algoritmos paralelos)
- Performance absoluta maior em workload contínuo
- Documentação vasta, exemplos abundantes
- Bindings Rust razoáveis: `cust` (wrapper de CUDA driver API), `rustacuda` (mais antigo)
- Target do mercado brasileiro: ~80% dos PCs gamer/workstation vendidos têm NVIDIA

**Contra**:
- NVIDIA-only. Descarta AMD (~15% do mercado), Intel Arc, Apple Silicon, GPUs integradas
- Instalação exige CUDA Toolkit ou runtime no sistema do cliente (~2GB)
- Licenciamento pra redistribuição é permissivo mas exige atenção
- Não roda em servidores ARM comuns (Raspberry Pi, etc.)

### Vulkan Compute

**A favor**:
- Portátil: NVIDIA, AMD, Intel, mobile
- Sem runtime pesado — driver do sistema basta
- Roda em Linux, Windows, Android
- Futuro-prova: Khronos tá investindo nele

**Contra**:
- Ecossistema DSP muito menos maduro que CUDA
- Você implementa muita coisa do zero (FFT, redes neurais)
- API muito mais verbosa que CUDA
- Performance típica 70-90% da CUDA na mesma GPU NVIDIA

### wgpu-rs (WebGPU em Rust)

**A favor**:
- Cross-platform real: Windows (DX12/Vulkan), macOS (Metal), Linux (Vulkan), Android (Vulkan), iOS (Metal), web (WebGPU)
- API moderna, segura, idiomática pra Rust
- Ativo desenvolvimento, usado pelo Firefox
- Permite um código fonte único pra todas as plataformas

**Contra**:
- Ainda menos maduro que Vulkan puro
- Compute shaders em WGSL (linguagem nova pra aprender)
- Performance overhead em relação a Vulkan/Metal nativos (~10-20%)
- Features avançadas (subgroups, tensor cores) chegando gradualmente

### Metal (macOS/iOS)

**A favor**:
- Performance excelente em Apple Silicon
- API bem desenhada
- Obrigatório se quiser rodar bem em Mac

**Contra**:
- Apple-only
- Você não vai escrever direto — seria via wgpu ou MoltenVK

### OpenCL

**Contra, simplesmente**:
- Tecnologia em declínio
- NVIDIA investe mal (implementação inferior à CUDA na mesma placa)
- Apple deprecated em 2018
- Não é investimento defensável em 2026

### Decisão prática

Arquitetura em Rust com **trait `GpuProcessor`** que abstrai backend. Implementações:

```rust
trait GpuProcessor {
    fn new(config: &Config) -> Result<Self>;
    fn process(&mut self, input: &[f32], output: &mut [f32]) -> Result<()>;
    fn latency(&self) -> usize;
}

// Backends possíveis (feature flags):
// - CudaProcessor (cust crate)
// - WgpuProcessor (wgpu crate)
// - VulkanProcessor (ash crate)
// - CpuFallback (sempre presente)
```

Estratégia recomendada:

1. **Primeiro protótipo em wgpu-rs** — portátil, testa o conceito em qualquer máquina, força disciplina de API abstrata desde o começo
2. **Se performance de wgpu for insuficiente, adicionar backend CUDA** — feature flag opcional, cliente com NVIDIA ativa
3. **Fallback CPU sempre presente** — cliente sem GPU adequada roda em modo degradado (sem efeitos pesados)

---

## Casos de uso priorizados

### Tier S — Diferenciais fortes de marketing

**Reverb de convolução longa (IR de 5-10s)**
- Impressionante visualmente e auditivamente
- CPU não dá conta em produção live sem FFT muito otimizada
- Perfeito pra igreja grande simular acústica de catedral
- IRs disponíveis gratuitamente (OpenAIR, ReverbHall)

**Denoise neural (DeepFilterNet ou RNNoise)**
- Remove ar condicionado, ruído ambiente, vazamento de monitor
- Enorme valor percebido em igrejas com equipamento não ideal
- DeepFilterNet já tem implementação de referência
- Diferencial real — concorrentes tradicionais não têm isso

### Tier A — Valor agregado

**Dereverb (reverb reduction)**
- Recupera voz "lavada" em ambientes reverberantes
- Útil pra igrejas com má acústica

**Separação de fontes em tempo real**
- Muito ambicioso, mas impressionante
- Isolar bateria/baixo/voz de um playback mono
- Útil pra karaokê, ensaios

### Tier B — Nice to have

**Upsampling de alta qualidade**
- Fonte 44.1 toca em pipeline de 48kHz sem artifact

**Análise espectral visual**
- Spectrogram tempo real na UI, com centenas de bins

---

## Arquitetura proposta

```
┌────────────────────────────────────────────────────────────┐
│  Motor principal (Rust, CPU, tempo real)                   │
│                                                             │
│  Audio callback → bus routing → ...                        │
│                                ↓                            │
│                  [ send pra GPU queue ]                    │
│                                ↓                            │
│                     (thread separada)                       │
└────────────────────────────────────────────────────────────┘
                                ↓
┌────────────────────────────────────────────────────────────┐
│  GPU Effects Worker Thread                                 │
│                                                             │
│  Input ring buffer (lock-free)                             │
│        ↓                                                    │
│  Batch accumulator (junta N blocos pra eficiência)         │
│        ↓                                                    │
│  GPU dispatch via trait GpuProcessor                       │
│        ↓                                                    │
│  Output ring buffer (lock-free)                            │
└────────────────────────────────────────────────────────────┘
                                ↓
                    [ volta pra motor principal ]
                                ↓
                    mix no bus de saída → interface
```

**Decisões importantes**:

- GPU **nunca no path crítico** — sempre em send/return paralelo
- Thread separada pra evitar acoplar audio callback com latência de GPU
- Lock-free queues (crate `rtrb` ou `ringbuf`) pra comunicação
- Batch accumulator pra amortizar custo de launch de kernel
- Latência declarada pro usuário (ex: "Reverb de catedral: +4ms") — operador compensa mentalmente

---

## Custos e riscos

### Custo de desenvolvimento
- **Mínimo pra ter algo funcionando**: 2-3 meses full-time de alguém com experiência em GPU compute
- **Production-ready (com UI, fallbacks, testes)**: 6-8 meses
- **Você sozinho com aprendizado**: provavelmente 12+ meses se for a primeira vez com GPU compute

### Custos de manutenção contínua
- Driver updates quebrando kernels (raro em CUDA, comum em Vulkan/wgpu)
- Novos modelos de GPU com features diferentes
- Suporte a bugs específicos de hardware de cliente (GPU antiga, driver desatualizado)

### Riscos de posicionamento
- Se você não **entrega** bem (latência glitchando, crashes em hardware obscuro), vira piada
- Pode assustar comprador conservador que associa GPU a "gaming" e não "profissional"
- Contra-argumento: NVIDIA RTX é padrão em estações de streaming profissional hoje

### Riscos técnicos
- Sincronização de clock entre áudio (sample-accurate) e GPU (milissegundos, não samples)
- Fallback robusto pra quando GPU não responde (driver crash, thermal throttle)
- Hot-swap de efeitos GPU sem glitchar (usuário liga/desliga reverb)

---

## Próximos passos quando retomar

### Fase 0 — Pesquisa (1-2 semanas)
- [ ] Compilar e rodar DeepFilterNet standalone (Rust puro existe)
- [ ] Medir performance de FFT em wgpu-rs vs cuFFT vs rustfft (CPU)
- [ ] Benchmark de convolução longa em GPU vs CPU
- [ ] Ler paper de algoritmos de baixa latência pra GPU (partitioned convolution)

### Fase 1 — Protótipo isolado (1 mês)
- [ ] Implementar reverb de convolução simples em wgpu-rs
- [ ] Medir latência real round-trip CPU↔GPU com blocos de 128 samples
- [ ] Comparar qualidade com reverb CPU equivalente
- [ ] Decidir se wgpu é suficiente ou se precisa CUDA

### Fase 2 — Integração com motor (1-2 meses)
- [ ] Definir trait `GpuProcessor` final
- [ ] Implementar worker thread + ring buffers
- [ ] Hot reload de IR sem glitch
- [ ] Fallback pra CPU quando GPU não disponível

### Fase 3 — Segundo efeito (1 mês)
- [ ] Denoise neural (DeepFilterNet portado)
- [ ] UI de before/after pro operador

### Fase 4 — Beta com usuário real (2-3 meses)
- [ ] Testar com igreja parceira
- [ ] Hardening de edge cases
- [ ] Documentar requisitos de sistema

---

## Referências pra estudo futuro

### Papers relevantes
- "Real-Time Convolution Reverberation on GPU" (Savioja, 2011)
- "Low-Latency Audio on GPU" (vários, DAFx conferences)
- DeepFilterNet paper (Schröter et al.)

### Projetos existentes pra inspiração
- **DeepFilterNet** — denoise neural Rust open source (CPU e GPU)
- **rustfft** / **cuFFT** — FFT libraries
- **Bungie's GPU audio engine** (palestra GDC) — processamento de áudio de jogo em GPU
- **NVIDIA Broadcast** — SDK comercial, denoise + dereverb
- **Brave Browser audio** — experimenta wgpu pra audio

### Crates Rust
- `wgpu` (0.19+) — WebGPU implementation
- `cust` — CUDA bindings
- `ash` — Vulkan bindings raw
- `candle` — framework de redes neurais Rust (Hugging Face)
- `tract` — inference engine Rust (ONNX)

---

## Nota sobre priorização

**Este documento existe pra preservar a ideia, não pra forçar execução.**

Prioridade atual do projeto:
1. V1 do motor em Rust rodando passthrough → DSP clássico → saída RTP/OBS
2. Validação com 1-2 igrejas parceiras
3. Monetização inicial

GPU acceleration é **pós-MVP**. Só faz sentido quando o motor básico estiver estável e o produto tiver usuários reais que justifiquem investir mais 6-12 meses nesse tier de features.

Quando chegar a hora, esse documento serve como ponto de partida pra não começar do zero.
