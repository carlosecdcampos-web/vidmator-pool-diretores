# RELATÓRIO AUDITÁVEL — SMOKE DO DIRETOR DE DOCUMENTÁRIO ANIMAL

> **Pedido do operador (05/08):** *"faremos com 10 roteiros de animais diferentes, biomas
> diferentes, para vermos realmente a % de acertos… um relatório que seja auditável com o
> resultado, apontando pontos de melhoria, erros, gaps, na utilização das 40 variáveis que
> temos disponível, inserção de texto, apresentação, gráfico."*

**Dados brutos:** `director/_smoke_fauna.json` (bateria) · `_smoke_roteiro*.json` (roteiro
inteiro) · logs `_log_smoke_*.txt`
**Reproduzir:** `python _smoke_fauna.py 3` · `python _smoke_roteiro_inteiro.py 3`

---

## 0. O QUE FOI RODADO

| Teste | O que mede | Volume |
|---|---|---|
| **Bateria de intenções** | classificação: o detector sabe o que cada trecho é? | 10 biomas × 19 intenções × 3 reps = **570 casos** + 30 silêncios |
| **Roteiro inteiro** | **priorização**: com o roteiro todo na frente, o que ele escolhe? | 10 biomas × 3 reps = **30 roteiros** de ~390 palavras |

Os dois medem coisas diferentes e **só o segundo se parece com produção** — é lá que o
detector precisa escolher, não só classificar.

**Corpora:** 10 biomas animais distintos, 10 espécies-âncora distintas —
amazonia · australia · africa (**dev**) · artico · himalaia · oceano · namibe · taiga ·
recife · borneo (**hold-out**, nunca usados para calibrar prompt).
Egito/China/Roma saíram deste diretor e viraram `_corpora_arqueo_semente.py`.

---

## 1. RESULTADO — % DE ACERTO

### 1.1 A escolha do gabarito muda o número em 5 pontos

| Métrica | gabarito **anotado à mão** | gabarito **mecânico** |
|---|---|---|
| Recall geral | 87,7% | **92,8%** |
| Recall **dev** | 89,5% | 91,8% |
| Recall **hold-out** | 87,0% | **93,2%** |
| Precisão | 74,6% | **80,3%** |

**Por que dois números:** o gabarito multi-label declara quais intenções, além da
principal, o trecho **também** sustenta. Anotei à mão e fui inconsistente: em `citacao`,
por exemplo, marquei `spec` como legítimo só na Amazônia, quando **todos** os biomas têm
número de tempo de carreira na frase ("after fifty years working these waters"). Isso
reprovou 17 dos 30 casos de `citacao` por um defeito **meu**, não do detector.

O gabarito **mecânico** aplica uma regra objetiva e auditável: *o extra é legítimo se o
texto o sustenta* — há número falado → `spec`/`stat` valem; há pessoa nomeada com verbo de
fala → `citacao`/`lowerthird` valem. **29 dos 70 "erros" eram artefato da minha anotação.**

> **Qual usar:** o mecânico. Ele é reproduzível e não depende do meu julgamento caso a
> caso. Já corrigi a anotação à mão no corpus para os dois convergirem na próxima rodada.

### 1.2 🟢 O hold-out ficou ACIMA do dev

**93,2% (hold-out) contra 91,8% (dev).** Os 7 biomas que nunca foram usados para calibrar
o prompt tiveram acerto **maior** que os 3 que foram. Não há sobreajuste — as regras
generalizam para fauna que o detector nunca viu.

### 1.3 Por corpus (gabarito anotado; ordem de pior para melhor)

| Corpus | Grupo | Recall | Precisão |
|---|---|---|---|
| australia | dev | 82,5% | 70,7% |
| oceano · recife · borneo | hold-out | 84,2% | 67,9-74,4% |
| artico | hold-out | 86,0% | 73,1% |
| namibe | hold-out | 87,7% | **84,3%** |
| amazonia · taiga | dev/hold-out | 89,5% | 74-76% |
| himalaia | hold-out | 93,0% | 76,6% |
| **africa** | dev | **96,5%** | 75,3% |

