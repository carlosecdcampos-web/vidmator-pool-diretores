# ESPEC — ACERVO: 40 VARIANTES + CHAPTER + MAPAS

> **Status:** ✅ **IMPLEMENTADA (GO 04/08) — passos 0-7 no código, branch `acervo-mapas-chapter`.**
> Validada: tsc na baseline (47) em todos os passos, PRE-RENDER REPORT limpo (plano de ouro
> `c7433ae0`), render da Austrália com capítulo + transição Apagão + variantes, auditoria
> Vision sem regressão (3 achados, todos pré-existentes; baseline do ouro = 4).
> O §8 (regra de uso das 40 variantes) segue FORA — espec própria, aguardando rodada.
>
> **Como implementar:** siga o
> [RELATORIO-IMPLEMENTACAO-ACERVO.md](RELATORIO-IMPLEMENTACAO-ACERVO.md) — ordem dos passos,
> arquivos exatos, erros a evitar e plano de validação.
>
> **Espec IRMÃ:** [ESPEC-EDICAO-ADITIVOS.md](ESPEC-EDICAO-ADITIVOS.md) — aditivos de edição
> do Sistema 1 (apresentações, entradas de texto, ilustrações). Seus passos 0-5 **já foram
> implementados** (branch `edicao-aditivos`); o Item 1 dela (transições) segue pendente e
> está resumido aqui no §7.1.
>
> **Regra-mãe (a mesma da espec irmã):** nada aqui pode alterar o que já está validado
> (tag `diretor-doc-v1-validado`). Tudo entra aditivo e gated.

> ⚠️ **O título ficou menor que o conteúdo.** A espec nasceu só com as 40 variantes de
> texto/gráfico, mas cresceu: hoje cobre também o `ChapterTitle` e **todo o subsistema de
> mapas** (cor, valores, inventário de 21, seleção de 14, filtro de elegibilidade e barrinha).
> O nome do arquivo foi mantido para não quebrar os links já existentes.

## 📑 Sumário

| § | Assunto | Estado |
|---|---|---|
| 4 | Valores da família **TEXTO** (10) | ✅ fechado |
| 5 | Valores da família **OVERLAY** (14) | ✅ fechado |
| 6 | Família **GRÁFICO** (16) | ✅ aprovada sem alteração |
| 6.1 | **`ChapterTitle`** (capítulo/tópico) | ✅ fechado · gatilho decidido |
| 7.1 | Transições **Apagão/Brilho** | ✅ regra fechada · **entra NESTE GO** (passo 7 do relatório) |
| 7.2.1 | Mapas: **cor** vermelho → `#f59e0b` | ✅ fechado |
| 7.2.1b | Valores do **`MapAnimation`** | ✅ fechado |
| 7.2.2 | Mapas guiados pelo **tema** do canal | ⏳ escopo futuro |
| 7.2.3 | **Inventário** dos 21 mapas | ✅ levantado |
| 7.2.4 | **Seleção**: 14 dos 21 + filtro de elegibilidade | ✅ fechado |
| 7.2.5 | **Barrinha** no `CountryCharacterMap` | ✅ fechado |
| 7.2.6 | **Valores** dos mapas + regra do kicker 33 | ✅ fechado |
| 8 | Pendências abertas | ⏳ 2 — **fora deste GO** (rodada própria) |

---

## Contexto

**Trazido em:** 04/08/2026.

### Inventário real (conferido no código — são **40**, não 38)

| Família | Arquivo | Qtd | Variantes |
|---|---|---|---|
| **Texto** (tela cheia) | `AcervoTexto.tsx` | **10** | `Texto01_Typewriter` · `02_HighlightSweep` · `03_WordPop` · `04_EditorialSerif` · `05_BoxedKicker` · `06_SplitBar` · `07_StampImpact` · `08_GradientGlow` · `09_UnderlineDraw` · `10_LetterCascade` |
| **Overlay** (sobre o vídeo) | `AcervoTextoOverlay.tsx` | **14** | `Ovl01_ChapterBig` · `02_SubchapterLine` · `03_LowerThird` · `04_FootnotePill` · `05_CornerTag` · `06_CenterPunch` · `07_QuoteAttribution` · `08_SideNote` · `09_TickerCaption` · `10_NumberBadge` · `11_SpecBadge` · `12_GiantStat` · `13_PriceTag` · `14_PillVerdict` |
| **Gráfico** | `AcervoGraficos.tsx` | **16** | `Graf01_CounterGlow` · `02_Odometer` · `03_DonutPercent` · `04_GaugeMeter` · `05_VersusBars` · `06_VersusTug` · `07_TimelineRise` · `08_LinePulse` · `09_RankList` · `10_BigStatCard` · `11_PieSlices` · `12_MultiBars` · `13_DualLine` · `14_OvlCounterPunch` · **`15_OvlStatCorner`** · **`16_OvlProgressBar`** |

> Correção do inventário anterior: eram citadas 38; o acervo tem **40** (16 gráficos, não 14).

### Método diferente deste item (e o porquê)

Os outros refinos foram feitos com HTML replicando o componente. Aqui são **40 componentes
React animados** — replicar à mão seria caro e, pior, **infiel**. Então:

1. **Catálogo visual** — cada variante RENDERIZADA de verdade pelo Remotion
   (`_render_acervo_catalogo.mjs` → `remotion/_catalogo_acervo/*.jpg`), com o mesmo conteúdo
   de exemplo, montado num HTML com o nome ao lado de cada uma.
2. **Operador escolhe** quais entram no repertório do Director (não faz sentido plugar 40).
3. **Refino interativo só das escolhidas**, no formato dos itens anteriores.

### 3. — Refino COM CONTROLES (pedido do operador)

O catálogo (stills) serve para escolher; para **personalizar** cada variante o operador pediu
os pontos editáveis. Como são 40 componentes, o refino sai **por família**, 2 por linha:

| Família | Arquivo de refino | Estado |
|---|---|---|
| **TEXTO** (10) | `Desktop/acervo_TEXTO_refino.html` | ✅ **valores fechados pelo operador** (04/08) — ver §4 |
| **OVERLAY** (14) | `Desktop/acervo_OVERLAY_refino.html` | ✅ **valores fechados pelo operador** (04/08) — ver §5 |
| **GRÁFICO** (16) | `Desktop/acervo_GRAFICO_refino.html` | ✅ **aprovada SEM alterações** (04/08) — ver §6 |

Cada variante expõe: tamanho do kicker/texto, espaçamento entre letras, espaço abaixo do
kicker, altura de linha, margens, e os parâmetros próprios do efeito (velocidade de
digitação, altura do marca-texto, damping/stiffness do pop, largura da régua, cantoneiras,
barra, inclinação do carimbo, velocidade do brilho, sublinhado, cascata...).

> ⚠️ Nota de fidelidade: o HTML replica os componentes com as fontes do tema **serif**
> (Playfair/Lora/IBM Plex Mono, via Google Fonts). O que se ajusta ali são **números**
> (tamanhos/espaçamentos) — o render final usa as fontes reais do projeto.

