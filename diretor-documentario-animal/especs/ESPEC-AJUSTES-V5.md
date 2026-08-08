# ESPEC DE AJUSTES v5 — ESPEC FINAL DE IMPLEMENTAÇÃO (revisão do operador sobre os v5 finais)

> **Status:** ✅ **EXECUTADA (06/08).** GO dado e os 17 itens implementados; a PARTE 3 no
> fim registra o que a execução PROVOU, o que ela REFUTOU (uma hipótese minha caiu) e o
> que ficou fora do alcance deste render.
>
> Investigação feita DIRETO NO CÓDIGO e nos render-json entregues antes de qualquer
> conserto — método Sherlock: primeiro o PORQUÊ, depois o fix. Tudo distingue **FATO**
> (evidência citada) de **HIPÓTESE** (assim marcada).
>
> Veredito dele: *"NO GERAL, ESTÁ PERTO, MUITO PERTO DA PERFEIÇÃO, MUITO MESMO."*

---

# ORDEM DE ANÁLISE (e por quê nessa ordem)

| # | Item | Gravidade | Causa raiz |
|---|---|---|---|
| S1 | Tela preta 3:06→3:15 (urso) | 🔴 bug na tela | `pop_bg_*` órfão de run anterior + `return null` no BrollTest |
| S2 | Sistema de volume dos SFX (prints 2/3/6 + 5:07) | 🔴 recorrente | loudness dos ARQUIVOS varia 18 LU; multiplicador uniforme não é calibração |
| S3 | Transição no meio da cena (1:26 + OBS geral) | 🔴 lei violada | par foto/vídeo do MESMO asset em cenas adjacentes + guarda D13 só existe p/ tópicos |
| S4 | Filmadora na configuração errada (print 4) | 🔴 | filmadora escolhida p/ cena que DEPOIS virou herança-com-moldura |
| S5 | SFX solto no print 1 (0:12) | 🟠 | overlay+transição+SFX debaixo de card tela-cheia (D3, já corrigido pós-render) |
| S6 | LowerThird anuncia capítulo 4 antes do card | 🔴 | embargo por INTENÇÃO não cobre ilustração; falta embargo por CONTEÚDO |
| S7 | Crocodilo revelado 1 cena antes do top 1 | 🔴 | cena atravessa o marcador "Number one"; embargo só vive no resolver (nunca re-validado) |
| S8 | SFX específicos por elemento (Typewriter/GradientGlow/SpecBadge/BoxedKicker) | 🟠 | pares errados na tabela `sfx_por_variante` |
| S9 | Ápice do SFX de impacto na ENTRADA | 🟠 lei nova | áudio começa no frame de entrada; pico do arquivo cai depois |
| S10 | 🔴 LEI DOS NÚMEROS (global, nova) | prioridade | cota Pop=2 recusou 3 elementos numéricos no urso ENTREGUE |

---

# S1 · TELA PRETA 3:06→3:15 (urso) — CAUSA FECHADA COM EVIDÊNCIA

**Sintoma:** *"3:06 é o segundo momento que ficou preto, cena nenhuma, só voltou a cena
em 3:15. ERRO."* Frame extraído em 188s = preto puro (FATO).

**Cadeia da causa (FATO, lida do `_render_urso_polar_v5_final.json`):**
```
173.7-180.3  pop_bg_dono=10 (ate=194.86)  E AO MESMO TEMPO  pop_bg_membro=9
180.3-186.4  pop_bg_membro=10
186.4-194.9  pop_bg_membro=10
```
`BrollTest.tsx:793` — `if (c.pop_bg_membro != null) return null;` — membro NÃO renderiza
(confia que o dono cobre). A cena 173.7 é DONA do grupo 10 **e MEMBRO do grupo 9**: o
check de membro vem primeiro → a dona devolve `null` e **nunca renderiza a cobertura até
194.86**. As duas cenas seguintes são membros → `null` → **nada na tela de ~180s a 194.9s**
(o trecho 180-186 foi parcialmente coberto por outro dono; 186.4→194.9 ficou 100% preto).

**De onde vieram grupos 9 e 10 se o vídeo final só tem 2 janelas de word-pop?** Do run
ORIGINAL das `enumeracoes` (21 janelas → grupos 1-20). O `edicao` do run final podou para
2 janelas mas **reescreve só `tl["enumeracoes"]` — as marcas `pop_bg_*` POR CENA ficam
órfãs**. O W6 do preparar (linhas 543-545) limpa as marcas apenas das janelas que ELE
MESMO descarta. Mesma família do E19: *passe que não limpa a própria saída não é
re-executável*.