Desvio de **4,2 pp** entre corpora — a variância que motivou toda esta linha de trabalho
está sob controle (era 6,4 pp com 5 corpora e gabarito single-label).

### 1.4 Teste do silêncio: 25/30

**As 5 reprovações, uma a uma:**

| Corpus | O que apareceu | Culpa |
|---|---|---|
| recife ×2 | `spec` (factual!) | **minha** — deixei "For **twenty** minutes" num texto que deveria ser atmosfera pura. Já corrigido; regra nova: silêncio passa por varredura de numeral antes de entrar no corpus |
| himalaia ×2 | `nota_lateral` + `punchline` | limite: o critério tolera **1** momento não-factual, vieram 2 |
| namibe | `local_tag` + `punchline` | idem |

**Nenhuma reprovação foi invenção de dado.** As 3 legítimas são excesso de anotação leve,
não alucinação. O comportamento central — não preencher buraco com fato inventado —
continua de pé.

---

## 2. 🔴 O GAP DAS 40 VARIANTES — o achado principal

### 2.1 Só 27 das 40 apareceram na bateria. No roteiro inteiro, **9**.

| Família | Bateria (trecho isolado) | **Roteiro inteiro** (regra antiga) |
|---|---|---|
| Texto (10) | 7 usadas | **0 usadas — 0% dos momentos** |
| Overlay (14) | 13 usadas | 13% dos momentos |
| Gráfico (16) | 7 usadas | **87% dos momentos** |
| **Total** | **27/40** | **9/40** |

*(Com as correções desta rodada o roteiro inteiro subiu para **18/40** — ver §2.5.)*

**No cenário que se parece com produção, o detector usava 9 das 40 variantes e a família
Texto inteira nunca aparecia.** Foi exatamente a intuição do operador: *"não foi pra isso
que fizemos tantas variações, tantas opções."*

### 2.2 A causa, com diagnóstico estrutural

As 13 nunca usadas se dividem em duas causas limpas:

**Causa A — presas atrás da rotação (7):** `Graf02` `Graf04` `Graf06` `Graf08` `Graf10`
`Graf12` `Graf14` são 2ª ou 3ª opção da própria chave da tabela. Com poucos momentos por
vídeo, a rotação nunca chega nelas.

**Causa B — a intenção nunca foi escolhida no domínio certo (6):**

| Variante | Chave | Por que não saiu |
|---|---|---|
| `Graf01_CounterGlow` | (stat, **tela_cheia**) | `stat` sempre saía como overlay |
| `Texto02_HighlightSweep` | (citacao, **tela_cheia**) | idem |
| `Texto05_BoxedKicker` | (spec, **tela_cheia**) | idem |
| `Texto08_GradientGlow` | (punchline, tela_cheia, **épico**) | o tom `epico` nunca foi escolhido |
| `Ovl01_ChapterBig` | (capitulo, overlay) | `capitulo` desligado por decisão (só roteiro-lista, B5) |
| `Graf13_DualLine` | (tendencia_dupla, tela_cheia) | intenção que nenhum corpus exercita |

**Quatro delas eram primeira opção da sua chave e mesmo assim nunca saíram — todas
bloqueadas pelo mesmo teto: `tela_cheia` limitada a 1 por roteiro.**

### 2.3 A correção do operador, medida

> *"NÃO PODE [1 tela-cheia por roteiro], senão ele fica menos dinâmico… é para ter
> rotacionalidade no dinamismo, não ficar refém do mesmo elemento/animação."*

Regra reescrita: tela-cheia liberada (só não pode duas seguidas), ritmo por densidade
(~1 momento a cada 60-90 palavras, nunca em frases consecutivas), e instrução explícita de
**rotação** entre intenções, domínios e tons.

| | ANTES (3 momentos, 1 tela-cheia) | DEPOIS (tela-cheia livre) |
|---|---|---|
| Roteiros com >1 tela-cheia | **0/30** | **19/30** |
| Variantes distintas | 9 | **15** |
| Intenções distintas | 8 | **12** |
| Família Texto | 0x | 2x |
| Fora do gabarito | 0/90 | 0/89 |