---

## 4. ✅ VALORES FECHADOS — família TEXTO (operador, 04/08)

> Refinados em `acervo_TEXTO_refino.html`. **Três variantes ficaram como estão** (`OK` = os
> defaults do componente já foram aprovados, não mexer). Onde o valor coincide com o default,
> está marcado *(igual)* — na implementação, só altera o que muda de verdade.

### Texto01_Typewriter — *dossiê/documento*
| Parâmetro | Valor |
|---|---|
| kicker | **34** (era 30) |
| espaço letras kicker | 6 *(igual)* |
| espaço abaixo kicker | 28 *(igual)* |
| texto | **64** (era 60) |
| altura da linha | **1.30** (era 1.45) |
| margem lateral | 180 *(igual)* |
| velocidade digitação | 1.40 *(igual)* |

### Texto02_HighlightSweep — *citação/estudo, fundo papel*
| Parâmetro | Valor |
|---|---|
| kicker | **28** (era 26) |
| espaço letras kicker | 8 *(igual)* |
| espaço abaixo kicker | **22** (era 30) |
| texto | **80** (era 76) |
| altura da linha | **1.20** (era 1.30) |
| altura do marca-texto | **58** (era 46) |
| posição do marca-texto | **56** (era 78) |
| opacidade do marca-texto | 0.40 *(igual)* |
| margem lateral | **140** (era 160) |

### Texto03_WordPop — *punchline/hook*
| Parâmetro | Valor |
|---|---|
| kicker | **28** (era 26) |
| espaço letras kicker | 8 *(igual)* · espaço abaixo kicker | **30** (era 34) |
| texto | 88 *(igual)* · espaço entre palavras | 26 *(igual)* |
| altura da linha | 1.25 *(igual)* · atraso entre palavras | 5 *(igual)* |
| damping | 11 *(igual)* · stiffness | 160 *(igual)* · deslocamento | 46 *(igual)* |

> Só o kicker (26→28) e o espaço abaixo dele (34→30) mudam. A física do pop fica intocada.

### Texto04_EditorialSerif — *reflexão/transição elegante*
| Parâmetro | Valor |
|---|---|
| largura da régua | **380** (era 320) |
| espessura da régua | 3 *(igual)* · espaço abaixo da régua | 40 *(igual)* |
| kicker | **26** (era 24) · espaço letras kicker | 10 *(igual)* |
| espaço abaixo kicker | **14** (era 22) |
| texto | **80** (era 78) · altura da linha | 1.35 *(igual)* |
| margem lateral | 220 *(igual)* |

### Texto05_BoxedKicker — ✅ **OK, sem alteração**
### Texto07_StampImpact — ✅ **OK, sem alteração**
### Texto08_GradientGlow — ✅ **OK, sem alteração**

### Texto06_SplitBar — *afirmação direta / abre tópico*
| Parâmetro | Valor |
|---|---|
| largura da barra | 10 *(igual)* · altura da barra | 360 *(igual)* |
| espaço barra↔texto | 54 *(igual)* |
| kicker | **30** (era 26) · espaço letras kicker | 7 *(igual)* |
| espaço abaixo kicker | 18 *(igual)* |
| texto | **96** (era 80) · altura da linha | 1.25 *(igual)* |

### Texto09_UnderlineDraw — *frase com UMA palavra decisiva*
| Parâmetro | Valor |
|---|---|
| kicker | **28** (era 24) · espaço letras kicker | 11 *(igual)* |
| espaço abaixo kicker | **24** (era 34) |
| texto | **86** (era 78) · peso do texto | 300 *(igual)* |
| altura da linha | 1.35 *(igual)* |
| espessura do sublinhado | 7 *(igual)* · distância do sublinhado | −10 *(igual)* |

### Texto10_LetterCascade — *revelação gradual / suspense*
| Parâmetro | Valor |
|---|---|
| kicker | **30** (era 26) · espaço letras kicker | 8 *(igual)* |
| espaço abaixo kicker | **24** (era 32) |
| texto | 86 *(igual)* · altura da linha | 1.25 *(igual)* |
| velocidade das letras | 1.10 *(igual)* · desfoque inicial | 10 *(igual)* |

> 📐 **Padrão que emergiu dos ajustes** (vale como diretriz para as próximas famílias): o
> operador **subiu os kickers** (24-26 → 28-34) e **encolheu o espaço abaixo deles**, e
> **subiu o corpo do texto** onde o componente é o dono da tela. Efeito: kicker mais
> presente, mais colado no texto, e frase maior.

---

## 5. ✅ VALORES FECHADOS — família OVERLAY (operador, 04/08)

> Refinados em `acervo_OVERLAY_refino.html`, sobre um frame real do render da Austrália.
> **Três ficaram OK** (sem alteração): `Ovl09_TickerCaption`, `Ovl12_GiantStat`,
> `Ovl13_PriceTag`. Onde o valor coincide com o default, está marcado *(igual)*.

### 🌑 Regra GLOBAL da família: `dim = 0.15`

**Todas as 11 variantes que o operador ajustou receberam o mesmo `escurecer o fundo` = 0.15**
(a única exceção é a `Ovl14_PillVerdict`, que já nasce em 0.45 por tomar a tela). Isso não é
um valor por variante — é **decisão de família**: o footage entra 15% mais escuro sob
qualquer overlay de texto, somando-se ao scrim próprio de cada um.

> ⚠️ **ONDE o dim mora (importante para o GO atual):** o `dim` é aplicado pelo **CALLER**
> que compõe o overlay sobre o footage — **não** pelos componentes (eles são transparentes;
> assar o dim dentro deles quebraria as variantes OK e misturaria responsabilidades). Como a
> família OVERLAY **ainda não está plugada** e o caller só nasce no passo 8 (fora deste GO),
> **o dim=0.15 NÃO tem o que implementar agora** — fica registrado aqui como regra pronta
> para a espec do passo 8. No GO atual, o passo 3 aplica só os valores por variante.

> ✅ **RESOLVIDO (operador, 04/08):** *"OK é que é para manter como já está, sem alterar
> nada. Não englobe o 0.15"*. As três marcadas OK (`09`, `12`, `13`) ficam com **`dim` 0** —
> o 0.15 vale só para as 11 listadas abaixo. **"OK" nesta espec significa sempre: default
> intocado, nenhuma regra global se aplica por cima.**

