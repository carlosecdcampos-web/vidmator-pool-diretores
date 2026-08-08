# ESPEC FINAL — EDIÇÃO DINÂMICA DO DIRETOR DE DOCUMENTÁRIO ANIMAL

> **Status:** 🟢 **GO DADO (operador, 05/08)** — implementação na ordem do §8.
> Decisão do polaroid: vídeo HABILITADO como fallback (ver §3.5).
>
> Documento definitivo que absorve a
> `ESPEC-DINAMISMO-E-CORRECOES.md` (agora rascunho histórico) e incorpora:
> a perícia print-a-print dos dois vídeos E2E, as regras ditadas pelo operador em 05/08,
> e o cross-check final contra a decupagem medida do vídeo de referência do Piter.
>
> **Fontes de verdade dos vídeos julgados:**
> `_saida/urso_polar_e2e.mp4` (CONTAMINADO — áudio e preset errados, não julga pipeline) ·
> `_saida/predadores_e2e.mp4` (teste limpo) · `remotion/timeline_render.json` (timeline
> exato do render dos predadores, usado em toda a perícia).

---

## 0. O QUE NÃO MUDA (ordens explícitas do operador)

| Item | Estado |
|---|---|
| **Abertura** (nome em Anton, gradiente, take fixo, corte seco) | 🔒 **CONGELADA** — *"NÃO IREMOS ALTERAR NADA NOSSA ABERTURA"* |
| **Trilha sonora** | ✅ **ELOGIADA** — *"está realmente muito boa, foi escolhida cirurgicamente"* — o pass `trilha.py` não é tocado por esta espec |
| Vídeo do Piter | referência de ritmo, **não** modelo fixo e imutável |

---

## 1. 🔬 PERÍCIA DOS PRINTS — cada defeito com causa raiz PROVADA no timeline

### P1 · CHAPTER 05 sozinho, sem nome do animal (0:37)

**Fato no timeline:** `{tipo: capitulo, num: 5, titulo: ''}` — o meu código B5 grava
`titulo: ""` sempre.
**Regra do operador:** *"ele deveria colocar embaixo do capítulo o lugar que tem para
escrever o nome mesmo… o nome seria o nome do animal."*
**Correção:** o detector de roteiro-lista extrai o **nome do item** da própria narração —
a frase imediatamente após o marcador ("Number five. **The tiger shark**.") nomeia o
animal; vira `titulo`. Componente já suporta (o card do Piter tem exatamente essa área).

### P2 (2:04) · P3 (~3:40) · P6 (7:05) + urso 1:23 · ELEMENTOS DO ACERVO VAZIOS — o defeito mais grave

**Prints:** cantoneiras amarelas sem nada (2:04) · caixa amarela de borda vazia (o print de
referência `Captura ... 154137.png` mostra o MESMO card no vídeo do Piter, **cheio**:
kicker "PEOPLE KILLED & INJURED" + título grande) · uma linha amarela sozinha (7:05).

**Fato no timeline — 8 dos 9 elementos do acervo entraram SEM texto:**

```
 33.4s Ovl11_SpecBadge        texto=None kicker=None valores=[230]
108.9s Ovl10_NumberBadge      texto=None kicker=None valores=[4]
122.2s Texto05_BoxedKicker    texto=None kicker=None valores=[40]   <- P2/urso
225.6s Ovl12_GiantStat        texto=None kicker=None valores=[1]
231.5s Ovl06_CenterPunch      texto=None kicker=None valores=None
257.6s Texto07_StampImpact    texto=None kicker=None valores=None
329.4s Texto10_LetterCascade  texto=None kicker=None valores=None
422.6s Texto04_EditorialSerif texto=None kicker=None valores=None   <- P6
```

**Causa raiz, em TRÊS camadas (todas provadas):**
1. **Detector** — para intenções numéricas devolve só `valores`, nunca monta o texto de
   tela. As minhas regras diziam *"all verbatim from the script or absent"* → o LLM
   aprendeu a omitir.
2. **Contrato de props quebrado** — as famílias Texto e Overlay aceitam **só**
   `{text, kicker, accent}`; `values` é ignorado. O `valores=[230]` **nunca tinha como
   chegar à tela** num `SpecBadge`. Verificado no código dos três arquivos de componentes.
3. **Elegibilidade cega a conteúdo** — nem `edicao.py` nem `escolher_variante` exigem os
   campos que a variante precisa para RENDERIZAR. O I13 validava dado semântico, não
   conteúdo de tela.

**Correção (invariante novo, o mais importante desta espec):**

> ### 🚨 I16 — NENHUM ELEMENTO RENDERIZA VAZIO
> Cada variante declara seus **campos obrigatórios de render** (tabela abaixo). Elemento
> sem os campos: (a) o detector é reorientado a SEMPRE montar `texto`/`kicker` a partir
> do trecho falado (verbatim continua lei — montar ≠ inventar: "230" + "newtons" falados
> viram `texto="230 NEWTONS"`); (b) se ainda faltar, o `edicao.py` **descarta ANTES do
> cronograma** — chrome vazio na tela nunca mais. O render (`BrollTest`) ganha a guarda
> final: sem conteúdo → `return null`.

