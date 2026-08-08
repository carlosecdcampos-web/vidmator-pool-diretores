# ESPEC — VALIDAÇÃO FINAL DO B1 (detector do acervo): execução passo a passo

> **Status:** 🟡 rascunho para o GO do operador (05/08).
> **Origem:** a contra-auditoria do [RELATORIO-AUDITORIA-ACERVO.md](RELATORIO-AUDITORIA-ACERVO.md)
> achou 4 problemas graves (C-1 contaminação · C-2 precisão não medida · C-3 regressões
> colaterais · C-4 ganho não provado) e 2 bugs de grounding (C-5, C-6). Esta espec é o
> caminho de B1 do estado atual até "pronto para plugar no pipeline", **sem os vícios da
> medição anterior**.
>
> **Regra-mãe:** aditivo e gated (I1). O detector continua FORA do pipeline até o aceite
> do §6 + GO explícito.

---

## PASSO 0 — Correções determinísticas de grounding (sem LLM, sem decisão)

Bugs com fix objetivo, testável em milissegundos. Em `acervo_texto.py`:

### 0.1 Separador de milhar em dígitos (C-5)
- **Hoje:** `"40,000"` → falados `{40.0}` — o número REAL (40000) é rejeitado.
- **Regra:** vírgula entre grupos de 3 dígitos é MILHAR (`1,200` → 1200; `40,000` → 40000);
  vírgula decimal só com 1-2 dígitos após (`3,5` → 3.5, padrão europeu raro em roteiro EN);
  ponto decimal (`1.5`) permanece decimal.
- **Teste de aceite:** `1,200→1200 · 40,000→40000 · 1.5→1.5 · 3,5→3.5 · 12,34→12.34` +
  regressão dos 9 casos do C1.

### 0.2 Ligação `and` só após ESCALA (C-6)
- **Hoje:** `"between ninety and seventy"` → falados incluem **160** (nunca falado) —
  o validador aceitaria um 160 alucinado. I14 afrouxado no sentido perigoso.
- **Regra:** `and` continua a corrida **somente se o token anterior da corrida foi uma
  escala** (`hundred/thousand/million/billion`). Entre dois números soltos → fecha.
- **Teste de aceite:** `two hundred and fifty→250 ✓ · ninety and seventy→{90,70} SEM 160 ·
  a thousand→1000` + regressão completa.

### 0.3 Escopo da whitelist de rótulos (R3 com o ajuste do C-8)
- **Hoje:** `_ROTULOS_NEUTROS` vale para os campos `texto`/`kicker`/`autor` inteiros.
- **Regra:** whitelist só se aplica a **palavra única** e **nunca ao campo `autor`**
  (autor inventado é exatamente o que I14 protege). Adicionar eixos de dimensão:
  `length, height, weight, speed, depth, distance, age, duration` (+ pt equivalentes).
- **Teste de aceite:** kicker `"Source:"` passa · texto `"Length"` passa · autor
  `"Source"` REPROVA · texto `"Length of the wall"` (multi-palavra) segue validando verbatim.

### 0.4 Critério do silêncio: alinhar código e doc (C-7)
- **Decisão embutida (proposta):** a folga de 1 momento não-factual vale para
  **qualquer classe não-factual** (não só punchline) — é o que o código já faz; o DOC
  que estava errado. Registrar no runner o critério por extenso.

## PASSO 1 — Métrica de PRECISÃO no runner (C-2)

`_smoke_variancia.py` passa a reportar, por caso e agregado:
- **recall** (o atual "acerto");
- **precisão**: `extras = obtidos − {esperado}`; separar **extras factuais**
  (stat/percentual/comparacao/… — dado na tela) de **extras não-factuais**;
- **regra dura nova:** extra FACTUAL não-gabaritado = reprova o caso (dado indevido na
  tela é pior que ausência); extra não-factual conta na precisão, não reprova.
- Baseline já conhecido para comparação: precisão 57% (antes) → 74% (depois).

## PASSO 2 — Descontaminar as regras (C-1)

1. Reescrever os 3 exemplos embutidos em `_REGRAS` com fraseados **sintéticos** que não
   ocorrem em nenhum corpus (`"rarely passes five"` → outro par de medidas; `"this
   footage"` → descrever o critério sem citar a abertura; `"of every ten"` → idem).