| Variante | Valores |
|---|---|
| **Ovl01_ChapterBig** | dim **0.15** · margem esq. 90 *(igual)* · margem baixo 90 *(igual)* · kicker **32** (30) · esp. letras 8 *(igual)* · esp. abaixo 14 *(igual)* · régua **200** (150) · espessura 5 *(igual)* · esp. abaixo da régua 20 *(igual)* · título **92** (88) · altura linha 1.12 *(igual)* · largura máx **1350** (1150) · scrim **0.80** (0.78) |
| **Ovl02_SubchapterLine** | dim **0.15** · topo **100** (96) · barra larg. 10 *(igual)* · barra alt. **126** (84) · esp. barra↔placa **16** (26) · margem vert. 18 *(igual)* · margem dir. **42** (40) · kicker **24** (21) · esp. letras 6 *(igual)* · texto **66** (44) · opac. placa 0.62 *(igual)* |
| **Ovl03_LowerThird** | dim **0.15** · margem esq. 90 *(igual)* · margem baixo 110 *(igual)* · barra **8** (9) · margem vert. 16 *(igual)* · margem dir. 44 *(igual)* · texto **68** (40) · kicker **30** (22) · esp. letras 3 *(igual)* · opac. 0.78 *(igual)* · canto 10 *(igual)* |
| **Ovl04_FootnotePill** | dim **0.15** · margem baixo **90** (64) · margem vert. 12 *(igual)* · margem horiz. 30 *(igual)* · asterisco 30 *(igual)* · esp. asterisco 14 *(igual)* · texto **34** (25) · opac. pill 0.68 *(igual)* |
| **Ovl05_CornerTag** | dim **0.15** · margem dir. 84 *(igual)* · topo **80** (78) · margem vert. 10 *(igual)* · margem horiz. 22 *(igual)* · ponto 12 *(igual)* · esp. ponto 12 *(igual)* · texto **34** (26) · esp. letras 3 *(igual)* · canto 8 *(igual)* · opac. 0.60 *(igual)* |
| **Ovl06_CenterPunch** | dim **0.15** · kicker **32** (26) · esp. letras 9 *(igual)* · esp. abaixo 16 *(igual)* · texto **120** (110) · altura linha 1.10 *(igual)* · largura máx 1300 *(igual)* · scrim larg. 46 *(igual)* · scrim alt. **35** (34) · força scrim **0.70** (0.72) · glow 44 *(igual)* |
| **Ovl07_QuoteAttribution** | dim **0.15** · margem esq. 110 *(igual)* · margem baixo 100 *(igual)* · aspas 120 *(igual)* · esp. abaixo das aspas **−12** (8) · texto **70** (52) · altura linha **1.36** (1.35) · autor **30** (26) · esp. acima do autor 18 *(igual)* · largura máx 1250 *(igual)* · scrim 0.80 *(igual)* |
| **Ovl08_SideNote** | dim **0.15** · margem dir. 80 *(igual)* · largura máx **900** (560) · borda 6 *(igual)* · margem vert. **28** (26) · margem horiz. **50** (34) · kicker **24** (22) · esp. letras 4 *(igual)* · esp. abaixo 10 *(igual)* · texto **44** (30) · altura linha **1.46** (1.45) · opac. 0.74 *(igual)* |
| **Ovl09_TickerCaption** | ✅ **OK — sem alteração** |
| **Ovl10_NumberBadge** | dim **0.15** · margem esq. 90 *(igual)* · margem baixo **100** (96) · número 150 *(igual)* · esp. número↔texto 34 *(igual)* · divisor 3 *(igual)* · esp. após divisor 30 *(igual)* · texto 52 *(igual)* · altura linha 1.20 *(igual)* · largura máx 1000 *(igual)* · scrim **0.65** (0.66) |
| **Ovl11_SpecBadge** | dim **0.15** · margem dir. **72** (70) · topo 64 *(igual)* · margem vert. 14 *(igual)* · margem horiz. 30 *(igual)* · valor 54 *(igual)* · esp. entre partes 16 *(igual)* · unidade 27 *(igual)* · esp. letras unidade 4 *(igual)* · canto 10 *(igual)* · opac. 0.72 *(igual)* |
| **Ovl12_GiantStat** | ✅ **OK — sem alteração** |
| **Ovl13_PriceTag** | ✅ **OK — sem alteração** |
| **Ovl14_PillVerdict** | dim 0.45 *(igual)* · nome **60** (46) · margem vert. pill 14 *(igual)* · margem horiz. pill 46 *(igual)* · canto pill 12 *(igual)* · esp. entre as duas 22 *(igual)* · veredito **32** (28) · margem vert. veredito 12 *(igual)* · margem horiz. veredito 34 *(igual)* · borda 1.50 *(igual)* |

> 📐 **O padrão do TEXTO se confirmou aqui, e mais forte:** o operador **subiu o corpo do
> texto em quase todas** — os saltos grandes são `Ovl03` (40→68), `Ovl02` (44→66) e `Ovl08`
> (30→44), todos overlays de placa/caixa. Leitura: as placas nasceram **pequenas demais para
> TV/celular**; o texto agora ocupa a placa. Os kickers também subiram (21-26 → 24-32),
> mesmo movimento da família TEXTO.
>
> Ajustes de posição/estrutura ficaram mínimos (1-4px), o que confirma que o layout base
> estava certo — o problema era **escala de leitura**, não composição.

---

## 6. ✅ Família GRÁFICO (16) — **APROVADA SEM ALTERAÇÕES** (operador, 04/08)

> *"gráficos estão todos ok, não vou querer alterar nada"*.
>
> As **16 variantes** ficam com os valores atuais de `AcervoGraficos.tsx`, **byte a byte** —
> incluindo o `dim` das três de overlay (`Graf14`, `Graf15`, `Graf16`), pela mesma regra do
> §5: **OK = default intocado, nenhuma regra global por cima**.
>
> Este item **não gera trabalho de implementação**. Registrado para que ninguém "melhore" o
> que já foi aprovado — mesma proteção do Item 4 da espec irmã (entradas de texto).

### 🏁 Refino do acervo: **40/40 concluído**

| Família | Qtd | Resultado |
|---|---|---|
| TEXTO | 10 | 7 ajustadas · 3 OK (§4) |
| OVERLAY | 14 | 11 ajustadas · 3 OK (§5) |
| GRÁFICO | 16 | **0 ajustadas · 16 OK** (§6) |
| **Total** | **40** | **18 ajustadas · 22 intocadas** |

O que falta neste bloco **não é mais refino visual** — é a decisão de **quando cada variante
entra** (regra de uso) e **como o LLM preenche** as props. Ver "Pendências" abaixo.

---

## 6.1 ✅ VALORES FECHADOS — `ChapterTitle` (operador, 04/08)

> Componente do print do operador: abertura de **capítulo/tópico** (label CHAPTER + número
> gigante + réguas + título + subtítulo). Mora **solto** em
> `remotion/src/compositions/ChapterTitle.tsx` — **não** está nas 3 famílias do acervo, por
> isso não apareceu nos refinos anteriores. Hoje **não é usado em produção** (o `BrollTest`
> não o importa).
>
> Refinado em `Desktop/mapa_chapter_refino.html`. Veredito: *"aparece e vou utilizar como
> está agora"*.

⚠️ **Atenção — os valores aprovados NÃO são os do componente atual.** O HTML de refino foi
uma réplica, e vários números dele já saíram diferentes do `.tsx`. A tabela abaixo é a
verdade a aplicar; a coluna "hoje" é o que está no código:

| Parâmetro | ✅ Aprovado | Hoje no `.tsx` | Muda? |
|---|---|---|---|
| **cor de destaque** | `#f59e0b` | `#f59e0b` | — |
| label "CHAPTER" — tamanho | **26** | 34 | **sim** |
| label — espaço entre letras | **14** | 16 | **sim** |
| label — espaço abaixo | **10** | 6 | **sim** |
| ⚠️ label — `paddingLeft` | *(não exposto no refino)* | **16** | ver nota |
| número — tamanho | **210** | 300 | **sim** |
| número — glow | **44** | `30 + glow*40` (30-70) | **sim** |
| número — zoom de entrada | 1.50 | 1.5 | — |
| réguas — largura | **230** | 220 | **sim** |
| réguas — espessura | 2 | 2 | — |
| réguas — espaço até o ponto | **20** | 30 | **sim** |
| ponto — tamanho | **6** | 8 | **sim** |
| réguas — espaço vertical | **38** | `34px 0 30px` (assimétrico) | **sim** |
| título — tamanho | **104** | 108 | **sim** |
| título — espaço entre letras | **0** | 6 | **sim** |
| título — altura da linha | **1.06** | *(não definida)* | **sim** |
| título — largura máx | **1500** | *(sem limite)* | **sim** |
| subtítulo — tamanho | **34** | 44 | **sim** |
| subtítulo — espaço acima | 26 | 26 | — |
| subtítulo — espaço entre letras | **4** | 3 | **sim** |

> 📐 **Leitura:** o operador **encolheu o conjunto todo** — número (300→210), título
> (108→104), label (34→26), subtítulo (44→34) — e **zerou o espaço entre letras do título**
> (6→0). O resultado é um card menos "estourado" e mais legível, coerente com o restante da
> identidade.

**Notas de implementação (3 pegadinhas):**
1. **`paddingLeft: 16` no label** existe no `.tsx` e **não foi exposto** no refino — serve
   para compensar opticamente o `letterSpacing` (a última letra empurra o texto para a
   esquerda). Ao aplicar `letterSpacing 14`, **recalcular ou manter proporcional**; não
   remover sem olhar, ou o "CHAPTER" fica descentralizado.
2. **Margem das réguas é assimétrica** hoje (`34px 0 30px`); o refino usa um valor único
   (**38**) em cima e embaixo. Aplicar como `38px 0`.
3. **Subtítulo é itálico peso 300** no `.tsx` — o refino não mexeu nisso; **manter**.

### 🚨 GATILHO CORRIGIDO PELO OPERADOR (04/08, depois do render de prova)

O gatilho "fronteira de tópico" estava **ERRADO** para o capítulo — produziu um "CHAPTER 04"
solto num vídeo de narrativa contínua. A regra verdadeira, nas palavras do operador:

> *"capítulo só quando o roteiro FOR DIVIDIDO POR CAPÍTULOS, PARTES OU TÓPICOS — tipo 5
> tópicos, 7 tópicos, 5 civilizações, aí entra 'CAPÍTULO 1 - MAIAS'. Não tem capítulo na
> introdução; se fosse ter, seria depois da introdução. Ele entra sempre que o roteiro se
> tratar de tópicos, lista, quantidade: 5 predadores da Amazônia, 7 maravilhas do mundo
> antigo, as 7 feras mais aterrorizantes dos 7 mares — onde cada fera é um capítulo. O
> capítulo deverá ser entendido ÚNICA E EXCLUSIVAMENTE nesses pontos. Se o vídeo é só sobre
> anaconda, não tem capítulo, não tem divisão."*

**Tradução em regra:**
1. **Capítulo é para ROTEIRO-LISTA, não para narrativa contínua.** O gatilho é a **estrutura
   do roteiro** (título/estrutura tipo "Os N …" com N itens enumeráveis), não a segmentação
   de tópicos do `topicos.py` — que existe para trilha/glitch e segmenta QUALQUER narrativa.
2. **Cada ITEM da lista = um capítulo**, com o nome do item ("CAPÍTULO 1 - MAIAS").
3. **Nunca na introdução** — o capítulo 1 entra quando o item 1 começa, depois da intro.
4. **Numeração completa obrigatória**: *"NUNCA deve aparecer um número solto — se aparecer
   capítulo 6, obrigatoriamente deve ter aparecido o 1, 2, 3, 4 e 5"*. Ou a lista inteira
   capitula (1..N), ou nenhum card entra.

**Estado:** o gatilho antigo foi **DESLIGADO** no preset (`marcacoes_topico.capitulo: false`).
O `ChapterTitle` fica pronto no acervo (valores aprovados, dono-da-tela implementado)
esperando o **detector de roteiro-lista** — item novo, a especificar: provavelmente um pass
que reconhece a estrutura enumerada no roteiro (o `enumeracoes.py` já acha listas FALADAS;
isto é a lista ESTRUTURAL do vídeo inteiro) e marca o início de cada item.

*(Nota histórica: a versão abaixo descreve o gatilho antigo por fronteira — mantida só como
registro da mecânica de tela cheia/precedência, que continua válida QUANDO o capítulo entrar.)*

**Cadeia de precedência na fronteira de tópico** (um elemento por emenda, sempre):

> ### `ChapterTitle` > transição Apagão/Brilho > glitch

Assim o **I3** (transição única, nunca empilhar) fica satisfeito **por construção**, não por
checagem posterior — que é a forma robusta. Nas fronteiras sem capítulo, a transição entra
normalmente; nas sem transição, o glitch segue como sempre.

**Mecânica fechada (especificidade para a implementação, 04/08):**

| Aspecto | Regra | Por quê |
|---|---|---|
| **Duração** | **3,0 s** (90 frames) | a animação interna completa no frame 62 (~2,1 s: label→número→réguas→título→subtítulo); 90 dá ~1 s de leitura após assentar. Respeita I5 com folga |
| **Nunca no tópico 1** | capítulo só em fronteiras **internas** (tópico 2 em diante) | o tópico 1 começa em 0,0 s — card em cima do HOOK mataria a retenção; o hook é sagrado |
| **Props** | `title` = `topico.titulo` · `chapterNumber` = índice do tópico (2, 3, …) · `subtitle` = **vazio** | título e índice já vêm do `topicos.py`; subtítulo não tem fonte factual — inventar violaria o Contrato Editorial (grupo B) |
| **É elemento de TELA CHEIA** | entra pelas mesmas regras do mapa: a cena por baixo troca em **corte seco** (invisível) e, se terminar num corte, **sai POR CIMA** da cena entrante | regra do operador de 03/08 v3 que eliminou o vazamento das ruínas — vale para QUALQUER tela cheia |
| **Respeita I6** | fronteira dentro de janela de word-pop/enumeração → capítulo **não entra** (cai para a transição) | o word-pop é dono da tela; a checagem de janelas já existe no `pkgScenes`/`legibilidade` |
| **Quantos por vídeo** | os que os tópicos derem (num vídeo de 100 s: 4 tópicos → **até 3** capítulos internos) | densidade natural do `topicos.py`, mesma lógica das transições |

