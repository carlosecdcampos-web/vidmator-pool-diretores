# RELATÓRIO DE AUDITORIA — detector do acervo de texto (B1)

> **Pedido do operador (05/08):** *"rode 5 testes paralelos com roteiros do tamanho que
> rodou esses testes, em biomas diferentes, 3 com animais 2 com documentário de um local
> específico… analise o resultado dos 5 testes, documente tudo, deixe o resultado pronto
> para auditoria, quero um relatório especificado dos resultados, citando os erros e
> apontando métodos de corrigir cada um, para que não aconteça mais."*
>
> **Motivo:** a medição anterior (94-97%) veio de **2 rodadas do MESMO corpus** — mede
> instabilidade do LLM, não generalização.

---

## 1. MÉTODO

| | |
|---|---|
| Corpora | **5** temáticos: `amazonia` · `australia` · `africa` (animais) · `egito` · `china` (lugar histórico) |
| Casos por corpus | 19 intenções + 1 teste do silêncio |
| Repetições | 2 |
| **Total** | **190 casos + 10 silêncios**, 6 em paralelo, ~6 min por rodada |
| Código | `director/_corpora_acervo.py` (textos+gabarito) · `director/_smoke_variancia.py` (runner) |
| Bruto | `director/_auditoria_acervo.json` (depois) · `_auditoria_acervo_ANTES.json` (antes) |
| Reproduzir | `cd director && python _smoke_variancia.py 2` |

### 1.1 Métricas — o que cada uma quer dizer

- **acerto** — a intenção esperada apareceu entre os momentos que **sobreviveram ao
  grounding**. É o número comparável com os 94-97% anteriores.
- **classe_errada** — sobreviveu algo, com outro rótulo. Erro de **classificação**.
- **morto_grounding** — o detector ACERTOU, mas o **validador matou**. Instrumentado nesta
  auditoria: antes isso se confundia com "não achou", e são erros de natureza oposta —
  um se corrige no prompt, o outro no validador.
- **vazio** — nada sobreviveu, nem certo nem errado.
- **silêncio** — atmosfera pura; qualquer intenção **factual** ali = "preencheu buraco" = reprovado.

### 1.2 ⚠️ Viés declarado (leia antes dos números)

**Eu escrevi os textos e o gabarito.** Um trecho pode ter saído mais didático para a classe
que eu tinha em mente do que um roteiro real seria — isso **infla o acerto**. Estes números
são um **teto**, não a expectativa de produção.

Mitigações aplicadas: os 5 corpora **não** são o mesmo texto com substantivo trocado
(estrutura, ordem e registro variam); números compostos entraram de propósito como
regressão viva. Mitigação **não** é eliminação — ver §5.

---

## 2. RESULTADO — ANTES × DEPOIS DAS CORREÇÕES

| Corpus | Tipo | Antes | Depois | Δ |
|---|---|---|---|---|
| amazonia | animais | 97,4% | 97,4% | = |
| australia | animais | 89,5% | 92,1% | +2,6 |
| africa | animais | 94,7% | 94,7% | = |
| egito | lugar | 86,8% | **97,4%** | **+10,5** |
| china | lugar | 78,9% | 86,8% | +7,9 |
| **MÉDIA** | | **89,5%** | **93,7%** | **+4,2** |
| **DESVIO-PADRÃO** | | **6,4 pp** | **3,9 pp** | **−2,5** |

**Teste do silêncio: 10/10 nas duas rodadas.** Zero momentos factuais em atmosfera pura,
nos cinco temas. A propriedade que mais importava (não inventar para preencher) se sustenta.

### 2.1 Os dois achados que só 5 corpora revelariam

**(a) Os 94-97% eram otimistas.** Com 5 temas em vez de 1, a média real ficou em **89,5%**
antes das correções. Um corpus só media o corpus.

**(b) Havia um buraco de DOMÍNIO, não de intenção:**

| Grupo | Antes | Depois |
|---|---|---|
| animais | 93,9% | 94,7% |
| lugares históricos | **82,9%** | **92,1%** |

Onze pontos de diferença. O detector estava afinado em documentário de animais — as regras
de desempate assumiam sujeito vivo. **As correções fecharam o buraco para 2,6 pp.**

---

## 3. ERROS ENCONTRADOS — causa e correção, um a um

### 🔴 E1 · BUG NO PRÓPRIO C1: a pontuação não fechava o número composto

**Evidência:** `china/comparacao` morreu com *"número NÃO falado: 800"* — mas 800 **estava**
falado (*"The road beside it covered eight hundred."*).

**Causa raiz:** o tokenizador era `re.findall(r"[a-z]+", …)`, que **apaga o ponto final**:

