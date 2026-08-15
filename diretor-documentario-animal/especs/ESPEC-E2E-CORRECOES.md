# ESPEC v3 — Correções 1-8 para o E2E de espécie nova

> v3 = v2 + parecer final do Codex (12/08, leu 103 fontes, 0 falhas): itens 1/3/4
> APROVADOS com checklist; 2/5/6/7/8 com defeitos de contrato PAGOS abaixo.
> Deltas v3 marcados com [v3].

> Decisão do operador (12/08): implementar **1-8 de uma vez** e validar com UM E2E
> completo — espécie nova, outro bioma, `VM_MODO=fresh` — porque o que se quer medir
> é a **velocidade total de fabricação** (meta: 30 min). Sem render intermediário da orca.
>
> Histórico: v1 reprovada pelo Codex (parecer em `scratchpad/codex_parecer.md`);
> esta v2 incorpora as DUAS rodadas do contraditório. Cada item sai com prova de
> bancada offline (sem VEO, sem rede) antes do E2E.

Convenções: **FONTE** = o problema deixa de existir · **REMÉDIO** = dano limitado,
declarado · **DIAGNÓSTICO** = só enxerga. Todo item lista os consumidores tocados.

---

## ITEM 1 — Captura por passo nos fechos (morte muda NUNCA MAIS)

**Fato:** `_fecho_orca_v2.py:112` roda `subprocess.run` SEM capturar nada e o `run()`
reporta "saída completa em X.live.log" **sem nunca escrever nele** — todo log saiu
"(vazio)", inclusive nos sucessos. A tentativa 1 do resolver morreu 6min24s com rc=1
e zero rastro.

**Mudanças (em `run()` do fecho, e espelhadas no `vidmator_bridge._rodar_passo`):**
1. stdout+stderr → `_workspace/logs_fecho/{label}.attempt-{N}.log` (um por TENTATIVA —
   Codex: um único live.log é sobrescrito entre tentativas).
2. Filho roda com `PYTHONUNBUFFERED=1` e `-X faulthandler` (crash nativo 0xC0000142
   pode não produzir traceback Python; faulthandler dá o stack C quando dá).
3. Registrar SEMPRE: comando, cwd, rc, duração, exceção. Na falha, o tail de 15
   linhas viaja na mensagem do diário.
4. `TimeoutExpired`/erro de spawn TAMBÉM anexam tail (Codex: o bridge atual só
   anexa quando tem returncode) e matam a ÁRVORE do processo no Windows
   (`taskkill /T /F`) — filho de filho não pode sobreviver ao timeout.
5. **`rc=5` deixa de ser sucesso.** Censo Codex: nenhum exit(5) no director; no
   piter-repo `_whisper_subprocess.py:80` usa 5 como FALHA de transcrição,
   `_smoke_tudo.py:55` como timeout. Tolerância global REMOVIDA de
   `_fecho_orca_v2.py:116` e `vidmator_bridge.py:132`; se um passe legitimamente
   precisar, whitelist explícita `rc_ok_extra={...}` por passo.

Classificação: DIAGNÓSTICO (e é o pré-requisito de todos os outros).
Consumidores: fechos (`_fecho_*.py`), `vidmator_bridge.py` (AC-Automator, aditivo).
Bancada: passo que imprime e explode → log existe por tentativa, tail na exceção,
rc=5 reprovado; passo que trava → árvore morta, tail presente.

---

## ITEM 2 — `scene_id` imutável + divisão não herda o que é do ELEMENTO

**Fato:** a divisão anti-loop (`reconciliar_cenas.py:511`, `nova = {**c, ...}`) copia a
cena inteira; esqueceu os campos do elemento e o `idx`. Timeline entregue da orca: idx
duplicados [16, 46, 69]; idx 16 com o MESMO `texto_impacto` nas duas metades (texto em
dobro na tela + troca de take no meio = o "microtake" do operador).

**Mudanças:**
1. **`scene_id` imutável** nasce no `montar_timeline` (`scene_id = str(idx)`); a divisão
   deriva `"16.a"`, `"16.b"`. **`idx` NUNCA é renumerado** (o manifesto do resolver
   grava por `str(c["idx"])` — resolver_cascata.py:2324; renumerar quebra replay).