---

## 7. 📌 PENDENTES HERDADOS DO 1º GO (registrados aqui a pedido do operador)

> Os dois itens abaixo **não são do acervo** — a regra canônica de ambos mora na espec irmã
> [ESPEC-EDICAO-ADITIVOS.md](ESPEC-EDICAO-ADITIVOS.md). Ficam listados aqui **como pendência
> viva**, para não se perderem entre uma espec e outra. **Fonte da verdade continua sendo a
> espec irmã** — se divergirem, ela vence.

### 7.1 — Transições Apagão de Filme + Brilho 2 ⏳

**Estado:** ficou **fora do 1º GO** por decisão do operador (a regra de frequência foi
adiada). Todo o resto já está resolvido e documentado na espec irmã (Item 1):

- Assets no acervo, durações medidas (**1,000 s / 0,708 s**, ambos 24 fps → **30 e 21
  frames** no timeline de 30 fps).
- 🚨 **Blend por clipe** (medido, não suposto): `Brilho 2` → `mixBlendMode: "screen"`;
  **`Apagão de Filme` → composição NORMAL** — em `screen` ele ficaria invisível, porque é
  preto na maior parte dos 24 frames.
- SFX das duas: `foley_camera_shutter_01.mp3` a **0.15**.
- Guardas: **I3** (transição única, nunca empilhar) e **I5** (estado mínimo ~1s).

✅ **A regra de frequência foi FECHADA em 04/08** (detalhe completo na espec irmã, Item 1):
entram **na fronteira de tópico**, **substituindo** o glitch que hoje ocupa esse lugar, e a
escolha entre as duas vem do **mood do tópico que entra** (`tense`/`dark`/`somber` → Apagão;
`mysterious`/`neutral` → Brilho). Guardas: I3 (cenas viram corte seco), I5 (pula se a cena
entrante < ~2 s) e nunca junto de pacote só-filtro ou saída de mapa.

→ ✅ **A "rodada própria" é ESTE GO: as transições entram pelo passo 7 do relatório**, junto
com o gatilho do `ChapterTitle` (a precedência dos dois na fronteira de tópico foi decidida
em conjunto — ver §6.1). O detalhe técnico (blend por clipe, 24→30 fps, shutter 0.15) segue
na espec irmã, Item 1 — o passo 7 aponta para lá.

### 7.2 — MAPAS ⏳ *(este § é a norma dos mapas — a espec irmã aponta para cá)*

#### 7.2.1 — 🎨 COR DE DESTAQUE: vermelho → **`#f59e0b`** *(operador, 04/08)*

> *"hoje os mapas destacam os locais em vermelho, não gosto, quero que destaquem daqui para
> frente nessa cor: #f59e0b"*

**Onde mora (levantado no código):** `remotion/src/compositions/MapAnimation.tsx` — é o
componente que **entra no vídeo hoje** (`BrollTest.tsx:627`). A cor é a constante
[`ACCENT = "#e0452e"`](remotion/src/compositions/MapAnimation.tsx#L9) e alimenta **5 pontos**:

| # | Elemento | Linha |
|---|---|---|
| 1 | preenchimento do país destacado — ⚠️ **hardcoded** como `rgba(224,69,46,…)`, não usa a constante | :68 |
| 2 | contorno do país destacado | :69 |
| 3 | anel pulsante do pin | :80 |
| 4 | pin | :81 |
| 5 | barra sob o título grande | :113 |

⚠️ **Armadilha:** trocar só a constante **não basta** — a linha 68 tem o vermelho escrito à
mão em `rgba`. Tem que virar `rgba(245,158,11,…)` (que é o `#f59e0b`), senão o país fica
preenchido de vermelho com contorno âmbar.

✅ **Escopo confirmado — e mais forte do que parecia.** Levantando TODOS os componentes de
mapa do projeto (04/08), o resultado é este:

| Componente de mapa | Accent hoje |
|---|---|
| `CountryCharacterMap` · `MapRoute` · `MultiCountryOutline` · `RegionLocationText` · `SatelliteDrawPath` · `SatelliteLocationPin` | **`#f59e0b`** ✅ |
| **`MapAnimation`** (o único que entra no vídeo) | **`#e0452e`** ❌ |

**Os 6 outros mapas do acervo já nasceram em `#f59e0b`.** O `MapAnimation` é a **exceção** —
e é justamente o que o operador vê no vídeo. Ou seja: a troca não introduz um padrão novo,
ela **conserta o outlier** e alinha a produção com o acervo inteiro (as 40 variantes, o
`ChapterTitle`, os gráficos, e os 6 mapas acima).

❓ **Fora do escopo (a decidir):** o outro caminho de mapa, `SatelliteZoom.tsx`, usa **verde
`#34d399`** ("verde scanner de satélite") — **não é vermelho**, então não entra nesta troca
por padrão. Se o operador quiser unificar também, é decisão dele.

#### 7.2.1b — ✅ VALORES FECHADOS do `MapAnimation` (operador, 04/08)

> Refinado em `Desktop/mapa_chapter_refino.html` (geografia real, d3-geo + world-atlas).
> A maioria dos valores **confirma** o que já está no código; muda o pin/anel e o card.

| Parâmetro | ✅ Aprovado | Hoje no `.tsx` | Muda? |
|---|---|---|---|
| **cor de destaque** | **`#f59e0b`** | `#e0452e` | **sim** (§7.2.1) |
| preenchimento do país | 0.90 | `0.12 + 0.78·hl` = 0.90 | — |
| contorno do país | 1.40 | 1.4 | — |
| zoom inicial (mundo) | 230 | 230 | — |
| zoom final (país) | 1500 | 1500 | — |
| **tamanho do pin** | **9** | 7 | **sim** |
| **alcance do anel** | **30** | 16 | **sim** |
| **espessura do anel** | **2.50** | 2 | **sim** |
| **espessura da linha tracejada** | **1.50** | 2 | **sim** |
| card — posição horizontal | 70% | `W·0.70` | — |
| card — posição vertical | 14% | `H·0.14` | — |
| **card — largura** | **500** | 380 (com `local`) / 300 (sem) | **sim** |
| **foto — altura** | **280** | 224 (com `local`) / 180 (sem) | **sim** |
| foto — borda / canto | 3 / 6 | 3 / 6 | — |
| legenda sob a foto / espaço acima | 28 / 10 | 28 / 10 | — |
| título grande — tamanho / letras | 96 / 3 | 96 / 3 | — |
| título — margem esq. / base | 6% / 16% | 6% / 16% | — |
| barra — largura / espessura | 150 / 8 | 150 / 8 | — |

> 📐 **Leitura:** o pin ficou **maior e mais visível** (pin 7→9, anel 16→30, mais grosso) e a
> linha tracejada **mais fina** (2→1.5) — ou seja, mais destaque no ponto, menos no traço até
> o card. E o card da foto **cresceu** (380→500, foto 224→280), coerente com o padrão das
> outras famílias: o operador vem aumentando tudo que carrega informação.

**⚠️ 2 pontos de atenção na implementação:**
1. **O componente tem DOIS layouts** — com `local` (card 380/foto 224) e sem `local`
   (300/180). Os valores aprovados são do layout **com `local`**. Falta decidir o que
   acontece com o layout sem `local`: escala na mesma proporção (≈ 395/230) ou fica como
   está? *(o layout sem `local` é o dos nichos do Piter)*.
2. **"clareza dos outros países" = 0.10** no refino equivale a ≈ `rgb(33,40,59)`, enquanto o
   código usa `#151b29` = `rgb(21,27,41)`. O controle do HTML **não partia do valor do
   código** — se a intenção era manter os outros países como estão, o valor a aplicar é
   `#151b29`; se era clarear de leve, é o `rgb(33,40,59)`. **Confirmar com o operador.**

#### 7.2.2 — Mapas guiados pelo TEMA do canal ⏳

**Estado:** escopo futuro **anunciado pelo operador** no GO do tema claro/escuro, e
explicitamente **fora daquele GO**. A regra do tema já está implementada e validada para as
apresentações com fundo (`film` + `polaroid` + `grid`); falta estendê-la à representação de
mapas — as cores do mapa passam a seguir o tema do canal (claro/escuro), do mesmo cadastro.

Ponto de partida quando for implementar: `director/detectar_mapas.py` + a camada 1.5 de
mapas no `BrollTest.tsx`, e o `tema_visual` que o `preparar_render.py` já expõe.

> Nota: 7.2.1 (cor) e 7.2.2 (tema) mexem no MESMO componente. Se forem implementados juntos,
> a cor de destaque provavelmente vira **parte do tema** em vez de constante fixa — vale
> decidir isso antes de escrever as duas coisas separadas.

#### 7.2.3 — 🗺️ INVENTÁRIO COMPLETO DOS MAPAS (levantado 04/08)

> O operador procurou "as animações de mapas de **viagem/locomoção**" e não achou o caminho.
> Elas existem — estão em **dois lugares diferentes**, e nenhum deles é o mapa que entra no
> vídeo hoje. Inventário completo para nunca mais procurar:

**A) Em PRODUÇÃO** (`BrollTest` importa, vira vídeo):