**Correção (4 camadas):**
1. `edicao.py`: ao reescrever `enumeracoes`, limpar `pop_bg_dono/ate/membro` de TODAS as
   cenas e re-marcar só para as janelas sobreviventes.
2. Invariante nova (I17): **cena nunca é dono e membro ao mesmo tempo** — o marcador das
   `enumeracoes` deve recusar encadeamento de grupos sobrepostos.
3. `BrollTest` defensivo: membro sem dono vivo cobrindo o intervalo **renderiza normal**
   — cena com mídia NUNCA devolve `null`.
4. Portão: o D5 (frames pretos) passou com **"0 amostras"** — não amostrou nada e não viu
   9s de preto. Passa a amostrar o vídeo INTEIRO (1 frame a cada 2s, luma média < 3 = preto).

---

# S2 · SISTEMA DE VOLUME DOS SFX — a resposta ao "conferir se todos estão com default 0.20"

**Auditoria dos multiplicadores (FATO):** `sfx_vols` + `sfx_vol_max=0.2` estão presentes
nos DOIS render-json entregues; todo `<Audio>` do BrollTest passa pelo clamp `sfxVol()`…
**exceto UM**: `BrollTest.tsx:1294` — o 2º click do CTA tem `volume={0.7}` **hardcoded**,
fora do clamp. Corrigir para `sfxVol(timeline,"click",0.15)`.

**Então por que os prints 2/3 soaram ALTOS se o multiplicador era ≤0.20?** Porque o que
varia é o **loudness do ARQUIVO** (medido, ebur128):

| Arquivo | LUFS integrado | Observação |
|---|---|---|
| ui_click_01 (calibrado 0.15 pelo operador) | **-10.9** | referência: arquivo ALTO × mult. baixo |
| glitch_digital_01 | -12.1 | tão alto quanto o click, mas roda a 0.20 |
| riser_uplifter_01 | -12.3 | alto E LONGO (crescendo de segundos) |
| impact_hit_02 (print 2, CenterPunch 29.2s) | -14.4 | sustentado |
| whoosh_air_02 | -16.4 | |
| typewriter_tapping (novo, do operador) | -23.5 | arquivo BAIXO |
| counter_digital | **-29.3** | 18 LU mais baixo que o click! |

**FATO:** 0.20 × riser (-12.3) é ~7× mais alto ao ouvido que 0.20 × counter (-29.3). Os
multiplicadores estão uniformes; a CALIBRAÇÃO POR ARQUIVO nunca existiu — o operador
calibrou de ouvido os 2 primeiros (click, transição) e os ~15 novos entraram crus.

**Correção definitiva (uma vez, para sempre):**
1. **Normalizar OS ARQUIVOS** do `acervo/sfx/` para um alvo comum (one-shots curtos por
   PICO, sustentados por LUFS integrado; alvo escolhido para que 0.20 soe como os já
   aprovados). Backup dos originais em `acervo/sfx/_originais/`.
2. Multiplicadores ficam simples e previsíveis: **0.20 default · 0.15 clique/transição ·
   0.13 typewriter** (regra permanente do operador, `sfx_vol_max=0.2` continua).
3. Script `_calibrar_sfx.py` grava `catalogo_sfx.json` com LUFS/pico/offset-do-pico por
   arquivo — vira insumo do S9 e prova de auditoria.

---

# S3 · TRANSIÇÃO NO MEIO DA CENA (1:26 urso + OBS geral do operador)

**Sintoma (OBS verbatim):** *"ele entra com o overlay, sua transição e seu sfx e mantém
O MESMO TAKE DE FUNDO por mais 5 segundos, isso é erro."*

**FATO (frames extraídos de 84.6s e 86.5s):** a MESMA paisagem dos dois lados da fronteira
85.7s onde a transição `brilho` disparou (entrada da moldura film). E no timeline:
```
77.4s  p126de87a4617.jpg   «Polar bear slowly walking sea ice»
85.7s  vee4777d5ee11.mp4   ← VÍDEO do asset ee4777d5ee11
92.1s  pee4777d5ee11.jpg   ← FOTO DO MESMO ASSET ee4777d5ee11
```
O pool entrega o **par foto+vídeo do MESMO asset em cenas adjacentes** → corte invisível.
A guarda **D13** ("transição só em corte real") **existe apenas para transições de
TÓPICO** (preparar ~850-869); os emissores **R2 (par da moldura, linha 1081-1090)** e
**B5 (par do overlay de take, linha ~1160)** adicionam transição SEM nenhuma checagem de
troca de take. E o D13 compara `clip_id` — que não enxerga o par p/v (ids diferentes,
mesmo conteúdo).

