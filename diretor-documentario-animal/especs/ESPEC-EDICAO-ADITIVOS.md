# ESPEC — ADITIVOS DE EDIÇÃO (acumulativa, em coleta)

> **Status:** 🔴 **FECHADA PARA IMPLEMENTAÇÃO — AGUARDANDO GO EXPLÍCITO DO OPERADOR.**
> A lista de itens está completa e revisada (04/08), **e as perguntas A/B/D foram
> respondidas pelo operador** (registradas no relatório §4 e no Item 2). O 1º GO cobre os
> **itens 2-5** (passos 0-5 do relatório); o **Item 1 (transições) ficou para rodada
> própria** — regra de frequência adiada pelo operador. **NENHUMA LINHA DE CÓDIGO pode ser
> escrita a partir desta espec sem o operador dizer GO.** Ele revisa antes.
>
> **Como implementar:** siga o [RELATORIO-IMPLEMENTACAO-EDICAO.md](RELATORIO-IMPLEMENTACAO-EDICAO.md)
> — ordem dos passos, arquivos exatos, erros a evitar e plano de validação.
>
> **Espec IRMÃ (separada):** [ESPEC-ACERVO-40-VARIANTES.md](ESPEC-ACERVO-40-VARIANTES.md)
> — as 40 variantes do acervo seguem em refino pelo operador, com cronograma próprio.
>
> Ponto de partida: tag `diretor-doc-v1-validado` (Amazônia + Austrália + África aprovados).
> **Regra-mãe desta espec:** nada aqui pode alterar o que já está validado. Todo item entra
> **aditivo e gated** (knob no preset; ausente = comportamento atual byte a byte).
>
> Contexto do operador: *"eu também sou editor de vídeo, e sei que hoje não dá para fazer
> edição automática com o mesmo nível de complexidade que fazemos manualmente"* — portanto o
> objetivo é ampliar o REPERTÓRIO do diretor (recursos aplicados por regra), não replicar
> julgamento fino de editor.

---

## 📋 ESTADO DA IMPLEMENTAÇÃO (o que já está no código × o que está só registrado)

> Tabela de controle: na hora do GO, tudo que estiver como **registrado** precisa ser
> implementado. Nada some.

| # | Item | Estado |
|---|---|---|
| 1 | Transições Apagão + Brilho (+ SFX shutter 0.15) | 📋 **PRONTO PARA IMPLEMENTAR** — ficou fora do 1º GO, mas a regra de frequência foi **fechada em 04/08** (ver Item 1). Aguarda GO da rodada própria |
| 2 | `film`: tremor removido · borda 8px/19px · largura 80% · `fundoRel` (prop opcional) | ✅ **no código** |
| 2 | `film`: SFX de projetor 0.20 automático | ✅ **implementado** (GO 04/08, `39a73ce`) |
| 2 | Tema do canal (claro/escuro) alimentando o grid — `film`+`polaroid`+`grid` | ✅ **implementado** (GO 04/08, `228707f`; interino `tema` na config até a UI de cadastro) |
| 3 | Valores dos outros 8 templates (lupa, spotlight, polaroid, reveal, split, grid) | ✅ **implementado** (GO 04/08, `5c84ab3`) |
| 3.1 | Legenda do polaroid (fonte + valores + LLM preencher) | ✅ **implementado** (GO 04/08, `5c84ab3`) |
| 4 | Entradas de texto | ✅ **aprovadas como estão — nada a fazer** |
| 5 | Ilustrações (13) — valores fechados | ✅ **implementado** (GO 04/08, `8d77689`) |
| 5 | Correção do `stat` (linha centralizada) | ✅ **implementado** (GO 04/08, `8d77689`) |
| — | Acervo de 40 variantes | ➡️ **espec própria** (`ESPEC-ACERVO-40-VARIANTES.md`) |

> **Branch da implementação:** `edicao-aditivos` (a partir de `2b957fa`, que sucede a tag
> `diretor-doc-v1-validado` só com docs). Validação em curso: tsc 47 = baseline em todos os
> passos; report da Austrália (plano de ouro `c7433ae0`) limpo com 22 PASS / 0 erro; render
> de prova comparando com `_renders_aprovados/2026-08-04_australia_croc_FINAL/`.

### 🔬 MÉTODO adotado (decisão do operador, 04/08)

