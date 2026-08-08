# ESPEC — REGRA DE USO DO ACERVO (o "passo 8"): quando cada variante entra + como o LLM preenche

> **Status:** 🟢 **ALINHADA (04/08) — P1-P4 fechados; execução em fases (§5). Fase 1 em curso.**
> Nada se implementa sem GO. O objetivo declarado pelo operador (04/08):
> *"discutindo qual melhor abordagem para fazer um trabalho mais perto da perfeição logo de
> primeira, mitigando os erros e aumentando o mais próximo possível de 100% os acertos."*
>
> **Origem:** é o §8 da [ESPEC-ACERVO-40-VARIANTES.md](ESPEC-ACERVO-40-VARIANTES.md), que o
> operador tirou do GO anterior (resposta E) justamente por ter tamanho de espec própria.
>
> **Regra-mãe:** nada aqui altera o que já está validado. Tudo aditivo e gated (I1). O
> `ilustrar.py`/Sistema 1 **continua funcionando** — convivência, não substituição (decisão D).

---

## 0. HERDADOS APROVADOS DA RODADA ANTERIOR (implementar JUNTO desta rodada)

Aprovados no teste isolado de 04/08 (*"ficou PERFEITO"*) — pequenos, entram no primeiro
commit desta rodada:

| # | Ajuste | Onde |
|---|---|---|
| H1 | **SFX shutter 0.15 → 0.13** (alto de ouvido no teste) | `BrollTest` (marcações) + `TransTest` |
| H2 | **`film` + Apagão de ENTRADA obrigatório** — SEMPRE que a apresentação `film` aparecer: Apagão (clarear, pico no início da cena) + shutter na entrada; saída não precisa | `BrollTest` (cenas com `presentacao.tipo === "film"`) |
| H3 | `dim = 0.15` da família OVERLAY (regra do §5 da espec do acervo, esperando o caller — o caller nasce AQUI) | o compositor de overlay desta espec |

---

## 1. O PROBLEMA, COM PRECISÃO

40 variantes prontas e aprovadas visualmente (10 **Texto** tela-cheia · 14 **Overlay** sobre
o vídeo · 16 **Gráfico**, sendo 3 overlay). Falta o que decide **quando** cada uma entra e
**com que conteúdo** — sem quebrar as leis já pagas:

- **I13** — variante sem os dados exigidos NÃO pode ser escolhida (filtro ANTES da escolha).
- **I14 / R-32 / Contrato Editorial grupo B** — dado que não está no roteiro **não existe**.
  O LLM não inventa número, nome ou fonte. As variantes de gráfico **recusam** sem dado real.
- **I6** — o word-pop é dono da tela; overlay não entra na janela dele (guarda no último pass).
- **I5** — nada aparece por menos de ~1s.
- **I3** — um dono por emenda/momento; nunca empilhar.
- **I15** — variantes marcadas OK ficam byte a byte.

## 2. ARQUITETURA PROPOSTA — "o LLM detecta, a regra escolhe"

> Princípio (provado nos mapas e no `apresentar.py`): **o LLM nunca escolhe a variante
> diretamente.** Ele faz o que faz bem — entender o texto — e devolve MOMENTOS com intenção
> e dados. A variante sai de **tabela determinística** + filtro de elegibilidade. Isso é o
> que aproxima de 100%: o espaço de erro do LLM cai de "40 opções × props livres" para
> "classificar a intenção de um trecho falado".

### Camada 1 — DETECTOR (pass novo `director/acervo_texto.py`, LLM Gemini como os demais)

Lê `roteiro + words.json + timeline.json` e devolve **momentos**:

```json
{ "trecho": "…verbatim do roteiro…",      // âncora p/ localizar o timestamp (padrão dos passes)
  "intencao": "stat",                      // vocabulário FECHADO (§3)
  "dados": { "texto": "…", "kicker": "…", "valores": [23,18], "labels": ["…"], "sufixo": "%" },
  "fonte_dados": "falado" }                // sempre "falado" — validado depois, não confiado
```

### Camada 2 — VALIDADOR DE GROUNDING (determinístico, sem LLM)