| Família | Campos obrigatórios p/ renderizar |
|---|---|
| Texto01-10 | `texto` (kicker opcional) |
| Ovl (todos) | `texto` (kicker opcional) |
| Graf (todos) | `valores` + `labels`/`sufixo` conforme o manifesto de cada um; **Graf de valor ÚNICO (counter/odometer/stat) exige `sufixo` OU `kicker`** — número gigante sem unidade nem rótulo é ruído, não dado (o "5 months" do Piter tem os dois) |

**+ mapeamento de props (auditoria 7.5-A):** `autor` → `kicker` quando kicker vier vazio
— hoje o autor validado pelo grounding morre no caminho e a citação sai sem atribuição.

### P4 · "1 kilometer" duas vezes seguidas (4:57 e 5:01) — e um terceiro counter logo após

**Fato no timeline:** três cenas CONSECUTIVAS com infográfico `counter`:
`293.0s {1 kilometer · Darkness Depth}` → `300.6s {1 kilometer · Hunting Depth}` →
`307.2s {1 hour · Breath Hold}`. Dois deles com o MESMO valor+sufixo.
**Causa:** o `montar_timeline` marca infográficos por cena sem olhar vizinhança; o Diretor
de Edição **não coordena** esses elementos (ver P5).
**Correção:** infográfico entra no cronograma único com as regras de todos: **nunca a
mesma animação em cenas consecutivas; nunca o mesmo conteúdo 2× no vídeo** (dedup por
valor+sufixo).

### P5 · Texto antigo aparecendo sob a caixa amarela (6:16) — dois donos na tela

**Fato no timeline:** cena `371.2-379.5s` tem `texto_impacto='Hunts Land and Water'`
(até 379.5) e o acervo coloca `Ovl14_PillVerdict` em `375.7s` — **3,8s de sobreposição**
de dois elementos de texto (viola I3).
**Causa raiz estrutural:** o `edicao.py` coordena apresentações + ilustrações + acervo,
mas **não coleta** `texto_impacto`/`infografico`/word-pop como candidatos — metade dos
elementos do vídeo está FORA do cronograma único.
**Correção:** ver §4 (Cronograma Absoluto).

### P-urso · "Solid (October) → Open Water (June)" estourando as caixas

O `BeforeAfterArrow` (ilustração) sem clamp: texto maior que a caixa vaza. **Correção:**
mesmo padrão da abertura — encolher fonte até caber; e o texto de cada lado limitado a
~14 caracteres pelo detector (é rótulo, não frase).

### P-extra · Transição Apagão "fora de corte real"

**Fato:** as marcações caem em fronteiras de cena — mas nem toda fronteira é um **corte
visível**: fronteira entre cena herdada e a dona mostra o MESMO clipe (herança de
hard-miss), e fronteira com crossfade dissolve. Um Apagão ali parece disparado no meio da
cena.
**Correção:** transição só em fronteira de **corte seco com troca de clipe**
(`transicao=none/whip` E `clip_id` diferente dos dois lados). Fronteira herdada ou em
crossfade não recebe transição — recebe nada.

### P-extra · CHAPTER 03 antes da fala

Já registrado como parte do D8: encaixe de 3,0s é frouxo. **Aperta para 1,5s** e, na
dúvida, o capítulo **atrasa** (nunca adianta — anunciar antes do narrador é spoiler de
1 palavra; depois é eco natural).

---

## 2. 🔬 WORD-POP → TRECHO-POP: a praga quantificada

**Fato no timeline dos predadores: 13 janelas de word-pop em 456s** — praticamente todo o
primeiro terço do vídeo é pop:

```
24-31s · 43-56s · 58-65s · 84-92s · 93-102s · 199-209s · 235-243s ·
243-251s · 315-322s · 356-363s · 363-372s · 414-421s · 446-457s (o ENCERRAMENTO!)
```

E os "itens" são orações: `'shark eats fish'`, `'included licence plate'`,
`'reach five metres'`, `'entire ocean basins'`, `'animal ever measured'`,
`'time not improving'`…

**Regra do operador (verbatim):** *"WORD-POP É QUANDO FOR UMA ÚNICA PALAVRA, NUMA
SEQUÊNCIA SEPARADA POR VÍRGULA, TIPO: CARRO, MOTO, ÔNIBUS E AVIÃO."*

**Regras duras do word-pop (novas):**

| # | Regra |
|---|---|
| W1 | Item = **1 palavra** (2 apenas para nome próprio composto). Item maior → a janela inteira é DESCARTADA, não adaptada |
| W2 | Só **lista verdadeira**: 3+ itens separados por vírgula na MESMA frase |
| W3 | **MÁXIMO 2 JANELAS POR VÍDEO** *(operador 05/08)* — e word-pop **NÃO conta como dinamismo**: o Piter não usa nenhuma. É tempero, nunca prato |
| W4 | **Nunca** no encerramento (última cena) nem na abertura |
| W5 | Janelas **nunca adjacentes** (mín. 60s entre duas) |
| W6 | Fundo do pop **obrigatório e validado** — pop sobre fundo preto = janela descartada |
| W7 | Word-pop entra no **cronograma único** e consome orçamento como qualquer elemento |