| Componente | O que faz | Accent |
|---|---|---|
| `MapAnimation.tsx` | mundo → zoom no país + pin + card com foto + título grande | `#e0452e` → **trocar** |
| `SatelliteZoom.tsx` | pilha de imagens de satélite (zoom Google-Earth) | verde `#34d399` |

**B) COMPONENTES SOLTOS** em `remotion/src/compositions/` — prontos, **não plugados**, todos
já em `#f59e0b`. **É aqui que estão as de viagem/locomoção:**

| Componente | O que faz |
|---|---|
| **`MapRoute.tsx`** | 🧳 **VIAGEM A→B**: mapa enquadrado nos 2 pontos, pins de origem/destino, **arco curvo desenhado** + **marcador viajante** percorrendo + labels |
| **`SatelliteDrawPath.tsx`** | 🧳 **TRAJETÓRIA**: mapa zoomado no local com uma **rota curva sendo desenhada** atravessando o quadro (vibe "traçar rota") |
| `SatelliteLocationPin.tsx` | mapa zoomado no ponto + pin |
| `CountryCharacterMap.tsx` | país destacado + personagem |
| `MultiCountryOutline.tsx` | vários países contornados (enquadra a região) |
| `RegionLocationText.tsx` | país destacado + texto de região |

**C) ACERVO `AcervoMapas.tsx`** — 13 variantes catalogadas com manifesto (`quando` usar cada
uma), **não plugadas**. As de deslocamento:

| Variante | Quando (do manifesto) |
|---|---|
| **`Map03_RouteArc`** | 🧳 rota/ligação **A→B** (2 pontos lat/lon) |
| **`Map06_PathTrail`** | 🧳 **expedição/jornada multi-parada** (3+ pontos) |
| **`Map11_StreetRoute`** | 🧳 **trajeto urbano A→B estilo navegação** (km real calculado) |
| `Map01_CountryFocus` · `Map02_MultiHighlight` · `Map04_PinCallout` · `Map05_RegionZoom` · `Map07_RadarSweep` · `Map08_StatMap` | país/ponto/região/radar/stat |
| `Map09_EarthZoom` · `Map10_SatPinApp` · `Map12_SatTarget` · `Map13_CineLocation` | satélite (precisam de imagery) |

> Total de mapas no projeto: **2 em produção + 6 soltos + 13 no acervo = 21**.

**D) 🚢 ENGINE PRÓPRIO DE ROTAS** — `acervo/mapas/` (fora do Remotion; Playwright + d3).
Não é componente React: é um renderizador próprio com `map_core.js` (matemática pura) +
`map_render.py`. Tem **12 receitas** e **10 previews renderizados**, e cobre **8 modos de
transporte**: `aviao` · `navio` · `trem` · `carro` · `bicicleta` · `cavalo` · `a_pe` ·
`exercito` — com rota marítima por **A\* com penalidade de costa** (o navio navega mar
aberto, não colado na terra) e seta de guerra para exército.

> A [ESPEC-MAPAS.md](ESPEC-MAPAS.md) que o descreve foi **declarada obsoleta** pelo operador
> em 04/08 (a arquitetura escolhida foi a dos componentes React). **Mas o artefato existe e
> funciona**, e cobre uma capacidade que **nenhuma** das 14 selecionadas tem: rota **por modo
> de transporte**. O `Map03_RouteArc` desenha um arco genérico; ele não sabe se foi de navio
> ou a pé.
>
> Registrado aqui para não se perder. Se um roteiro disser *"a expedição partiu de navio do
> Brasil até a Austrália"*, a resposta visual existe — só não está plugada nem selecionada.
> **Não é pendência desta espec** (o operador não o incluiu na seleção); é um ativo conhecido.

#### 7.2.4 — ✅ SELEÇÃO PARA PRODUÇÃO (operador, 04/08) — **14 dos 21**

> Escolhidos no catálogo `Desktop/mapas_21_catalogo.html`. Estes ficam **à disposição do
> Diretor de Documentário**; os outros 7 permanecem no acervo, sem regra de uso.
>
> *(Houve uma 1ª lista com 11 antes desta. O operador **confirmou** a diferença: trocou o
> `MapRoute` pelo `Map03_RouteArc` — "faz basicamente a mesma coisa" — e acrescentou
> `Map04`, `Map05` e `Map06` por interesse editorial. **Esta lista de 14 é a definitiva.**)*