```
"covered eight hundred. One was built"  →  corrida contígua "eight hundred one" = 801
```

O valor 800 nunca entrava no conjunto de falados, e o validador matava um momento legítimo.
**O fix do C1 entregue em 04/08 criou uma variante nova do problema que ele resolvia.**

**Correção aplicada** (`acervo_texto._compostos`): a pontuação vira token e **fecha a
corrida**. Protege também `"ninety, seventy, twenty"` de virar um número só.
**Verificação:** 9/9 casos, incluindo as regressões de `six`/`ninety`/`fifteen`.
**Status: ✅ corrigido e testado.**

### 🔴 E2 · `comparacao` → `spec`/`stat` (6 erros em 10)

**Evidência:** *"A green anaconda can reach nine meters. A black caiman rarely passes five."*
→ classificado como dois `spec`.

**Causa raiz:** a regra definia comparação como "A vs B", mas **não dizia o que fazer quando
as duas medidas vêm em frases separadas**. Cada frase, isolada, É uma medida — então `spec`
era uma leitura defensável.

**Correção aplicada** (regra no prompt): *duas medidas da MESMA dimensão postas lado a lado,
mesmo em frases separadas, são `comparacao` com os DOIS valores — nunca dois `spec`.*
`spec` só quando a medida está sozinha, sem nada contrastado.
**Status: ✅ 4/10 → 9/10.**

### 🔴 E3 · `local_tag` → `legenda_doc` (4 erros em 10)

**Evidência:** *"This was filmed in Kakadu, in the far tropical north…"* → `legenda_doc`.

**Causa raiz:** as duas frases **começam igual** (*"this footage…"*), e a abertura da frase
virou o critério de fato. A regra antiga dizia "a line that merely names where/when" — mas
"merely" não é operacional para o modelo.

**Correção aplicada:** decidir por **conteúdo, não por abertura** — se a única informação é
ONDE e/ou QUANDO foi filmado → `local_tag`; só se a frase faz uma **afirmação sobre a
imagem** (rara, nunca vista, onde câmeras não sobrevivem) → `legenda_doc`.
**Status: ✅ 6/10 → 10/10.**

### 🟡 E4 · `ranking` → `distribuicao` (2 erros em 10)