**Trecho 1:23-1:31 decupado** (pedido do operador): janela `83.58-92.26s` com 4
pseudo-itens sobre fundo único ('reach five metres', 'entire ocean basins'…) — viola W1 e
W2; e a janela seguinte começa 0,7s depois (93.0s) — viola W5. Depois de 1:31, fundo
preto — viola W6.

---

## 3. AS REGRAS DE APRESENTAÇÃO (operador, 05/08 — agora com mecânica exata)

| # | Regra | Mecânica |
|---|---|---|
| R1 | `film` → **sempre** entra com **Brilho 2** + SFX fixo dele | a marcação nasce JUNTO da apresentação, no mesmo passe — nunca por inferência de fronteira |
| R2 | `quadro` → **sempre** entra com **Apagão de Filme** + SFX fixo dele | idem |
| R3 | **Nenhuma apresentação subsequente a outra** — nem igual, nem diferente | guarda no cronograma único, aplicada DEPOIS da herança de hard-miss (a herança foi quem violou no urso) |
| R4 | SFX preso ao **evento**, jamais ao tempo | SFX só nasce como PAR de um elemento visível; `glitch_topico` permanece morto no default novo |
| R5 *(nova, do teste)* | O mesmo elemento/animação **nunca 2× consecutivas** e conteúdo idêntico nunca 2× no vídeo. **Exceção (operador 05/08): infográficos/gráficos consecutivos PODEM quando medem DIMENSÕES DIFERENTES** ("um falando de peso e outro de distância") — dimensão = sufixo (kg ≠ km ≠ h); mesma dimensão colada segue proibida | dedup no cronograma (caso "1 kilometer") |

---

## 3.5 🔒 TETOS DE REPETIÇÃO E RODÍZIO DAS MOLDURAS (operador, 05/08 — LEI)

> *"com exceção da animação de capítulos, que se tiver 7 capítulos usará a animação 7
> vezes, mudando só o nome do capítulo, qualquer outra, deve aparecer no máximo 2 vezes
> por roteiro. […] o quadro, film e polaroid podem aparecer cada um até 4x por roteiro.
> NUNCA CONSECUTIVAMENTE."*

| Elemento | Teto por roteiro |
|---|---|
| **Capítulo** (ChapterTitle) | **ISENTO** — 1 por capítulo; 7 capítulos = 7 usos, só muda o nome |
| **quadro · film · polaroid** (molduras) | **até 4× CADA** |
| **Qualquer outra variante/animação** | **máximo 2×** |

**Escopo do "qualquer outra" (interpretação adotada, explícita para veto):** vale para
TODO formato visual do cronograma — as 40 variantes do acervo, cada tipo de ilustração
(`ilu_*`), cada apresentação não-moldura (lupa, spotlight, reveal, parallax, split, grid),
cada tipo de `infografico` (counter, ring…) **e o `texto_impacto`** (é uma animação como
qualquer outra — nos Predadores apareceu 12×; sob esta lei, 2×). Transições e SFX ficam
FORA (não são elementos; o operador já os excluiu do dinamismo). Word-pop já tem lei
própria mais dura (W3: 2 janelas).

**RODÍZIO ESTRITO DAS MOLDURAS (verbatim):** *"se aparecer um quadro, antes de aparecer
ele outra vez, deve aparecer um film e um polaroid; se o primeiro a aparecer for o film,
antes de ele reaparecer deverá aparecer o polaroid e o quadro; se o polaroid for o
primeiro, deverá aparecer o quadro e o film antes de aparecer outro polaroid."*

Mecânica: um **ledger global de molduras** na ordem de aparição no vídeo (cronograma +
herança de hard-miss, que também usa moldura). Regra: o tipo só pode reaparecer depois
que os OUTROS DOIS tipos apareceram desde o seu último uso (ordem entre os dois é livre).
Nunca duas molduras consecutivas (já era o R3, que proíbe qualquer apresentação
subsequente a outra).

### SUBSTITUIÇÃO, NÃO REMOÇÃO (operador, 05/08)

> *"os outros 10 pontos devem ter elementos dinâmicos do mesmo jeito — não que onde
> apareceu ficaria sem aparecer nada, mas sim deixaria espaço para aparecer o que não
> apareceu ainda."*

**O teto recusa a VARIANTE, nunca o MOMENTO.** Um ponto do roteiro que merecia elemento
continua merecendo — o que muda é a roupa. Mecânica em 3 camadas:

1. **Re-vestir no mesmo ponto:** candidato recusado por teto não é descartado — o
   coordenador tenta OUTRA variante elegível abaixo do teto para o MESMO instante e o
   MESMO conteúdo falado. O 3º `texto_impacto` vira candidato do acervo com o mesmo
   texto (punchline/afins — a tabela já rotaciona variante); o 3º `counter` vira
   `Ovl12_GiantStat`/`Graf15` etc. Só depois de esgotar as variantes compatíveis o
   momento fica sem elemento.
2. **Oferta superdimensionada:** o detector passa a ser chamado com **~1,5× o
   orçamento** (hoje é chamado com o orçamento exato — sem excedente não há substituto
   de outra família por perto). O coordenador continua decidindo; sobra é descartada.