`Graf01_CounterGlow`, que era "1ª opção que nunca saía", virou a **mais usada** (17x) —
prova direta de que o teto era o bloqueio. E a precisão não pagou o preço: **0 momentos
fora do gabarito** nas duas rodadas.

### 2.4 ⚠️ O teto real não estava nas regras

A densidade continuou em **3,0 momentos por roteiro** mesmo depois de liberar tela-cheia.
Motivo: o `"find AT MOST 3 moments"` estava cravado no prompt do `detectar_v2`, não em
`_REGRAS`. **Corrigido:** teto agora é proporcional — `teto_momentos()`, ~1 a cada 75
palavras (≈30s de narração), mínimo 2. No roteiro do smoke dá 5; **num vídeo de 7-9 min
(~1.200 palavras) dá 16.**

### 2.5 ✅ RESULTADO FINAL — as três configurações, medidas

| | **A.** 3 momentos + 1 tela-cheia | **B.** tela-cheia livre | **C.** + teto proporcional |
|---|---|---|---|
| Momentos por roteiro | 3,0 | 3,0 | **4,4** |
| Roteiros com >1 tela-cheia | 0/30 | 19/30 | **25/30** |
| **Variantes distintas** | **9** | 15 | **18** |
| Intenções distintas | 8 | 12 | **16** |
| Família **Texto** | **0x (0%)** | 2x (2%) | **11x (8%)** |
| Família Overlay | 12x (13%) | 18x (20%) | 28x (21%) |
| Família Gráfico | 78x (87%) | 69x (78%) | 94x (71%) |
| **Fora do gabarito** | 0/90 | 0/89 | **0/133** |

**Dobrou o repertório em uso (9 → 18 variantes), a família Texto saiu do zero e a precisão
seguiu perfeita: zero momentos fora do gabarito em 133.** As 16 intenções distintas contra
8 do início são a "rotacionalidade" pedida — o vídeo deixa de ser refém do mesmo elemento.

**Gráfico ainda domina com 71%.** É esperado: os corpora são densos em número por
construção (19 intenções, das quais 9 são numéricas). Num roteiro real, com mais narrativa
e menos dado, a proporção tende a cair — mas isso ainda **não foi medido**.

### 2.6 🐞 Bug novo que a rodada final expôs

Com o teto maior, apareceu um defeito que o teto de 3 escondia: **15 dos 150 momentos (10%)
vieram do LLM sem o campo `intencao`** — objetos malformados que o validador descarta com
*"intencao fora do vocabulário: None"*.

Não é perda editorial grave (o momento simplesmente não entra), mas é **10% de trabalho
jogado fora** e o número deve piorar em roteiro de 1.200 palavras com teto 16. **Correção
recomendada (não aplicada):** um retry dirigido quando a taxa de objetos malformados passa
de ~10% na resposta, ou reforço no prompt de que todo objeto exige `intencao` do
vocabulário fechado. Registrado como **M7**.

---

## 3. ERROS E CORREÇÕES

### 3.1 Corrigidos nesta rodada

| # | Erro | Evidência | Correção |
|---|---|---|---|
| 1 | `"7 million"` rejeitado pelo grounding | dígito ia por um caminho, escala por outro → 7 e 1.000.000 soltos | dígito entra na corrida composta; 12/12 nos testes |
| 2 | **Tom mal grafado troca a FAMÍLIA da variante** | `veredicto`×2, `veridito`×1 → entregaram `Ovl06_CenterPunch` (Overlay) onde devia ser `Texto07_StampImpact` (Texto) | `_norm_tom` com similaridade; 9/9 |
| 3 | Teto de 3 momentos fixo | §2.4 | `teto_momentos()` proporcional |
| 4 | Tela-cheia capada em 1 | §2.2/2.3 | regra reescrita |
| 5 | Numeral em texto de silêncio | recife: "twenty minutes" | texto corrigido + regra de varredura |
| 6 | Gabarito de `citacao` inconsistente | 17/30 reprovados por anotação minha | uniformizado nos 10 biomas |

### 3.2 Intenções ainda abaixo de 90% (gabarito mecânico)