O ponto mais importante para a assertividade. Cada momento só sobrevive se:
- `trecho` localiza no `words.json` (mesmo mecanismo do `detectar_mapas.localizar`);
- **todo número em `dados.valores` aparece FALADO** na janela do trecho (busca no words,
  com variantes: "23", "twenty-three", "23%") — **um número não-falado = momento DESCARTADO
  inteiro** (hard fail, não "conserta");
- `texto`/`kicker` são substring/quase-verbatim do roteiro (fuzzy ≥ threshold — mesmo padrão
  da legenda do polaroid: melhor vazio que inventado).

### Camada 3 — ESCOLHA + COLOCAÇÃO (determinística)

- Tabela `intencao → variantes candidatas` (§3) + **filtro de elegibilidade** (I13): a
  intenção `comparacao` sem 2 valores falados não vira `Graf05` — cai para `Ovl11` (1 valor)
  ou não entra.
- **Rotação** dentro das candidatas (variedade sem repetir a mesma variante no vídeo; mesmo
  espírito do pool de apresentações).
- **Knobs no preset** (I1): `acervo_texto: { max_por_video, min_gap_s, familias: [...] }`.
  Preset sem o knob = pass não roda = zero mudança.
- **Colisões**: janela do momento não pode tocar word-pop (I6), mapa, cold-open, CTA, cena
  com `film`/apresentação especial, nem cena que já tem `ilustracao` do Sistema 1
  (convivência = **um dono por cena**). A guarda final roda no `legibilidade` (último pass),
  como manda a lição do "1 Million".
- **Texto tela-cheia (família Texto)** é caso especial: cobre o b-roll → entra pelas regras
  de elemento de tela cheia (corte seco por baixo, saída por cima — as do mapa). Por isso a
  proposta é **começar só com OVERLAY + os 3 Graf-overlay** e liberar tela-cheia depois
  (ver pergunta P2).

## 3. TAXONOMIA — vocabulário fechado de intenções

O campo `quando` dos manifestos (`TEXTO_MANIFEST`, `OVERLAY_MANIFEST`, `GRAFICOS_MANIFEST`)
vira este vocabulário — o LLM só pode responder com uma destas:

| intencao | O que é na fala | Variantes candidatas (ordem de preferência) | Exige (I13) |
|---|---|---|---|
| `stat` | número dramático falado | `Ovl12_GiantStat` · `Graf15_OvlStatCorner` · `Graf14_OvlCounterPunch` | 1 número falado |
| `percentual` | % falada | `Graf16_OvlProgressBar` · `Graf03_DonutPercent`* | 1 % falada |
| `comparacao` | A vs B com valores | `Graf05_VersusBars`* · `Graf06_VersusTug`* | 2 valores falados |
| `ranking` | top-N com valores | `Graf09_RankList`* | 3+ itens com valores |
| `citacao` | fala citando alguém/algo | `Ovl07_QuoteAttribution` | trecho + autor falado |
| `punchline` | frase de impacto curta | `Ovl06_CenterPunch` | trecho curto (≤8 palavras) |
| `subcapitulo` | virada de assunto nomeada | `Ovl02_SubchapterLine` | título curto do trecho |
| `nota_fonte` | "according to…", fonte/ressalva | `Ovl04_FootnotePill` | fonte falada |
| `local_tag` | lugar/época pontual | `Ovl05_CornerTag` | nome falado |
| `spec` | medida/unidade ("6 meters long") | `Ovl11_SpecBadge` | valor + unidade falados |
| `item_lista` | item de enumeração estrutural | `Ovl10_NumberBadge` | posição + nome falados |
| *(nenhuma)* | o normal — a maioria das cenas | — | — |

\* = tela cheia ou semi: só entram se P2 liberar; senão a intenção cai para a alternativa overlay.

