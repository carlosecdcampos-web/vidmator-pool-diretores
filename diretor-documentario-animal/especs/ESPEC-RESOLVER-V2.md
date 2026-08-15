# ESPEC — RESOLVER v2.5: a violação e o desperdício não nascem

> **v2.5 = v2.4 + PRESCRIÇÃO do Codex (5ª rodada) incorporada.**
> ⚙️ **O CONTRATO DE IMPLEMENTAÇÃO VIVE EM `ANEXO-PRESCRICAO-RESOLVER.md`** —
> arquivo/função/assinatura/ordem prescritos pelo fiscal, ancorados no código real
> (58 referências arquivo:linha). Esta espec é o PORQUÊ e as REGRAS; o anexo é o
> COMO. Divergência entre os dois = o anexo manda (ele foi conferido no código).
> Bate-voltas 1-3 cumpridos por ordem do operador. Esta versão vai a implementação.
>
> **DECISÃO DO OPERADOR (12/08, palavra final): a COTA CRIATIVA do VEO fica INTACTA**
> — as cenas planejadas para o VEO continuam sendo geradas normalmente (é a
> identidade visual do canal). O "VEO é último recurso" vale EXCLUSIVAMENTE para o
> GAP DE HARD-MISS: cena de banco sem material → tenta CÓPIA pelas diretrizes de
> reuso seguro (≥90s, máx 2, tratamento diferente) → só se a cópia for impossível,
> VEO socorro. **No full_flow, a MESMA lógica**: 150/200 cenas, faltaram 10/15 →
> duplica dos assets já produzidos pelas mesmas diretrizes — re-gerar no VEO vira
> exceção, não rotina. Consequência: a 2ª passada do VEO morre como caminho normal
> nos DOIS formatos.
> Classificação obrigatória: FONTE / REMÉDIO declarado / DIAGNÓSTICO.
> Fora do escopo (operador): render e narração (servidor novo + Skia).

---

## 0. EVIDÊNCIA (corrigida pela rodada 2)

- O manifesto da onça registra 28 cenas de identidade, 246 `POSTER_BATCH`, soma
  1.445 em `julgados_no_poster` — a métrica "558" era a contagem ERRADA (entradas de
  `tentativas`, não candidatos). **PASS@k recontado pelo fiscal: mediana 12
  julgados/cena, máx 18 — 3 LOTES cobrem os 21 casos com PASS.**
- **D0 (novo, DIAGNÓSTICO obrigatório antes de codar): script `passk_onca.py`
  reproduzível**, versionado no repo, lendo o decision_manifest e imprimindo
  PASS@k por lotes E por candidatos; o ledger novo passa a registrar TODO lote
  (inclusive all-pass e falha de infra, que hoje somem).

## R1 — ORÇAMENTO ÚNICO POR CENA (`Budget`)

- **Unidade: LOTE de Vision** (até 6 posters/1 chamada). **Teto: 4 lotes/cena**
  (dado real: 3 cobrem tudo; 1 de folga), knob `VM_LOTES_CENA`.
- **Aritmética honesta** (corrigida): 30 cenas × até 4 lotes = **até 120 chamadas
  no PIOR caso** — o ganho não vem do teto e sim de (a) eliminar re-resoluções
  (hoje ilimitadas) e (b) descoberta compartilhada. O teto é a REDE, não o motor.
- **Objeto `Budget` persistente por `scene_id`** (no manifesto): sobrevive a
  restart/replay, **inclui irmãs pós-reconciliação** (nascem com saldo próprio de
  1 lote — o pai já gastou o dele).
- **Transação do Budget (v2.3 — 16 workers no mesmo saldo):** WAL por
  `operation_id`; **reserva + fsync ANTES do submit**; a resposta grava
  `COMPLETED` ou `INFRA_REFUND`; estado `UNKNOWN` pós-crash CONSOME por segurança;
  read-modify-write sob lock do estado do job.
