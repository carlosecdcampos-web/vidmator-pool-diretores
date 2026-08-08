# ESPEC-V8 — as correções do veredito do v7 PIRANHA, ancoradas no código real

> Veredito do operador (07/08, assistindo o `piranha_v7.mp4`): **"PARABÉNS PARA NÓS,
> ficou muito, muito bom"** · *"NENHUM take apareceu com CENA+número"* · *"as inserções
> de texto ficaram ótimas"* · *"a citação do Theodore Roosevelt, gosto muito desse
> estilo"* · **"esse é o ÚNICO ERRO"** (SFX/edição durante a abertura) + a cor
> monocromática global. Esta espec corrige exatamente o que ele citou — nada além.

---

## V8-1 · A LEI DA COR — vídeo COLORIDO por padrão; monocromático é ARMA PONTUAL

**Palavras dele**: *"o vídeo ficou quase preto e branco, e deveria ter cor normal.
GOSTO DESSE EFEITO MONOCROMÁTICO PARA ALGUNS PONTOS DE MAIS TENSÃO — traz dinamismo,
mais um efeito de edição para usarmos como arma para nosso editor... ainda mais quando
se tratar de alguma parte histórica, que aconteceu muitas décadas atrás. Mas em takes
pontuais; um vídeo inteiro assim não."*