> `Ovl01_ChapterBig`, `Ovl03_LowerThird`, `Ovl08/09/13/14` e as famílias Texto/Graf
> tela-cheia ficam **fora do vocabulário inicial** (menos opções = mais assertividade) e
> entram em rodadas seguintes, com dados de smoke test na mão.
>
> ⚠️ **REQUISITO do operador (04/08): as 40 variantes têm que ser ENCONTRÁVEIS ao final.**
> *"temos 40 variáveis, temos que garantir que todas sejam encontráveis."* O recorte de 12 é
> estratégia de rodada, não teto — o vocabulário cresce por fases até cobrir as 40 (cada
> expansão passa pelo mesmo ciclo: smoke → HTML de revisão → aprovação). A tabela do §3
> ganhará as intenções que faltam (capitulo_full, lowerthird, nota_lateral, ticker, preço,
> veredito, contador, odômetro, gauge, timeline, linha, pizza, multi-barras, dual-line,
> big-stat, e as 10 de texto tela-cheia) conforme as rodadas avançam.

> 📌 **Sequência revista pelo operador (04/08):** os roteiros de 7 minutos (Fase 2) estão
> ADIADOS — *"tem mais ajustes que quero fazer no hook primeiro"*. A Fase 1 continua aberta
> recebendo ajustes até ele liberar.

## 4. 🧪 SMOKE TESTS — a parte central desta espec (SEM RENDER)

> Pedido do operador: *"mandar um smoke test para ver se o llm irá captar a inserção correta
> de cada uma das variantes, talvez mais de um, com roteiros, letras, palavras diferentes, e
> analisar o grau de assertividade na correlação."*

### 4.1 Corpus (diversidade proposital)

| # | Roteiro | Por quê |
|---|---|---|
| 1-3 | os 3 biomas reais (Amazônia, Austrália, África) — roteiro + words REAIS | material de produção, com gabarito visual conhecido |
| 4 | um roteiro-LISTA real ou escrito p/ o teste ("5 predadores…") | estrutura enumerada — testa `item_lista` e a fronteira com o ChapterTitle |
| 5 | um roteiro DENSO em números (estatísticas, %) | estressa grounding: números falados vs números "arredondados" pelo LLM |
| 6 | um roteiro SEM números e SEM citações | o teste do silêncio: o pass tem que devolver QUASE NADA — falso positivo aqui é o erro mais perigoso |

### 4.2 Gabarito

Para cada roteiro, uma lista `esperado.json`: `{trecho, intencao_esperada}` + trechos onde
**NADA** deve entrar. **Quem faz:** eu proponho o gabarito dos 6 → **o operador revisa** (é
o julgamento editorial dele que define "certo"). ~15 min de revisão por roteiro.

### 4.3 Métricas (por rodada, por roteiro)

| Métrica | Alvo | Hard fail? |
|---|---|---|
| **Grounding**: 0 token/número inventado nas props | **100%** | 🔴 SIM — um inventado reprova a rodada |
| **Precisão de momento**: momentos achados ÷ gabarito | ≥ 85% | não |
| **Precisão de intenção**: classe certa nos momentos achados | ≥ 90% | não |
| **Falso positivo** (momento onde o gabarito diz "nada") | ≤ 1 por vídeo | 🔴 2+ reprova |
| **Colisões** (word-pop/mapa/ilustração/densidade) | **0** | 🔴 SIM (mas é bug de código, não de LLM) |

### 4.4 A/B de abordagem de prompt (a "discussão da melhor abordagem")

Três estratégias, mesmas entradas, mesmas métricas — a vencedora é a que ganhar **na média
dos 6 roteiros**, não num só:

- **V1 — cena a cena**: classifica cada cena isolada. Simples; tende a falso positivo
  (quer achar algo em tudo).
- **V2 — roteiro inteiro, uma chamada**: vê o todo, escolhe os N melhores momentos. Doseia
  melhor; prompt maior; é o padrão da casa (`apresentar`, `efeitos`, `topicos`).
- **V3 — duas etapas**: (1ª) "liste APENAS números/citações/fontes FALADOS, verbatim" —
  extração pura; (2ª) classifica a intenção só dos extraídos. Máximo grounding por
  construção; 2× chamadas.

Aposta inicial (a confirmar nos dados): **V3 para gráficos/stat** (o que não pode errar
número) e **V2 para texto/overlay** (o que precisa de senso do todo).

### 4.5 Harness