2. **Sonda automática no runner:** nenhuma 4-gram das `_REGRAS` pode ocorrer em corpus de
   avaliação — falhou, o teste ABORTA acusando contaminação. (Vale para sempre: impede
   que a próxima rodada de "melhorias" repita o vício.)

## PASSO 3 — Corpora HOLD-OUT (a medição que vale)

- **2 corpora novos**, escritos **DEPOIS** do Passo 2, mesmo formato (19 intenções +
  silêncio): 1 animal (`oceano/ártico` — bioma não usado) e 1 lugar histórico (`roma
  antiga` — domínio fraco na auditoria).
- Regras de construção: as mesmas 4 do cabeçalho de `_corpora_acervo.py` + sonda do
  Passo 2 + **números em DÍGITOS em pelo menos 3 trechos** (cobre o C-5, que o corpus
  atual não exercitava).
- Os 5 corpora atuais viram **conjunto de desenvolvimento** (para iterar regra);
  os 2 novos são **teste** — regra nunca é editada olhando para eles.

## PASSO 4 — Rodada robusta (o "teste mais robusto" que o operador pediu)

- `7 corpora × 19 intenções × 3 reps = 399 casos` + 21 silêncios (~15 min, mesma infra).
- **Relatório final** com recall + precisão por corpus/intenção, dev × hold-out separados,
  e significância (McNemar contra a rodada anterior quando aplicável).

## PASSO 5 — Gabarito CEGO nos roteiros reais (remove o viés de autor)

1. Pego os 3 roteiros validados (amazônia/austrália/áfrica).
2. **O operador marca**, num HTML simples, os momentos que ELE esperaria na tela
   (trecho + família), **sem ver a saída do detector**.
3. Roda o detector nos mesmos roteiros; comparação lado a lado; divergências viram
   decisão editorial dele (o gabarito é ele, não eu).
- Este passo também mede o que a bateria não mede: **priorização** (máx 3 momentos num
  roteiro inteiro, não trecho isolado).

## PASSO 6 — CRITÉRIOS DE ACEITE para plugar no pipeline (proposta a bater)

| Métrica | Meta |
|---|---|
| Recall médio no **hold-out** | ≥ 90% |
| Desvio entre corpora | ≤ 5 pp |
| Extras FACTUAIS não-gabaritados | **0** (reprova direto) |
| Precisão geral | ≥ 80% |
| Silêncio | 21/21 |
| Gabarito cego (Passo 5) | aprovação qualitativa do operador |

Aceite atingido → implementação no pipeline segue a própria ESPEC-REGRA-DE-USO-ACERVO
(caller com H3 `dim=0.15`, knobs `acervo_texto` no preset, guarda no `legibilidade`) —
**com novo GO explícito**.

## ✅ DECISÕES DO OPERADOR — TRAVADAS (05/08, GO dado)

| # | Decisão | Resposta do operador |
|---|---|---|
| D-a | Aceitar "one metre eighty"=1,80 no grounding? | **NÃO por ora** (*"vou seguir sua recomendação"*) — medir frequência em roteiro real antes de afrouxar o validador |
| D-b | Extra factual não-gabaritado reprova o caso? | **SIM** (*"concordo com você"*) — dado factual indevido na tela é pior que ausência |
| D-c | Folga do silêncio = 1 não-factual de qualquer classe? | **SIM** (*"concordo também"*) |
| D-d | Quem escreve o hold-out? | **"Você + Sonda"** — eu escrevo, a sonda automática de 4-grams vigia a contaminação |

---

# 📊 RESULTADO DA EXECUÇÃO (05/08 — passos 0-4 concluídos)

**399 casos + 21 silêncios · 7 corpora (5 dev + 2 hold-out) × 19 intenções × 3 reps · 12 min**
Bruto: `director/_auditoria_acervo.json` · log: `_log_auditoria_final.txt`

## A pergunta do C-1 foi respondida: o ganho ERA real

| Grupo | Recall estrito (D-b) | Métrica antiga |
|---|---|---|
| **dev** (ensinaram as regras) | 87,0% | 93,0% |
| **hold-out** (nunca vistos) | **83,3%** | **91,2%** |
| gap dev→hold-out | **3,7 pp** | 1,8 pp |

**Um gap de 3,7 pp é generalização, não memorização.** Se as regras tivessem apenas decorado
o corpus, o hold-out desabaria. A contaminação existia (C-1 era procedente), mas a correção
real dominou o efeito. **Silêncio: 21/21** — nos 7 temas, incluindo os 2 inéditos.