- Consome: lote submetido; vet completo (1). NÃO consome: infra (429/503/timeout/
  Vision fora) — registrada à parte no ledger (`infra_failures`).
- Vale em TODOS os call-sites: `resolver_identidade`, cascata `resolve()` L1-L5,
  anti-repetição, micro-takes, pai-neutra, reparo — todos recebem o `Budget` da
  cena; **caminho sem Budget não pode julgar** (assert).

## R2 — ALOCAÇÃO GLOBAL

- **Fase A: cardinalidade máxima** (ninguém fica sem material se existir casamento
  completo); **fase B: dentro das soluções de cardinalidade máxima, maximizar
  score** — implementação: `scipy.optimize.linear_sum_assignment` com custo
  `(-score)` e aresta proibitiva para pares não-aprovados (**scipy já está em
  `requirements-director.txt:31`** — zero dependência nova). Desempate final
  determinístico por `provider:id`. Cena não casada → R3 → R4, em ordem de
  escassez (menos alternativas primeiro).
- **Score 0-100 nasce no gate** (prompt versionado `gate_v2`, persistido por
  candidato no manifesto). Falha do Vision → score ausente ≠ 0 (infra).
- **Identidade em 2 níveis, schema versionado** (`ident_v2`):
  `pre = provider:id` (Pexels/Pixabay id, URL canônica Commons, `prompt_hash` VEO);
  `pos = sha1 + dHash 64-bit` calculado no download.
- **Clustering (v2.3 — Hamming ≤8 não é transitivo, não pode ser chave):**
  **union-find persistido com representante canônico**; asset novo que une dois
  clusters dispara revalidação das reservas afetadas; **vídeo usa amostragem
  TEMPORAL versionada** (dHash em 3 frames 20/50/80% + duração), nunca 1 frame.
- **Exceções (as únicas, declaradas):** janela de word-pop = **1 alocação lógica**
  (a identidade entra no livro global; proibido vazar para fora da janela);
  **irmãs 16.a**: micro-alocador pós-reconciliação com MESMAS regras + Budget
  próprio; **pai-neutra: sem exceção** — vira consumidor do alocador.
- **Morte real dos alocadores paralelos** (rodada 2 provou que "morrer" não estava
  especificado): `reuse-fill` (resolver_cascata.py:2559-2575), `abre-swap`
  (:2588-2642), reparo (:2777-2787), `reuse-fill-emergencia` do veo_gerar
  (veo_gerar.py:87-123) — **todos passam a chamar `alocador.pedir(cena)`** (que
  aplica R2→R3→R4); o código de escolha própria deles é REMOVIDO, não desviado.

## R3 — REUSO SEGURO

- `inicio_2 − fim_1 ≥ 90s` (tela limpa) · máx 2 aparições · tratamento final
  DIFERENTE. Identidade = `ident_v2` (pega gêmeos e cross-provider — o F15 atual
  compara `clip_rel` e é cego a isso: `_decupagem_video.py:294-310`).
- **Acoplamento (protocolo definido):**
  0. (v2.3) a linhagem vive em **`allocation_group_id`** (não só `_copia_de` por
     scene_id): sobrevive à absorção/fusão de cenas pelo reconciliador via
     `scene_aliases`; e o REPARO pós-congelamento só pode trocar por material que
     CAIBA na duração congelada — nunca provocar nova divisão/reconciliação;
  1. alocador grava na cena-cópia: `_copia_de: <scene_id>` + `allocation_group_id`;
  2. `edicao`, ao FIM da alocação de molduras/overlays, roda `resolver_copias()`:
     calcula o **tratamento canônico** `(moldura|None, overlay|None, entrada|None)`
     da original e da cópia; iguais → troca o da cópia pela alternativa de MENOR
     custo de cota; **sem alternativa viável → devolve a cena ao alocador com o
     material vetado** (reuso proibido);
  3. `reconciliar_cenas` pode mudar molduras DEPOIS (:588-598) → **verificador
     final** (novo passe curto pós-reconciliador, timeline congelada): re-agrupa
     por `ident_v2`, valida 90s/2×/tratamento; violação → **REPARO automático**
     (re-aloca a cópia via alocador); irreparável → **REPROVA pré-render**.