**CAUSA RAIZ (medida)**: [preparar_render.py:765](remotion/preparar_render.py#L765)
```python
render["periodo"] = "vintage" if (_preset.get("vintage") or (_anos and min(_anos) <= 1970)) else None
```
UM ano ≤1970 em qualquer ponto do roteiro (aqui: **1913**, Roosevelt) liga `periodo=
"vintage"` GLOBAL → [BrollTest.tsx:774-778](remotion/src/compositions/BrollTest.tsx#L774)
aplica `grayscale(0.9) sepia(0.18) contrast(1.12)` em **toda cena** (`periodFilter`
entra no `vfilterDe` de todas), mais grão/vinheta do `PeriodOverlay`. Uma anedota
histórica pintou o documentário inteiro de época.

**A LEI**:
1. `periodo` global **morre** como gatilho automático (o knob `_preset.vintage`
   continua existindo para o futuro "diretor treinado para isso" — decisão explícita,
   nunca inferência por ano).
2. Nasce **`cena.look_vintage` POR CENA** — dono ÚNICO: o passe **`epoca`**
   (que já é dono de `presentacao` etc. em
   [_prova_idempotencia.py:88](director/_prova_idempotencia.py#L88); `look_vintage`
   entra no mapa OWNED dele — idempotência garantida por reconstrução). Marcado
   APENAS na **janela histórica**: cenas cujo texto narra o evento de décadas atrás
   (régua: a janela do card de `datas` ± cenas do mesmo trecho, tipicamente 2-5).
3. **Arma de tensão**: o diretor de edição ([edicao.py](director/edicao.py)) ganha o
   direito de marcar `look_vintage` em 1-2 picos de tensão do vídeo (mood `tenso`,
   mesmo predicado do pacote "cena só filtro" de
   [BrollTest.tsx:779-791](remotion/src/compositions/BrollTest.tsx#L779) — que já
   garante janela sem elemento por cima). Teto: **≤10% das cenas** com look mono no
   vídeo inteiro; violou → o preparar corta os excedentes (barreira, não estilo).
4. Render: [BrollTest.tsx:774](remotion/src/compositions/BrollTest.tsx#L774) troca
   `const bw = timeline.periodo === "vintage"` por leitura POR CENA
   (`c.look_vintage`), aplicando `periodFilter` só nelas — e o
   `{bw ? <PeriodOverlay /> : null}` global de
   [BrollTest.tsx:876](remotion/src/compositions/BrollTest.tsx#L876) (grão/vinheta)
   passa a montar DENTRO da Sequence da cena marcada, nunca sobre o vídeo todo. A
   apresentação `film` (que já é P&B por conta própria via
   [apresentar.py:29](director/apresentar.py#L29)) segue intocada — ela é moldura de
   época legítima.

**Bancada obrigatória**: piranha re-preparada sob a lei → contar cenas com filtro
mono no render json: esperado ~4-8 (janela Roosevelt + até 2 picos), **nunca 96**.

---

## V8-2 · A LEI DA ABERTURA — a edição começa DEPOIS dela ("o único erro")

**Palavras dele**: *"logo no início fala de um ano, dá para escutar o SFX de número,
mas a tela é a abertura com o take escrito AMAZON... é como se o vídeo tivesse sido
montado e a abertura colocada por cima. A edição/SFX deve acontecer APÓS A ABERTURA."*

**CAUSA RAIZ (medida no render json do v7)**: com a abertura cobrindo `0 → 5,78s`
(+fade 1,5s), entraram por baixo dela:
- card de **`datas` a t=0,95s** (o "1913" — é o SFX de número que ele ouviu);
- **mapa a t=4,20s**.
Cada passe agenda seus elementos olhando só a fala; **nenhum** conhece a janela da
abertura. O diagnóstico dele está literalmente certo: a edição é montada e a abertura
é sobreposta ([BrollTest](remotion/src/compositions/BrollTest.tsx) renderiza a
abertura como camada superior — os elementos e SFX continuam tocando embaixo).

**A LEI** (na BARREIRA do [preparar_render.py](remotion/preparar_render.py), junto de
V1-V6 — roda em 100% dos pipelines, inclusive reuso):
1. Janela morta = `0 → abertura.fim + abertura.fade_s` (campos já existentes;
   sem abertura → lei no-op).
2. TODO elemento com instante inicial dentro da janela morta — `datas`, `mapas`,
   `acervo_texto`, `imagens`, `enumeracoes`, `ctas`, kinetic/infográfico via
   `aparece_em`, e os SFX atrelados a eles — é **EMPURRADO** para
   `abertura.fim + fade + 0.3s`, com DUAS condições (senão **DERRUBA com log alto**,
   `V7: N elemento(s) movido(s)/removido(s) da janela da abertura`):
   · a âncora ainda faz sentido (a cena/janela de fala ainda cobre o novo instante);
   · **teto de atraso ≤4s** em relação ao instante FALADO original — elemento que
     chegaria mais de 4s atrasado vira ruído fora de hora (o "1913" a 8s de distância
     da fala confunde mais do que ajuda): derruba, não empurra.
3. Denominação: **V7 da barreira** (segue a numeração V1-V6 existente).
4. Transição de saída da abertura é a única coisa audível/visível permitida no
   cruzamento (o `fade_s` já cuida).

**Bancada obrigatória**: re-preparar o v7 → `datas@0,95` tem que virar `datas@7,58+`
e o mapa@4,20 mover/dropar; zero elementos na janela morta no render json.

---

## V8-3 · Fluxo VEO consolidado (o que o v8 roda de ponta a ponta)

Ordem dele, já toda em lei — o v8 é a primeira rodada 100% limpa:
1. **Projeto NOVO por produção** ([veo_projeto_novo.py](_import/2026-08-06-veo-flow/veo_projeto_novo.py)).
2. **TODAS as imagens → depois TODOS os vídeos**
   ([veo_ciclo.py](_import/2026-08-06-veo-flow/veo_ciclo.py) `--tipo ambos`: passe
   imagem → passe vídeo, `garantir_modo` com Nano Banana Pro confirmado no chip).
3. **Download UM A UM, nunca coleção/zip**
   ([veo_download.py](_import/2026-08-06-veo-flow/veo_download.py) `baixar_por_cena`,
   número lido NO TEXTO do card; o zip está morto no código — commit `eb08e2b`).
4. **Curadoria canônica** ([veo_curador.py](_import/2026-08-06-veo-flow/veo_curador.py)):
   só **"CENA + número"** reprova (Vision transcreve, regex decide; texto didático/
   livro/prancheta fica) → regenera 1× → veio queimada de novo → **entra pronta que
   sirva** → **usa TODO o resto**. Zero geração para buracos.
5. **Prompts**: espécie EXATA + bioma travado explícitos no cabeçalho
   (`SPECIES:`/`BIOME:` em [veo_pedido.py](_import/2026-08-06-veo-flow/veo_pedido.py),
   GUIA de 11 leis + `_validar_prompt` fiscal + rodízio de câmera + 1-em-4 atmosfera).
   *"se está falando de Amazônia, ele não vai meter uma girafa"* — a lei 2 é
   literalmente essa, e o fiscal reprova antes de enviar.
6. **CENA+número MANTIDO** no prompt (decisão 07/08): casamento exato e legibilidade
   pro operador valem o ~10% de queima, que o curador já paga sozinho. Observação de
   futuro: se o Flow permitir nomear card separado do prompt, migrar e zerar o custo.

---

## V8-4 · EXPURGO do pool + sidecar com identidade de canal

**Fato do v7**: tubarões/baleias apareceram porque os 14 takes de prompt contaminado
entraram no `_cache_stock` com sidecar carregando a `busca_original` da cena de
piranha (o filtro "contém piranha" passou). Além do expurgo pontual:
1. **Expurgo**: remover do pool todo asset cujo sidecar (`.veo.json`) tenha prompt de
   espécie marinha/fora-de-bioma da leva de 07/08 00:39-00:45 (whale/shark/ocean/
   crocodile-in-ocean) — script `_expurgo_pool.py`, com lista impressa antes de apagar.
2. **Sidecar ganha identidade**: `canal` + `bioma` gravados na entrega
   ([veo_entrega.py](_import/2026-08-06-veo-flow/veo_entrega.py)) — o socorro passa a
   filtrar por canal/bioma ANTES da régua de espécie (asset de outro canal nunca é
   "pronta" de ninguém).

---

## V8-5 · Blindagens estruturais (a lição dos zumbis — prioridade máxima)

1. **Workspace-por-job (E18, promovido a OBRIGATÓRIO)**: cada produção em
   `_workspace/<job>/`; até lá, **lock de dono**: `workspace_lock.json` com
   `{job, pid, ts}` escrito pelo runner; a checagem mora no CHOKE POINT
   **`producao.run()`** ([producao.py](director/producao.py) — todos os runners passam
   por ele; foi o heartbeat dele que entregou o zumbi `{"pass": "render"}` do v6b):
   job diferente do dono do lock → **aborta alto ANTES de rodar o passe**. Execuções
   manuais avulsas (`python passe.py`) ficam fora do lock — aceito e documentado;
   o E18 completo (workspace-por-job) fecha esse resto.
2. **Kill por ÁRVORE, nunca por padrão de nome**: `taskkill /T /F` no PID do runner
   (registrado no lock/heartbeat); memória `feedback-taskstop-nao-mata-filhos`
   atualizada com o caso `_rodar_pos_resolver`.
3. **Gate do ciclo perde o `text-card`** ([veo_driver.py](_import/2026-08-06-veo-flow/veo_driver.py)
   `_aprovado` / [vision_gate.py](_import/2026-08-06-veo-flow/vision_gate.py)): no v7
   ele reprovou 3 cenas LIMPAS (28/32/38 — falso positivo com quadro escuro) e deixou
   passar as queimadas de verdade. **Um juiz por crime**: off-topic/child/talking-head
   ficam no gate; texto queimado é exclusivo do curador.

---

## 🏆 PADRÃO CANONIZADO — o card do Roosevelt (elogio do operador)

*"gostei muito de como ele fez o elemento para citar o Theodore Roosevelt, fundo de
grid e tal... gosto muito desse estilo: a pessoa de um lado e o nome do outro."*

É o **[PersonCard.tsx](remotion/src/compositions/PersonCard.tsx)**: retrato recortado
à esquerda, NOME em display à direita com subtítulo (PRESIDENT 1858-1919), **grid de
fundo** sutil ([linha 44](remotion/src/compositions/PersonCard.tsx#L44), `gridC` com
variante clara/escura), consumido em
[BrollTest.tsx:732](remotion/src/compositions/BrollTest.tsx#L732) a partir do passe
[pessoas.py](director/pessoas.py). **Fica registrado como padrão-referência**: toda
citação de pessoa histórica usa este layout; NÃO mexer no componente (está aprovado
como está).

---

## Ordem de execução do V8

1. V8-4 expurgo do pool (5 min — higiene antes de qualquer reuso futuro).
2. V8-2 Lei da Abertura na barreira (é "o único erro" — primeiro).
3. V8-1 Lei da Cor (global morre; janela por cena + arma de tensão + teto 10%).
4. V8-5.1 lock de dono do workspace (barato; E18 completo pode vir depois).
5. V8-5.3 gate sem text-card.
6. Bancadas: re-preparar o v7 e provar V8-1/V8-2 nos números (sem re-render).
7. **v8 fresh** (animal a definir pelo operador) com o fluxo V8-3 completo.

**Decisões do operador embutidas (nenhuma pendente)**: mono pontual ≤10% ✓ · edição
só pós-abertura ✓ · download 1-a-1 ✓ · CENA+número mantido ✓ · PersonCard intocável ✓.

*Criada 07/08/2026, ancorada no código real e no render entregue (`piranha_v7.mp4`).*

---

## ✅ EXECUTADA 07/08/2026 — bancada VERDE (números medidos no re-preparo do v7)

- **V8-4**: expurgo aplicado — **10 assets fora** (2 lixos determinísticos + 4 tubarões,
  2 jubartes, 1 peixe-de-recife, 1 clownfish/Nemo); bisturi da bancada POUPOU 3 botos
  cor-de-rosa e 1 arraia potamotrygon (fauna amazônica real — "dolphin/stingray" só
  caem com ambiente marinho na descrição do Vision). **5 cenas do timeline v7**
  (s004/s015/s050/s054/s057 — onde os tubarões apareciam) re-apontadas para prontas
  limpas. **133 sidecars** com retrofit `nicho/especie/bioma`; entrega grava identidade
  em todo sidecar novo; socorro filtra bioma ANTES da espécie (mesa 5/5, incl.
  congo≠amazon).
- **V8-2**: V7 na barreira. Re-preparo do v7: **ZERO elementos na janela morta**
  (0→7,28s). `datas@0,95` ("October 1913") DERRUBADA por atraso >4s — era o SFX que o
  operador ouviu; `mapa@4,20` DERRUBADO pela guarda anti-colisão (empurrado bateria no
  PersonCard @10,34); 2 SFX de troca de cena sob a abertura silenciados. PersonCard do
  Roosevelt e data "2012"@159 intactos.
- **V8-1**: `periodo` global = None; **4/96 cenas mono** (3 históricas na janela do
  Roosevelt [s0-s2] + 1 pico de tensão [s78 @367,3s]) — teto 9 (10%), nunca 96.
  2ª rodada de epoca+edicao reconstrói estado idêntico (idempotência provada na prática).
  Blindagem extra na execução: **uma arma por cena** — cena com look mono sai do pacote
  de wash vermelho (doutrina "não sobreponha overlays" do operador).
- **V8-5.1**: lock em `producao.py` (`reivindicar`/`liberar`/`_conferir_lock` em
  `run()` E `render()`; `fazer_job` reivindica). Memória do TaskStop atualizada com o
  caso `_rodar_pos_resolver`. **V8-5.3**: `text-card` filtrado no `_aprovado` do driver
  (cobre veo_driver e veo_ciclo — mesmo chamador).
- Guarda extra descoberta na execução (5ª blindagem): **empurrão que cria colisão de
  cards DERRUBA** — sem ela, o mapa empurrado empilharia no PersonCard.
- **Sem re-render por design**: a prova visual (cor pontual + abertura limpa) aparece
  no próximo render (v8, ou re-render do v7 sob demanda — o workspace já está pronto).

## V8-6 · LEI DO ÂNGULO & DIREÇÃO (adendo pós-espec — VALIDADA AO VIVO 07/08)

**Palavras dele**: *"os takes de piranha, mesmo diferentes, PARECIAM MUITO ENTRE SI —
a piranha nadava sempre para o mesmo lado. Ele trabalhou movimentos de câmera
diferentes, sim, mas deveria ter trabalhado ÂNGULO, PERSPECTIVA... prompts
consecutivos pareciam o mesmo take com leves diferenças."*

**Causa**: o rodízio fiscalizado só existia para o MOVIMENTO (regra 7); ponto de
vista era a regra 5 "soft" (sem lista, sem fiscal) → o modelo caía no default
(perfil, mesmo lado). **A lei** (commit `69ded79`, mesma arquitetura da câmera):
regra 12 do GUIA — 8 ângulos + 4 direções em lista fechada, declaração obrigatória
quando o animal está no quadro, e o fiscal de sequência rodizia as TRÊS dimensões
(movimento, ângulo, direção — nunca o mesmo em 2 shots consecutivos).

**Bancada ao vivo** (15 fotos consecutivas de piranha, s026-s041, projeto T15ANGULO):
fiscal reprovou 5 tentativas e o autor corrigiu; 15/15 sem fallback, zero repetição
consecutiva. Operador acompanhou a geração em tempo real: **"estão ficando muito
boas, muito mesmo, exatamente como deveriam ser."** ✅ LEI VALIDADA.