`director/_smoke_acervo.py`: roda o detector nos 6 corpus × K rodadas (variação real do
LLM), compara com o gabarito, imprime a matriz por métrica e por estratégia. Sem render,
custo ~zero (Gemini Flash). Reprodutível: mesmo comando, tabela nova.

### 4.6 Critério de aprovação → PoC → GO de produção

```
smoke: 2 rodadas consecutivas com grounding 100% + intenção ≥90% + FP ≤1
  → PoC DE RENDER: mini-render (~20s, footage da Austrália) com 3-4 inserções reais
    → operador assiste e aprova o VISUAL em contexto
      → GO de produção (pass plugado, gated no preset doc_realista)
```

## 5. ✅ ALINHAMENTO FECHADO (operador, 04/08)

**P1 — Densidade: máx 3 inserções, espaço ≥ 20s** → knob `acervo_texto: {max_por_video: 3,
min_gap_s: 20}`.

**P2 — ~~Rodada 1 só com overlays~~ SUPERADA pelo operador (04/08):** *"não faz sentido —
devemos implementar tudo já. Vamos validar o overlay sendo que depois, ao inserir o texto,
pode ter que reentender o overlay, e a mesma coisa com o gráfico. Gerar 2 roteiros de 7 min
para depois ter que regerar não faz sentido."* → **o vocabulário cobre as 40 DESDE JÁ**; o
smoke valida o repertório completo de uma vez. (O argumento dele é o correto: as famílias
INTERAGEM na escolha — validar em fatias obrigaria a revalidar a cada fatia nova.)

**P3 — Gabarito: proposta minha + revisão dele. E um GATE NOVO no pipeline** (palavras do
operador): *"antes de rodar o render final, para ser mais rápido, vamos fazer aquela versão
de html, cena a cena, e vamos ver se ele entendeu os lugares corretos; caso não tenha
entendido, podemos corrigir daí mesmo, sem esperar o render total."*
→ **HTML DE REVISÃO CENA A CENA** entra como etapa fixa: depois do detector rodar num
roteiro, gera-se um HTML mostrando cada cena com a inserção proposta (variante + props +
posição na fala); o operador **aprova/corrige ali** e só então o render acontece. Mesmo
método dos refinos anteriores — validado no navegador antes de entregar, com export das
correções.

**P4 — Roadmap em FASES (definido pelo operador):**

| Fase | O quê | Objetivo |
|---|---|---|
| **1** | Refino ISOLADO, começando pela **INTRODUÇÃO/HOOK** — smoke do detector só no trecho de abertura dos roteiros reais | mesma disciplina dos testes isolados das transições: uma parte pequena, dominada por completo |
| **2** | **2 roteiros NOVOS de ~7 minutos**, com listas: **"Os 4 predadores mortais da Amazônia"** (animais) e **"5 curiosidades sobre o Egito antigo"** (SEM animais — *"para vermos se esse diretor roteirista servirá para os dois"*) | generalidade de domínio + estrutura-lista de verdade (cada item = capítulo → conecta com o detector de roteiro-lista do ChapterTitle) |
| **3** | Smoke completo nos 6 corpus + A/B de prompt + HTML de revisão + PoC de render | aprovação final e GO de produção |

> ⚠️ **Escala nova embutida na Fase 2:** 7 minutos ≈ **4× a duração** dos vídeos validados
> (~100s). Isso estressa TUDO (tópicos, mapas, orçamento de repetição, duração de render) —
> não é só o acervo. Tratar como marco próprio: o primeiro vídeo de 7 min é um evento de
> validação em si.
>
> Nota: os 2 roteiros da Fase 2 serão **gerados** (roteirista) — nenhum existe hoje. Eles
> substituem o corpus #4 sintético do §4.1 e viram o material canônico do teste de lista.

---

*Criada em 04/08/2026, na sequência da aprovação das transições. Espec irmãs:
[ESPEC-EDICAO-ADITIVOS](ESPEC-EDICAO-ADITIVOS.md) ·
[ESPEC-ACERVO-40-VARIANTES](ESPEC-ACERVO-40-VARIANTES.md).*