## O preço da regra D-b, medido

| Métrica | Valor |
|---|---|
| recall **antigo** (ignora extras) | **92,5%** |
| recall **estrito D-b** (extra factual reprova) | **86,0%** |
| custo da regra | **−6,5 pp** = 26 casos com dado factual a mais na tela |

Extras que sujaram: `spec`×19 · `stat`×8 · `comparacao`×2.

## 🔴 ACHADO QUE INVALIDA PARTE DO 86,0%: o gabarito é single-label

`lowerthird` despencou para **7/21** — mas **13 dos 21 casos são `acerto_sujo`**, não erro.
Causa, com evidência:

```
"Doctor Sarah Kimani, the ecologist who has followed this pride for ELEVEN SEASONS…"
        ↑ lowerthird esperado          ↑ número REALMENTE falado -> spec/stat legítimo
```

**Os 7 trechos de `lowerthird` que escrevi contêm um número de tempo de carreira.** O
detector marca o lowerthird **e** o número — e o número **foi falado** (passou pelo
grounding, logo não é invenção). O gabarito, que só admite 1 rótulo por trecho, pune o
detector por achar um segundo fato **que está lá**.

- `lowerthird` responde por **13 dos 26 sujos** (metade).
- Excluindo `lowerthird`, o recall estrito sobe para **88,9%**.

**Conclusão honesta: 86,0% é piso pessimista; 92,5% é teto otimista; o número verdadeiro
está no meio e esta bateria não consegue decidi-lo** — porque separar "extra factual
legítimo" de "extra factual editorialmente lixo" é **julgamento editorial**, não medição.
Quem decide é o operador, no Passo 5.

**Correção do método (para a próxima rodada):** gabarito **multi-label** — cada trecho
declara TODAS as intenções legitimamente marcáveis; só extra FORA desse conjunto suja.

## Erros remanescentes com correção proposta

| Erro | N | Diagnóstico | Correção |
|---|---|---|---|
| `spec` 12/21 | 5× nada detectado | efeito colateral do D-a: "one metre eighty"/"3 metres" pulados por não poderem ser escritos como falados | revisitar D-a **com dado**: agora sabemos que custa ~5 casos em 21 |
| `citacao` 15/21 | 6 sujos | a citação vem junto de um número ("after thirty winters") | multi-label resolve |
| `punchline` 15/21 | 3× nada | último recurso apertado demais (R4 confirmada) | afrouxar só se incomodar no vídeo |
| `tendencia` | 1 morto: *"número NÃO falado: 7000000"* | "7 million square kilometres" → o LLM montou 7000000; `_compostos` dá 7 e 1000000 separados | **bug real ainda aberto**: dígito + escala por extenso ("7 million") não compõe |
| `item_lista`, `ranking` | 1 cada | rótulos `'ITEM 5'`, `'Deadliest Coastal K…'` inventados | whitelist não cobre rótulo COMPOSTO; avaliar regex de rótulo curto |

## Aceite do §6 — situação

| Métrica | Meta | Real | |
|---|---|---|---|
| Recall no hold-out | ≥ 90% | 83,3% estrito · 91,2% antigo | ⚠️ depende do multi-label |
| Desvio entre corpora | ≤ 5 pp | **4,8 pp** | ✅ |
| Extras factuais | 0 | 26 (metade = artefato do gabarito) | ⚠️ |
| Precisão | ≥ 80% | 65-79% | 🔴 |
| Silêncio | 21/21 | **21/21** | ✅ |

**Veredito: NÃO plugar ainda.** Faltam o gabarito multi-label (mede o que existe) e o
Passo 5 (define o que o operador quer na tela). Precisão de 65-79% é o número que mais
importa e o único claramente abaixo da meta.

---

## ORDEM DE EXECUÇÃO

```
0. Passo 0 (4 fixes determinísticos + testes)      ── minutos, sem decisão
1. Passo 1 (métrica de precisão)                   ── precisa D-b
2. Passo 2 (descontaminar + sonda automática)
3. Passo 3 (2 corpora hold-out)                    ── precisa D-d
4. Passo 4 (rodada 399 casos -> relatório final)
5. Passo 5 (gabarito cego do operador)             ── precisa DELE na frente do HTML
6. Aceite §6 -> GO -> implementação no pipeline
```

*Criada 05/08/2026 a partir da contra-auditoria. Nada roda sem o GO.*