| Mapa | Origem | Serve para |
|---|---|---|
| `MapAnimation` | produção | mundo→país + card com foto (o de hoje) |
| `SatelliteZoom` | produção | mergulho de satélite |
| `CountryCharacterMap` | **solto** | país + nome grande + barrinha (§7.2.5) |
| `Map01_CountryFocus` | acervo | um país acende |
| `Map02_MultiHighlight` | acervo | vários países + valores |
| **`Map03_RouteArc`** | acervo | 🧳 **rota A→B** *(substituiu o `MapRoute`)* |
| `Map04_PinCallout` | acervo | ponto específico + card com coordenadas |
| `Map05_RegionZoom` | acervo | mergulho cinematográfico mundo→país |
| **`Map06_PathTrail`** | acervo | 🧳 **expedição multi-parada** |
| `Map08_StatMap` | acervo | país + stat gigante |
| `Map09_EarthZoom` | acervo | mergulho Google-Earth |
| `Map10_SatPinApp` | acervo | pin estilo app de mapas |
| `Map12_SatTarget` | acervo | vibe recon militar |
| `Map13_CineLocation` | acervo | cartão cinematográfico do lugar |

**Fora (7):** `MapRoute`, `SatelliteDrawPath`, `SatelliteLocationPin`, `MultiCountryOutline`,
`RegionLocationText`, `Map07_RadarSweep`, `Map11_StreetRoute`.

✅ **Sobreposição de viagem RESOLVIDA por decisão explícita:** ficou o `Map03_RouteArc`
(acervo); o `MapRoute` (solto) saiu. Uma só rota A→B no repertório, sem risco de divergência
— e com o `Map06_PathTrail` junto, o repertório cobre também **expedição multi-parada**.
*(Sem trajeto urbano: o `Map11_StreetRoute` ficou fora.)*

🎯 **Consequência arquitetural — o repertório ficou quase todo no acervo.** Dos 6 componentes
soltos, **só o `CountryCharacterMap` entrou**; os outros 5 saíram. Ou seja: **11 das 14
variantes vêm do `AcervoMapas.tsx`**, que compartilha base visual (`Base`), validação
geográfica (`resolverPais`, que rejeita continente) e enquadramento automático
(`projecaoFit`). Isso torna a implementação **muito mais barata do que 14 componentes
independentes**: uma mudança no `Kicker` ou no `Mundo` alcança 11 de uma vez — é o que faz a
regra do kicker 33 (§7.2.6) ser **uma edição só**.

> 💡 **Opcional:** o `MapRoute` tinha um **marcador viajante** percorrendo o arco, que o
> `Map03_RouteArc` não tem. Portar custa ~4 linhas e deixaria o `Map03` com o melhor dos
> dois. Sugestão, não bloqueio.
⏳ **Pendências da seleção:**
1. **6 das 14 dependem de imagem** (`SatelliteZoom`, `Map09`, `Map10`, `Map12`, `Map13` de
   satélite/foto; `MapAnimation` de foto do local) — o `satelite_fetch.py` precisa cobrir
   todas, senão a variante é recusada em runtime.
2. **Falta a regra de uso** (qual situação chama qual) — hoje só `MapAnimation`/`SatelliteZoom`
   têm gatilho, via `detectar_mapas.py`. As **12 novas** precisam entrar nesse pass. O
   `MAPAS_MANIFEST` já traz o campo `quando` de cada uma: é o insumo pronto para essa regra.
3. ✅ **FILTRO DE ELEGIBILIDADE — decidido pelo operador (04/08).** *"é só se tiver essas
   informações mesmo que cada animação dessa deverá ser usada, um filtro para elas não
   entrarem em lugar errado."*

   **Regra:** o gatilho **só pode escolher** uma variante se os dados exigidos por ela
   existirem. A checagem roda **antes** da escolha, não depois — a variante nunca deve ser
   selecionada e só então recusar.

   | Variante | Só entra se tiver | Fonte |
   |---|---|---|
   | `Map02_MultiHighlight` | **2+ países** resolvidos | `validarGeo().paisesOk.length >= 2` |
   | `Map03_RouteArc` | **2 pontos** com lat/lon | `pontosOk.length >= 2` |
   | `Map06_PathTrail` | **3+ pontos** | `pontosOk.length >= 3` |
   | `Map08_StatMap` | país **+ valor ancorado** no roteiro | `temPais && valores[0]` |
   | `Map01` · `Map05` · `CountryCharacterMap` | **1 país** | `temPais` |
   | `Map04` · `Map09` · `Map10` · `Map12` | **1 ponto** lat/lon | `temPonto` |
   | `Map09` · `Map10` · `Map12` | **+ imagem de satélite** | `sat.length` (ver pendência 1) |
   | `Map13_CineLocation` | **1 foto** + país ou ponto | `images[0] && (temPais\|\|temPonto)` |

   ⚠️ **O "valor ancorado" do `Map08` é regra editorial, não só técnica:** o Contrato
   Editorial (grupo B) proíbe inventar dado que não está no roteiro. O valor tem que **vir
   da fala**, não do LLM — se não houver número dito, o `Map08` não entra. Mesma disciplina
   do `R-32` que já existe no acervo ("sem dado real = nada").

   💡 **A função já existe:** `validarGeo()` é exportada pelo `AcervoMapas.tsx`
   justamente "p/ o registry/Diretor decidir ANTES de escolher a variação" (comentário no
   próprio arquivo). O filtro **não precisa ser escrito do zero** — é ligar o que já está lá.

5. 🔒 **GATING DOS MAPAS (I1, mecanismo exato):** o schema `MapaSeg` do timeline ganha o
   campo **opcional `variante?`**. **Entrada SEM `variante` = comportamento de hoje, byte a
   byte** (`tipo: satelite` → `SatelliteZoom`; senão → `MapAnimation`). As 12 variantes
   novas só entram quando o `detectar_mapas.py` escrever o campo — e ele só escreve com o
   knob do preset ligado. Timeline antigo, preset sem knob, replay de manifesto: tudo
   continua rendendo idêntico. *(É o mesmo padrão do `fundoRel` das apresentações.)*
   Nota: o `Map13_CineLocation` precisa de **foto** (`images[0]`) — reutilizar o
   `imagem_rel` que o pipeline do mapa **já busca** para o card do `MapAnimation`, não criar
   um segundo caminho de foto.
4. ✅ **`Map10_SatPinApp` MANTÉM o `#ea4335`** *(decidido 04/08)* — o vermelho é o do Google
   Maps e a imitação do app é o ponto do componente (pin gota + card branco); em âmbar ele
   perde a referência. **Exceção deliberada à regra do §7.2.1** — anotar no código para que
   ninguém "corrija" depois achando que passou batido.
   *(O `Map11`, que tinha o azul `#2f6fed`, ficou fora da seleção.)*

#### 7.2.6 — ✅ VALORES FECHADOS dos mapas (export do HTML, 04/08)

> Exportado com o botão "copiar TODOS os valores". **12 dos 21 ficaram sem alteração** —
> defaults aprovados como estão.