**Correção — com uma HIPÓTESE MINHA REFUTADA no caminho:**

Eu havia proposto "identidade de asset = `pXXX`/`vXXX` → `XXX`, proibir o par em cenas
adjacentes". **Medido na implementação: está ERRADO.** A foto e o vídeo do mesmo id do
Pexels têm aHash com **41 bits de diferença** (`p126de87a4617.jpg` × `v126de87a4617.mp4`)
— são imagens bem distintas. Cortar por id igual removeria transições legítimas, e ids
diferentes podem trazer a mesma paisagem. Faixa medida entre assets distintos: 24-41 bits.

1. **Guarda D13 universal e PERCEPTUAL** (implementada): todo emissor de transição
   (tópico, moldura R2, overlay B5) passa pela mesma barreira, e o critério é o PIXEL —
   aHash 8×8 dos dois lados da emenda, hamming ≤ 6 = a tela não mudou = sem transição e
   sem SFX.
2. **Exceção**: se a MOLDURA muda na fronteira (herança de hard-miss embrulha de
   propósito), a mudança visual existe e a transição é merecida mesmo com o take igual.
3. Pool: evitar assets perceptualmente próximos em cenas adjacentes fica como melhoria do
   resolver (não bloqueia — a barreira já impede o sintoma).

---

# S4 · FILMADORA ERRADA (print 4) vs CERTA (7:00 e exemplo.mp4)

**FATO (frames comparados):** o `exemplo.mp4` do operador = footage FULLSCREEN + apenas
o viewfinder (REC, bateria, colchetes) por cima. O uso de **417.4s (7:00) saiu idêntico ao
exemplo** ✓. O de **194.9s saiu errado** porque a cena embaixo estava com
`presentacao=quadro` + foto herdada (`scene_34.jpg`): viewfinder sobre uma FOTO ENCOLHIDA
dentro de moldura.

**Causa raiz (FATO):** o `edicao` escolheu filmadora para uma cena que naquele momento
**não tinha take** (hard-miss). O preparar depois herdou o take do vizinho **dentro de
moldura quadro** (regra obrigatória da herança, preparar 426-527) — e ninguém reconciliou:
moldura + filmadora empilharam. A config do asset (screen 0.6) está CERTA — não mexer.

**Correção:**
1. `edicao`: cena **sem `clip_path`** não é candidata a overlay de take.
2. Herança no preparar: ao embrulhar em moldura, **remove `overlay_take`** da cena
   (moldura obrigatória vence).
3. Validação final no preparar: **filmadora só sobre cena fullscreen** (`presentacao`
   None/kenburns, media vídeo). Violou → degrada para sem-overlay, com log. **"Nunca mais
   do jeito do print 4"** vira invariante checada, não intenção.

---

# S5 · SFX SOLTO NO PRINT 1 (0:12 urso)

**FATO (render json):** na cena 11.82s empilharam: overlay `barulho` (multiply) +
marcação `transicao apagao 11.8 origem=overlay_barulho` + SFX glitch — TUDO DEBAIXO do
card tela-cheia `Texto02_HighlightSweep` (9.1s→13.6s; o acervo renderiza POR CIMA da
cena). Overlay invisível, transição invisível, sons tocando = "SFX solto".

**Correção:** o **D3** (commitado pós-render: overlay de take não entra em cena coberta
por elemento tela-cheia) já mata o caso — e o par transição+SFX do B5 **nasce do mesmo
if**: overlay vetado = par vetado junto. Nada de som sem dono visível (reforça o I3).

---

# S6 · LOWERTHIRD ANUNCIANDO O CAPÍTULO 4 ANTES DO CARD (predadores)

**FATO (render json):** cena 104.0-109.8 carrega
`ilustracao = card lowerthird {titulo:"4. Great White Shark", sub:"Apex Predator"}`;
o card do capítulo 4 entra em **109.8**. Um elemento de OUTRO sistema (`ilustrar.py`,
texto vindo do LLM) nomeou o item numerado ANTES do anúncio falado.

**Causa raiz:** o veto A2/P1b do `edicao` filtra momentos do ACERVO com intenção
`capitulo`/`item_lista` — mas **ilustração não passa por esse filtro**, e não existe
embargo por CONTEÚDO (texto que nomeia um item de capítulo).

