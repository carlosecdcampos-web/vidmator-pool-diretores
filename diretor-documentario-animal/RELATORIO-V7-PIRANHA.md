# RELATÓRIO DE PRODUÇÃO — v7 PIRANHA (06/08/2026)

> Primeira produção FRESH com o funcionário do VEO contratado (`VM_VEO=1`).
> Ordem do operador: *"monte um monitor que esteja 100% atento ao trabalho, catalogue
> cada log... pode ir corrigindo simultaneamente caso dê algum erro, o importante é o
> job não morrer, catalogue logs de erro como fez para corrigir e instruções para
> blindar caso aconteça outra vez."*
> Este arquivo é a matéria-prima da ESPEC pós-job.

## Ficha do job

| Campo | Valor |
|---|---|
| Nome | `piranha_v7` |
| Título | **PIRANHA: The Deadliest Secret of the Amazon Rivers** (4ª opção, fundindo as 3 do operador) |
| Roteiro | `_workspace/roteiro_piranha.txt` — 1211 palavras (urso: 1187 → 454s) |
| Voz | `ashtar_voice` (Chatterbox) |
| Nicho | `doc_realista` · modo `fresh` |
| Knobs | `VM_VEO=1` · `VM_VEO_CANAL=PIRANHA` · `VM_VEO_COLECAO=piranha-v7` |
| Projeto Flow | `c18e71d4-6665-4145-8f74-e6492fde08b1` (**NOVO**, ordem: 1 projeto por produção) |
| Commit base | `89786da` |

### Conteúdo do roteiro (o "educar de verdade" pedido)
Fatos verificáveis, todos com fonte real: expedição **Roosevelt (out/1913)** e o
espetáculo ENCENADO que criou o mito (rio bloqueado, peixes famintos por dias, a vaca)
· dentes em **lâmina contínua** trocados em **quadrantes** (o peixe nunca fica banguela)
· **mordida medida em campo** (~320 N num exemplar de ~1 kg ≈ 30× o próprio peso, a
mais forte por tamanho já medida num peixe ósseo) · **cardume como estrutura defensiva**
(o peixe do meio é o mais seguro) · **lepidofagia / comer nadadeiras** (a presa sobrevive
e o "prato" se regenera) · **três sons** produzidos contra a bexiga natatória · **pulso
de cheia × seca** como a real janela de mordidas em humanos · papel **sanitário** no
ecossistema · fecho: *tudo que Roosevelt viu era verdade; toda conclusão tirada dali era
falsa.*

---

## Blindagens que ENTRARAM antes do disparo (aprendidas na prova geral)

| # | Blindagem | Motivo |
|---|---|---|
| B1 | `veo_projeto_novo.py` — projeto novo por produção, id registrado em `projetos.json` | ordem do operador + evita mistura de coleções entre vídeos |
| B2 | Coleção identificada por **ID**, rename só cosmético | o rename da UI do Flow não confirma no Enter e BLOQUEAVA o ciclo |
| B3 | Porta do pool desembrulha **ZIP disfarçado** de mp4/png | o Flow entrega mídia dentro de zip; o pool comeria lixo |
| B4 | **Rede de segurança** no `veo_gerar` (reuso de emergência + log gritado) | cena VEO sem take = buraco preto = reprova no D5 e mata o job |
| B5 | `vision_gate` traduzido para o container | faltava na frota; o ciclo morria em `ModuleNotFoundError` |

---

## Log de execução (cronológico)