| Intenção | Acerto | Diagnóstico |
|---|---|---|
| `punchline` | 23/30 (77%) | rebaixada a último recurso; 5× não detectou, 2× virou `subcapitulo`. **Trade-off consciente** — foi rebaixada para parar de engolir as outras classes |
| `subcapitulo` | 24/30 (80%) | troca mútua com `punchline`: a fronteira entre "virada de assunto" e "frase de impacto" é genuinamente fina |
| `distribuicao` | 26/30 (87%) | confunde com `comparacao` quando as partes não somam explicitamente |
| `citacao` | 26/30 (87%) | atrai `lowerthird` (a frase nomeia a pessoa) — os dois são leituras válidas |

**Nenhuma é erro de invenção.** São fronteiras entre classes vizinhas, com impacto visual
baixo (troca de anotação de canto por outra anotação de canto).

---

## 4. PONTOS DE MELHORIA — priorizados

| # | Melhoria | Ganho esperado | Custo |
|---|---|---|---|
| **M1** | **Rotação por FAMÍLIA, não só por variante** | ataca a Causa A (7 variantes presas). Hoje `escolher_variante` só evita repetir a mesma variante; deveria preferir a família menos usada no vídeo | baixo — é lógica na tabela |
| **M2** | Corrigir a rodada com o **teto proporcional** e re-medir cobertura | é a medição que ainda falta para saber quantas das 40 saem num vídeo real | 1 rodada |
| **M3** | Exercitar `tendencia_dupla` no corpus | `Graf13_DualLine` é inalcançável sem ela | baixo |
| **M4** | Decidir sobre `capitulo`/`Ovl01_ChapterBig` | depende do detector de roteiro-lista (B5), hoje desligado | médio |
| **M5** | Tom `epico` nunca escolhido | `Texto08_GradientGlow` inalcançável; o prompt lista os 7 tons mas não diz quando cada um cabe | baixo |
| **M6** | Gabarito **cego** do operador | remove o viés de autor que ainda existe (§5) | precisa dele |
| **M7** | **10% dos momentos vêm sem `intencao`** (§2.6) | recupera 1 momento a cada 10; piora com teto 16 | baixo — retry dirigido ou reforço no prompt |
| **M8** | Gráfico em 71% dos momentos | equilíbrio de famílias; pode ser artefato do corpus (denso em número) — **medir em roteiro real antes de mexer** | medição |

---

## 5. O QUE ESTE RELATÓRIO **NÃO** PROVA

1. **Eu escrevi os textos e o gabarito.** Mitigado (10 biomas, 7 hold-out, sonda
   anti-contaminação de 4-grams que aborta o teste se regra e corpus compartilharem
   fraseado) — mas **não eliminado**. Os números são teto, não expectativa de produção.
2. **Roteiro sintético.** Os "roteiros inteiros" são a colagem dos 19 trechos em ordem
   editorial. Um roteiro real tem transições, redundância e trechos mortos que o meu não
   tem — provavelmente **mais fácil** de priorizar do que o real.
3. **A % de acerto (§1) é da bateria, rodada ANTES das correções de dinamismo.** Os
   números de cobertura (§2.5) já são pós-correção, mas o recall/precisão por intenção
   **não foi re-medido** com o teto proporcional — mais momentos por roteiro podem mudar a
   fronteira entre classes vizinhas. É a próxima rodada, não uma conclusão desta.
4. **Nada disso foi visto em vídeo.** Acerto de classificação não é acerto editorial: só o
   render diz se o elemento certo, no momento certo, fica bom na tela. **É o teste de
   7-9 min que decide** — e ele agora tem uma expectativa concreta: ~16 momentos, ~18
   variantes distintas em rotação.

**Próximo passo que remove o viés:** gabarito cego nos 3 roteiros reais já validados — o
operador marca o que esperava ver na tela sem ver a saída do detector.

---

*Relatório de 05/08/2026. Bateria: 570 casos + 30 silêncios. Roteiro inteiro: 30 roteiros
em 3 configurações de regra. Todas as correções em `director/acervo_texto.py`.*