**Correção — embargo de conteúdo universal (edicao):** elemento de QUALQUER sistema
(ilustração, acervo, texto_impacto) cujo texto casa com item de capítulo (`^\d+[\.\)]` ou
o título do item, case-insensitive) com `t < inicio_do_card` → **descartado** (ou
re-vestido sem o texto do item). O anúncio de capítulo tem UM dono: o ChapterTitle.

---

# S7 · CROCODILO REVELADO 1 CENA ANTES DO TOP 1 (predadores)

**FATO:** cena 330.2-337.1 carrega o take `id3621e56df2b0` (saltwater crocodile) e começa
**4s antes** do marcador falado (~333.5s, *"Number one, the saltwater crocodile"*) e do
card (334.4s). A primeira frase da cena ainda é do cachalote (*"We know they happen
because of the marks…"*).

**Causa raiz (2 camadas):**
- (a) `montar_timeline` **não corta cena no marcador "Number X"** — a revelação cai no
  MEIO de uma cena, e o take (do sujeito novo) aparece desde o início dela;
- (b) o embargo do passo F vive **só no resolver** — este pipeline REUSA takes sem
  re-resolver, então **nada re-valida** (mesma família do E18/E20: validação apenas na
  origem morre no reuso).

**Correção:**
1. `montar_timeline`: **marcador de item = corte obrigatório de cena** (bônus direto no
   F2 cortes/min, o único fixador que ainda reprova).
2. **Validação de embargo no preparar** (roda em TODO pipeline, incluindo reuso): take
   cujo asset pertence ao sujeito do capítulo N não pode começar antes do card N. Cena
   violando → recebe o take da cena ANTERIOR (neutro) até o card.

---

# S8 · PARES SFX↔ELEMENTO (correções pontuais na tabela `sfx_por_variante`)

| Elemento | Hoje (FATO, tabela/preset) | Passa a ser | Origem do pedido |
|---|---|---|---|
| `Texto01_Typewriter` (e QUALQUER variante com "Typewriter") | ui_keyboard_02 | **`typewriter_tapping.mp3` @ 0.13** (✅ arquivo já copiado p/ `acervo/sfx/ui/`) | *"SEMPRE que usar qualquer efeito com Typewriter no nome"* |
| `Texto08_GradientGlow` | riser_uplifter_01 (-12.3 LUFS, crescendo) | **impact_boom_01 @ 0.20**, ápice na entrada (S9) | print 3 urso (2:54) E 5:07 predadores — o MESMO elemento reclamado nos 2 vídeos |
| `Ovl11_SpecBadge` | ui_beep_01 | **ui_click_01 @ 0.15** | print 6 predadores (4:55, "Sunlight not for a kilometre") |
| `Texto05_BoxedKicker` | foley_camera_shutter_01 (**pico -1.6 dB, o mais alto do acervo**) | **ui_click_01 @ 0.15** | proposta NOVA da vistoria (correção: o U-1 FOI aplicado — `Ovl05_CornerTag`/`Ovl08_SideNote` já têm clique; o BoxedKicker nunca foi alvo do U-1, mas carrega o arquivo de pico mais alto do acervo) |
| CTA 2º click | `volume={0.7}` hardcoded (BrollTest:1294) | `sfxVol("click", 0.15)` | auditoria S2 |

---

# S9 · LEI NOVA: ÁPICE DO SFX DE IMPACTO NA ENTRADA DO ELEMENTO

*"sfx de impacto deve ter seu ápice no momento de entrada dessa animação — o ápice do sfx
de impacto é sempre na entrada do elemento."*

**Causa de soar errado hoje:** o `<Audio>` começa NO frame de entrada; se o arquivo tem
build-up (riser tem SEGUNDOS de subida), o pico cai depois da entrada — som "atrasado/solto".

**Implementação:** o `_calibrar_sfx.py` (S2) mede o **offset do pico** de cada arquivo;
`preparar` grava `sfx_offsets`; BrollTest inicia o áudio em
`entrada_do_elemento − offset_do_pico` (clamp ≥ 0). Pico cai exatamente na entrada.

---

# S10 · 🔴 LEI DOS NÚMEROS (global — prioridade do operador)

*"peso, altura, força, distância, quantidade — SEMPRE apresentadas por elementos
dinâmicos, não importa se repete 2, 3 ou 4 vezes por vídeo; o que não pode é o mesmo
elemento em cascata subsequente."*

**Verificação no código — por que números morrem hoje (FATOS):**
1. `edicao.py` cotas §7: **Pop = 0.05** → orçamento 34 = **teto 2 por vídeo** (log do
   render entregue: `cotas v2: … Pop 2`). Recusas do urso final: **`cota Pop: 3`** — três
   elementos numéricos recusados NO VÍDEO ENTREGUE.
2. Word-pops pré-aceitos consomem a mesma cota Pop ANTES dos infográficos.
3. `montar_timeline` prompt: *"choose AT MOST ONE overlay: A) INFOGRAPHIC … B)
   TEXTO_IMPACTO"* — a 2ª medida da mesma frase morre na ORIGEM (o G/infografico2 só
   cobre a 2ª dimensão de UMA frase).