3. **Aritmética que sustenta:** 40 variantes de acervo ×2 + ~8 apresentações ×2/4 +
   ilustrações ×2 ≈ **100+ vagas de formato** para um orçamento típico de 25-35
   elementos — o teto nunca é o gargalo da densidade, só da repetição. F3/F4 (piso por
   minuto) continuam sendo o juiz final: se um minuto ficar abaixo do piso POR CAUSA de
   teto, é bug de substituição, não de oferta.

**⚠️ Caso-limite descoberto no código (precisa de decisão):** `polaroid` hoje só aceita
IMAGEM — [Presentacao.tsx:106-129](remotion/src/compositions/Presentacao.tsx) renderiza
`<Img>` fixo, e [apresentar.py:23](director/apresentar.py) exclui polaroid de cena de
vídeo (`APLICA_EM_VIDEO = {quadro, film}`). Com `video_frac 0.6` no doc_realista, num
roteiro pobre de imagens o rodízio estrito TRAVA: quadro aparece 1×, film 1×, e sem cena
de imagem para o polaroid nenhum dos dois pode reaparecer.
**✅ DECISÃO DO OPERADOR (05/08):** *"pode habilitar, mas sempre preferencialmente usar
imagem no polaroid, o vídeo deve ser fallback, exceção, não regra."*
Mecânica que implementa exatamente isso:
- O caminho NORMAL não muda: o LLM do `apresentar.py` só recebe polaroid como opção
  para cena de IMAGEM (`APLICA_EM_VIDEO` continua `{quadro, film}` para a escolha dele).
- Polaroid com VÍDEO só nasce do **ledger/rodízio** (§3.5) ou da **herança de hard-miss**,
  quando o rodízio exige polaroid e não há cena de imagem elegível — a exceção, com
  origem registrada no timeline (`_polaroid_video: true`) para o report auditar.
- `Presentacao.tsx` ganha o ramo de vídeo no polaroid (mesmo `Midia`/`ehVideo` do
  `quadro`); o visual com imagem não muda um pixel.

## 4. 🏗️ CRONOGRAMA ABSOLUTO — a correção estrutural que amarra tudo

**Diagnóstico-síntese da perícia:** os defeitos P4, P5, o excesso de word-pop e o
"dinamismo pobre" têm UMA causa comum — o Diretor de Edição de hoje coordena só 3 dos 6
sistemas que põem coisas na tela.

| Sistema | Hoje | Espec |
|---|---|---|
| Apresentações | ✅ coordenado | ✅ |
| Ilustrações | ✅ coordenado | ✅ |
| Acervo (40) | ✅ coordenado | ✅ |
| `texto_impacto` (montar_timeline) | ❌ FORA | **entra** |
| `infografico` (montar_timeline) | ❌ FORA | **entra** |
| Word-pop / micro-takes (enumeracoes) | ❌ FORA | **entra** |

O `edicao.py` passa a ser o **único dono da tela**: todo elemento visual é candidato, o
cronograma decide TUDO — orçamento, cotas, respiro, rotação, dedup, janelas exclusivas,
I16. O que ele não aprovar é REMOVIDO do timeline (como já faz com apresentações).

**Prioridade quando disputam a mesma janela:** capítulo > acervo com dado falado >
apresentação > ilustração > texto_impacto > infografico > word-pop.
(Word-pop por último de propósito: é o elemento mais barato e o que mais saturou.)

---

## 5. DEFEITOS CONSOLIDADOS (rascunho anterior + perícia nova)

| # | Defeito | Correção | Origem |
|---|---|---|---|
| D1 | trecho-pop | §2 (W1-W7) | ambos os vídeos |
| D2 | apresentações rarefeitas | §6 fixadores | urso |
| D3 | apresentações adjacentes | R3 no cronograma | ambos |
| D4 | texto estourando caixa | clamp universal (P-urso) | urso |
| D5 | fundo preto (7,6% do vídeo; 13s contínuos no pior caso) | I16 + W6 + conferência: **nenhum frame sem imagem** — cena/janela sem mídia validada não entra | ambos |
| D6 | mapa/ilustração sem conteúdo | elegibilidade I13/I16 para mapas e ilustrações | ambos |
| D7 | acervo entregou 6-9 de ~34 | teto vira meta com 2ª passada; I16 remove a causa de descarte silencioso | ambos |
| D8 | capítulos: 2 de 5 + timing | precedência capítulo>transição; encaixe 1,5s; título do item (P1) | predadores |
| D9 | concorrência do render vs molduras com vídeo | calibração progressiva 6→8→…; já na espec | predadores |
| D10 | **elementos vazios** | **I16** (P2/P3/P6) | ambos |
| D11 | dois donos simultâneos | Cronograma Absoluto (P5) | predadores |
| D12 | conteúdo repetido consecutivo | R5 dedup (P4) | predadores |
| D13 | transição sem corte visível | regra do corte seco real (P-extra) | predadores |

---

## 5.5 📐 DENSIDADE DE ELEMENTOS — decupagem completa do Piter (as 4 folhas)

> **Ordem do operador:** *"só transições NÃO VÃO SER CONSIDERADO DINAMISMO"* — ilustrações,
> acervo TEXTO e acervo OVERLAY entram na conta. **Word-pop NÃO conta** e tem teto de
> **2 janelas por vídeo**.