- `pre_render_report` e L05/F15 migram de índice/path para **segundos + ident_v2**
  (leitor único — abaixo).

## R4 — ORDEM DE DECISÃO + COORDENAÇÃO VEO (v2.3: protocolo corrigido pela rodada 3)

- **Cota criativa do VEO: INTACTA** (decisão do operador acima). R4 governa SÓ o
  gap de hard-miss das cenas de banco (e, no full_flow, os buracos de geração):
  `material único → (identidade hard-miss: atmosfera única) → CÓPIA por reuso
  seguro (R3) → VEO socorro → nunca vazio`.
- A "aceitação com violação por prazo" (:2222-2247) MORRE — prazo esgotado cai
  para R3/R4, nunca para violação.
- **Coordenação banco→VEO (corrigida — o ACK-antes-do-envio da v2.2 perdia ou
  duplicava trabalho):**
  - **`coord_dir` ÚNICO do job**, FORA de `_ws_banco`/`_ws_veo` (os ramos rodam em
    workspaces separados — um mailbox por ramo nasceria duplicado e invisível);
  - eventos com **`request_id` + `attempt_id`** e máquina de estados
    `REQUESTED → CLAIMED(lease) → SENT → DONE|FAILED`;
  - **claim sob lock/CAS** (não append cego); **lease com expiração** — crash após
    claim devolve a cena à fila; ACK significa CONCLUSÃO, nunca reivindicação;
  - o merge decide pela **última geração do `request_id`**, não por scene_id;
  - **[v2.4 — DECISÃO DO OPERADOR] CICLO DE VIDA: o MERGE RELANÇA o consumidor.**
    O ramo VEO NÃO fica residente esperando (custo e complexidade sem retorno: no
    desenho novo o socorro é raro). Fluxo: os dois ramos terminam → o merge lê o
    `coord_dir` → **se houver `REQUESTED` sem `DONE`, relança um ciclo VEO curto
    só para esses pedidos** → merge final. É a "última barreira" e praticamente
    não será usada;
  - **FENCING (furo apontado): `attempt_id` é MONOTÔNICO por `request_id`**; `DONE`
    de uma tentativa cujo lease expirou é REJEITADO (chega tarde = descartado, o
    pedido já foi reivindicado por outra tentativa); `SENT` sem `DONE` dentro do
    lease volta a `REQUESTED` com `attempt_id+1`.
- **Contador persistente ÚNICO de falhas por `scene_id`** (`veo_falhas.json`, com
  CAS/lock e `attempt_id` — hoje o contador é por FILENAME, veo_ciclo.py:206-216).
  Teto 3 → DESISTE declarado → cena volta ao alocador (R3). **Loop da lontra
  impossível por construção.**
- **Circuit-breaker de INFRA persistente**: infra não consome Budget, mas ganha
  teto próprio entre restarts (senão retry infinito — furo da rodada 3).

## R5 — DISCIPLINA DO VEO