4. Teto §3.5 de 2× por variante recusa o 3º número que caia na mesma variante.

**Correção:**
1. Elemento com **dado numérico mensurável = OBRIGATÓRIO**: fora da cota Pop e fora do
   orçamento (mesmo mecanismo da exceção B — invioláveis continuam: dono da tela,
   capítulo soberano, respiro).
2. **Rodízio de variantes numéricas** (mesma mecânica do rodízio de molduras):
   `Graf01_CounterGlow · Graf02_Odometer · Graf03_DonutPercent · Graf04_GaugeMeter ·
   Graf10_BigStatCard · Graf14/15/16 · Ovl10/12/13 · infografico counter/odometer` —
   nunca a MESMA variante em números subsequentes; repetição permitida com outras no meio.
3. `montar_timeline`: prompt devolve **LISTA de medidas** (todas as dimensões da frase),
   não "no máximo uma".
4. Portão ganha **F13**: toda medida falada no roteiro (regex de unidades sobre o words)
   tem elemento dinâmico na janela da fala — reprova se faltar.

---

# ORDEM DE IMPLEMENTAÇÃO PROPOSTA

```
1. S1  tela preta (pop_bg órfão + BrollTest defensivo + D5 do portão)   [bug na tela]
2. S2  normalização dos ARQUIVOS SFX + volume hardcoded + catálogo      [base p/ S8/S9]
3. S8  pares SFX↔elemento (tabela)                                      [config]
4. S9  ápice na entrada (offsets)                                       [depende do catálogo]
5. S3  transição só com troca de take (guarda universal + par p/v)      [lei do operador]
6. S4  filmadora × herança (3 camadas)                                  [inclui D3 já commitado]
7. S6  embargo de conteúdo (capítulo tem um dono)                       [predadores]
8. S7  corte no marcador + embargo validado no preparar                 [predadores + F2]
9. S10 lei dos números                                                  [prioridade global]
10. Re-render dos dois + portão (com D5 e F13 novos)
```

*Aberto em 06/08/2026, sobre a revisão dos `urso_polar_v5_final` / `predadores_v5_final`.
AGUARDANDO GO.*


---
---

# PARTE 2 — 🔍 VISTORIA DAS ESPECS V3/V4 CONTRA O CÓDIGO (pedido do operador, 06/08)

> Pergunta-guia: *"os erros foram resolvidos NA FONTE, para sempre, independente de
> implementações futuras?"* Cada compromisso das duas especs foi conferido no código atual
> e nos render-json ENTREGUES. Resultado em 4 categorias.

## 2.1 · ✅ RESOLVIDO NA FONTE (verificado, com âncora)