> *"antes de renderizar, você pode gerar uma 'imagem' html para refinarmos, depois partimos
> para o render final"*

Todo ajuste visual segue: **HTML interativo com controles ao vivo → operador ajusta e manda
print com os valores → registro na espec → só então implementa/renderiza**. Economiza render
e evita ciclo de tentativa e erro. (Os HTMLs também são validados num navegador headless
antes de irem para o operador — lição do `textos_refino.html`, que foi entregue quebrado.)

### 🧪 Infra de teste criada (fora da produção)

| Arquivo | Para quê |
|---|---|
| `remotion/src/compositions/FilmGridTest.tsx` | composição de prova do `film` (grid + SFX) |
| `remotion/_render_filmgrid.mjs` | renderiza as duas provas (escuro/claro) |
| `remotion/public/teste_grid/` | grids, foto e 4 imagens demo do refino |
| `Desktop/film_moldura_refino.html` | refino da borda do `film` |
| `Desktop/apresentacoes_refino.html` | refino dos 8 templates |
| `Desktop/polaroid_legenda.html` | refino da legenda do polaroid |
| `Desktop/textos_refino.html` | entradas de texto (já aprovadas) |
| `Desktop/ilustracoes_refino.html` | ilustrações (valores fechados) |
| `Desktop/acervo_catalogo.html` + `acervo_TEXTO_refino.html` | acervo (espec própria) |
| `remotion/_render_acervo_catalogo.mjs` | renderiza os stills do catálogo |

---

## Item 1 — Overlays de TRANSIÇÃO (Apagão de Filme + Brilho 2)

**Trazido em:** 04/08/2026

### Assets (já movidos para o acervo) — **medidos com ffprobe em 04/08**

| Asset | Caminho | Duração real | Frames | fps do arquivo |
|---|---|---|---|---|
| Apagão de Filme | `acervo/transitions/CapCut/Apagao de Filme - CapCut.mp4` | **1,000 s** | 24 | 24 |
| Brilho 2 | `acervo/transitions/CapCut/Brilho 2 - CapCut.mp4` | **0,708 s** | 17 | 24 |
| SFX das duas | `acervo/sfx/foley/foley_camera_shutter_01.mp3` | 0,34 s | — | — |

> ⚠️ **Os clipes são 24 fps e o projeto é 30 fps.** No timeline: Apagão = **30 frames**,
> Brilho = **21 frames** (0,708 × 30 = 21,25 → arredondar). O `<OffthreadVideo>` reamostra
> sozinho; o que NÃO pode é assumir a contagem de frames do arquivo.

### 🚨 MODO DE COMPOSIÇÃO — as duas NÃO são iguais (medido, não suposto)