- **FONTE já parcialmente existente** (rodada 2): `veo_pedido` cria slot
  `sNNN`↔cena; `veo_entrega` rejeita conteúdo entregue a outra cena
  (veo_entrega.py:131-185); gate de completude exige material próprio
  (veo_gerar.py:127-142). O que falta e entra:
  1. **`prompt_hash` ponta a ponta**: já existe prompt-store por idx + sidecar
     arquivo→prompt (veo_entrega.py:214-223, veo_download.py:268-330) — nasce o
     **LEITOR ÚNICO `identidade_de(path) → ident_v2`** (módulo `ident.py`) usado
     por alocador, leis, report e decupagem. Gêmeos (mesmo prompt_hash) = mesmo
     material.
  2. **Tombstones por `generation_id` FÍSICO** (v2.3 — o tombstone por prompt_hash
     da v2.2 mataria o card novo junto com o velho, pois a regeneração conserva o
     prompt). **[v2.4] CONTINUIDADE BIFÁSICA (furo apontado):** o `generation_id` é
     **imutável e nasce no DOM**: `gid = sha1(src_canônico)`; as representações
     seguintes são ALIASES gravados no MESMO registro —
     `{gid, src, hash_bruto (download), hash_normalizado (pós-transcode do pool)}`.
     A chave de contabilidade e de tombstone é SEMPRE o `gid`; hash bruto e
     normalizado são só evidências. Sidecar OBRIGATÓRIO em toda fronteira
     (download E entrega) carregando `gid + src + prompt completo + hashes`; tombstone + fsync ANTES de mover o asset para `_aposentados/`;
     recuperação pós-crash completa movimentos interrompidos; o download ignora
     `generation_id` tombstonado (o src aparece na varredura DOM —
     veo_download.py:172-211).
  3. **Auditoria contábil por `generation_id`** com estados terminais MUTUAMENTE
     EXCLUSIVOS (`usado | reprovado | aposentado`): `gerados == Σ estados`; sobra
     sem classificação = falha listada no diário.
  4. **Sidecar completo no download** (v2.3 — a rodada 3 provou que
     veo_download.py:268-330 NÃO grava sidecar arquivo→prompt, só o prompt truncado
     no filename): o download passa a gravar o sidecar com o prompt COMPLETO, no
     mesmo formato do veo_entrega (:214-218); o leitor único `ident.py` consome os
     dois.
- Reuso de asset VEO para tampar buraco: 1×, critérios R3, nunca sequencial.

## R6 — CENA-ELEMENTO LIMPA + REGIME DE BLOQUEIO

1. **Normalização NA FONTE** (não só irmãs): todo passe que cria/marca cena-elemento
   (edicao C4/kinetic, capítulos, teases) grava `transicao="corte"`, `sfx=False`,
   `fade=0`, `presentacao=None-sonora` no ATO. O corte tardio do film
   (reconciliar:659-675) vira cinto, não fonte.
2. **Bloqueio por ID de lei, com estado versionado**: novo campo em `LEIS`:
   `regime: "bloqueia" | "avisa"` + `resolvida_em: <data/commit>`. L01/L02/L03/L05
   (e a futura L14) nascem `bloqueia`. `aprovar_render`: violação de lei
   `bloqueia` → **não escreve o lock** (hoje escreve sempre — :129-135); modo
   automator: quarentena SEM produzir. Violação de lei `avisa` → opção A (sai,
   nunca calado). Sem regex frágil: o ID da lei é o contrato.

## TRANSVERSAIS

1. **Manifesto `scene_id`** (schema `manifest_v2`): chave nova; leitura legado
   (`scene_id` ausente → `str(idx)`); `pre_render_report.py:258` migra junto.
2. **Allowlist do merge** ganha: `_copia_de`, **`allocation_group_id`**,
   **`scene_aliases`**, `prompt_hash`, **`gid`**, `ident_v2`, score, reservas do
   alocador, Budget. **[v2.4 — erro corrigido]** a v2.3 exigia
   `allocation_group_id` na R3 e o esquecia aqui: a linhagem morreria no merge.
3. **Telemetria por operação** (`telemetria.jsonl` por job): busca, poster-download,
   Vision (fila/latência/batch), download final, ffmpeg, vet, re-resolução —
   com `op, scene_id, dur_ms, bytes, resultado`. **A meta ≤25 min só vira aceite
   depois desta telemetria numa run.**
4. **Ledger de arestas**: falha SEMÂNTICA pós-vet queima a aresta cena↔material
   (persistida); falha de INFRA não queima nada; re-matching é transacional
   (reservas das cenas não afetadas preservadas).