**Causa raiz:** valores absolutos por item (*"ninety carts, thirty, four"*) lidos como
partes de um todo.
**Correção aplicada:** `distribuicao` **só** quando as partes somam UM todo ("of every
ten…", percentuais de um total); valor absoluto por item **nunca** é distribuição.
**Status: ✅ ranking 7/10 → 10/10.** *(o erro migrou para `distribuicao` — ver R2.)*

### 🟡 E5 · `nota_fonte` morto por *"kicker inventado: 'Source:'"* (1 em 10)

**Causa raiz:** o validador tratava o rótulo de tela `"Source:"` como afirmação inventada.
Mas `"Source:"` **não afirma nada sobre o mundo** — é cromo, o mesmo papel de "FONTE:".
I14 protege **dado** inventado, não etiqueta.

**Correção aplicada:** whitelist `_ROTULOS_NEUTROS` (source, fonte, nota, note, dado, data,
ref, via) em `validar_grounding`. **Status: ✅ corrigido.**

### 🟡 E6 · `spec` morto por *"número NÃO falado: 1.8"* (2 em 10)

**Causa raiz:** *"one metre eighty"* foi convertido pelo LLM em `1.8`. O validador estava
**certo** em rejeitar: 1.8 não foi falado.

**Correção aplicada:** regra explícita — *nunca converter fala em decimal; se o número não
puder ser escrito exatamente como falado, pule o momento.*
**Status: ⚠️ parcialmente resolvido.** O momento deixou de ser **morto** e passou a ser
**não detectado** (vazio). Trocamos "dado inventado" por "momento perdido" — o lado certo
do trade-off, mas o dado continua fora da tela. **Ver R1.**

---

## 4. FALHAS REMANESCENTES (12 de 190 = 6,3%)

| Corpus | Esperado | Virou | Leitura |
|---|---|---|---|
| china ×2 | `spec` | nada | E6 — "one metre eighty" agora é pulado |
| africa, egito | `punchline` | nada | punchline virou último recurso; o pêndulo passou do ponto |
| australia | `punchline` | subcapitulo | idem |
| australia ×2 | `distribuicao` | stat / comparacao | R2 |
| china | `nota_fonte` | citacao | "According to X…" lido como citação |
| china | `comparacao` | morto: *texto inventado 'Length'* | R3 — mesmo caso do E5 |
| china | `tendencia` | nada | isolado |
| amazonia | `lowerthird` | punchline | isolado |
| africa | `item_lista` | subcapitulo | isolado |

### Recomendações abertas (não aplicadas — precisam de decisão ou de mais medição)

- **R1 · decimal falado** (`spec`, 2 casos). Ensinar o grounding a aceitar o padrão
  `número + unidade + dezena` ("one metre eighty" = 1,80) OU manter o descarte. Aceitar
  recupera o dado; ampliar o validador é o caminho mais arriscado da lista, porque cada
  padrão novo abre uma fresta para número não falado. **Recomendo medir antes de mexer.**
- **R2 · `distribuicao` ficou instável** (9/10 → 8/10): a correção do E4 endureceu a
  fronteira e o erro migrou de lado. Precisa de um exemplo positivo explícito de
  distribuição sem a fórmula "of every ten".
- **R3 · whitelist de rótulo é curta demais**: `'Length'` caiu pelo mesmo motivo que
  `'Source:'`. Estender com rótulos de dimensão (length, height, weight, speed, depth,
  distance, age, duration). Baixo risco: são palavras de eixo, não dados.
- **R4 · `punchline` perdeu recall** (8/10 → 7/10). Foi rebaixada a último recurso para
  parar de engolir as outras classes — funcionou, mas agora frases de impacto genuínas
  ficam sem texto. Dano baixo (ausência de overlay, não erro na tela). Reavaliar só se
  incomodar no vídeo.

---

## 5. O QUE ESTE RELATÓRIO **NÃO** PROVA

1. **Não prova 93,7% em produção.** Textos e gabarito são meus (§1.2). O número é teto.
2. **N=2 por célula.** 5 corpora × 2 rodadas dá boa leitura de *tendência* e de *erro
   sistemático*, mas cada intenção individual tem só 10 amostras — uma falha isolada
   (`tendencia` china) pode ser ruído.
3. **Testa trecho isolado, não roteiro inteiro.** Na produção o detector vê o roteiro
   completo e escolhe **no máximo 3 momentos** — ou seja, ele também precisa *priorizar*,
   e isso esta bateria não mede.

### Próximo passo que remove o viés

Rodar sobre **roteiros reais do canal** (os 3 biomas já validados servem), com **gabarito
cego**: o operador marca o que esperava ver na tela **sem ver a saída do detector**; só
depois se compara. É a única forma de medir sem o autor do teste ser o autor do texto.

---

---

# ⚔️ CONTRA-AUDITORIA (05/08, a pedido do operador)

> O operador mandou: *"não assuma como fonte da verdade afirmações do relatório,
> contraponha, procure erros e gaps"*. Cada afirmação acima foi re-verificada contra o
> código real e o bruto. **O relatório acima tem 4 problemas graves e 3 menores.**
> Os agregados numéricos (89,5%→93,7%, tabelas por corpus e por grupo) **conferem** com o
> bruto recontado célula a célula — os problemas estão no que o relatório **não mediu** e
> no que **maquiou sem intenção**.

## C-1 🔴 CONTAMINAÇÃO: as correções embutem frases do próprio teste

Verificado por sonda textual: as regras novas em `_REGRAS` contêm fraseados
**quase-verbatim do corpus de avaliação**:

| Sonda | Está na regra? | Está no corpus? |
|---|---|---|
| `"rarely passes five"` | ✅ | ✅ (amazonia/comparacao) |
| `"this footage"` | ✅ | ✅ (local_tag, 5 corpora) |
| `"of every ten"` | ✅ | ✅ (distribuicao, 5 corpora) |

Isso é **ensinar para a prova**. O ganho de `comparacao` (4→9) e `local_tag` (6→10) foi
medido **no mesmo corpus cujas frases entraram na regra** — parte do ganho é
generalização real, parte é memorização do exemplo. **O antes/depois do §2 está inflado
em magnitude desconhecida.** Só um corpus hold-out (escrito DEPOIS das regras, com
fraseados distintos) separa as duas coisas.

## C-2 🔴 PRECISÃO NUNCA FOI MEDIDA — e o número é feio

A métrica "acerto" pergunta só *"o esperado apareceu?"* — **ignora o que apareceu junto**.
Recontado no bruto:

| | acertos c/ momento EXTRA de outra classe |
|---|---|
| antes | **73/170 (43%)** — punchline×45, stat×11, lowerthird×6… |
| depois | **47/178 (26%)** — punchline×28, spec×8… |

Em produção o detector escolhe **até 3 momentos**: o extra não é inofensivo, **vira texto
na tela**. O relatório mede recall e chama de acurácia. (As correções melhoraram a
precisão de 57%→74% como efeito colateral — mas isso nunca foi reportado nem virou meta.)

## C-3 🔴 REGRESSÕES REBATIZADAS DE "isolado"

O §4 lista `lowerthird→punchline`, `item_lista→subcapitulo` e `tendencia→nada` como
"isolado". Recontagem por intenção mostra que **as três eram 10/10 ANTES**:

```
melhoraram: comparacao +5 · local_tag +4 · ranking +3 · stat +1      (+13)
PIORARAM:   tendencia −1 · lowerthird −1 · item_lista −1
            · distribuicao −1 · punchline −1                          (−5)
```

São **regressões colaterais das minhas mudanças** (ou ruído), não falhas pré-existentes.
O rebaixamento da punchline e as regras novas de desempate mexeram no equilíbrio de
classes vizinhas — o relatório só admitiu isso para `distribuicao` (R2) e `punchline` (R4).

## C-4 🔴 O GANHO NÃO É ESTATISTICAMENTE PROVADO

+4,2pp = 8 casos em 190. Não-pareado: **1,48σ**. Pareado (McNemar, 16 melhoraram × 8
pioraram): **χ²=2,67 < 3,84 (p≈0,10)**. Leitura honesta: *consistente com melhora,
não demonstrada* — ainda mais somando C-1. O relatório afirma "+4,2" sem qualificar.

## C-5 🟡 BUG REAL que a auditoria NÃO pegou (gap do corpus): separador de milhar

```
"a wall of 1,200 kilometres"  -> falados = {1.2}      (1200 rejeitado!)
"40,000 jaguars remain"       -> falados = {40.0}     (40000 rejeitado!)
```

O regex de dígitos trata a vírgula de milhar como decimal. Roteiro real escreve números
em dígitos; **o corpus só usa números por extenso** — por isso passou ileso. Grounding
rejeitaria dado legítimo em produção.

## C-6 🟡 A ligação `and` cria soma espúria — afrouxa o I14 no sentido perigoso

```
"between ninety and seventy years" -> falados = {90, 70, **160**}
```

O 160 **nunca foi falado** e entra no conjunto de aceitos: se o LLM alucinar 160, o
validador aprova. Regra correta do inglês: `and` só liga **imediatamente após escala**
("two hundred AND fifty"); entre dois números soltos, fecha a corrida.

## C-7 🟡 Divergência doc×código no teste do silêncio

O relatório/§1.1 diz *"aceita-se no máximo 1 punchline"*. O código aceita **1 momento
não-factual de QUALQUER classe** (`len(outros) <= 1` — um `subcapitulo` solitário também
passaria). Nos dados só apareceu punchline, mas o critério escrito ≠ critério executado.

## C-8 · Veredito sobre R1-R4

| Rec | Veredito | Ajuste |
|---|---|---|
| R1 (decimal falado) | ✅ correta, e "medir antes de mexer" é a postura certa | — |
| R2 (distribuicao instável) | ✅ confirmada no bruto (9→8) | é regressão colateral, não "instabilidade" espontânea (C-3) |
| R3 (whitelist de rótulos) | ⚠️ correta MAS sem escopo | whitelist hoje vale p/ `texto`/`kicker`/`autor` inteiros; estender às cegas afrouxa o validador. Escopo: só palavra ÚNICA de eixo, e nunca no campo `autor` |
| R4 (punchline recall) | ⚠️ correta mas SUBdimensionada | a queda não foi só punchline: o rebaixamento derrubou `lowerthird` e `item_lista` junto (C-3) |

**Faltou:** R para `nota_fonte→citacao` (menor, 1/10) e — muito mais importante — faltavam
**C-1, C-2, C-5 e C-6 inteiros**, que esta contra-auditoria adiciona.

## C-9 · O que segue VÁLIDO do relatório original

Aritmética dos agregados ✓ (recontada do bruto) · o achado do buraco de domínio
animais×lugares ✓ (não depende das correções) · **silêncio 10/10 nas duas rodadas ✓** ·
o bug E1 do C1 ✓ (real, corrigido, 9/9) · a instrumentação `morto_grounding` ✓ · a
declaração de viés do §1.2 ✓ (a contra-auditoria a confirma e amplia com C-1).

*Contra-auditoria de 05/08/2026, verificada por recontagem do bruto + sondas no código.
Plano de correção: [ESPEC-B1-VALIDACAO-EXECUCAO.md](ESPEC-B1-VALIDACAO-EXECUCAO.md).*

---

*Auditoria de 05/08/2026. Bruto em `director/_auditoria_acervo.json` (+ `_ANTES`).
Correções E1-E6 aplicadas em `director/acervo_texto.py`.*
