# RELATÓRIO COMPLETO — E2E LEOPARDO-DAS-NEVES (13/08/2026)

> Escrito a pedido do operador após a falha do ramo do banco. Contém TUDO:
> cronologia passo a passo, cada erro com causa raiz e `arquivo:linha`, telemetria
> integral, orçamento cena a cena, ledger de envios, o que funcionou (medido), os
> meus erros de execução E os meus erros de leitura durante o acompanhamento, o
> estado exato de cada artefato agora, e o que está pendente de decisão.
>
> Fontes primárias (todas preservadas em disco):
> - `_workspace/_leopardo_neves_cronometro.jsonl` — tempos de cada etapa
> - `_workspace/_leopardo_neves_diario.jsonl` — diário do fecho
> - `_workspace/logs_fecho/paralelo_veo_banco.attempt-1.log` (64.375 B) — o log do paralelo
> - `_workspace/logs_fecho/veo_gerar.attempt-1.log` (10.253 B) — o que a série fez
> - `_ws_banco/_jobs/leopardo_neves/estado/budget.json` — orçamento cena a cena
> - `_ws_banco/_jobs/leopardo_neves/estado/telemetria.jsonl` — 542 operações medidas
> - `_coord/leopardo_neves/veo_falhas.json` — ledger de envios ao VEO

---

## 1. VEREDITO EM UMA LINHA

O ramo do VEO entregou **48/48 takes, perfeito**. O ramo do banco entregou **0/32**
porque **dois bugs meus** (unidade errada do orçamento + exceção não capturada num
caminho) derrubaram o processo — e um **terceiro problema de desenho** (fallback
para série) começou a **regerar takes do VEO já prontos** até eu matar o job.
Nenhum asset foi perdido. Nenhum arquivo de produção anterior foi tocado.

---

## 2. CRONOLOGIA COMPLETA (todos os horários reais)

| hora | t+ | evento | resultado |
|---|---|---|---|
| 12:51:10 | 0,0 | START do `_fecho_leopardo` | voz ashtar, VEO on, coleção "teste 05 - leopardo das neves" |
| 12:51:10 | 0,0 | narração (Chatterbox) inicia | |
| 12:58:18 | 7,1 min | narração OK | **428,2s** — onça: ~12 min. 991 palavras, 6,2 MB |
| 12:58:36 | 7,4 min | whisper OK (17,2s) + deloop OK (0,2s) | words.json gerado |
| 13:03:35 | 12,4 min | montar_timeline OK | **299,1s** — 80 cenas, 389,44s de vídeo |
| 13:08:25 | 17,3 min | contrato_visual OK | **290,4s** — 54/80 cenas com sujeito obrigatório (onça: 28/82) |
| 13:08:25 | 17,3 min | enumeracoes OK (0,1s) | |
| 13:08:26 | 17,3 min | veo_alocacao OK (0,2s) | **banco 32 · veo_video 29 · veo_imagem 19** |
| 13:08:26 | 17,3 min | paralelo VEO×banco inicia | dois processos, workspaces `_ws_veo` e `_ws_banco` |
| ~13:08–13:10 | | **ramo BANCO roda e MORRE em ~2 min** | crash por `SemSaldo` não capturado (seção 3.2) — 0/32 cenas |
| 13:16:05 | | ramo VEO: ciclo de 19 IMAGENS inicia | fiscal de prompts: rodada 1 = 19 aprov/28 reprov; rodada 2 = 11/17; rodada 3 = 3/14 |
| 13:26:10 | | imagens: **19/19 prontas em 1 rodada** | `FIM imagem: 19/19 | desistidos no gate: 0` |
| 13:26:15 | | ramo VEO: ciclo de 29 VÍDEOS inicia | |
| 13:41:50 | | vídeos: 28/29 na rodada 1 | |
| 13:43:02 | | rodada 2: pediu **1**, colheu **1/1**, parou | sem loop — o desenho novo funcionando |
| 13:45:32 | | `FIM video: 29/29 | desistidos: 0` · curador: 47 limpos, 0 regen · `[V9-G4] takes VEO: 48/48` | ramo VEO **rc=0** |
| 13:48:01 | 56,9 min | paralelo termina: `VEO rc=0, BANCO rc=1` | **guard do merge segurou**: "a timeline principal NÃO foi tocada" |
| 13:48:01 | | 🔴 o fecho cai no **fallback SÉRIE** (`veo_gerar` avulso) | seção 3.3 — começa a regerar o que já existia |
| 14:02:40–14:05:30 | | série reenvia **12 de 19** prompts de imagem à coleção | ledger registra: 12 cenas passam a 2 envios |
| ~14:05:45 | | **eu mato a árvore inteira** (ordem do operador: opção A) | 48 assets preservados em `_ws_veo` |