5. **Correção imediata** (independe do resto): `c["inicio"]` inexistente em
   `resolver_identidade` (crash com pool ativo — `resolver_cascata.py:1109-1112`);
   assinatura ganha `t_cena` explícito.

## ACEITE (E2E de espécie nova)

| Métrica | Onça | Aceite |
|---|---|---|
| Lotes de Vision/cena | s/ teto real | ≤ 4 em TODOS os caminhos (ledger prova) |
| F15 por `ident_v2` (incl. gêmeos) | ≥1 | **0** |
| Repetição: tela limpa | n/medida | ≥ 90s, máx 2×, tratamento ≠ (verificador final) |
| Assets VEO sem classificação contábil | 54 | **0** |
| Elemento com sfx/film por baixo | 10 | **0** (lei `bloqueia`) |
| 2ª passada VEO | 26,3 min | só cenas não-ACKadas; loop impossível (contador) |
| Cenas vazias | 0 | 0 |
| Resolver | 61,3 min | meta experimental ≤25 — telemetria decide o aceite |

## SEQUÊNCIA DE PRs (rodada 3 — cada um com bancada offline)

| PR | Conteúdo | Bancada de aceite |
|---|---|---|
| **PR0** | `passk_onca.py` + ledger completo (all-pass e infra) + fixture congelada da onça | números reproduzíveis |
| **PR1** | núcleo de estado: scene_id/manifest_v2, writer atômico, WAL, locks, Budget, telemetria, edge ledger | 32 threads + kill em cada ponto de persistência + restart: nunca >4 débitos, JSON sempre legível |
| **PR2** | identidade: ident.py, prompt_hash ponta a ponta, fingerprint temporal, union-find, generation_id, tombstones, contabilidade | cópias exatas, transcodes, cross-provider, cadeia A≈B≈C, 2 cards do mesmo prompt, crash entre tombstone/move |
| **PR3** | alocador global: gate_v2, matching 2 fases, remoção dos 4 alocadores paralelos | grafos adversariais + brute-force em matrizes pequenas |
| **PR4** | coordenação VEO: coord_dir, claim/lease, request/attempt id, contador único, merge por estado | 2 consumidores, crash antes/depois do claim e do envio, ACK duplicado |
| **PR5** | reuso/editorial: allocation_group, resolver_copias, micro-alocador de irmãs, aliases, verificador final, L05/F15/report por ident_v2+segundos | original absorvida, 16.a, word-pop, 89.99/90.00s, reparo sem mudar topologia |
| **PR6** | leis: normalização na fonte, regime bloqueia/avisa + resolvida_em, L14, lock bloqueante | cada lei isolada; automator em quarentena não renderiza |
| **PR7** | E2E fake offline (providers determinísticos, restart/replay, contabilidade zero-sobra) → só depois canário real de espécie nova | telemetria + meta de 25 min medida |

---

## R7 [v2.4] — OS 4 WRITERS DE REGERAÇÃO: TETOS DECIDIDOS PELO OPERADOR

> Furo apontado na 4ª validação: a regra "duplica em vez de re-gerar" ficaria letra
> morta porque **4 caminhos re-geram sozinhos**, sem consultar o alocador. Tetos
> aprovados pelo operador (12/08):

| Writer | Hoje | v2.4 | Depois do teto |
|---|---|---|---|
| `veo_ciclo` (`veo_ciclo.py:213,280`) | reenvia faltantes em até **6 rodadas** | **2 tentativas** | cena → `alocador.pedir()` (duplicação R3) |
| `veo_driver` (`veo_driver.py:447,508`) | re-enfileira card falhado/reprovado (até `--regen`) | **1 re-envio** | duplicação — se o Flow recusou o prompt (política), insistir raramente resolve |
| `veo_socorro` (`veo_socorro.py:154-178`) | gera socorro por conta própria (MD5, 2 usos, 7 índices) | **não gera**: vira consumidor de `alocador.pedir()` | as diretrizes R3 (≥90s, máx 2, tratamento ≠) substituem os critérios próprios |
| `veo_recolher_perdidas` | 2ª passada cega, sem teto entre execuções | só o que o **mailbox não cobriu**, com o contador persistente (teto 3) | DESISTE declarado → duplicação |