Todo overlay CapCut hoje no `BrollTest` entra com `mixBlendMode: "screen"`
([BrollTest.tsx:799](remotion/src/compositions/BrollTest.tsx#L799), o "Linhas de TV"). Em
`screen`, **preto é neutro** — some. Luminância média medida (`signalstats`, escala 0-255):

| Clipe | Base | Picos | Conclusão |
|---|---|---|---|
| Brilho 2 | 13,9 (preto) | **197** no meio | é um **estouro de luz** → `screen` funciona ✅ |
| Apagão de Filme | 13,9 (preto) na **maior parte dos 24 frames** | 131 / 92 / 60 | é um **apagão** → em `screen` ficaria **INVISÍVEL** ❌ |

**Regra — CORRIGIDA pelo operador (04/08, depois do render de prova):** os DOIS clipes
são **overlay em "clarear"** (lighten/screen): **a parte preta NUNCA aparece** — só a luz do
filme queimado. A conclusão anterior ("Apagão em composição normal") estava **errada
editorialmente**: em modo normal o preto cobria a tela e dava sensação de
corta-volta-corta (visto no render de 04/08). O nome "Apagão" engana — o efeito útil dele
são os LAMPEJOS, não o preto.

**⏱️ TIMING (regra nova do operador):** *"o momento onde o brilho fica todo na tela é o
momento exato onde o corte do vídeo deve ser"*. O clipe começa ANTES da emenda, de modo que
o PICO de luz caia exatamente no corte e o esconda:
- `Brilho 2`: pico ≈ 0,27-0,33 s do clipe (medido: YAVG 197 no frame 8 de 17) → começar
  ~0,3 s antes do corte.
- `Apagão`: lampejo principal ≈ 0,33 s (YAVG 131 no frame 8 de 24) → começar ~0,33 s antes.

**Status:** ✅ **APROVADAS pelo operador (04/08, teste isolado `transtest_sequencia.mp4`):**
*"ficou PERFEITO, é isso que quero"*. Duas correções finais, herdadas para a PRÓXIMA rodada:

1. **SFX shutter 0.15 → 0.13** — *"o sfx achei alto nesse teste"*. Mesmo padrão do projetor:
   o valor de ouvido no material pronto vence a calibração teórica.
2. **REGRA NOVA — `film` + Apagão de ENTRADA obrigatório:** *"SEMPRE, SEMPRE que aparecer a
   apresentação film ela deve obrigatoriamente ter a transição de entrada do apagão de
   filme com seu sfx, de saída não precisa."* → toda cena com `presentacao.tipo === "film"`
   ganha o Apagão (clarear, pico no início da cena) + shutter na ENTRADA; a saída fica
   como está. Nota: com isso a cena de `film` pode ter Apagão de entrada + SFX de projetor
   ao longo — são camadas diferentes (transição × som contínuo), não empilhamento de
   transição (I3 preservado).

### Regra (palavras do operador)

- Os dois clipes **são transições** (não overlays de cena).
- Uso: **quando houver corte seco de um take para outro** — o overlay entra **por cima**
  dos dois takes, fazendo a transição entre eles.
- **Não obrigatoriamente sempre**: é um recurso do repertório, usado com critério.
- SFX que acompanha **as duas**: `Camera Shutter` a **volume 0.15**.

### Notas de implementação (a confirmar com o operador na hora do GO)

- Volume 0.15 respeita a política de SFX (teto 0.20; calibrados a 0.15 são imutáveis).
- Entra como novo tipo de transição no `BrollTest` (camada acima das duas cenas), não como
  camada de cena — a transição própria da cena vira corte seco (**regra da transição única
  já validada**: entrada/saída de overlay É transição, nunca empilhar).
### ✅ REGRA DE FREQUÊNCIA — FECHADA (operador, 04/08)

Ficou fora do 1º GO; a regra foi decidida depois e está completa abaixo. **Este item agora
tem tudo que precisa para ser implementado** (falta só o GO da rodada própria).

**1. Gatilho — fronteira de TÓPICO.** As duas transições entram **onde o assunto muda**, não
a cada N cortes. O pass `topicos.py` já marca as fronteiras com mood dominante; na Austrália
deu **4 tópicos / 3 fronteiras em 102 s** (≈ 1 a cada 25-35 s), que é a densidade certa para
uma transição de 1 segundo. Razão da escolha: na fronteira o corte **já é conceitual**, então
a transição pesada tem justificativa narrativa em vez de virar enfeite.

**2. Colisão com o glitch — a transição nova SUBSTITUI o glitch da fronteira.** Hoje o SFX de
glitch toca "só na fronteira de tópico"; por **I3** os dois não podem coexistir ali. Na
fronteira passa a entrar Apagão/Brilho **no lugar** do glitch — uma marcação por fronteira,
nunca empilhada. O glitch **continua existindo** no pacote só-filtro (cena tensa sem texto),
que é outro gatilho e não é afetado.

**3. Qual das duas — pelo mood do tópico que ENTRA.** O `topicos.py` já devolve o mood com
vocabulário fechado (`tense · dark · mysterious · somber · neutral`):

| Mood do tópico entrante | Transição | Porquê |
|---|---|---|
| `tense` · `dark` · `somber` | **Apagão de Filme** | escurece e **fecha** — combina com peso/tragédia |
| `mysterious` · `neutral` | **Brilho 2** | estoura luz e **abre** — combina com revelação/respiro |

> Mood ausente ou fora do vocabulário → `neutral` → **Brilho** (é o mesmo fallback que o
> `topicos.py` já aplica na linha 81-83).

**4. Guardas obrigatórias (não negociáveis):**
- **I3** — a cena que entra e a que sai viram **corte seco** nessa fronteira (a transição
  própria delas é suprimida), exatamente como já se faz com `pkgScenes` e `mapaCorteEntra`.
- **I5** — se a cena entrante for curta demais, a transição comeria a cena inteira: **pular
  a transição** quando a cena entrante durar menos que ~2 s (Apagão = 30 frames = 1 s).
- **Nunca** numa fronteira que também tenha pacote só-filtro ou saída de mapa — os dois já
  são transição, e empilhar já foi reprovado em vídeo pelo operador.

**5. Cuidados técnicos já resolvidos** (medidos, não supostos): blend por clipe
(`Brilho 2` = `screen`; **`Apagão` = normal**, senão fica invisível), 24 fps → **30 e 21
frames**, SFX `foley_camera_shutter_01.mp3` a **0.15** nas duas.

---

## Item 2 — Apresentação `film`: sem tremor + fundo de GRID por tema do canal

**Trazido em:** 04/08/2026

### Nomenclatura (levantada no código, para não divergir do Piter)

O efeito é uma **APRESENTAÇÃO** (`presentacao`) — cada variação é um **"template de
apresentação"**. Pass: `apresentar.py` ("o Gemini escolhe COMO cada imagem se MOVE na tela,
por contexto da fala"). Campo no timeline: `presentacao: {tipo, extras, foco}`. Componente:
`remotion/src/compositions/Presentacao.tsx`. O log chama de *"cenas com template especial"*.
Templates: `kenburns` (default) · `lupa` · `spotlight` · `polaroid` · **`film`** · `parallax`
· `reveal` · `split` · `grid`. Valem **só para cenas de IMAGEM**, ~1 em 3, nunca 2 seguidas.

### O que o operador NÃO gostou (template `film`)

1. **TREMOR — tirar.** *"isso eu não gostei NENHUM POUCO"*. Hoje: `translate(sin(frame*1.7)*2,
   cos(frame*2.3)*2)` em [Presentacao.tsx:100](remotion/src/compositions/Presentacao.tsx#L100).
   **Não é preciso preservar a versão original** — o operador não quer correr o risco de o
   formato atual ser usado de novo (é substituição, não variante nova).
2. **Fundo preto chapado — trocar por GRID.**

### Assets (já movidos)

| Arquivo | Tema | Descrição |
|---|---|---|
| `acervo/fundos/grids/grid_escuro.jpg` | escuro | preto com linhas finas claras |
| `acervo/fundos/grids/grid_claro.jpg` | claro | branco/cinza-claro com linhas finas escuras |

2560×1440 (16:9), feitos no Canva pelo operador. README em `acervo/fundos/grids/README.md`.

### Regra do TEMA DO CANAL (estrutural — vale para todo o Vidmator)

**✅ Decidido pelo operador (04/08, pré-GO):**
- O **cadastro do canal, na UI**, ganha a marca do tipo de edição: **"edição simples"** (o
  fluxo de hoje) ou **"Vidmator"**. Quando Vidmator, aparece o campo **`tema`**: `claro` |
  `escuro`.
- O diretor **leva o tema em conta ao montar o vídeo**: tema claro → usar **somente** as
  variantes com `grid_claro` no fundo; tema escuro → `grid_escuro`.
- **Toda apresentação que tiver FUNDO** (mídia que não ocupa a tela inteira) usa o grid do
  tema do canal.
- Não é decisão do LLM nem do nicho: vem do cadastro do canal.
- **Escopo futuro anunciado pelo operador:** o tema vai guiar também a **representação de
  MAPAS** (cores do mapa pela cor do tema). Registrado — NÃO entra neste GO.
  → **A norma dos mapas mora em [ESPEC-ACERVO §7.2](ESPEC-ACERVO-40-VARIANTES.md)**: lá estão
  a troca da cor de destaque (vermelho → `#f59e0b`, com a armadilha do `rgba` hardcoded) e o
  tema. Os dois mexem no mesmo componente — decidir juntos.
- Interino até a UI de cadastro existir: campo na config do canal que o bridge injeta no
  timeline (`tema_visual`); a UI, quando nascer, vira a fonte sem mudar o encanamento.
- Alcance: começa pelo `film`, mas a regra é para **todos** os templates com fundo.
  Conferido no código (04/08) — quem tem fundo visível de fato:

  | Template | Tem fundo? | Hoje |
  |---|---|---|
  | `film` | ✅ | `#000` (+ `fundoRel` já implementado) |
  | `polaroid` | ✅ | `radial-gradient(#1a2230 → #070b12)` [:85](remotion/src/compositions/Presentacao.tsx#L85) |
  | `grid` | ✅ | `#0b1020` aparece no `gap` e no `padding` [:142](remotion/src/compositions/Presentacao.tsx#L142) |
  | `split` | ❌ | **duas imagens cobrem 100% da tela — não há fundo.** Fora do alcance |
  | `lupa`/`spotlight`/`reveal`/`parallax`/`kenburns` | ❌ | imagem em tela cheia |

  → o item cobre **`film` + `polaroid` + `grid`**. `split` não entra.

### ✅ VALORES FECHADOS (refinados no HTML interativo e aprovados — "MUITO MELHOR")

| Parâmetro | Valor |
|---|---|
| Borda de acabamento | **8px sólida `#000000`** |
| Canto (border-radius) | **19px** |
| Largura do filme | **80% da tela** (1536×864 em 1920×1080) |
| Tremor | **removido** |
| Fundo | grid do **tema do canal** |
| **SFX (novo)** | **`acervo/sfx/machine/machine_projetor_filme_01.mp3` a volume **0.13** — SEMPRE que a apresentação `film` for usada** |

> 🔊 **Volume revisado DE OUVIDO (operador, 04/08, depois do render de prova):** *"achei que
> em 0.20 ela ainda ficou alta, pode normalizar o volume dela, sempre para 0.13"*.
> **0.13 é o valor definitivo** — o 0.20 inicial foi estimativa, este foi ouvido no vídeo
> pronto. Fica abaixo do teto de 0.20 e um pouco abaixo dos calibrados de 0.15, o que é
> coerente: o projetor é ruído contínuo por trás da narração, não um acento pontual.

Sem a borda, a moldura "cortava" seco contra o grid claro (sem acabamento). As coordenadas
das perfurações eram de um filme de 1500px e passam a ser escaladas por `1536/1500`.
Provas renderizadas: `Desktop/film_grid_{escuro,claro}.mp4` (5s cada, com o SFX).

### Notas de implementação (a confirmar no GO)

- Onde o tema mora: cadastro de canal do Vidmator (ainda a definir — hoje o preset é por
  NICHO, e tema é por CANAL; provavelmente um campo novo que o bridge injeta no timeline,
  ex.: `tema_visual: "escuro"`). **Ponto a decidir com o operador.**
- Fallback sem tema definido: manter `escuro` (o que mais se parece com o comportamento atual).
- Os grids são imagens estáticas: entram como camada de fundo da apresentação, cobrindo a
  tela inteira (atenção à regra N9 já validada: camada que se move tem que ser maior que o
  quadro — aqui a camada é estática, então basta cobrir 100%).
- Os demais elementos do `film` (perfurações, filtro sépia/dessaturação, riscos horizontais)
  **ficam como estão**: o operador viu o render final com todos eles e aprovou. Não foram
  questionados nem alterados — se um dia incomodarem, viram item novo.

---

## Item 3 — Valores fechados dos outros 8 templates de apresentação

**Trazido em:** 04/08/2026 · refinados no HTML interativo `apresentacoes_refino.html`.

> ⚠️ **Só o `film` já está aplicado no código** (foi preciso para renderizar a prova). Os
> valores abaixo estão REGISTRADOS e entram no código no GO da implementação.

| Template | Parâmetro → valor |
|---|---|
| **kenburns** | **IGUAL** (zoom inicial 1.04 · zoom total +0.12 · velocidade 1) |
| **lupa** | raio **280** · zoom da lente **2.10** · brilho do fundo **0.60** · amplitude **12** · borda **5** |
| **spotlight** | raio do facho **500** · escuro no meio **0.10** · escuro na borda **0.75** · parada 1 **42%** · parada 2 **85%** · amplitude **14** |
| **polaroid** | largura **900** · altura **610** · margem **14** · margem de baixo **90** · inclinação **−3°** · zoom **+0.05** |
| **parallax** | **IGUAL** (zoom 1.12 → +0.16 · deslocamento 22 · vinheta 0.5 / abertura 40) |
| **reveal** | fim da varredura **0.50** · brilho do lado apagado **0.50** · zoom **+0.06** · largura da linha **7** · brilho da linha **26** |
| **split** | posição base **46%** · amplitude **15** · largura da linha **7** |
| **grid** | espaço entre **4** · margem externa **8** · atraso entre fotos **4** · zoom de entrada **0.20** |

### 🗺️ MAPA DE APLICAÇÃO — rótulo do HTML → variável real no código

> **Por que esta tabela existe:** os valores acima são os **rótulos em português** dos
> controles do HTML de refino. O HTML mora no Desktop (fora do git) e **não é** o código.
> Sem este mapa, quem implementar teria que adivinhar — e três rótulos são ambíguos
> (marcados 🔎). Tudo abaixo foi conferido linha a linha em `Presentacao.tsx` (04/08).

| Template | Rótulo | Valor | Onde no código | Hoje → depois |
|---|---|---|---|---|
| lupa | raio (px) | 280 | `R` [:33](remotion/src/compositions/Presentacao.tsx#L33) | 175 → **280** |
| lupa | zoom da lente | 2.10 | `Z` :33 | 2.1 → *igual* |
| lupa | brilho do fundo | 0.60 | `brightness()` :36 | 0.82 → **0.60** |
| lupa | 🔎 amplitude | 12 | `sin(...)*amp` **e** `cos(...)*(amp×0.64)` :33 | 14 / 9 → **12 / 7.68** |
| lupa | borda (px) | 5 | `border: Npx solid` :39 | 4 → **5** |
| spotlight | raio do facho | 500 | `circle Npx` :51 | 300 → **500** |
| spotlight | escuro no meio | 0.10 | 1º stop `rgba(0,0,0,N)` :51 | 0.25 → **0.10** |
| spotlight | escuro na borda | 0.75 | 2º stop `rgba(0,0,0,N)` :51 | 0.85 → **0.75** |
| spotlight | parada 1 | 42% | % do 1º stop :51 | 42 → *igual* |
| spotlight | parada 2 | 85% | % do 2º stop :51 | 74 → **85** |
| spotlight | 🔎 amplitude | 14 | `sin(...)*amp` **e** `cos(...)*(amp×0.66)` :47 | 12 / 8 → **14 / 9.24** |
| polaroid | largura foto | 900 | `.jan width` :87 | 820 → **900** |
| polaroid | altura foto | 610 | `.jan height` :87 | 600 → **610** |
| polaroid | margem | 14 | `padding` lateral e topo :86 | 22 → **14** |
| polaroid | margem de baixo | **40** | `padding-bottom` :86 | 70 → **40** |
| polaroid | 🔎 inclinação | −3° | **fim** do spring `[-9,-3]` :82 | já é −3 → ***IGUAL*** |
| polaroid | zoom da foto | +0.05 | `kb` :83 | 0.05 → *igual* |
| reveal | fim da varredura | 0.50 | `seg(pp,0.04,0.5)` :69 | 0.5 → *igual* |
| reveal | brilho do lado apagado | 0.50 | `brightness()` :72 | 0.45 → **0.50** |
| reveal | zoom (+) | 0.06 | :74 | 0.06 → *igual* |
| reveal | largura da linha | 7 | `width` :76 | 6 → **7** |
| reveal | 🔎 brilho da linha | 26 | `boxShadow: 0 0 {26}px {round(26/4)=7}px` :76 | blur *igual*, spread 6 → **7** |
| split | posição base | 46% | `x = 50 + ...` :126 | 50 → **46** |
| split | amplitude | 15 | `*18` :126 | 18 → **15** |
| split | largura da linha | 7 | `width` :133 | 4 → **7** |
| grid | espaço entre | 4 | `gap` :142 | 8 → **4** |
| grid | margem externa | 8 | `padding` :142 | 8 → *igual* |
| grid | atraso entre fotos | 4 | `frame - i*N` :144 | 6 → **4** |
| grid | zoom de entrada | 0.20 | `0.8 + s2*0.2` :145 | 0.2 → *igual* |
| **kenburns** · **parallax** | — | — | — | ***NÃO TOCAR*** |

**Os três 🔎 (sem isto, o implementador chuta):**
1. **`amplitude` é UM controle e o código tem DUAS oscilações** (horizontal e vertical). O
   HTML deriva a vertical: `lupa = amp × 0.64` · `spotlight = amp × 0.66`. Aplicar o número
   nas duas iguais muda o movimento.
2. **`inclinação −3°` é o valor FINAL**, que o código já atinge (spring de −9 → −3, mais uma
   oscilação de ±1,2°). O HTML de refino não simulava a entrada com mola. **Conclusão: não
   mexer na animação de entrada do polaroid** — ela está no render aprovado.
3. **`brilho da linha` alimenta dois números** do `boxShadow`: o blur (26) e o spread
   (`round(26/4)`).

⚠️ **Só alterar as linhas marcadas em negrito.** Sete valores já são iguais ao código —
"aplicar" todos mexeria à toa em coisa validada.

⚠️ **`spec.foco` não pode virar constante.** O HTML fixou o centro em `50 / 58` porque era
uma demo; o código usa `fx/fy` vindos do `foco` que o **LLM escolhe por cena**
([:29](remotion/src/compositions/Presentacao.tsx#L29)). Trocar isso por 50/58 mata o
enquadramento inteligente sem dar erro nenhum — regressão silenciosa.

### 3.1 — Legenda do `polaroid` (pedido novo, VIÁVEL)

Operador: *"ver a viabilidade de colocar na parte branca de baixo o nome da imagem, tipo
como se fosse a informação da foto ('— Amazônia'), com fonte de máquina de escrever"*.

**Viável — o campo JÁ EXISTE**: `PresentacaoSpec.legenda` é renderizado na tarja branca
([Presentacao.tsx:90](remotion/src/compositions/Presentacao.tsx#L90)). Falta só:
1. **Preencher**: `apresentar.py` nunca gera `legenda` — o LLM passa a devolver o nome do
   lugar/assunto da imagem (curto, 1-3 palavras, no formato `— Amazônia`).
2. **Trocar a fonte**: hoje é manuscrita (`Bradley Hand/Segoe Script`) → **máquina de
   escrever** (monospace tipo `Courier New`/`IBM Plex Mono`, que já está no projeto).
3. A margem de baixo abre o espaço da legenda.

#### ✅ Valores fechados da legenda (refino `polaroid_legenda.html`)

| Parâmetro | Valor |
|---|---|
| Texto (exemplo) | `— Amazônia` (LLM gera: nome do lugar/assunto, curto) |
| Fonte | **Courier New** (clássica de máquina de escrever) |
| Tamanho | **49** |
| Peso | **400** |
| Espaço entre letras | **2.5** |
| Distância da foto | **18** |
| Cor | **#2a2a2a** |
| Moldura (revisada) | largura **900** · altura **610** · margem **14** · **margem de baixo 40** · inclinação **−3°** |

> ⚠️ A margem de baixo caiu de **90 → 40** com a legenda em cena (o item 3 tem 90; vale
> **40**, que é o valor testado COM texto).

---

## Item 4 — Entradas de TEXTO (`KineticText`) — ✅ **APROVADAS COMO ESTÃO**

**Veredito do operador (04/08):** *"entradas de texto estão ótimas, vamos manter todas
assim"* → **nenhuma alteração**. Os valores atuais do código são os definitivos; este item
não gera trabalho no GO. Registrado para que ninguém "melhore" o que já foi aprovado.

**Nomenclatura levantada no código:** as animações de texto se chamam **"entradas de
texto"** — campo `entrada_texto` no timeline, componente **`KineticText`** (texto cinético)
em `BrollTest.tsx`. Não confundir com o **word-pop** (`EnumWord`), que é outra coisa e já
está validado.

**As 7 entradas (todas mantidas):** `words` (default) · `cascade` · `pop` · `blur` · `up` ·
`slam` · `typewriter` (esta última entra pelo tema de fonte, não pelo campo).

Refino conferido em `Desktop/textos_refino.html`.

---

## Item 5 — ILUSTRAÇÕES (`ilustracao` → `Illustration.tsx`) — ✅ valores fechados

**Trazido em:** 04/08/2026.

### ⚠️ Descoberta importante: existem DOIS sistemas de inserção no projeto

| Sistema | Onde | Estado |
|---|---|---|
| **1. Ilustrações** — pass `ilustrar.py` → campo `ilustracao` → `Illustration.tsx` | no pipeline | **é o que o Director usa hoje** |
| **2. Acervo de texto/gráficos** — `AcervoTexto.tsx` (10) + `AcervoTextoOverlay.tsx` (14) + `AcervoGraficos.tsx` (14) | biblioteca | **pronto, porém NÃO plugado no Director** |

Os prints que o operador mandou misturam os dois:
`Ovl03_LowerThird` · `Ovl04_FootnotePill`/`Ovl09_TickerCaption` · **`Graf11_PieSlices`** ·
**`Ovl01_ChapterBig`** · `Texto05_BoxedKicker`/`Ovl02_SubchapterLine`.
→ **Espec própria:** o acervo virou [ESPEC-ACERVO-40-VARIANTES.md](ESPEC-ACERVO-40-VARIANTES.md)
(são **40**, não 38) — em refino pelo operador, cronograma separado.

### Taxonomia do Sistema 1 (o que está em refino agora)

| tipo | variantes |
|---|---|
| **card** | `lowerthird` · `quote` · `definition` · `steps` · `beforeafter` · `vs` |
| **grafico** | `bar` · `line` · `donut` · `stat` · `comparison` · `progress` |
| **icone** | ícone + legenda |

### ✅ VALORES FECHADOS (refino `ilustracoes_refino.html`, 04/08)

#### Cards

| Variante | Valores |
|---|---|
| **lowerthird** | título **46** · subtítulo **26** · barra **7×92** · padding **18 / 28** · canto **12** · opacidade do fundo **0.90** |
| **quote** | aspas **96** · texto **40** · autor **24** · largura máx **640** |
| **definition** | termo **40** · texto **26** · caixa **720×180** · borda **2** · canto **14** · padding **30 / 38** |
| **steps** | raio **30** · distância entre passos **300** · número **40** · rótulo **20** · linha **4** |
| **beforeafter** | painel **250×120** · rótulo **36** · borda **3** · canto **12** · espaço da seta **160** |
| **vs** | caixas **310** · altura mín **140** · título **44** · detalhe **22** · selo VS **68** · borda **3** |

#### Gráficos

| Chart | Valores |
|---|---|
| **bar** | largura **500** · altura das barras **240** · barra **80** · espaço **36** · número **26** · rótulo **20** · canto **7** |
| **line** | largura **680** · altura **280** · linha **5** · área **0.14** · ponto **10** |
| **donut** | raio **120** · anel **16** · número **70** · rótulo **24** |
| **stat** | número **130** · rótulo **24** · linha **5** · largura da linha **300** |
| **comparison** | barra **520×32** · rótulo **24** · espaço entre linhas **80** |
| **progress** | barra **600×26** · rótulo **28** |

#### Ícone

| Parâmetro | Valor |
|---|---|
| **icone** | ícone **370** · traço **3** · legenda **34** · espaço até a legenda **0** |

### 🐞 5.1 — CORREÇÃO no `grafico · stat` (apontada pelo operador)

**Sintoma:** *"a linha de baixo do número está descentralizada; a linha deve estar
completamente centralizada com o número, e o rótulo centralizado com a linha e o número"*.

**Causa raiz (confirmada no código):** em
[Illustration.tsx:123](remotion/src/compositions/Illustration.tsx#L123) a linha vai de
`x1=120` a `x2=300` — centro **210**, exatamente onde estão o número e o rótulo. Ou seja,
**no estado final ela ESTÁ centralizada**. O que quebra é a **animação**: `dashP(p)` revela
o traço **da esquerda para a direita**, então durante toda a entrada a linha fica deslocada
para a esquerda (foi esse instante que o operador pegou no print).

**Correção:** a linha passa a **crescer do CENTRO para os dois lados** — em qualquer
instante da animação ela fica centralizada com número e rótulo:
```
cx = 210  (centro do viewBox 420×230, mesmo x do número e do rótulo)
x1 = 210 − 150·p        x2 = 210 + 150·p
```
(no lugar do `dashP(p)`, que é revelação unidirecional).

> 🔎 **Desambiguação obrigatória:** o controle **"largura da linha" = 300** é a largura
> **TOTAL** da linha (por isso o ±150 acima), **não** o `x2=300` que está hoje no código.
> Confirmado nos controles do `ilustracoes_refino.html`, onde `linha`=espessura (5) e
> `lw`=largura (300). Com 300 a linha vai de x=60 a x=360 e cabe no viewBox de 420.
> O outro par que engana: em `bar`, `larg`(500) é a largura do gráfico e `bw`(80) a da barra.

---

## Item 6 — *(aguardando o operador)*