##### 🔤 Regra de família: **kicker 25 → 33**

Em **todos os 4 mapas que têm kicker** e foram ajustados (`Map01`, `Map02`, `Map05`,
`Map08`), o operador subiu o kicker de **25 para 33** — sempre o mesmo número. Não é valor
por variante, é **decisão de família**: aplicar 33 no `Kicker` compartilhado do
`AcervoMapas.tsx` (linha ~19), que serve as 13 variantes de uma vez.

##### Alterações por mapa

| Mapa | O que mudou |
|---|---|
| **`Map01_CountryFocus`** | kicker **33** (25) |
| **`Map02_MultiHighlight`** | kicker **33** (25) · nome no card **36** (28) · valor no card **44** (34) |
| **`Map05_RegionZoom`** | kicker **33** (25) · label **72** (58) · altura do label **92** (90) |
| **`Map08_StatMap`** | kicker **33** (25) · nome do país **40** (32) · stat **200** (150) · legenda do stat **34** (30) · largura da barra **170** (130) |
| **`Map13_CineLocation`** | nome do lugar **96** (76) · letterbox **140** (138) · legenda **30** (29) |
| `SatelliteZoom` · `Map12_SatTarget` | só o zoom (1.35→1.36 / 1.15→1.16) |
| **Sem alteração (12)** | `MapAnimation`*, `MapRoute`, `SatelliteDrawPath`, `SatelliteLocationPin`, `CountryCharacterMap`, `MultiCountryOutline`, `RegionLocationText`, `Map03`, `Map04`, `Map06`, `Map07`, `Map09`, `Map10`, `Map11` |

> \* `MapAnimation` aparece "sem alteração" porque o HTML **já partia dos valores aprovados
> no §7.2.1b** (pin 9, anel 30, card 500, foto 280, cor `#f59e0b`). Esses continuam valendo —
> a referência é o §7.2.1b, não os defaults do componente.

##### 🔇 Deltas de 1-2px = ruído de slider, **não implementar**

`SatelliteZoom` zoom 1.35→**1.36** · `Map12` zoom 1.15→**1.16** · `Map13` letterbox 138→**140**
e legenda 29→**30** · `Map05` altura do label 90→**92**.

São variações de um passo do controle, **invisíveis em vídeo**. Registrado para que ninguém
gaste tempo aplicando — e para não poluir o diff com mudanças sem efeito. Se algum deles for
intencional, o operador avisa.

##### 📐 Leitura

Mesmo padrão de todas as famílias anteriores: **tudo que carrega informação cresceu** — stat
150→200, nome do lugar 76→96, label 58→72, valor no card 34→44, nome do país 32→40. E o
kicker subiu em bloco. Geometria (enquadramento, contornos, atrasos, deslocamento) ficou
**intocada** em todos: o layout estava certo, o problema era **escala de leitura** — exatamente
o que já havia acontecido na família OVERLAY.

#### 7.2.5 — ➕ Barrinha no `CountryCharacterMap` (operador, 04/08)

> *"tem como adicionar essa barrinha que tem ali de detalhe embaixo do nome do país no
> MapAnimation"* — sim. É o mesmo elemento do `MapAnimation` (`width 150 · height 8 ·
> background accent · marginTop 12 · borderRadius 4`), aplicado sob o nome do país.

Implementação: envolver o nome num container e adicionar a barra logo abaixo (o nome hoje é
um `div` solto posicionado por `left/top`).

✅ **Valores fechados (export de 04/08): os defaults**, ou seja, **idênticos aos do
`MapAnimation`** — largura **150** · espessura **8** · distância do nome **12** · canto **4**.
O operador não mexeu em nenhum dos 4 controles, o que confirma a intenção original: *a mesma
barrinha*, não uma variação.

### 7.3 — ✅ Regra de frequência: DECIDIDA (04/08)

Fechada com o operador nas três dimensões — gatilho, colisão e escolha entre as duas. O
texto normativo está na espec irmã (Item 1, "REGRA DE FREQUÊNCIA — FECHADA"); resumo em
§7.1 acima. **Nada mais a decidir neste ponto.**

---

## 8. ⏳ PENDÊNCIAS ABERTAS — **FORA DESTE GO** (rodada e espec próprias)

> Revisado em 04/08. As três pendências originais viraram **duas** — a terceira foi
> respondida pelo operador.
>
> ✅ **Decidido (04/08): este bloco NÃO entra no GO atual.** O GO cobre os passos 1-7 do
> relatório (aparência + mapas plugados + capítulo/transições). A regra de uso das 40
> variantes vira espec dedicada.
>
> 🛡️ **Por que isso é bom:** com o §8 fora, o **`ilustrar.py` não é tocado**. Ele é o pass do
> Sistema 1, que está em produção e **acabou de receber** os valores novos das 13 ilustrações
> (espec irmã, passo 3). Mexer nele agora empilharia mudança sobre mudança recém-feita —
> exatamente o que o `feedback_aditivo_apenas` proíbe.

**8.1 — Quando cada variante de TEXTO/GRÁFICO entra (regra de uso).** Qual situação chama
`Ovl01_ChapterBig`, qual chama `Graf11_PieSlices`. Hoje o `ilustrar.py` só conhece o
Sistema 1 (13 ilustrações). **Insumo pronto:** cada família exporta seu manifesto
(`TEXTO_MANIFEST`, `OVERLAY_MANIFEST`, `GRAFICOS_MANIFEST`) com o campo `quando` de cada
variante — é a base da regra, do mesmo jeito que o `MAPAS_MANIFEST` é para os mapas.

> ✅ **Para os MAPAS isso já está resolvido** (§7.2.4, filtro de elegibilidade). O modelo
> vale para as outras famílias: *checar os dados exigidos ANTES de escolher a variante*.

**8.2 — Como o LLM preenche as props.** Cada família tem contrato próprio:
`{text, kicker, accent}` para texto/overlay; `{title, kicker, labels, values, suffix}` para
gráficos; `{paises, pontos, valores, titulo, kicker}` para mapas. Precisa de um pass (ou
extensão do `ilustrar.py`) que devolva o shape certo por variante.

⚠️ **O `R-32` do acervo é lei aqui:** as variantes de gráfico e mapa **recusam** (retornam
`null`) sem dado real. Combinado com o Contrato Editorial (grupo B: não inventar informação
fora do roteiro), isso significa que **o LLM não pode "preencher" número que não foi falado**
— ou vem da fala, ou a variante não entra.

---

### ✅ Pendência RESPONDIDA (registro)

**Convivência com o Sistema 1 → CONVIVEM** *(operador, 04/08)*. O acervo **não substitui** as
13 ilustrações; entra como **adição**. Por isso o §5 da espec irmã (valores das ilustrações)
foi implementado normalmente, sem esperar por esta espec. Ver §4-D do
[RELATORIO-IMPLEMENTACAO-EDICAO.md](RELATORIO-IMPLEMENTACAO-EDICAO.md).