**Invariante que nasce daqui:** nenhum caminho re-gera no VEO sem antes perguntar ao
alocador se a **duplicação segura** resolve. O VEO é socorro, não reflexo.
**Contabilidade:** toda tentativa de regeneração incrementa `veo_falhas.json` por
`scene_id` (CAS/lock) — os tetos acima são verificáveis no ledger, não confiança.

---

## §8 [v2.5] CONTRATOS FECHADOS PELA PRESCRIÇÃO (resumo — detalhe no ANEXO)

| Contrato | Decisão prescrita |
|---|---|
| **Mailbox** | módulo novo `director/coord_veo.py`; `REQUESTED` publicado **no mesmo ato da realocação** (resolver_cascata.py:2808) — não no fim; `coord_dir` compartilhado passado aos dois ramos; claim atômico no Windows via `msvcrt.locking`; `DONE_REJECTED` para tentativa com lease expirado; comando **dirigido** (consome só os pedidos); teto persistente do próprio relançamento |
| **`gid`** | **nasce UMA vez** em `veo_download`, no par atômico do DOM (`src+prompt` juntos, :172-200); `casar_espelho` passa a copiar o **sidecar** junto da mídia (:309-332); a **entrega REJEITA mídia sem sidecar** e preserva o `gid` — nunca derivar outro ID dos bytes; `veo_curador` (:139-152) para de apagar antes do tombstone |
| **Linhagem no merge** | separar `CAMPOS_MEDIA` de **`CAMPOS_META`**; META copiado **mesmo sem `clip_path`** (hoje o merge exige mídia e descartaria reservas/aliases/Budget); absorção do reconciliador faz **união ordenada** de `scene_aliases`; irmã herda o grupo do **doador**, não do pai |
| **R7 (tetos)** | os tetos são REMÉDIO; a **FONTE é um chokepoint ÚNICO de submissão VEO** com consulta ao alocador + reserva persistente **imediatamente antes de `fd.enviar_prompt()`**. Pré-requisitos achados: o lote **não carrega `scene_id`** (veo_pedido.py:833) — precisa carregar; `veo_driver` tem **DOIS** reenfileiradores (:447 e :508) que dividem o mesmo teto; o writer do socorro está em `veo_socorro.py:211-228` (não :154); o **gate final (veo_gerar.py:131-142) precisa aceitar duplicação segura**, senão reprova o que o alocador aprovou; `_fecho_*.py` não pode mais repetir o recolher 3× (conflita com o contador interno) |

## §9 [v2.5] BUGS REAIS ACHADOS DE PASSAGEM (fora da espec — corrigir junto)

1. **Lock do ciclo VEO não é atômico** (`veo_ciclo.py:143-154`): lê e depois
   `write_text()` — dois processos podem reivindicar o mesmo ciclo. Mesmo
   `msvcrt.locking` do mailbox.
2. **`_srcs.json` envenenado** (`veo_download.py:251-254`): o `src` entra no
   conjunto **antes** do HTTP; se o download falha e outro persiste o conjunto
   (:277), aquele src **nunca mais é tentado**. Só inserir após binário+sidecar
   fsyncados. ← candidato a explicar assets perdidos.
3. **Workspaces paralelos com nome global fixo** (`_rodar_paralelo.py:29,40-44`):
   `_ws_veo`/`_ws_banco` são apagados recursivamente — dois runners simultâneos se
   destroem. Usar `_jobs/<job>/workspaces/veo|banco`.
4. **Promoção não atômica** (:151-183): staging por `merge_id` + manifesto +
   `os.replace()` como commit final.
5. **`state_io.py`**: `atomic_write_json()` + `locked_update_json()` como primitiva
   única de timeline, manifesto, mailbox, alocador e ledger.