Decupagem das 595s inteiras (4 folhas de contato, 1 frame a cada 4s), contando **elemento
distinto** (elemento que persiste em 2 quadros conta 1 vez):

| Minuto | Elementos | O que aparece |
|---|---|---|
| 1 | **7** | donut c/ legenda · ticker · punchline tela cheia · CHAPTER 01 · kicker+parágrafo · split · CHAPTER 02 |
| 2 | **6** | legenda de rodapé · ticker · rótulo âmbar · foto emoldurada ×2 · legenda |
| 3 | **5** | foto emoldurada · punchline · serif "The Root Cause" · cantoneiras · ticker |
| 4 | **8** | emoldurado ×3 · gráfico teal · "PREDA" + frase · typewriter · kicker "Brazil records" · frase |
| 5 | **3** | "PREDA" · kicker "income" · frase "wandering spider" |
| 6 | **2** | **card de fundo CLARO** · CHAPTER 05 |
| 7 | **0** | só footage — sustentado pelo corte |
| 8 | **3** | serif antivenom · caixa âmbar "UNDER SIX" · serif malária |
| 9 | **5** | gráfico teal · serif · número "4" tela cheia · cantoneiras · "4 SECONDS — JAGUAR KILL TIME" |
| 10 | **1** | serif "The forest ranks its dangers" |

**Total ≈ 40 elementos distintos em 9:55 → média 4,0/min · hook (min 1-2) 6,5/min.**

> ⚠️ **Este número é PISO, não exato.** A amostragem de 4s perde elemento que dura menos
> que isso. A contagem real deve ser ~1,3× maior. Uso o piso de propósito: meta baseada em
> número subestimado é meta segura.

### 5.5.1 Comparação com o nosso teste limpo (mesmo critério, sem word-pop)

| | Piter | Predadores (nosso) |
|---|---|---|
| Elementos/min (média) | 4,0 | **5,1** |
| Hook (0-120s) | 6,5/min | **7,0/min** |
| Por minuto | 7·6·5·8·3·2·**0**·3·5·1 | 10·4·7·4·6·5·3·**0** |
| **Word-pop** | **zero** (não usa) | **13 janelas** 🔴 |
| **Elementos VAZIOS** | zero | **8 de 9 do acervo** 🔴 |

**A revelação:** em QUANTIDADE bruta nós já superamos o Piter (5,1 vs 4,0/min). O que
falta não é volume — é **qualidade e variedade**:

1. **8 dos nossos elementos eram chrome vazio** (I16). Descontando, caímos para ~4,0/min —
   empate técnico, mas com 8 defeitos visíveis na tela.
2. **Nossa composição é pobre:** 12 `texto_impacto` + 9 `infografico` (os dois mais
   simples) = 54% do total. O Piter distribui entre ~10 formatos distintos por vídeo.
3. **13 janelas de word-pop** que ele simplesmente **não usa** — e que o operador acaba de
   limitar a **2**.

> **Conclusão que muda a implementação:** o problema nunca foi "usar mais elementos". É
> **trocar 13 word-pops e 21 elementos-genéricos por ilustrações + acervo TEXTO + acervo
> OVERLAY variados**, e garantir que nenhum entre vazio.

### 5.5.2 📐 QUADRO + FILM + POLAROID — quantos o Piter usa por minuto

Medição pedida pelo operador, feita por **detecção automática + conferência visual de 100%
dos candidatos** (não por amostragem de folha de contato).

**Método.** 595 frames (1 por segundo). Para cada um, mede-se a *variância local* e extrai-se
a caixa do conteúdo com textura. Moldura = mídia densa num retângulo **recuado**, com placa
lisa em volta. Histogramando a geometria dos 595 frames aparecem **dois clusters com recuo
vertical baixo** (`vert=0.093` e `vert=0.056`) — são as molduras; **todo o resto do vídeo tem
recuo vertical ≥0.26**, que é card de texto (texto fica muito mais centralizado). Essa
separação é limpa e foi o que tornou a contagem confiável.

⚠️ **Duas armadilhas do método, registradas para não repetir:**
- **Falso positivo:** vinheta escura de borda em take de tela cheia imita "placa lisa".
  6 dos 13 candidatos eram tela cheia (jaguar 27s, mão 108s, sucuri 192s, aéreo 292s,
  subaquático 480s, mosquito 487s). **Conferência visual é obrigatória — o detector sozinho
  erra 46%.**
- **Falso negativo:** moldura em *fade* perde densidade e cai no filtro (o peixe em 512s
  quase escapou). Não filtrar por densidade de textura ao contar.

**Resultado — 8 molduras confirmadas em 9:55:**

| # | tempo | dur | conteúdo |
|---|---|---|---|
| 1 | 36s | 1s | formiga (macro) |
| 2 | 93-98s | 6s | onça na vegetação |
| 3 | 120-122s | 3s | aéreo da floresta |
| 4 | 190-191s | 2s | interior da mata |
| 5 | 198-202s | 5s | ferrão da arraia (macro) |
| 6 | 215-216s | 2s | thumbnail "10 ANIMAIS PERIGOSOS" |
| 7 | 232-235s | 4s | anta/mata |
| 8 | 511-514s | 4s | peixe (piranha) |

