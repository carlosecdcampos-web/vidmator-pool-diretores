# ESTUDO — Por que os takes erram (análise pós-rodada 3, PoC Amazônia)

**Criado:** 03/08/2026 · **Status:** estudo aprovado → vira ESPEC → OK explícito → implementar
**Método:** cada apontamento do operador rastreado até a LINHA de código/prompt que o causou.
Nada foi alterado. Regra do operador: a solução deve valer para QUALQUER roteiro do nicho
documentário realista (animais E lugares), nunca otimização para este trecho.

---

## Os 3 casos apontados, com o culpado real de cada um

### CASO 1 — A capa de livro sobre "It is a refuge for jaguars" (55–61s)

**Culpado: o pass `imagens.py` — NÃO o resolver de takes.**

Evidência dura: a query do resolver para a cena 16 era **BOA** (`"jaguar walking jungle
close up"`, vídeo L3v) — o jaguar estava lá embaixo. O que apareceu por cima foi um
**overlay do pass `imagens`**: ele inseriu aos 57,68s uma "imagem PD do Commons" com
legenda *"Scientific classification provides order to countless species"* — uma capa de
livro de taxonomia.

Por quê: o pass foi desenhado para OUTRO nicho — o prompt dele pede *"a newspaper
clipping, a press photo, a historical record, an object"* (história/true crime). Num
documentário de natureza ele procurou "classificação científica" no Commons e achou um
livro. **E ele roda sem nenhum gate de preset** (verificado: zero referência a preset no
arquivo).

### CASO 2 — 1 take genérico onde a fala nomeia 3 animais (61–67s)

**Culpado: o prompt do `montar_timeline` INSTRUI o Gemini a genericizar.**

Linha do prompt real: *"**Avoid named people/places** (those are handled separately) —
describe a filmable **GENERIC** scene"*. Resultado medido: a fala "harpy eagles, and pink
river dolphins, for birds and butterflies" virou a query `"diverse amazon wildlife montage
vibrant"` — as entidades nomeadas são **descartadas por design**.

Faz sentido no nicho do Piter (true crime: nomes viram cards de pessoa/mapa). Em
documentário realista é o OPOSTO do certo: **as entidades faladas SÃO o b-roll ideal**
(jaguar → take de jaguar; Saara → take do Saara; Atlântico → take do oceano).

### CASO 3 — Pessoa nadando no pântano em "beings watching back" (85–93s)

**Culpado: vet de vídeo leniente por design + nenhuma regra de conteúdo por nicho.**

Query da cena: `"dark swamp mysterious eyes watching"` (razoável). O Pexels devolveu uma
pessoa nadando entre vitórias-régias. O vet existente (`#5b`) só rejeita nota **"none"**
(comentário no código: *"Lenient de propósito"*) — pessoa em água escura tem fit parcial
("weak") com a query → **passa**. Não existe em lugar nenhum a regra "documentário de
natureza não põe humanos em cena, salvo se a fala pedir gente/tribo".

---

## Direcionamentos (o que a ESPEC vai conter)

| # | Direcionamento | Ataca | Generaliza porque |
|---|---|---|---|
| **D1** | **Perfil de b-roll por NICHO no prompt de queries**: o preset ganha um bloco `broll_diretrizes` injetado no prompt do montar_timeline. Para `natureza`: entidades faladas (espécies/lugares/fenômenos) SÃO o alvo da query; vida selvagem/paisagem REAL; SEM humanos salvo a fala mencionar pessoas/tribos; NUNCA objetos, documentos, livros, estúdio | Caso 2 | é dado do preset, não código — cada nicho escreve o seu (crime, história, ciência...) |
| **D2** | **Micro-takes por entidade nomeada**: quando a fala enumera entidades FILMÁVEIS ("jaguars, harpy eagles, and pink river dolphins" / "do Saara à Amazônia"), cortar a cena nos timestamps das palavras (words.json — a MESMA técnica do word-pop) e resolver 1 take POR entidade | Caso 2 (o pedido central) | detector de enumeração + casamento por timestamp JÁ existem (enumeracoes.py); a régua é linguística (substantivo concreto), não temática |
| **D3** | **Vet content-aware por nicho**: o gate de Vision (vídeo E foto, unificando com o vet_imagem da rodada 3) recebe REGRAS do preset — natureza: humano visível sem a fala pedir = reprova; objeto/livro/interior = reprova; espécie citada = tem que ser A espécie | Casos 2 e 3 | regras vêm do preset; a régua "weak passa" morre onde o nicho exigir |
| **D4** | **Pass `imagens` gated por preset**: `natureza` desliga (`usar_imagens_pd: false`). O pass é legítimo — no nicho certo (história/crime) | Caso 1 | knob por nicho; nichos do Piter intocados |
| **D5** | **Rodada de reconstrução TOTAL**: zerar `index_cascata` + cache de mídia da PoC → o Director re-decide TUDO (queries novas, downloads novos). Regra do operador: ele não deve "consertar só o apontado" — queremos ver se acerta do zero | teste honesto | procedimento de rodada, documentado no runbook |

### Decisão em aberto para a ESPEC (sua)

Quando uma enumeração é de entidades filmáveis (D2), o que fazer com o word-pop?
- **(a)** micro-takes de footage SUBSTITUEM o word-pop (a tela mostra o animal, não a palavra)
- **(b)** os dois juntos: footage da entidade ATRÁS + palavra em caixa alta na frente
- **(c)** régua automática: itens abstratos/sensoriais (water, shadow, silence) = word-pop;
  entidades concretas (jaguar, harpy eagle) = micro-take de footage

A PoC sugere (c): o word-pop em "Water, canopy, shadow..." você aprovou como perfeito; o
que faltou foi footage nos ANIMAIS. Mas a escolha é sua.

## O que NÃO muda (aprovado e protegido)

Word-pop (caixa alta, SFX-0127 a 0.15) · legibilidade 2s/palavra · abertura sempre vídeo ·
cor do preset natureza · trilha mood-matched · tudo gated por preset, nichos do Piter imunes.