| Hora | Etapa | Estado |
|---|---|---|
| 20:0x | projeto Flow criado (`c18e71d4`) | ✅ |
| 20:0x | pipeline disparado (`_rodar_e2e producao_jobs_piranha.json`) | ✅ |
| 20:1x | narração Chatterbox/ashtar: **457,3s** (7min37), loudnorm -14, 561s de geração | ✅ sem loop (guarda de ratio passou) |
| 20:1x | deloop (E20): nenhum loop detectado | ✅ |
| 20:1x | whisper 11s · montar_timeline 71s → **96 cenas** | ✅ |
| 20:1x | **E1 em produção real**: inícios/min = `[17,12,12,12,12,12,12,7]` (último = minuto parcial de 37s) | ✅ **nenhum minuto cheio abaixo de 12** — era o F2 que reprovou os dois v6b |
| 20:1x | contrato_visual 113s (29 cenas com sujeito obrigatório) · enumeracoes 18s (2 enumerações) | ✅ |
| 20:2x | **veo_alocacao (lei da divisão) em produção real**: 38 banco / 35 veo_video / 23 veo_imagem = **39,6% / 36,5% / 24,0%** · **10 cenas de anatomia pinadas** · fita intercalada `B V I B V I B V B V I…` | ✅ cravado na lei |
| 20:5x | resolver_cascata 2176s — buscou stock SÓ das 38 do banco (C2 funcionando; com 96 cenas teria levado ~2,5× mais) | ✅ |
| 21:4x | **veo_gerar 2939s (49 min)** — o funcionário trabalhando sozinho: 49 mídias geradas e baixadas (30 vídeos + 19 imagens), **5 rodadas de vídeo + 3 de imagem** | ✅ |
| — | **vision_gate**: 25 vídeos aprovados de primeira, **5 reprovados e re-gerados** (voltaram aprovados), **0 desistidos**; 19/19 imagens aceitas | ✅ o gate do gerado trabalhou |
| — | **veo_socorro**: 13 cenas resolvidas por REUSO de asset que serve à narrativa (4 do banco sem match + 9 de geração que não fechou) | ✅ a lei do socorro em ação real |
| — | **rede de segurança: NÃO precisou** (0 cenas) — nenhuma emergência | ✅ |
| — | **timeline final: 0 órfãs** · 34 banco · 49 VEO gerado · 13 VEO reúso · 36 vídeos + 26 imagens do VEO | ✅ |
| 21:4x | detectar_mapas: 0 mapas (roteiro sem lugar nomeado com país) · pessoas: 1 card (Roosevelt) | ✅ A4 sem gatilho aqui |

## Erros encontrados → causa → blindagem

### E-01 · Plano com 35 erros → lock em QUARENTENA (o job seguiu, mas ia entregar defeito)

`pre_render_report`: `limpo=False | erros=35 | avisos=63`. Autópsia item a item:

**(a) 17 erros FALSOS — "tem sujeito obrigatório mas resolveu por [vazio]: caminho de
IDENTIDADE foi pulado"**
· CAUSA: o gate de identidade foi feito para STOCK DE TERCEIROS (foi ele que pegou a
"leoa africana num doc da Austrália"). Cena do VEO nunca passa pelo resolver — logo não
tem `nivel=ID-*` — e o relatório a acusava. Mas o take fabricado **nasce com a espécie
exata no prompt** (lei do take nº1) e ainda passa pelo `vision_gate` no ciclo.
· BLINDAGEM (aplicada): `pre_render_report` pula o gate de identidade para cena com
`fonte` começando em `veo` — *a identidade do fabricado é a fabricação; o sidecar
`.veo.json` é a prova.*

**(b) 3 erros OBSOLETOS — `cena 21/41/54: semantic_status=HARD_MISS / estado=degraded`**
· CAUSA: vêm do `decision_manifest` do resolver, que congela o estado ANTES da fase VEO.
As três foram cobertas por take fabricado depois (hoje têm `fonte=veo` + clip).
· BLINDAGEM (para a espec): a fase VEO precisa **reabrir o manifesto** e regravar
`estado/semantic_status` das cenas que cobriu (hoje o manifesto mente para frente).