```
minuto:  1  2  3  4  5  6  7  8  9 10
moldura: 0  2  1  4  0  0  0  0  1  0     = 8 em 9:55
```

- **Média: 0,81 moldura/min** · duração mediana **3,5s** (faixa 1-6s)
- **Distribuição real é em RAJADA, não uniforme:** minutos 2-4 concentram **7 das 8**
  (1,75/min ali); os minutos 5-8 e 10 têm **zero**.
- Conteúdo: 7 de 8 são **foto/vídeo de fauna ou paisagem**; 1 é uma imagem de referência
  (thumbnail). Nenhuma moldura dele enquadra texto — **moldura é o formato de MÍDIA**.

**Cruzando com o nosso:** nos Predadores geramos **15 apresentações em ~8 min = 1,9/min** —
mais que o **dobro** do Piter. Somando isso ao fato de que 2 formatos já ocupavam 54% do
vídeo, confirma-se o diagnóstico do §5.5.1: **o excesso não é de quantidade, é de
repetição do mesmo formato**.

**→ Consequência para a cota de Apresentação (§7):** a cota de 20% que o operador fixou em
D4 está **coerente com o Piter** e não deve subir. O alvo operacional passa a ser
**0,8-1,2 moldura/min**, com teto de **2 por minuto** para não virar tique — e a rajada é
permitida (concentrar num trecho é o que ele faz), desde que o vídeo inteiro respeite a
média. Isso vira o fixador **F11**.

## 6. 🎯 FIXADORES DE DINAMISMO v2 — cruzados com a decupagem do Piter

A decupagem medida (150 cortes = 1/4,0s constante · elementos front-loaded · persistência
4-8s) confrontada com o teste limpo (74 cortes = 1/6,2s decaindo · 26 elementos com 8 dos
9 do acervo vazios):

| Fixador | Alvo | Piter (medido) | Nosso teste limpo | Ação |
|---|---|---|---|---|
| F1 · cena máxima | **6s** | ~4s típico | mediana 6,3s, 59% acima | cortar cena longa em 2 no montar_timeline |
| F2 · cortes/min | **≥12 em TODO minuto** *(operador)* | 12-17 sempre | 17→2 decaindo | F1 aplicado uniformemente; verificação no pre_render_report |
| F3 · elementos no hook (0-120s) | **≥6/min** | 6,5/min | 7,0/min ✅ | manter |
| F4 · elementos no miolo | **≥3/min** + TODOS os capítulos | 3,6/min | 4,2/min mas cai a 0 no fim | piso por minuto, não média |
| F5 · persistência do elemento | **4-6s** | 4-8s | 2,8-3,2s | duração default sobe |
| **F6 · VARIEDADE (novo, o que faltava)** | **≥8 formatos distintos por vídeo**; nenhum formato > **25%** do total | ~10 formatos | **2 formatos = 54%** 🔴 | cotas §7 + rotação por formato, não só por variante |
| F7 · fundo claro | 1 card claro por vídeo | usa como quebra | não existe | liberar 1 variante clara |
| F8 · capítulo completo | número + **nome do item** | sempre | número solto | P1 |
| **F9 · word-pop (operador)** | **máximo 2 janelas por vídeo**, e **não conta como dinamismo** | **zero** | **13** 🔴 | W3 revisado para 2 |
| **F10 · zero vazio** | **0** elementos sem conteúdo | 0 | **8** 🔴 | I16 |
| **F11 · moldura (quadro/film/polaroid)** | **0,8-1,2/min**, teto **2 num mesmo minuto**, dur **3-5s**, sempre com MÍDIA (nunca texto); **teto duro: 4× cada tipo + rodízio estrito (§3.5)** — em vídeo >15 min o teto manda e o piso 0,8 deixa de ser exigível | **0,81/min** (8 em 9:55), mediana 3,5s | **1,9/min** 🔴 (dobro) | cota Apresentação 20% mantida (§7); rajada permitida, média não |
| **F12 · teto por variante (§3.5, operador)** | **≤2× por roteiro** toda variante/animação; molduras ≤4×; capítulo isento | (implícito: ~10 formatos, nenhum dominante) | texto_impacto **12×** · counter **9×** 🔴 | counter no cronograma (D) + ledger (E) |

### Checklist de paridade com a referência (o que o vídeo precisa exibir para "passar")

- [ ] Cadência de corte constante, **sem minuto abaixo de 12**
- [ ] Card de capítulo com número E nome, nas 5 posições, numeração completa
- [ ] Pelo menos 1 uso de: split (comparação real), card claro, moldura (`quadro`/`film`),
      lower-third/kicker, ticker, stat com NÚMERO VISÍVEL
- [ ] Zero chrome vazio · zero sobreposição de textos · zero pop de oração
- [ ] Elementos vivem 4-6s
- [ ] **≥8 formatos distintos**, nenhum passando de 25% do total
- [ ] **≤2 janelas de word-pop** no vídeo inteiro
- [ ] **≥6 elementos/min no hook · ≥3/min em TODO minuto do miolo**
- [ ] **Molduras entre 0,8 e 1,2/min**, nunca mais de 2 no mesmo minuto, todas com mídia
- [ ] **Nenhuma variante >2× no vídeo** (molduras ≤4× cada · capítulo isento) e **rodízio
      estrito das molduras respeitado** (§3.5)