| Compromisso | Prova |
|---|---|
| A1 título de capítulo MAIÚSCULO | `preparar_render.py:907` `.upper()` + FATO nos 5 cards do predadores |
| A2 `item_lista` fora do acervo em roteiro-lista | `edicao.py` filtro `capitulo`+`item_lista` |
| A3 capítulo obrigatório (sem corte → insere no marcador) | bloco A3 no preparar |
| A4 `antecipa_max_s = 0` (texto nunca adianta) | `presets.json:55/414` |
| A5 respiro por domínio (overlay 3s · tela-cheia 4s) | `_conflito_respiro` + log "respiro 4.0s (overlay 3.0s)" |
| B1 overlay de capítulo 1 por roteiro (hash) | FATO: vapor idêntico nos 5 capítulos do predadores |
| 1B.6/1B.9 blends calibrados (vapor screen 25 · barulho/quadro multiply 1.0 · escala 1.45) | `_OV_CAPITULO`/`_OV_TAKE`/`ChapterTitle` conferidos |
| C1 pisos de texto/min | logs do render final `[4,3,5,2,2,2,2]` / `[3,3,4,2,2,2,5,2]` |
| C2 rajada de famílias diferentes permitida | `_cabe` só pune família REPETIDA |
| C3 peso nas variantes preferidas | `acervo_texto.py:286-304` `PREFERIDAS` |
| C6 mood vermelho 2/3/4 por duração | `efeitos.py` bloco C6 |
| D1 rodízio de moldura CRONOLÓGICO com heranças | portão mediu sequência perfeita nos DOIS vídeos |
| D2/E pool com vet de sujeito + atmosfera | `pool_take_identidade`/`pool_take_atmosfera` (⚠️ ver 2.3) |
| U-1 clique no elemento do canto | `Ovl05_CornerTag`/`Ovl08_SideNote` → ui_click ✓ (correção: ontem afirmei que U-1 não fora aplicado — foi, no elemento certo; a proposta p/ BoxedKicker é NOVA) |
| U-4 whoosh no card serifado | `Texto04_EditorialSerif` → whoosh_air_02 (commit 4a3e6ea) |
| B exceção dos 3 min (aditiva, fora do teto E do orçamento, rodízio) | logs + json (14 `_exc` no urso) |
| `ilustrar` limpa a própria saída | `ilustrar.py:142` ("único dono do campo") |
| E4 sorteio da abertura persistido (duração) | `abertura.py:201-209` |

## 2.2 · 🟡 IMPLEMENTADO MAS **NUNCA EXERCITADO** — "feito" sem prova de vida

| Item | FATO | Risco |
|---|---|---|
| **G · multi-número por frase (infografico2)** | `infografico2` preenchidos nos vídeos ENTREGUES: **0 no urso, 0 no predadores** — os timelines vêm de resolves antigos e o `montar_timeline` (onde o prompt mudou) NUNCA re-rodou | o U-2/U-3 ("peso não entrou depois da altura") segue NÃO demonstrado; a 1ª produção fresh pode revelar defeito no caminho novo |
| **Lei dos números (S10)** | idem — depende do prompt novo do montar | mesma |

**Regra que a vistoria impõe:** item marcado "feito" só fecha com **prova de vida num
render** (o G está "pronto" há 2 dias e nunca produziu 1 frame).

## 2.3 · 🟠 RESOLVIDO SÓ NO CAMINHO OPCIONAL — morre no REUSO (a família de furo mais perigosa)

**O padrão:** a validação vive no passo que DECIDE (resolver/montar) — mas o pipeline de
produção REUSA decisões antigas e esses passos não rodam. FATOS desta rodada:

| Lei | Onde vive hoje | Como furou |
|---|---|---|
| **F · embargo de spoiler** | `resolver_cascata.py:1668-1691` (populado, funcional) | o take do crocodilo foi atribuído num resolve ANTERIOR ao F; o reuso nunca re-validou → spoiler no vídeo entregue (S7) |
| **E · vet de sujeito/atmosfera do pool** | `pool_take_*` no resolver | idem: take atribuído antes do vet fica para sempre |
| **D13 · transição só em corte real** | só no emissor de TÓPICO | R2 (moldura) e B5 (overlay) emitem sem guarda → 1:26 (S3) |
| **filmadora fullscreen** | escolha no edicao | herança do preparar mudou a cena DEPOIS → print 4 (S4) |