**(c) 15 erros REAIS e graves — repetição de CONTEÚDO** ⚠️ *o relatório estava certo e
eu quase os descartei como falso positivo*
· FATO medido: 13 cenas (11,13,19,27,35,37,39,44,50,57,59,62,92) tinham **arquivos com
nomes diferentes e bytes IDÊNTICOS** (mesmo md5 completo, mesmo tamanho: 7.118.591 b).
Mais 2 grupos vindos do VEO: uma imagem em 4 cenas e um vídeo em 3.
· CAUSA RAIZ (dupla, mesma doença — **identidade por NOME, não por CONTEÚDO**):
  1. **Stock**: o mesmo vídeo do banco é baixado sob nomes/ids diferentes por queries
     diferentes; a cota de reuso (`max_usos_por_fonte`) compara `pexels_id`/nome → nunca
     viu que era o mesmo material.
  2. **VEO**: o casamento do zip por TÍTULO entrega o **mesmo card** para prompts
     parecidos (as fotos de cota são quase gêmeas em texto) → arquivos `sNNN` distintos
     com bytes iguais → o pool (que nomeia por md5) colapsa tudo num asset só.
· BLINDAGENS (aplicadas):
  1. `veo_entrega`: **1 ASSET = 1 CENA** — conteúdo (md5) já entregue a outra cena é
     RECUSADO; a cena vira órfã e cai na lei do socorro.
  2. `veo_socorro`: passa a respeitar o **orçamento do portão** (máx 2 usos do mesmo
     conteúdo, distância ≥7 cenas), com contagem viva durante o próprio socorro.
  3. **PENDENTE para a espec (fonte do stock)**: dedup por CONTEÚDO na porta do
     `_cache_stock` + chave de orçamento = md5 do arquivo. Sem isso o resolver volta a
     servir 13 cópias do mesmo clipe em outro vídeo.
· REPARO DESTE VÍDEO: 15 cenas liberadas do conteúdo duplicado → socorro (já
  orçamento-aware) reusou 8 que servem à narrativa → **7 foram para geração sob medida**
  no Flow (coleção `piranha-v7-soc`), exatamente como a lei manda.

**Decisão de condução**: o render defeituoso foi INTERROMPIDO no meio (ele ia entregar
13 cenas com o mesmo clipe). Não é "o job morreu" — é o job voltando um passo para
entregar certo; a narração, os 96 cortes, os 49 takes gerados e todos os passes foram
preservados (só `preparar` + `render` serão refeitos).

### E-02 · CATÁSTROFE DOS ZUMBIS (00:36) — workspace da piranha sobrescrito pelo v6b

**FATO**: dois processos `_rodar_pos_resolver` (re-render do v6b dos PREDADORES,
jobs_v6b.json) ficaram vivos e invisíveis durante toda a produção da piranha e, às
00:36, seu fluxo re-escreveu o WORKSPACE (narração/words/timeline/payload remotion/
timeline_render.json) com o documentário errado. Heartbeat os entregou:
`{"pass": "render", "unidade": "50%"}` — estavam RENDERIZANDO o v6b em paralelo.
Sintomas na superfície: prompts de reposição com CACHALOTE/BALEIA-AZUL/"open ocean"
num vídeo de PIRANHA; regeneração em loop de cenas boas.

**CAUSA-RAIZ (dupla)**:
1. Meus kills usavam padrão de NOME (`_rodar_e2e|render|veo_ciclo`) — `_rodar_pos_resolver`
   nunca casou com nenhum. Lição TaskStop-não-mata-filhos, agora em dose dupla.
2. **WORKSPACE ÚNICO** é ponto único de falha: qualquer processo escreve
   `_workspace/timeline.json` sem dono, sem lock de JOB. Duas produções = roleta.

**BLINDAGENS para a espec (prioridade máxima)**:
- **E18 workspace-por-job** (já estava no backlog — vira OBRIGATÓRIO): cada job em
  `_workspace/<job>/`, passes recebem o diretório; colisão vira impossível.