Tempo total gasto até o kill: **~74 min**. Nenhum render aconteceu.

---

## 3. OS ERROS — CAUSA RAIZ, UM A UM

### 3.1 🔴 ERRO MEU #1 — a UNIDADE do orçamento está errada (o estrangulamento)

**O que a espec manda** (ESPEC-RESOLVER-V2.md §R1): a unidade do orçamento é o
**LOTE de Vision** — 1 chamada com até 6 posters. A medição PASS@k da onça ("3
lotes cobrem 21/21") foi feita **nessa unidade**, e o teto de 4 veio dela.

**O que eu implementei** (commit `5081f6e`): o débito em `_api_text` com imagem —
ou seja, **QUALQUER chamada de Vision debita 1**, não qualquer lote. Só que o
caminho de identidade faz, por variante de query:

```
1 poster-gate (o LOTE da espec)          = 1 débito
  └─ aprova até 2 candidatos
       vet_contrato do candidato 1        = 1 débito   ← NÃO é lote
       vet_contrato do candidato 2        = 1 débito   ← NÃO é lote
```

Uma única variante custa até 3 débitos. **O teto de "4 lotes" virou, na prática,
~1,3 lote por cena.**

**Prova nos números** (budget.json + log):
- 65 poster-gates para 32 cenas = **2,0 lotes reais/cena** (a espec queria até 4)
- 117 débitos = 3,7/cena — a diferença (52 débitos) são vets cobrados como se fossem lotes
- 23 de 32 cenas bateram no teto; 9 não bateram e TAMBÉM não resolveram (seção 4.3)

**Orçamento cena a cena** (gastos/infra):
```
cena 0:3/0   cena 3:3/0   cena 6:3/0   cena 8:3/0   cena 11:2/0  cena 13:4/0
cena 16:4/0  cena 19:4/0  cena 21:4/0  cena 23:4/0  cena 26:4/0  cena 28:4/0
cena 30:4/2  cena 31:4/0  cena 35:4/0  cena 38:4/0  cena 41:4/0  cena 43:4/0
cena 47:4/0  cena 48:4/0  cena 51:4/0  cena 52:4/0  cena 55:4/0  cena 57:4/0
cena 61:4/0  cena 63:4/0  cena 66:4/0  cena 68:2/0  cena 71:3/0  cena 73:4/0
cena 76:3/0  cena 78:3/0
```

**Por que o erro nasceu**: na revisão do item 1, o Codex apontou (corretamente)
que os vets chamavam Vision fora de qualquer teto, e mandou centralizar o débito
no submit. Eu obedeci — mas **mantive o número 4**, que tinha sido calibrado na
unidade antiga. Centralizei a cobrança e esqueci de recalibrar o teto para a
unidade nova. Erro de tradução entre espec e implementação, meu.

### 3.2 🔴 ERRO MEU #2 — `SemSaldo` não capturado num caminho = o processo INTEIRO morreu

Eu capturei `SemSaldo` nos dois pontos de entrada (`resolver_identidade` e
`resolve`), transformando saldo esgotado em ABSTENÇÃO da cena. **Mas há um
terceiro caminho que eu não cobri**: `_resolve_beat_cru` chama `vet_atmosfera`
DIRETO (resolver_cascata.py:2646), fora dos dois invólucros.

Traceback real (paralelo_veo_banco.attempt-1.log:435-462):
```
_resolve_beat → _resolve_beat_cru (:2646)
  → vet_atmosfera (:1118) → _vet_atmosfera_cru (:1153)
    → _api_text (:1765) → _orcamento (:1003)
      → budget.py:110  raise SemSaldo("cena 63: orçamento esgotado")
→ NINGUÉM captura → o worker morre → o resolver morre → rc=1
```

**Consequência**: não foram "32 cenas em abstenção" — foi **o processo do ramo
morrendo no meio**. Mesmo as 9 cenas que ainda tinham saldo perderam tudo, porque
a timeline do ramo nunca foi gravada (`_ws_banco/timeline.json`: 0/80 com clipe).

As dezenas de linhas `"ABSTENÇÃO (atmosfera/hard-miss decide)"` no log são as
cenas que degradaram CERTO pelos invólucros — até a cena 63 acertar o caminho
descoberto e derrubar tudo.

**Por que a bancada não pegou**: o `test_e2e_offline.py` dublava `_vet_atmosfera_cru`
com uma função que não passa por `_api_text` — então o caminho
vet_atmosfera→orçamento nunca foi exercitado com saldo esgotado. A bancada validou
o caminho feliz do próprio dublê. Erro de cobertura, meu.

### 3.3 🔴 ERRO DE DESENHO #3 — o fallback SÉRIE regenera o que o VEO já entregou

O `_fecho_leopardo` (herdado do `_fecho_e2e`) tem: "paralelo falhou → cai na
SÉRIE antiga" (`veo_gerar` + `resolver_cascata` avulsos). Esse fallback foi
desenhado para quando o paralelo **não consegue nem começar**. Hoje ele disparou
com o paralelo TENDO TERMINADO o ramo VEO com 48/48.

Em série, o `veo_gerar` olha `_workspace/_jobs/leopardo_neves/veo/assets` — que
estava **VAZIO** (os 48 assets estavam em `_ws_veo/...`, aguardando o merge que
não aconteceu). Resultado: ele considerou as 19 imagens "faltando" e **reenviou
12 de 19 prompts** à coleção do Flow entre 14:02:40 e 14:05:30, até eu matar.

Evidências:
- `veo_gerar.attempt-1.log`: `casamento offline: 0/19 | FALTAM: [2,5,10,14,17,22,27,29,32,39,44,45,49,56,58,65,69,74,77]`, depois `rajada: 12/19 enviados`
- **Ledger** (`_coord/leopardo_neves/veo_falhas.json`) registrou tudo:
  `{1 envio: 36 cenas, 2 envios: 12 cenas}` — as 12 são exatamente as reenviadas
  (cenas 2, 5, 10, 14, 17, 22, 27, 29, 32, 39, 44, 45), todas por `flow_driver`.

**Custo real**: 12 gerações de imagem duplicadas na coleção "teste 05" (cota do
Flow desperdiçada + cards duplicados que a próxima colheita precisa não-casar).
O teto do ledger (3) teria parado a TERCEIRA rodada — ele limita o loop, mas não
impede um replay legítimo-aos-olhos-dele. A decisão de reenviar foi do fallback,
não do ledger.

Houve ainda, dentro da série: 1 download do espelho com `HTTP 401` e 1
`Page.evaluate: Execution context was destroyed` (transitórios do Flow, sem
consequência além de log).

### 3.4 Erros MEUS DE LEITURA durante o acompanhamento (afetaram o que te reportei)

1. **"0 assets baixados"** — olhei a pasta da raiz do job durante o paralelo; os
   assets estavam em `_ws_veo` (o workspace do ramo). Alarme falso.
2. **"espelho: 0 mídias" como preocupação** — era a ÚLTIMA varredura, depois de já
   ter colhido tudo (os espelhos cheios estão nas linhas 546 e 650 do log).
3. **O mais grave**: às 13:38 eu te disse "o ramo do banco precisa fechar as 32
   cenas" — **o banco já estava morto desde ~13:10**. Eu não conferi o estado do
   processo do ramo, só o log agregado (dominado pelo VEO). Se eu tivesse visto o
   crash na hora, teria pausado 38 minutos antes e a série nunca teria rodado.
4. **"vision 14,5 min"** sem dizer que é SOMA de latência de 119 chamadas com 16
   workers em paralelo — em tempo de parede o ramo levou ~2 min. A frase te levou
   à conclusão errada de que "o Vision está demorando demais". Corrigido: a
   latência é ~7s por chamada (mediana 6,8s, p95 12,6s), normal para lote de
   posters; o problema nunca foi velocidade, foi a contagem.

---

## 4. TELEMETRIA INTEGRAL (a primeira run com T3 ligado)

`_ws_banco/_jobs/leopardo_neves/estado/telemetria.jsonl` — 542 operações:

| op | n | soma | mediana | p95 | resultado |
|---|---|---|---|---|---|
| vision | 119 | 14,5 min | 6,8s | 12,6s | 117 ok · 2 sem_resposta |
| vet | 34 | 3,5 min | 6,9s | 12,9s | |
| busca | 288 | 2,6 min | 0,4s | 1,2s | |
| download | 93 | 1,7 min | 0,5s | 4,3s | |
| vet_atm | 8 | 0,4 min | 3,6s | 4,2s | |

- **Tempo de parede do ramo banco: ~2 min** (13:08→13:10) — as somas acima são
  latência acumulada de 16 workers em paralelo.
- Cenas mais caras (soma de operações): cena 30 (98,9s — a única com infra=2),
  cena 61 (78,1s), cena 35 (54,6s), cena 38 (54,1s), cena 19 (53,1s).
- Leitura estrutural: **Vision = 85% do custo do resolver** (14,5 de 22,7 min de
  latência somada). Confirma que o R1 mira no lugar certo — com a unidade certa.
- Infra registrada sem consumir saldo: 2 ocorrências (cena 30), como desenhado.

Falhas de fonte no período (do log, sem consequência estrutural): SearxNG fora
(`WinError 10061`, dezenas de vezes — serviço local desligado) e Openverse/L2t
`HTTP 429` (rate limit). Ambas caíram nos fallbacks normais da cascata.

## 5. O QUE FUNCIONOU — MEDIDO, NÃO SUPOSTO

| item | evidência |
|---|---|
| **Ledger R7 em produção** | 48 envios na 1ª passada = 1 por cena, 0 reenvios; registro pela PORTA (`flow_driver`); e quando a série reenviou, o ledger CONTOU (12 cenas → 2 envios) — a conta é verificável, exatamente o objetivo |
| **VEO 48/48 sem loop** | imagens 19/19 em 1 rodada; vídeos 28/29 + rodada 2 pedindo SÓ 1 e parando; 0 desistidos; a onça teve cena com 5 envios e 0 entrega |
| **Guard do merge (fix do Codex, `fd7ff34`)** | `BANCO rc=1` → "a timeline principal NÃO foi tocada". Antes desse fix, o pai teria feito merge/recarga cega por cima |
| **Telemetria T3** | 542 ops com cena dona; apontou Vision=85% do custo — é a resposta de "onde foi o tempo" que nunca existiu |
| **Fiscal de prompts** | 3 rodadas (19/28, 11/17, 3/14 reprovações) — prompts ruins não chegaram ao Flow |
| **Curador** | 47 takes limpos, 0 texto queimado, 0 regeneração |
| **Contrato visual** | 54/80 cenas com sujeito obrigatório; prompts do VEO saíram com "warm brown rock and ochre alpine grass" — a camuflagem-contra-rocha do roteiro, não o clichê de neve branca |
| **Narração** | 7,1 min (onça: ~12) — GPU livre, sem contenção |
| **Abstenção por orçamento (onde coberta)** | dezenas de cenas degradaram declaradamente antes do crash — o mecanismo funciona nos caminhos cobertos |

## 6. ESTADO EXATO AGORA (14:10)

| artefato | estado |
|---|---|
| 48 assets do VEO | **SALVOS** em `_ws_veo/_jobs/leopardo_neves/veo/assets/` (19 png + 29 mp4) |
| narração + words | prontos (`_workspace/narracao.mp3`, `words.json`) |
| timeline principal | **intocada** — 80 cenas, 0 com clipe (o guard segurou) |
| timeline do ramo banco | 0/80 (processo morreu antes de gravar) |
| ledger de envios | 36 cenas×1 + 12 cenas×2 (o rastro fiel do que aconteceu) |
| coleção "teste 05" no Flow | 48 cards bons + **12 duplicatas de imagem** geradas pela série |
| processos | todos mortos (kill 14:05); nenhum zumbi |
| produções anteriores | intocadas (23 MP4 preservados) |
| tempo/cota gastos | 74 min de parede; 60 gerações VEO (48 úteis + 12 duplicadas); ~119 chamadas Vision |

## 7. PENDENTE DE DECISÃO (NADA será feito sem ordem do operador)

1. **Corrigir a unidade do orçamento**: débito de 1 lote no poster-gate; vets do
   candidato aprovado NÃO debitam do mesmo saldo (ou teto recalibrado por medição).
2. **Capturar `SemSaldo` no beat inteiro** (`_resolve_beat`), não só nos dois
   invólucros — saldo esgotado degrada A CENA, nunca mata o processo.
3. **Fallback série**: não pode disparar quando o ramo VEO terminou com rc=0
   (regenera o que já existe). Precisa distinguir "paralelo não começou" de
   "um ramo falhou".
4. **Retomada deste job**: promover os 48 assets → resolver as 32 cenas de banco
   (com orçamento desligado para MEDIR o consumo real) → seguir a cadeia. As 12
   duplicatas da coleção: a colheita futura tende a não-casá-las, mas podem ser
   apagadas à mão.

---
*Relatório gerado às 14:10 de 13/08/2026. Todos os arquivos-fonte citados estão
preservados nos caminhos indicados no topo.*