**Correção estrutural (é ISSO que responde "para sempre, independente de implementações
futuras"):** 🔴 **TODA lei ganha um VALIDADOR no caminho OBRIGATÓRIO** — o `preparar_render`
(único passe que roda em 100% dos pipelines) + o portão. O passo que decide continua
decidindo; o validador confere o RESULTADO FINAL, seja ele decidido agora ou reusado de
2 semanas atrás. Lista mínima de validadores a criar no preparar:
embargo de capítulo (conteúdo + take) · sujeito do take × sujeito da cena · atmosfera ×
setting · filmadora × fullscreen · transição × troca real de asset (hash p/v) · rodízio
cronológico (já existe — D1 é a PROVA de que o modelo validador funciona).

## 2.4 · ❌ FUROS REAIS ENCONTRADOS (novos, entram no GO da v5)

### S11 · A guarda anti-loop da narração NÃO PEGA os loops reais — prova aritmética
`narrar.py` re-gera chunk quando `ratio > 1.45`. Chunk = 400 chars ≈ 25-30s de áudio.
Os DOIS loops reais medidos: urso +5,22s → ratio ≈ **1,2**; predadores +2,18s → ratio ≈
**1,08**. **Nenhum dos dois dispararia a guarda.** O detector que pegou os dois foi o
TEXTUAL (`_deloop_audio.py`, ciclo+âncora, 0 falso positivo nos 3 roteiros-controle).
**Fix:** o deloop vira **passe automático pós-narração** (narra → transcreve → detecta →
excisa → só então segue); o ratio fica como telemetria. Sem isso, todo vídeo novo pode
nascer com loop de novo.

### S12 · `aprovar_render` assina "aprovado pelo OPERADOR" sem operador nenhum
`director/aprovar_render.py:34-39` (FATO): com erros no report, imprime *"aprovação
consciente do operador"* e concede o LOCK — **automaticamente**. Nos v5 finais: "Plano com
ERROS (10/11)" → LOCK no mesmo segundo. O carimbo mente sobre quem aprovou.
**Fix:** modo supervisionado → erros exigem confirmação real; modo automator → erros
estruturais CONHECIDOS (lista branca, ex.: hard-miss herdado) passam com carimbo
"auto-aprovado com exceções: […]"; erro fora da lista = **quarentena** (produz mas marca
NÃO-PUBLICAR).

### S13 · O portão reprova e NADA acontece
Os dois vídeos entregues FALHARAM no portão (F2) e seguiram para a pasta final normalmente.
Em produção autônoma isso é um portão decorativo.
**Fix:** portão FALHOU → vídeo vai para `_saida/_quarentena/` + notificação; exceções
estruturais aceitas pelo operador (F2 hoje) ficam numa lista explícita no preset, com data.

### S14 · Prova de IDEMPOTÊNCIA automatizada (fecha a família E19/E24 de uma vez)
3 incidentes da mesma família em 2 dias (`overlay_take` E19 · `pop_bg_*` E24 · suspeita em
`apresentar`). Um a um é enxugar gelo.
**Fix:** `_prova_idempotencia.py` — roda cada passe **2× sobre o mesmo timeline** e diffa o
resultado; qualquer campo que só cresce = reprovado. Vira parte da checagem seca pré-GO.
Cobre TODOS os passes de uma vez, incluindo os futuros.

### S15 · Palavra da abertura é loteria entre re-renders
FATO (V4 §8.1): mesmo roteiro rendeu 'ARCTIC' → 'ICE' em execuções distintas. O E4
persistiu a DURAÇÃO do sorteio, não a PALAVRA.
**Fix:** palavra escolhida por md5 do roteiro (mesma mecânica da trilha/cap-overlay) —
mesmo roteiro, mesma abertura, sempre. (Abertura é FROZEN por ordem do operador — isto não
muda o formato, só torna a escolha reprodutível.)

### S16 · SpecBadge sem a bolinha: causa achada, decisão pendente
`AcervoTextoOverlay.tsx:188-203` (FATO): a bolinha laranja só existe `{kicker ? …}` — badge
sem `kicker` (unidade) renderiza sem bolinha. Foi exatamente o print do v2 (C5, nunca
fechado).
**Fix:** I16 passa a exigir `kicker` para `Ovl11_SpecBadge` (sem unidade → re-veste noutra
variante). Badge de valor sem unidade não informa nada mesmo.

### S17 · Estruturais herdados que continuam abertos (não são desta espec, ficam nomeados)
- **E18 · workspace único** — jobs concorrentes ainda compartilham `_workspace/` (a guarda
  de identidade segura, mas o isolamento por job é a cura).
- **Zelador Z3/Z5/Z6/Z7** — degradações finas nunca implementadas.
- **VEO (E16)** — cobertura de repetição/espécie exata, adiado pelo operador.

## 2.5 · Ordem revisada (integra a PARTE 1)

```
0. S14 prova de idempotência (pega regressão dos itens abaixo DE GRAÇA)
1. S1  tela preta + D5 do portão
2. S2/S8/S9  SFX (arquivos normalizados + tabela + ápice + hardcoded 0.7)
3. S11 deloop automático pós-narração
4. 2.3 VALIDADORES no preparar (embargo/vet/filmadora/transição — inclui S3/S4/S6/S7)
5. S10 lei dos números + G exercitado em produção FRESH (2.2)
6. S12/S13 aprovar_render honesto + quarentena do portão
7. S15/S16 abertura determinística + SpecBadge
8. Re-render dos dois + portão novo
```

*Vistoria executada em 06/08/2026 sobre `ESPEC-AJUSTES-V3.md` e `ESPEC-AJUSTES-V4.md`,
ancorada no código real e nos render-json entregues. AGUARDANDO GO.*


---
---

# PARTE 3 — ✅ EXECUÇÃO (06/08): o que foi provado, o que caiu, o que ficou fora

## 3.1 · Descobertas que só apareceram IMPLEMENTANDO

| # | O que eu esperava | O que a medição mostrou |
|---|---|---|
| **S1** | marcas `pop_bg_*` órfãs | ...e mais: **4 das 17 donas eram TAMBÉM membras** de outro grupo. Como o BrollTest checa membro primeiro, a dona devolvia `null` e nunca cobria. Isso é o I17, que eu não tinha nomeado |
| **S1/D5** | detector de preto frouxo | o detector estava **CEGO por um caractere**: regex `YAVG:` contra o `YAVG=` do ffmpeg. Zero amostras → 0% → aprovava. E, corrigido, ele acusava **card escuro com texto** como tela preta — o que separa é o PICO (`YMAX 8` no preto real vs `244-255` no card) |
| **S2** | multiplicadores errados | os multiplicadores estavam **certos**; o problema eram os ARQUIVOS (27 LU de espalhamento). E nem pico nem LUFS integrado servem para medir isso — só o **loudness momentâneo (Mmax)** |
| **S3** | par foto/vídeo do mesmo id = corte invisível | 🔴 **HIPÓTESE MINHA REFUTADA**: `p126de87a4617` × `v126de87a4617` têm hamming **41** — são imagens BEM diferentes. Cortar por id igual removeria transições legítimas. O critério passou a ser 100% perceptual |
| **S11** | a guarda de ratio precisava de ajuste | ela **jamais** pegaria: os loops reais dão ratio 1,2 e 1,08 contra gatilho 1,45. Não era calibração, era o detector errado |
| **S12** | aprovação automática indevida | além disso, o report só gravava a **contagem** de erros — sem a lista, nem o carimbo cego nem a quarentena cega estariam certos |
| **E23** | `pais=None` num print | consertei 2 prints e quebrou em outros 3 escritos como **`m.get('pais','')`** — que parece protegido e não é: o default do `.get` só vale para chave AUSENTE, não para valor `None` |

## 3.2 · S14 — a prova de idempotência achou 3 casos no primeiro uso

Ela envenena os campos de DONO ÚNICO de cada passe, roda uma vez e cobra que o resíduo
tenha sumido (imune ao não-determinismo do LLM, que um diff de duas rodadas não seria).
Achou de cara: `produto_cta` (SKIP não apagava CTA de outro nicho), `legibilidade`
(`texto_ate` sobrevivia em cena que perdeu o overlay) e `edicao` — **reproduzindo sozinha
o E24**, a causa da tela preta.

**Resultado final: 12/12 passes limpam a própria saída.**

## 3.3 · O que este render NÃO valida (e por quê)

O v6 roda pelo `_rodar_pos_resolver`, que **reusa o timeline já resolvido**. Dois itens
vivem no `montar_timeline`, que só roda em produção FRESH:

- **corte obrigatório no marcador de item** (S7): implementado e medido em bancada —
  5/5 marcadores no primeiro frame da cena, contra 1 antes. Mas o v6 herda as fronteiras
  antigas. O `V4` da barreira cobre o sintoma (take do sujeito herda o vizinho).
- **prompt "toda medida = infográfico obrigatório"** (S10): a LEI no coordenador está
  ativa (número fora da cota, do teto e do orçamento + rodízio próprio); o que falta é a
  ORIGEM devolver mais medidas, e isso só na próxima produção do zero.

**Consequência honesta:** o F2 (cortes/min) deve continuar reprovando no v6 — ele depende
justamente do re-corte do timeline. Com o S13, o vídeo reprovado agora vai para
`_saida/_quarentena/` em vez da pasta de entrega.

## 3.4 · Continua em aberto (nomeado, não esquecido)

- **E18 · workspace único** — a guarda de identidade segura; o isolamento por job é a cura.
- **Zelador Z3/Z5/Z6/Z7** — degradações finas nunca implementadas.
- **VEO (E16)** — cobertura de repetição/espécie exata, adiado pelo operador.
- **`reuso a N cenas de distância`** — deliberadamente FORA da lista branca da aprovação:
  é repetição visível, e o operador tem que decidir se tolera ou se o pool precisa crescer.

*Executada em 06/08/2026.*