- **Lock de dono no workspace**: `workspace_lock.json` com PID+job; passe que não é
  o dono ABORTA alto (barato de implementar antes do E18 completo).
- Kill SEMPRE por árvore (`taskkill /T /F`) e por lista de PIDs registrada no
  heartbeat, nunca por padrão de nome.

**RECUPERAÇÃO** (o que salvou o dia): `_narr_chunks/*.wav` intactos → narração
reconstruída **bit-perfeita em duração (457,3s)** → mesmo áudio = mesmas cenas
determinísticas → os 65 assets `sNNN` continuam válidos; `_gerar.json` (prompts
certos) + `decision_manifest` (stock em modo reuso) + pool com sidecars completam.
14 mídias geradas com prompt contaminado foram DESCARTADAS e re-pedidas com o
timeline restaurado. Runner: `director/_recuperar_piranha.py`.

### E-03 · Zip do projeto PERDE o "CENA NNN" (o Flow re-titula o arquivo)

O card mostra "CENA 015" na UI, mas o arquivo do zip vem como
"Amazon_basin_riverbed_roots_tilting_<ts>.mp4" — o número NUNCA chega ao nome.
Consequência: casamento por número via zip é impossível; o de similaridade colidia
em prompts parecidos (cards trocados: «CENA 020» dentro de s028.mp4 etc.).
**BLINDAGEM**: rota de download EXATA = vista em lista, número lido NO TEXTO do card
(`_fechar_lote.py`); zip fica só como fallback de volume com dedup por conteúdo.

### E-04 · Curador de texto queimado — calibragem final do operador

Regra dele: **só reprova "CENA + número"**; qualquer outro texto (livro, prancheta,
rótulo didático M1, placa da cena) é LEGÍTIMO. Vision transcreve; regex local decide.
Teste cego: achou as 4 do operador (015/020/050/094 — e provou que 3 estavam em
ARQUIVOS trocados) + extra (080). Gate `text-card` do ciclo dá falso positivo e
NÃO pega o que importa → na espec: tirar text-card do gate, curador é o juiz único.


---

## 🌙 ESTADO AO FIM DA NOITE (07/08 ~02:30) — PRONTO PARA O GO DE RENDER

| Item | Estado |
|---|---|
| Timeline piranha | ✅ reconstruído (96 cenas, mesmas fronteiras — determinismo provado) |
| Elenco de mídia | ✅ 96/96: banco resolvido + 67 takes VEO prontos + buracos com PRONTAS (fluxo canônico, zero geração extra) |
| Curador (CENA+número) | ✅ **66/66 limpos — zero regeneração necessária** |
| Erros do plano | ✅ 4 reais consertados com prontas (reuso de conteúdo 4×/distâncias) — restam só os 10 obsoletos do manifesto (lista branca) |
| Lock | ✅ **auto-aprovado com exceções (SEM quarentena)** — plano `2e3e2009af51589e` |
| Payload de render | ✅ `timeline_render.json` gerado (457,34s · 96 cenas · barreira V1-V6: nenhuma violação) |
| Render | ⏸ **PAUSADO por ordem** (`_workspace/PAUSA_RENDER`) — GPU/CPU livres p/ o Automator |

**GO de amanhã** (1 comando): apagar `_workspace/PAUSA_RENDER` e rodar
`producao.render("piranha_v7")` — ~40 min → MP4.
⚠️ NÃO rodar nenhum pass antes do render (o timeline está selado no plano do lock).


## 🏁 ENTREGA (07/08 07:58)

**`D:\AC Creator HUB\Vidmator\_saida\piranha_v7.mp4`** — 457,4s (7min37) · 1920×1080
· 640 MB · render em **14 min** (840s) · lock limpo, FORA da quarentena.
Spot-check visual: apresentação "film" com moldura na época Roosevelt (P&B), take VEO
do rio alagado com word-pop "Amazon river flood pulse" — elementos e takes no lugar.