- [ ] SFX apenas pareado a evento visível

---

## 7. COTAS v2 (substituem as do D4 antigo no cronograma absoluto)

Com word-pop e texto_impacto agora DENTRO do orçamento (1 elemento / 35 palavras, D5 do
operador — inalterado):

| Família | Cota |
|---|---|
| Overlay (acervo) | 30% |
| Texto (acervo) | 25% |
| Apresentação (quadro/film/polaroid/split/grid/…) | 20% |
| Ilustração | 10% |
| Gráfico (só dado com eixo; `percentual` mantém) | 10% |
| Word-pop + infografico legado | **5%** (word-pop: máx **2 janelas**, W3) |

Regra de escassez inalterada: vaga vazia vence cota (2º passe), MAS word-pop nunca é
recuperado pelo 2º passe — escasso é escasso.

---

## 7.5 🔍 AUDITORIA PRÉ-GO — GAPS ANCORADOS NO CÓDIGO REAL (05/08)

Revisão da espec inteira contra o código, pedida pelo operador ("procure gap, erros,
inconsistências… veja as variáveis, como mandamos, se é o jeito certo que ele entende").
Cada item cita arquivo:linha. **Nada disto muda a espec — muda o TAMANHO do trabalho do
§8** (e dois itens pedem decisão do operador).

### A · Variável que MORRE no caminho (bug real, achado agora)

**`dados.autor` nunca chega à tela.** O detector declara `autor?` no schema
([acervo_texto.py:377](director/acervo_texto.py)), o grounding VALIDA autor
([acervo_texto.py:249](director/acervo_texto.py)) — mas o render não passa a prop:
[BrollTest.tsx:1181-1182](remotion/src/compositions/BrollTest.tsx) envia só
`text/title/kicker/values/labels/suffix`. E o componente de citação exibe a atribuição
via `kicker` ([AcervoTextoOverlay.tsx:132](remotion/src/compositions/texto/AcervoTextoOverlay.tsx):
`— {kicker}`). Resultado: toda `citacao` com `{texto, autor}` renderiza a frase SEM o
autor — em silêncio. **Fix (entra no passo 1/I16): mapear `autor` → `kicker` quando
kicker vier vazio.** No resto, o contrato confere: `texto→text/title`, `kicker`,
`valores→values`, `labels`, `sufixo→suffix` casam com os 3 arquivos de componentes.

### B · R1/R2 é SUBSTITUIÇÃO, não adição

O código de hoje implementa a regra ANTIGA (04/08): `film` → **Apagão**
([preparar_render.py:868-892](remotion/preparar_render.py), bloco H2) e `quadro` → nada.
A regra nova do operador (§3) é `film` → **Brilho 2** e `quadro` → **Apagão**. Quem
implementar o passo 4 achando que é só "adicionar o par do quadro" deixa o film com a
transição ERRADA. O bloco H2 tem de ser reescrito, não estendido.

### C · Word-pop: o cronograma não tem como matá-lo hoje

[edicao.py:85-87](director/edicao.py) trata TODA janela de enumeração como dona
intocável da tela — word-pop bloqueia o acervo e não gasta orçamento. Para W3/W7 valerem,
o `edicao.py` precisa de um caminho de REMOÇÃO de janelas (hoje não escreve em
`tl["enumeracoes"]`). Critério determinístico de escolha das 2 sobreviventes: mais itens
vence (pop mais rico), empate = a mais cedo; depois aplica W4 (nunca no encerramento) e
W5 (60s de distância).

### D · Teto de repetição: o código de hoje é 1×, a lei nova é 2×/4×

[edicao.py:181,201-202](director/edicao.py) usa um SET (`usadas_var`) — recusa qualquer
2ª aparição. A lei do §3.5 pede um COUNTER com teto por tipo (2 default, 4 molduras,
capítulo fora do cronograma = naturalmente isento). Detalhe já verificado: o fallback
final de `escolher_variante` ([acervo_texto.py:298-300](director/acervo_texto.py)) PODE
devolver variante já usada — o counter no coordenador é quem segura.

### E · Herança de hard-miss cria moldura POR FORA do coordenador

[preparar_render.py:476-479](remotion/preparar_render.py) alterna `film`/`quadro` nas
cenas herdadas, DEPOIS do `edicao.py` rodar. Viola potencialmente: os tetos de 4× (não
conta no ledger), o rodízio estrito (alterna só 2 tipos) e o R3 (foi a herança que criou
apresentações adjacentes no urso). **Fix: o ledger de molduras (§3.5) é escrito pelo
`edicao.py` no timeline e a herança CONSULTA e ATUALIZA o mesmo ledger**, escolhendo o
próximo tipo do rodízio compatível com o media_tipo. Validação final no
`pre_render_report` (que enxerga tudo por último).

### F · Cotas v2: a família Ilustração não existe no código

[edicao.py:68-75](director/edicao.py) dobra `ilu_*` em "Overlay" e
[edicao.py:53](director/edicao.py) tem as cotas antigas (35/30/20/15). As cotas v2 (§7)
têm Ilustração 10% como família própria — `familia()` e `COTAS` mudam juntos. E fica
explícito o que o §7 deixava implícito: **`texto_impacto` conta na cota Texto;
`infografico` conta na família de 5% (com word-pop)**.

### G · F5 (elemento 4-6s) QUEBRA o respiro atual

O respiro é distância entre INÍCIOS ([edicao.py:198](director/edicao.py), `abs(t_a-t_b)
< respiro`) e o default é 4,0s. Com duração default de 2,8-3,2s
([BrollTest.tsx:1176](remotion/src/compositions/BrollTest.tsx)) nunca houve sobreposição.
Subindo a duração para 4-6s (F5), dois elementos aceitos a 4s de distância se SOBREPÕEM
até 2s — dois tela-cheia emendados. **Fix: respiro passa a ser FIM→INÍCIO
(`t_b ≥ t_a + dur_a + respiro`), e o `edicao.py` grava `dur` em cada elemento** (hoje não
grava; o render usa default — decisão de duração é do diretor, não do materializador).

### H · Dois sistemas de capítulo podem duplicar o anúncio

A TABELA tem `("capitulo","overlay") → Ovl01_ChapterBig`
([acervo_texto.py:58](director/acervo_texto.py)) e o B5 insere ChapterTitle via
`marcacoes` ([preparar_render.py:709+](remotion/preparar_render.py)). A janela exclusiva
só bloqueia colisão de ±0,3s — um `capitulo` do detector ancorado 4s depois do marcador
renderiza DOIS anúncios do mesmo capítulo. **Fix: com o B5 ativo, `capitulo` sai do
vocabulário aceito do acervo (o detector pode devolver, o edicao descarta).** A animação
de capítulo é UMA, a isenta do §3.5.

### I · Oferta de apresentações não acompanha a demanda por variedade

O prompt do [apresentar.py:42-56](director/apresentar.py) pede "1 em N cenas" mas não
manda VARIAR template — o LLM pode escolher lupa 5× e spotlight 0×; com teto 2× a cota
de Apresentação morre de fome. Fix barato no passo 7: instrução de rodízio no prompt
(a decisão final continua determinística no coordenador).

### J · Aritmética dos tetos vs F11 (conferida)

4×3 molduras = máx 12/roteiro. Em 8 min → 1,5/min máximo possível; alvo F11 0,8-1,2/min
cabe folgado. Em 15-20 min → 0,8-0,6/min: **o teto do operador passa a mandar e o piso
de F11 deixa de ser exigível em vídeos longos** (anotado no F11; teto é lei, alvo é alvo).

## 8. ORDEM DE IMPLEMENTAÇÃO (pós-GO)

```
0. default do sistema = valores do doc_realista            (decisão já dada; mata a classe de erro do preset)
1. I16 + contrato de conteúdo por variante                 (detector monta texto; edicao valida; render guarda;
                                                            + autor->kicker [7.5-A] + Graf único exige contexto)
2. Cronograma Absoluto                                     (texto_impacto/infografico/word-pop entram;
                                                            + família Ilustração + cotas no código [7.5-F]
                                                            + teto por variante 2×/4× [7.5-D] + capitulo fora
                                                            do vocabulário do acervo [7.5-H]
                                                            + SUBSTITUIÇÃO no mesmo ponto + oferta 1,5× [§3.5])
3. W1-W7 (word-pop)                                        (mata o trecho-pop na fonte; caminho de REMOÇÃO de
                                                            janelas no edicao [7.5-C])
4. R1-R5 + D13 (transições/apresentações/SFX pareado)      (⚠️ SUBSTITUI o bloco H2: film Apagão->Brilho 2,
                                                            quadro ganha o Apagão [7.5-B])
5. D8 + P1 (capítulos: precedência, 1,5s, título do item)
6. F1/F5 (cena 6s · elemento 4-6s)  + D4 (clamp universal) (respiro vira FIM->INÍCIO e edicao grava dur [7.5-G])
7. F7 (card claro) + cotas v2                              (+ rodízio no prompt do apresentar [7.5-I]
                                                            + ledger de molduras compartilhado com a herança
                                                            de hard-miss [7.5-E] + polaroid em vídeo SE o
                                                            operador aprovar [§3.5])
8. RE-RENDER dos 2 roteiros (áudio certo, preset certo, concorrência 8)
   -> decupagem automática (cortes/min, % preto, elementos/min, tetos por variante,
      rodízio de molduras) ANTES de entregar
9. Avaliação do operador -> ajuste fino -> aí sim o vídeo de 15-20 min
```

O passo 8 inclui um **portão de autoavaliação**: o vídeo só é entregue ao operador se
passar nos fixadores mensuráveis (F1, F2, D5=0% preto, I16=0 vazio) — medidos por script,
não por confiança.

---

*ESPEC FINAL de 05/08/2026, consolidada após: 2 renders E2E, perícia print-a-print com
causa raiz provada no timeline, regras verbatim do operador e cross-check com a decupagem
medida do vídeo de referência. Substitui ESPEC-DINAMISMO-E-CORRECOES.md.*