2. Consumidores que hoje mapeiam por `idx` migram para `scene_id`:
   `_rodar_paralelo.py:91` (merge — com idx duplicado, uma cena SOME do mapa hoje),
   `_decupagem_video.py`, relatórios. O manifesto continua por idx (compat).
3. A parte nova da divisão faz **pop** de: `texto_impacto, infografico, infografico2,
   ilustracao, texto_pos, entrada_texto, palavra_chave, aparece_em, aparece_em_orig,
   texto_ate, pop_bg_dono, pop_bg_ate` (Codex B: herdar `pop_bg_dono` cria DOIS donos
   → fundos sobrepostos no render, BrollTest.tsx:868). Se a original era dona do
   word-pop, a nova recebe `pop_bg_membro` do mesmo grupo. `pop_bg_membro` existente
   é herdado (as duas metades seguem dentro da janela).
4. A parte nova nasce **incondicionalmente** com `transicao="corte"`, `sfx=False`,
   `fade=0` (Codex B: o guard atual só zera sob dono do universo velho).
5. `assert` de unicidade de `scene_id` fecha o passe do reconciliador.
6. [v3] Aceite corrigido: `scene_id` duplicado = 0; **`idx` duplicado é PERMITIDO só
   entre irmãos derivados** (16 + 16.a) — zero fora disso.
7. [v3] `_deloop_audio.py:296-297` RENUMERA todo idx (consumidor esquecido). Ele roda
   ANTES do resolver (idx ainda livre); se a timeline já tiver `scene_id`, ele os
   regenera junto — censo da ordem dos passes na implementação.
8. [v3] `dur_cena` é derivado (nasce em montar_timeline.py:435) e a divisão/fusão muda
   fronteiras sem recalcular — `pre_render_report.py:251,279` consome. Recalcular em
   TODA mudança de fronteira do reconciliador.
9. [v3] Migração explícita dos consumidores de idx: `_rodar_paralelo.py:91`,
   `_decupagem_video.py`, `pre_render_report.py:249-254` (nome de thumbnail/manifesto
   — com irmãos um sobrescreve o outro), `auditoria_composicao.py:75`.

Classificação: FONTE (o defeito deixa de ser possível por construção).
Bancada: dividir cena com texto_impacto+word-pop → metade nova sem elemento, sem
dono duplicado, corte seco; scene_id únicos; manifesto intocado.

---

## ITEM 3 — Função ÚNICA de janelas de elemento (as leis passam a ENXERGAR)

**Fato:** `leis.py:85-124` (universo das leis L01/L03) e `reconciliar_cenas.py:101`
(inventário de donos) montam CADA UM o seu universo — e NENHUM inclui os elementos
gravados na cena (`texto_impacto` etc., 18 na orca). L01 ("elemento é cena") e L03
("nada soa por baixo") estão certas e cegas. Origem do "som de projetor por baixo":
o rabo do `presentacao: film` vaza para dentro do elemento — exatamente o que L03
pegaria se visse a janela.

**Mudanças:**
1. Nova função `janelas_elementos(tl)` (módulo `leis.py`, exportada) que DERIVA, no
   ato da chamada, dos valores FINAIS (nada persiste — 4 writers mexem nesses campos
   depois do edicao: montar_timeline, edicao, legibilidade, ilustrar):
   - por cena: `texto_impacto` / `infografico` / `ilustracao` →
     `ini = aparece_em ?? inicio`, `fim = texto_ate ?? fim_da_cena`
     (a MESMA fórmula da legibilidade.py:158-168, confirmada no render
     BrollTest.tsx:1073/1253);
   - **`infografico2` → `ini = (aparece_em ?? inicio) + 3.0`, `fim = fim_da_cena`**
     (Codex A: janela própria, BrollTest.tsx:1115-1123);
   - mais o universo já existente: `marcacoes`/`acervo_texto`/`_capitulos_previstos`/
     `_janelas_curiosidade`/`mapas`/`pessoas` (código atual de `elementos_tela`).
2. `leis.elementos_tela()` delega para ela; o inventário de `donos` do reconciliador
   consome a MESMA função (fim dos dois universos).
3. A guarda final da legibilidade (`legibilidade.py:158`) passa a incluir
   `infografico2` (hoje o ignora — Codex A).

Classificação: FONTE (uma verdade, dois consumidores; a segunda lista da v1 morreu —
era dupla fonte de verdade).
Bancada: rodar `leis.py` na timeline REAL da orca → L01/L03 têm que acusar as
violações que o operador viu de olho (projetor sob elemento, elemento fora de
fronteira). A lei que não acusa no dado real conhecido está errada.

---

## ITEM 4 — `veo_recolher_perdidas` propaga erro (e só então retry)

**Fato:** `veo_recolher_perdidas.py:62-73` termina com `SystemExit(0)` SEMPRE — filho
falhou, cenas seguem vazias, VM_VEO desligado: tudo rc=0. Na orca: morreu com
0xC0000142 em 3,1s, `tentativas=1`, DESISTE → 9 de 94 cenas sem clipe.

**Mudanças (contrato validado pelo Codex, PONTO C):**
- rc=0: tudo recuperado (ou não havia perdidas);
- rc=2: filho `veo_gerar` falhou (inclui timeout/erro de spawn);
- rc=3: filho ok, mas restam cenas sem clipe (inclui `VM_VEO` desligado com perdidas);
- fecho: `tentativas=1 → 3`, **mantendo `essencial=False`** — o retry passa a existir
  e no esgotamento o job SEGUE com DESISTE honesto (lei do operador: "cota parcial é
  melhor que noite perdida").

Classificação: FONTE para o buraco silencioso (o sucesso falso deixa de existir);
o retry é REMÉDIO declarado para o 0xC0000142 (bug estatístico de spawn).
Bancada: simular filho rc!=0 → 2; timeline com buraco → 3; fecho retenta 3×.

---

## ITEM 5 — Gate pré-download por PROVEDOR (nada é baixado para ser julgado)

**Fato medido (orca):** 135 arquivos / 2,28 GB baixados → 25 usados (0,50 GB) = **78%
desperdício**; (a) 110 arquivos jamais usados = 1,78 GB (81% do ganho). Causas:
`resolve()` não tem gate — L3v baixa `vids[0]` direto (resolver_cascata.py:1662);
Commons busca+baixa na MESMA função e julga depois (`commons_image`,
resolver_cascata.py:1297: baixa → `vision_verifica_assunto` → reprovou → unlink).
E o `_poster_gate` atual **aprova automaticamente sem `required_subjects`**
(resolver_cascata.py:741) — reusá-lo cru para atmosfera seria no-op (Codex).

**Mudanças:**
1. **Interface comum de candidato** (pré-requisito do item 6 também):
   `{provider, id, poster_url, meta, download()}` — adaptadores para Pexels (já tem
   poster via `pexels_poster`), Pixabay (`_PIXABAY_POSTER`), Commons (separar a busca
   de metadados/thumb do download — a API do MediaWiki fornece `thumburl` sem baixar
   o original) e banco local.
2. **Gate de atmosfera por poster**: novo ramo do `_poster_gate` para cena SEM
   `required_subjects`, julgando pelo `visual_intent`/`stock_query` da cena (prompt
   próprio, em lote, mesmos knobs `VIS_POSTER`/`TETO_LOTE_GATE`). O gate de identidade
   continua o que é.
3. `resolve()` passa a: buscar candidatos (metadados) → gate por poster → **baixar SÓ
   o aprovado** → vet completo. Vale para L3v, L2, L2t.
4. **Download por trecho (F3.2 da v1): FORA deste E2E** (decisão com Codex: `-c copy`
   corta em keyframe, `clip_dur` falso quebra L04/Remotion, cache canônico seria
   envenenado). Re-medir o desperdício depois do gate; se ainda doer, desenho
   próprio (seek grosso antes do -i + reencode + ffprobe do resultado).
5. REMÉDIO: faxina do `stock/` do job — SÓ pasta privada do job, SÓ após render
   válido, marcação a partir da timeline final, NUNCA o cache canônico `pex/`.
6. [v3] O gate cobre TODOS os caminhos que baixam antes de julgar — não só L3v/L2/L2t:
   **L4i (resolver_cascata.py:1713), L4 (:1763), L5 (:1780)** entram, e
   `resolver_identidade()` (:995-1152) também passa a PRODUZIR a lista ranqueada —
   o árbitro (item 6) precisa das listas das cenas de identidade também.
7. [v3] O candidato é um DESCRITOR SERIALIZÁVEL `{provider, id, poster_url, meta}`
   (persiste no manifesto); o download é função à parte `baixar(descritor, dest)`
   — nunca método no objeto (o item 6 exige persistir as listas).

Classificação: FONTE (o download inútil deixa de existir por construção).
Bancada: mock de provedores → contar downloads = candidatos APROVADOS, nunca o lote;
Commons: zero download em reprova.

---

## ITEM 6 — Árbitro determinístico (o pós-loop vira conferência)

**Fato:** pós-loop serial custou ~73 dos 75 min do resolver da orca. Causa dupla:
(a) os 16 workers da fase 1 escolhem cegos uns aos outros (cache por QUERY —
resolver_cascata.py:1648 — dá o MESMO arquivo a queries iguais de cenas vizinhas);
(b) cada violação dispara até 4 re-resoluções completas em série
(resolver_cascata.py:2226-2275). A v1 propunha lock + "pegar o próximo candidato do
lote" — **reprovada**: a fila de candidatos é local e o download acontece dentro do
laço (não há "próximo pronto"), e lock entre workers quebra o determinismo do replay.

**Desenho (ditado pelo Codex, rodada 1):** resolução em TRÊS fases:
1. **Descoberta + poster-gate em PARALELO** (16 workers): cada cena produz uma lista
   RANQUEADA e ESTÁVEL de candidatos aprovados (ordenação determinística, desempate
   por `provider:id`). Nenhum download aqui (item 5).
2. **Atribuição GLOBAL serial e determinística**: um árbitro percorre as cenas em
   ordem de escassez (menos alternativas primeiro / maior "arrependimento" entre 1ª
   e 2ª opção — mata o viés "cena 1 pega o melhor, cena 89 pega a sobra") e atribui
   respeitando o orçamento INTEIRO (adjacência, `DIST_MIN_REUSO`, `MAX_USOS_ARQUIVO`)
   — as violações **não nascem**. Identidade pré-download = `provider:id`
   (pexels_id/pixabay id/URL canônica); hash de arquivo segue como verificação
   pós-download.
3. **Download + vet em PARALELO** só dos atribuídos. Falha de download/vet → a cena
   volta para a PRÓXIMA rodada determinística do árbitro (com o candidato queimado).
4. Listas ranqueadas + atribuições PERSISTEM no manifesto (replay/auditoria).
5. O laço de orçamento atual vira **CONFERÊNCIA** (DIAGNÓSTICO): espera-se ~0
   violações; >0 = bug do árbitro, com contadores no diário
   (`violacoes_nascidas` / `substituicoes_pos_loop`).
6. REMÉDIO: substituições residuais paralelas; teto por **deadline MONOTÔNICO DE
   PROCESSO** — [v3] CORREÇÃO FACTUAL: `_restante()` NÃO é global, é orçamento POR
   CENA em `threading.local` (resolver_cascata.py:267-287; init por worker em :2058).
   Nasce `DEADLINE_RESOLVER = time.monotonic() + teto` (global de módulo), consultado
   por busca, poster-gate, download, ffmpeg e Vision; estourou = registra e segue
   (nunca cancelamento de thread).
7. [v3] Prioridade do árbitro DETERMINÍSTICA e comparável: o gate persiste um score
   NUMÉRICO por candidato; arrependimento = `score(cand[0]) - score(cand[1])`;
   ordem das cenas = `(n_alternativas, -arrependimento, scene_id)`. Sem score
   numérico "arrependimento" não é comparável entre cenas.

Classificação: FONTE. Ganho esperado: ~70 min.
Bancada: mesma entrada 2× → MESMAS atribuições byte a byte (determinismo);
injetar 30 cenas com queries iguais → 0 violações na conferência.

---

## ITEM 7 — Política temporal ÚNICA (construtor e juiz param de discordar)

**Fato:** já EXISTE faixa especial na construção (`montar_timeline.py:24`:
primeiros 40s alvo 2,6s/teto 4s; resto alvo 4,5s/teto 6s) — a premissa "cadência
uniforme" da v1 estava ERRADA. Os furos reais: (a) o **merge de cenas curtas não
revalida o teto** após fundir (montar_timeline.py:121) — candidato causal da cena de
6s+ na abertura; (b) o fixador de cadência usa alvo uniforme 12 e metades ≥2,2s
(montar_timeline.py:215); (c) o juiz (`_decupagem_video.py`) cobra 12 cortes/min
uniforme — construtor e juiz usam números DIFERENTES; (d) passes posteriores
(reconciliador) mudam fronteiras sem revalidação final.

**Mudanças:**
1. **`politica_temporal(t)` única** (função exportada; provavelmente em `leis.py` ao
   lado das janelas): devolve `(alvo, piso, teto)` por faixa. Faixas (ordem do
   operador): 0-60s → alvo 4s, piso 3s, **teto 5s DURO**; 60-180s → teto 6s com piso
   de cadência maior; >180s → regra atual. Números finais calibrados na bancada
   contra o piso de cortes (Codex: teto 7s é incompatível com 15 cortes/min — máx
   ~8,6 inícios/min; os pares alvo×piso têm que fechar aritmeticamente).
2. `montar_timeline` consome a política em TODOS os pontos (targets, merge, fixador
   de cadência) e o **merge revalida o teto** — fusão que estoura é proibida.
3. `_decupagem_video` (F2) deriva o piso de cortes POR FAIXA da mesma função.
4. **Validação final pós-reconciliador**: depois de todos os passes estruturais, um
   check de política (mesma função) — construção não é invariante, entrega é.
5. **Teto F12 dinâmico**: `teto = max(2, ceil(inserções_da_família / variantes_elegíveis))`
   — [v3] âncoras corretas: produtor real = `edicao.py:643-649` + passe F12 em
   `edicao.py:1490-1509` (a :139 é `_PRIO`); juiz = `_decupagem_video.py:230`. Ambos
   consomem a MESMA função. Elemento longo na abertura: encurtar/deslocar antes de
   bloquear divisão (sem conflito com item 2).
6. [v3] Números CONCRETOS de todas as faixas (nada de "calibrar depois"): 0-60s =
   alvo 4s / piso 3s / teto 5s, F2 13 cortes/min; 60-180s = alvo 4,5s / piso 3s /
   teto 6s, F2 12/min; >180s = alvo 4,5s / piso 2,2s / teto 6s, F2 12/min (regra
   atual formalizada). A aritmética fecha: teto 5s permite 12+/min com mix de 3-4s.
7. [v3] O RECONCILIADOR consome a política: `TETO_VEO=7.5` (reconciliar_cenas.py:45,
   441, 472, 575, 650, 681) vira `min(7.5, teto_da_faixa(t))` — sem isso ele cria
   cena acima do teto duro na abertura.
8. [v3] A validação final é BLOQUEANTE: entra como lei em `leis.py` E o
   `aprovar_render` REPROVA com ela (hoje aprovar_render.py:112-127 só registra).

Classificação: FONTE (a discordância construtor×juiz deixa de existir).
Bancada: gerar timeline sintética → 0 cenas acima do teto por faixa APÓS
reconciliador; decupagem da timeline aprovada não reprova F2/F12.

---

## ITEM 8 — Paralelismo VEO×BANCO (portar o ganho da v9 para a cadeia)

**Fato:** todos os fechos rodam `veo_gerar` → `resolver_cascata` em SÉRIE
(`_fecho_orca_v2.py:180-181`; idem v12/v13/esteira). O paralelismo da v9 (33,7 min)
vivia na orquestração daquela rodada. Decisões são disjuntas (cenas veo_* excluídas
do stock/reuse-fill — resolver_cascata.py:1838), mas **escritas NÃO são**: ambos
fazem read-modify-write do `timeline.json` inteiro. E existe base pronta:
`_rodar_paralelo.py` (workspaces isolados, :73) — cujo merge por `idx` (:91) HOJE
perderia cenas (idx duplicados) e escreve timeline parcialmente mesclada antes de
validar rc.

**Mudanças:**
1. Reusar `_rodar_paralelo.py`: cada ramo (VEO / resolver) roda em WORKSPACE ISOLADO
   com snapshot da timeline (hash do snapshot base registrado).
2. **Merge de TRÊS VIAS por `scene_id`** (depende do item 2): base + ramo VEO + ramo
   banco; allowlist de campos por ramo (VEO escreve `clip_*`/`fonte` das cenas veo_*;
   resolver escreve os das cenas banco + `fonte_alvo`/`fonte_motivo` das realocações);
   conflito, ID faltando ou sobrando = FALHA (nada de merge parcial).
3. **Os dois processos precisam terminar com rc=0 antes de tocar a timeline
   principal** (hoje o merge legado escreve antes de validar).
4. **Realocações tardias banco→VEO** (`resolver_cascata.py:2578`, cena perdida
   marcada para VEO) entram no merge ANTES do `veo_recolher_perdidas` — é a
   dependência real entre os ramos (Codex, pergunta 5).
5. Junção: `veo_recolher_perdidas` (com o contrato do item 4) roda DEPOIS do merge.
6. [v3] Allowlist COMPLETA por ramo: VEO escreve `clip_path, clip_id, clip_dur,
   media_tipo, nivel, fonte` das cenas veo_* (veo_gerar.py:120-121); resolver escreve
   os mesmos + `real_query, pexels_id, fonte_alvo, fonte_motivo, fonte_alvo_original`
   das cenas banco (resolver_cascata.py:2318-2320, :2592-2595).
7. [v3] **PROMOÇÃO ATÔMICA dos artefatos**: os caminhos gravados pelo ramo apontam
   para o workspace isolado (`vmconfig.dir_job` ancora em TESTE → `_ws_banco/_jobs/...`)
   e a L11 REPROVARIA (leis.py:364-404). Após o merge: mover mídia + estado
   (decision_manifest.json, index_cascata.json, stock/, veo/, _prompts_cenas.json,
   retratos) para a raiz do job principal e REESCREVER os caminhos na timeline; só
   então publicar (o `_rodar_paralelo.py:113-126` atual copia só prompts+retratos).

Classificação: FONTE (a corrida deixa de existir por isolamento, não por sorte).
Ganho esperado: ~35 min. Bancada: merge com conflito fabricado → FALHA sem tocar a
principal; realocação tardia presente no resultado; rc!=0 de um ramo → principal intacta.

---

## ORDEM DE IMPLEMENTAÇÃO (Codex, rodada 1 — dependências explícitas)

1. **Item 1** (captura) — pré-requisito de diagnóstico de todos.
2. **Item 2** (scene_id + pop) — pré-requisito do merge (item 8) e da decupagem.
3. **Item 4** (contrato de erro do recolher).
4. **Item 5** (adaptadores + gate) — pré-requisito do árbitro (as listas ranqueadas
   nascem aqui).
5. **Item 6** (árbitro determinístico).
6. **Item 7** (política temporal única).
7. **Item 8** (paralelismo com merge 3 vias).
8. **Item 3** entra junto com o 2 (mesmos arquivos; a função de janelas é pequena).

Cada item: implementar → bancada offline → diff ao Codex → próximo. Commits atômicos
por item no repo do Vidmator.

## ACEITE DO E2E (espécie nova, bioma novo, VM_MODO=fresh)

| Métrica | Orca (12/08) | Aceite |
|---|---|---|
| Total | 154,7 min | **≤ 60 min** neste E2E; 30 min é a meta após item 8 estabilizar |
| Resolver | 81,9 min | ≤ 12 min |
| Conferência (ex-pós-loop) | ~73 min | ~0 violações, < 2 min |
| Downloads | 135 arq / 25 usados | baixados ≈ usados (± vet reprovado) |
| Cenas sem clipe | 9 | 0 (ou rc=3 EXPLÍCITO no diário) |
| scene_id duplicado | — | 0 (idx duplicado SÓ entre irmãos derivados) |
| Determinismo do árbitro | — | mesma entrada 2× = atribuições idênticas (bancada) |
| Ramos paralelos | — | merge só com rc=0 dos dois; assets na raiz do job (L11 verde) |
| L01/L03 sobre elementos da cena | cegas | acusam na timeline da orca (regressão-teste) |
| Morte muda | 2 ocorrências | impossível (log por tentativa + tail) |
