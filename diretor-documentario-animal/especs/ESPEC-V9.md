# ESPEC-V9 — a produção lateralizada + tudo que quebrou no v8

> **Escopo**: como o pipeline RODA — arquitetura de execução, ordem dos passes,
> e o catálogo COMPLETO dos defeitos da produção do v8 SERPENTES (07-08/08/2026).
> Banco de dados/acervo tem espec própria: `ESPEC-ACERVO-DRIVE-A-REFINAR.md`.
> Cada item aqui nasceu de um defeito OBSERVADO em execução real, com evidência.
> Nada implementado — é a lista de trabalho da v9.

---

# PARTE I — A LATERALIZAÇÃO (a arquitetura alvo)

## V9-1 · DOIS AGENTES QUE SAEM DO ESCRITÓRIO SABENDO SUAS CENAS

**Ordem do operador (literal, 08/08):**
> *"A divisão 40/60 deve ser feita ANTES DO ENVIO DOS PROMPTS. Todas as cenas
> catalogadas de acordo com o roteiro/SRT, cenas são divididas, são enviadas para
> os dois agentes diferentes — o do banco de estoque e o do VEO — AMBOS SABENDO DE
> QUAIS CENAS SÃO RESPONSÁVEIS. Eles não dependem do trabalho um do outro, pq eles
> já saem do escritório sabendo qual cena vão buscar."*
>
> *"Esperar um terminar para começar outro, cujo trabalho de cada um é
> independente, é perda de tempo, não é um trabalho otimizado."*

**Custo de não ter feito isso no v8**: VEO ~45 min + resolver ~40 min em SÉRIE.
Em paralelo seria `max(45, 40)` — **~40 min jogados fora num único vídeo.**

**A base JÁ EXISTE**: `veo_alocacao` roda antes de tudo e carimba cada cena com
`fonte_alvo` (banco / veo_video / veo_imagem). No v8: `102 cenas | {banco: 41,
veo_video: 37, veo_imagem: 24}`. Não falta informação — falta disparar em paralelo.

**Por que é seguro**: os conjuntos são **DISJUNTOS**. Nenhuma cena pertence aos dois
agentes, logo não existe corrida de conteúdo. O merge é união de conjuntos separados
(`idx` do VEO ∪ `idx` do banco = todas), determinístico e auditável.

**Desenho:**
```
                    ┌─ AGENTE VEO    (37 vídeo + 24 imagem) ─┐
alocação (40/60) ──►┤                                        ├──► merge ──► socorro
   [já existe]      └─ AGENTE BANCO  (41 cenas de stock)  ───┘   (idx)     (última rede)
```

**Disciplina obrigatória**: cada agente grava o SEU arquivo (`timeline_veo.json` /
`timeline_banco.json`); um passe de merge junta ao fim. **Nunca** os dois salvando
o mesmo `timeline.json` (aí sim vira corrida de escrita).

**Pré-requisito — ⚠️ CORRIGIDO NA REVISÃO (a 1ª versão desta espec ERRAVA aqui):**
o `resolver_cascata` **JÁ respeita a alocação**. Verificado em
[resolver_cascata.py:1611-1619]: com `VM_VEO=1` e timeline com `fonte_alvo`, ele
REMOVE as cenas `veo_video`/`veo_imagem` da busca e do reuse-fill ("C2/VM_VEO: N
cena(s) alocadas ao VEO ficam FORA da busca de stock"). Ou seja, **o lado do banco
já está pronto para o paralelo** — não há correção a fazer aqui.

O que falta é só a ORQUESTRAÇÃO: hoje `veo_gerar` e `resolver_cascata` são dois
passes chamados em sequência pelo runner. Basta dispará-los concorrentes (dois
processos), com escrita em arquivos separados + merge — o filtro de escopo que
tornaria isso seguro **já existe dos dois lados** (`veo_pedido` filtra `veo_*`
desde o v8; `resolver_cascata` filtra `banco` desde o C2).

**Extensão futura** (depende do acervo): enquanto o vídeo N renderiza, o agente já
prepara o cache do vídeo N+1 da fila — lateralização entre jobs, não só dentro.

## V9-2 · SOCORRO É A ÚLTIMA REDE — a barreira de junção

Com a lateralização, o socorro deixa de ser "mais um passe" e vira o **ponto de
encontro**: só roda quando VEO **e** banco terminaram. Ele cobre o que sobrou; não
compete com quem ainda está trabalhando.

**Prova do v8**: rodando ANTES do resolver, ele encheu **30 cenas de BANCO com
reuso de takes VEO** — o vídeo sairia ~90% VEO contra os 40/60 definidos. Precisou
limpar as 30 à mão e reordenar.

**⚠️⚠️ REVISÃO 08/08 — A CULPA ERA MINHA, NÃO DO PIPELINE.** (E a 1ª versão desta
espec acusava o código de produção — errado duas vezes.)

A ordem CANÔNICA, em [_rodar_v2abertura.py:47-56], é:
```
veo_alocacao → resolver_cascata → veo_gerar(pedido→ciclo→entrega→socorro) → ...
```
**O resolver vem ANTES do veo_gerar.** Logo, quando o socorro roda, as 41 cenas de
banco JÁ têm `clip_path` — não são órfãs, e ele nem as enxerga. O pipeline de
produção está correto e sempre esteve.

Quem inverteu foi o **meu runner do v8** (`_v8_retomar.py`), que chamou
`veo_gerar` antes do `resolver_cascata`. O defeito das 30 cenas foi consequência
direta disso. Erro de processo meu, catalogado em P2.

**MAS a correção continua NECESSÁRIA — e agora por outro motivo, mais forte:**
com a lateralização (V9-1), VEO e banco passam a rodar **ao mesmo tempo**. A ordem
deixa de ser garantida por construção: o socorro (que vive dentro do `veo_gerar`)
pode alcançar cenas de banco que o resolver ainda não terminou de preencher.

Hoje o socorro é seguro **por acidente de ordenação**. Sob paralelismo, precisa ser
seguro **por escopo**:
```python
# veo_socorro.py:133 — hoje
orfas = [c for c in cenas if not c.get("clip_path")]
# proposto: respeitar a alocação, como o resolver já faz (C2)
_veo = [c for c in cenas if str(c.get("fonte_alvo") or "").startswith("veo")]
orfas = [c for c in (_veo or cenas) if not c.get("clip_path")]
```
Duas linhas, simétricas ao filtro que o `resolver_cascata` já tem. Sem elas, o
paralelismo REINTRODUZ o bug que hoje não existe.

---

# PARTE II — CATÁLOGO DE DEFEITOS DO v8 (com evidência e status)

## A · O PIPELINE NÃO SABIA FAZER RANKING (multi-espécie)

Todos derivam da mesma raiz: o pipeline nasceu para documentário de UM animal
(v7 piranha). No v8 (5 serpentes), tudo que assumia "uma espécie / um lugar" quebrou.

| # | defeito | evidência | status |
|---|---|---|---|
| A1 | **Lei 2 travava UMA espécie global** (a mais citada) em todos os prompts | 17 de 20 prompts viraram "reticulated python"; mamba e víbora = 0 | ✅ corrigido no v8 (espécie POR CENA + detecção MULTI) |
| A2 | **`documentary_setting` genérico entrava literal** no prompt | "in its global snake habitats" — cenário que não existe | ✅ corrigido (`_habitat_de`: savana p/ mamba, arrozal p/ víbora…) |
| A3 | **Pedido ignorava a alocação** e o `--max` default (20) truncava | pegava as 102 cenas e cortava nas 20 primeiras → **o #1 e o #2 do ranking nunca eram pedidos** | ✅ corrigido (filtro `fonte_alvo` + max 300) |
| A4 | **Grade do fallback 100% aquática** (herança da piranha) | "gills working" numa cobra; "riverbed", "underwater" | ✅ corrigido (grade neutra + ângulo/direção) |
| A5 | **Adjetivo regional único** para doc multi-região | `contrato_visual` pediu "o adjetivo deste lugar" para `global snake habitats` → LLM devolveu **"Amazonian"** (com jaguar e caimão no glossário); o resolver buscava **"Amazonian Black Mamba"** e o poster-gate cortava 3/4 e 4/4 | ❌ **PENDENTE** — foi onde a produção parou |

### 🔧 A5 — o conserto, verificado no código (revisão 08/08)

**Rota completa do defeito**, confirmada linha a linha:
1. [contrato_visual.py:249-289] monta `gloss = {setting, adjetivo, termos}` e pede
   ao LLM *"o adjetivo regional DESTE lugar"*. Com `setting = "global snake
   habitats"` o LLM chutou **"Amazonian"** e encheu `termos` com fauna amazônica
   (jaguar, caimão-preto) — factualmente errado para 4 das 5 serpentes;
2. grava em `tl["glossario_regional"]` [linha 286];
3. [resolver_cascata.py:1715] o resolver carrega esse glossário;
4. [resolver_cascata.py:726-752] `ancorar_regional(q)` prefixa o adjetivo em toda
   query que não cite o setting nem uma das espécies "concretas" do glossário.
   Como as serpentes NÃO estavam nas `termos` (que tinham fauna amazônica), TODA
   query de mamba/víbora virou **"Amazonian Black Mamba"** — daí o poster-gate
   cortando 3/4 e 4/4.

**Conserto mínimo (duas guardas, nenhuma reescrita):**
- **No `contrato_visual`**: quando o `setting` for genérico/multi-região (contém
  `global`, `worldwide`, `various`, `several`, ou o roteiro tem ≥2 espécies
  protagonistas — a mesma detecção `MULTI` que o `veo_pedido` já usa), **não pedir
  adjetivo ao LLM**: gravar `adjetivo: ""`. Um ranking mundial não tem adjetivo.
- **No `resolver_cascata`**: `ancorar_regional` já é no-op quando não há `setting`
  nem `adjetivo` [linha 734] — mas o `setting` genérico continuaria entrando como
  âncora. Adicionar a mesma guarda: setting genérico ⇒ devolver a query intacta,
  porque quem ancora a cena é o **contrato de espécie dela**, não o lugar do doc.

**Por que isso basta**: cada cena do v8 já tem `required_subjects` correto
(medido: reticulated python 13, Russell's Viper 13, King Cobra 12, Black Mamba 11,
anaconda 12). A busca por espécie exata já é específica — o adjetivo global só
poluía. Sem ele, "Black Mamba savanna" busca o que existe.

**Lei geral**: quando o documentário é multi-espécie/multi-região, todo passe que
pergunta *"qual é A espécie / O lugar"* precisa perguntar **por cena** — ou não
perguntar. Auditar também: `inaturalist`, `abertura` (nome do lugar),
`detectar_mapas`, `epoca`.

## B · ESTADO COMPARTILHADO ENTRE PRODUÇÕES (contaminação)

| # | defeito | evidência | status |
|---|---|---|---|
| B1 | **`veo_job/assets` reutilizado entre jobs** (E18) | **54 takes da PIRANHA** ocupavam slots `sNNN` do v8: 8 imagens de piranha entrariam no vídeo e 16 vídeos nunca seriam gerados (o ciclo os deu como "prontos" pela mera existência do arquivo) | ⚠️ contornado à mão (`veo_job_v8/`) — **falta virar lei** |
| B2 | **`decision_manifest.json` do job anterior** vivo no workspace | manifesto do v7 (07/08 02:15) — o resolver reaproveitaria stock de piranha nas cenas de cobra | ⚠️ neutralizado à mão (`.bak`) |
| B3 | **Pool `_cache_stock` plano e global** | 133 assets de outros vídeos disponíveis ao socorro; barrados só por régua de TEXTO (que o v7 provou ser enganável: 14 sidecars diziam "piranha" com vídeo de tubarão) | 📋 espec própria (acervo/Drive) |

**Lei que falta**: `_workspace/<job>/` completo — assets, manifesto, `_gerar*.json`,
lock. O acervo é a ÚNICA exceção legítima, e com escrita curada.

## C · O CICLO VEO / DRIVER DO FLOW

| # | defeito | evidência | status |
|---|---|---|---|
| C1 | **Coleção se multiplica** | rename é cosmético e falha → card vira "Coleção sem título" → `abrir_colecao` procura pelo NOME, não acha, **CRIA OUTRA a cada rodada**. No v8 nasceram ~6; as mídias ficaram espalhadas; na rodada 2 o ciclo morreu (`botão '+' não encontrado`) | ⚠️ contornado (`_colher_colecoes.py` recuperou 27/27) — **causa exata na revisão abaixo** |
| C2 | **Organização correta a garantir** | operador: *"1 COLEÇÃO = 1 VÍDEO. O projeto é do diretor."* — a intenção do código já é essa; o rename quebrado é o que impede | ❌ pendente |

### 🔎 CAUSA EXATA do C1 (revisão 08/08) — o ID registrado é código MORTO

[veo_colecao.py, `abrir_colecao`] lê o ID logo no início:
```python
cid = (reg.get("colecoes") or {}).get(nome)     # linha 8 da função
```
…e **nunca mais o usa para achar a coleção**. A busca é feita só por RÓTULO
(`_label_da_colecao(page, nome)`), e na linha 43 o `cid` é SOBRESCRITO com o id
lido da URL depois de já ter entrado. Ou seja: o registro `nome → id` existe, é
gravado, e é inútil na hora que mais importa.

Encadeamento completo do defeito:
1. rename cosmético falha → card fica "Coleção sem título";
2. `abrir_colecao("teste 02 - serpentes")` procura esse rótulo → não acha;
3. `criar_se_faltar=True` → **cria outra coleção**;
4. repete a cada rodada do ciclo → N coleções órfãs, mídias espalhadas.

**Correção (a mínima que funciona)**: quando houver `cid` registrado, entrar pelo
card e **conferir se a URL bate com o `cid`**; se bater, é a coleção certa mesmo
com rótulo genérico. Só criar nova quando NÃO houver `cid` no registro — nunca
por rótulo ausente. Guarda extra: `criar_se_faltar` deve ser `False` por padrão
nas rodadas 2+ do mesmo ciclo (a coleção já foi criada na rodada 1).
| C3 | Falso positivo do lock de ciclo | o `python.exe` de venv no Windows é um LAUNCHER que spawna o interpretador real como filho → 2 pids com `veo_ciclo` → a varredura se auto-detectava e abortava TODO ciclo (3 vezes) | ✅ corrigido (exclui a própria árvore) |
| C4 | Espelho **mídia-primeiro** não acha nada | 0/15 em DUAS tentativas: texto e mídia são subárvores irmãs; subir da mídia nunca encontra o bloco de prompt | ✅ corrigido (texto-primeiro, provado 20/20) |
| C5 | Rota por URL da coleção = **estado zumbi** | URL certa, conteúdo errado (sem lista nativa) → download rodava no vazio | ✅ corrigido (entrada SEMPRE pelo card) |
| C6 | **Escape às cegas = VOLTAR** | dentro da coleção, `Esc` devolve ao projeto — era o "atualizou e voltou" que o operador via | ✅ corrigido (Esc só com dialog/menu aberto) |
| C7 | Clique do ⋮ pega o card **vizinho** | 3 pares de conteúdo idêntico por hash em 15 downloads (~20%) | ✅ resolvido pela rota linha-a-linha (zero cliques) |
| C8 | `scroll_into_view` em nó desmontado | timeout 6s em série na lista virtualizada | ✅ corrigido (rolagem incremental) |

## D · NARRAÇÃO

| # | defeito | evidência | status |
|---|---|---|---|
| D1 | **Guarda anti-loop mede a COISA ERRADA** (ver autópsia abaixo) | ratio >1,45 em **21/21 chunks** → 3 regenerações cada → narração levou **594s** em vez de ~150s, com o áudio CERTO (479,5s / 1.242 palavras = 155 wpm; transcrição 1237/1242 = razão 1.00) | ❌ pendente — **trocar o critério**, não calibrar |

### 🔴 AUTÓPSIA DO D1 (revisão 08/08 — a 1ª versão desta espec errava a causa)

Eu havia escrito *"a fórmula de duração esperada está calibrada para outra voz"*.
**Não existe fórmula de duração esperada.** O código real:

```python
# chatterbox_runner.py:265
"ratio": round(audio_duration / gen_time, 2)
```

`ratio` = **duração do áudio ÷ tempo de geração** — isto é, o **fator de tempo real**
(velocidade), não a razão entre áudio produzido e áudio esperado.

E o `narrar.py` trata `ratio > 1,45` como *"provável LOOP"*. Consequência lógica:
**quanto mais RÁPIDO o Chatterbox gera, mais o guarda acusa loop.** A própria memória
do operador registra que a velocidade normal é **~2,7× tempo real** — ou seja, o
guarda está condenado a acusar SEMPRE que a GPU estiver livre. No v8: 21/21 chunks.

O guarda nunca funcionou como anti-loop; ele funcionou como *detector de GPU rápida*.
E é pior que inútil: **triplica o custo da narração** (3 regenerações por chunk) e
some com o áudio bom, já que a última tentativa é a que fica.

**Correção**: apagar a checagem de ratio do `narrar.py` (não calibrar — o número não
mede o que se quer). O detector real de loop já existe e é TEXTUAL: `_deloop_audio`,
que roda depois do whisper e compara palavras. No v8 ele confirmou razão 1.00 em 0,2s.
Se quiser um guarda na geração, o critério certo é
`audio_duration` vs `len(texto)/chars_por_segundo_da_voz` — que é uma métrica que
hoje **não é calculada em lugar nenhum**.

## E · PEDIDO — pendências menores

- `_prompt_deterministico` ainda gera ambiente genérico em cena sem espécie
  ("a Earth wilderness global landscape") — 1 defeito em 61;
- anatomia sem brief ainda usa a espécie GLOBAL (`especie_doc`), não `_esp_cena(c)`;
- 6 fallbacks em 61 (10%): o fiscal reprovou 3 rodadas — provavelmente exigindo
  ângulo/direção em cenas de AMBIENTE, onde a lei não deveria se aplicar.

---

# PARTE III — ERROS DE PROCESSO (meus, não do código)

Registrados porque custaram mais tempo que os bugs.

| # | erro | custo real |
|---|---|---|
| P1 | **Mandei VEO e banco em SÉRIE depois de o operador ter decidido o paralelo** — cheguei a escrever a ideia na espec e executei o contrário | ~40 min no v8 + a raiva justa do operador |
| P2 | **Deixei o socorro rodar antes do resolver** | 30 cenas de banco viradas em VEO; limpeza manual |
| P3 | **Confiei em "arquivo existe = está pronto"** sem conferir a data | quase entregou piranhas no vídeo de serpentes |
| P4 | **Lancei ciclo por cima de ciclo vivo** (a notificação de conclusão nunca tinha chegado = ele ainda rodava) | prompts no projeto errado, 4 processos disputando o mesmo Chrome |
| P5 | **Misturei espec de acervo com espec de fluxo** | retrabalho de documentação |
| P6 | **Estimativas chutadas** ("5-8 min por take de vídeo" quando o Veo faz em ~1 min) | ruído na comunicação; corrigido por medição |

**Regra que sai daqui**: decisão do operador é lei imediata, não item de espec. Se
ele decidiu paralelo, o próximo run é paralelo — não se escreve "a refinar".

---

# PARTE IV — ORDEM DE IMPLEMENTAÇÃO (revisada 08/08 pelo custo/benefício real)

| # | item | esforço | por quê nessa ordem |
|---|---|---|---|
| 1 | **A5** — suprimir adjetivo em setting genérico | 2 guardas | **bloqueia a retomada do v8**; sem isso metade do vídeo busca errado |
| 2 | **D1** — apagar a checagem de ratio | 1 remoção | maior ganho por linha de todo o documento: devolve ~7 min e 2/3 da GPU por vídeo, e o detector certo (`_deloop_audio`) já existe |
| 3 | **V9-2** — guarda de escopo no socorro | 2 linhas | **pré-requisito do paralelismo** (hoje o socorro é seguro só por ordenação) |
| 4 | **C1** — usar o `cid` registrado | ~10 linhas | mata a multiplicação de coleções e o "botão + não encontrado" |
| 5 | **B1/B2** — workspace por job | médio | tira a contaminação da classe "sorte" |
| 6 | **V9-1** — lateralização | maior | ~40 min/vídeo; depende do item 3 |
| 7 | **A-geral / E** — auditoria multi-espécie + pendências | contínuo | |

**Nota de sequência**: o item 6 (paralelismo) **exige** o item 3. Fazer o 6 sem o 3
reintroduz o defeito das 30 cenas que hoje não existe.

---

# PARTE V — REVISÃO CONTRA O CÓDIGO REAL (08/08, madrugada)

Cada afirmação da 1ª versão foi conferida no código. **Três estavam erradas** — e
duas delas culpavam o pipeline por erro meu:

| item | o que a 1ª versão dizia | o que o código mostra |
|---|---|---|
| **V9-1 pré-req** | *"o `resolver_cascata` pega toda cena sem clip_path, precisa filtrar"* | ❌ **ERRADO**: ele JÁ filtra desde o C2 [resolver_cascata.py:1611-1619] — o lado do banco está pronto para o paralelo |
| **V9-2** | *"o bug é do código de produção"* | ❌ **ERRADO**: a ordem canônica é `resolver → veo_gerar` [_rodar_v2abertura.py:47-56], então o socorro nunca vê cena de banco órfã. **Quem inverteu foi o meu runner do v8** |
| **D1** | *"a fórmula de duração esperada está mal calibrada"* | ❌ **ERRADO**: não existe tal fórmula. `ratio = audio_duration / gen_time` é **velocidade** [chatterbox_runner.py:265] — o guarda acusa loop quanto mais RÁPIDA a GPU |
| **A5** | causa correta, conserto genérico | ✅ confirmado + rota completa mapeada (4 saltos) e conserto reduzido a 2 guardas |
| **C1** | *"reconhecer pelo ID registrado"* | ✅ confirmado + **causa exata achada**: o `cid` É lido e depois **sobrescrito sem nunca ser usado** — código morto |
| **B1** | workspace compartilhado | ✅ confirmado no código de produção: [veo_gerar.py:39] `assets = TESTE / "veo_job" / "assets"` — hardcoded |

**Verificações que NÃO acharam problema** (registradas para não repetir o trabalho):
- o lado VEO não consome nada que o resolver produza: `stock_query` nasce em
  `montar_timeline:354` e é refinada em `contrato_visual:232`, ambos ANTES dos dois
  agentes ⇒ **a independência do paralelismo é real, não presumida**;
- `glossario_regional` só é lido pelo `resolver_cascata` ⇒ o conserto do A5 não tem
  efeito colateral em outros passes;
- `fonte_alvo` é consumido por 5 arquivos (`veo_alocacao`, `veo_pedido`,
  `veo_socorro`, `resolver_cascata`, runner) — todos previstos nesta espec.

**Lição de método**: três dos meus diagnósticos vinham de leitura por memória, não do
código. A doutrina Sherlock Holmes do CLAUDE.md manda o contrário: evidência direta
antes de hipótese. Uma espec com causa errada é pior que nenhuma — ela faz consertar
o lugar errado com confiança.

---

*Aberta em 08/08/2026 durante a produção do v8 SERPENTES, atualizada a pedido do
operador para conter TODOS os erros do vídeo — inclusive os meus.*


---

# PARTE VI — GARANTIAS VERIFICÁVEIS DO TESTE v9 (pedido do operador, 08/08)

O operador zerou os assets do v8 (24 imagens + 37 vídeos, 578 MB) e o
`clip_path` das 61 cenas, **preservando** narração, `words.json`, `timeline.json`
(com a alocação) e `_gerar.json`. O teste do v9 parte daí e tem de provar 4 coisas.

## G1 · A DIVISÃO POR COTA JÁ ESTÁ PRONTA — ✅ verificado

O `timeline.json` preservado já traz `fonte_alvo` em toda cena:
`{banco: 41, veo_video: 37, veo_imagem: 24}` = 102. **Nenhum passe precisa
recalcular** — os dois agentes saem do escritório sabendo suas cenas porque a
alocação está gravada, cena a cena, desde o v8.

**Como verificar antes de rodar:**
```python
Counter(c.get('fonte_alvo') or 'banco' for c in tl['cenas'])
# esperado: {'banco': 41, 'veo_video': 37, 'veo_imagem': 24}
```

## G2 · OS DOIS AGENTES SIMULTÂNEOS

Cada um já filtra o próprio escopo (verificado na PARTE V): `veo_pedido` pega só
`fonte_alvo=veo_*`; `resolver_cascata` pega só `banco` (C2). Falta a orquestração:

```
alocação (pronta) ──┬─► AGENTE VEO   (61 cenas) ──► timeline_veo.json   ──┐
                    └─► AGENTE BANCO (41 cenas) ──► timeline_banco.json ──┴─► merge por idx ─► socorro
```

**Pré-requisito inegociável**: a guarda de escopo no `veo_socorro` (V9-2). Sem
ela, o socorro do agente VEO alcança cenas de banco que o outro ainda não
preencheu — o defeito das 30 cenas volta, agora por corrida em vez de ordem.

**Como verificar que funcionou:** os dois processos vivos ao mesmo tempo, e o
tempo total ≈ `max(VEO, banco)` e não `VEO + banco`. Baseline do v8 em série:
~45 min + ~40 min. Alvo: ~45-60 min.

## G3 · UMA COLEÇÃO POR VÍDEO — o algoritmo

**Regra do operador (literal):** *"1 coleção = 1 vídeo. Independentemente de ter 5
grupos de animais diferentes com 6 grupos de vídeos diferentes, elas pertencem ao
mesmo vídeo, então deverão estar todas juntas dentro de uma MESMA COLEÇÃO."*

**Por que quebrou no v8**: `abrir_colecao` é chamado **4+ vezes por rodada**,
dentro de `for tipo in (imagem, vídeo)` × até 6 rodadas — dezenas de chamadas. Como
o rename falha e a busca é só por rótulo, **cada chamada que não acha cria outra**.

**Algoritmo proposto (mínimo que resolve, sem sistema novo):**
1. **Um único ponto de criação por execução**: flag em memória do processo
   (`_colecao_criada`). A 1ª chamada pode criar; da 2ª em diante, **criar é
   proibido** — não achou, é erro, aborta alto. Isso sozinho já limita a UMA
   coleção por execução do ciclo.
2. **Entre execuções, manda o `cid` registrado**: hoje ele é lido e descartado
   (código morto — ver C1). Passa a ser a identidade: entra pelo card e confere se
   a URL contém o `cid`; se contiver, é a coleção certa, mesmo com rótulo
   "Coleção sem título".
3. `criar_se_faltar=False` em toda chamada que não seja a primeira do ciclo.

**Como verificar:** ao fim do teste, o projeto `DIRETOR VEO` deve ter
**exatamente UMA coleção nova** com as 61 mídias dentro (24 imagens + 37 vídeos,
das 5 serpentes). Duas coleções = falhou.

## G4 · TODOS OS TAKES GERADOS — gate de completude

No v8 o ciclo morreu na rodada 2 (`botão '+' não encontrado`) e ficou 10/37 —
descobriu-se só porque eu estava olhando. **Isso não pode depender de vigilância.**

**Falta um GATE explícito** ao fim do `veo_gerar`:
```
entregues = cenas com fonte_alvo=veo_* E clip_path preenchido
se entregues < total_veo:
    log ALTO com a lista exata das faltantes  →  NÃO seguir para os passes
```
Hoje o pipeline segue com buracos e o socorro tapa com reuso — o que mascara a
falha de geração. O gate transforma "faltou take" de silêncio em parada honesta.

**Como verificar:** ao fim, `61/61` cenas `veo_*` com `clip_path`, e as 5
espécies presentes (a checagem de distribuição do v8: víbora, anaconda, píton,
mamba, cobra-real — nenhuma zerada).

---

## RESUMO DO QUE PRECISA SER IMPLEMENTADO ANTES DO TESTE

| # | item | onde | tamanho |
|---|---|---|---|
| 1 | A5 — suprimir adjetivo em setting genérico | `contrato_visual` + `resolver_cascata` | 2 guardas |
| 2 | V9-2 — guarda de escopo no socorro | `veo_socorro.py:133` | 2 linhas |
| 3 | G3 — uma coleção (flag + cid) | `veo_colecao.abrir_colecao` | ~15 linhas |
| 4 | G4 — gate de completude | fim do `veo_gerar` | ~8 linhas |
| 5 | V9-1 — disparar os dois em paralelo + merge | runner | ~30 linhas |
| — | D1 (ratio) e B1 (workspace por job) | `narrar.py` / `veo_gerar` | fora do caminho crítico deste teste, mas baratos |

Sem os itens 1-4, o teste do paralelismo mede a coisa errada: pode "funcionar" e
ainda assim entregar coleção multiplicada, take faltando ou busca com adjetivo errado.

---

# PARTE VII — RODADA DE BLINDAGEM PRÉ-GO (08/08, manhã)

Verificação dos 3 bloqueadores declarados ("G3 nunca tocou o Flow", "merge nunca
rodou", "G4 pode ser rígido demais") — e o que a verificação REVELOU de novo.

## V7-1 · G4 verificado por leitura — posição CORRETA ✅
O gate roda DEPOIS de tudo: ciclo (6 rodadas internas) → socorro + re-ciclo →
2 voltas do curador → rede de emergência. Não aborta prematuramente. E distingue
`reuse-fill-emergencia` (não conta) de `veo_reuso` do socorro (conta, é lei).

## V7-2 · 🔴 DEFEITO NOVO: a regeneração do curador era um NO-OP
**FATO (rastreado no código):** o curador `--aplicar` zerava só o timeline. O take
queimado continuava vivo em TRÊS cópias, cada uma uma porta de retorno:
- `assets/sNNN.*` → o ciclo pula a cena ("arquivo existe", veo_ciclo faltam)
- `_espelho/<prompt>.*` → o casamento re-copia o velho (colheita prévia)
- `pool/veo<h>.*` → o socorro reusa como candidato (`pool.glob("veo*.*")`)
Resultado: "regenerar" devolvia o MESMO take queimado e a 2ª reprovação empurrava
pro socorro. **Fix: morte tripla no `--aplicar`** (assets + gêmeo do espelho por
md5 + pool se nenhuma outra cena referencia). veo_curador.py.

## V7-3 · 🔴 CAUSA RAIZ DO G3: o card de coleção é um `<a>` de TAMANHO ZERO
Sondas de DOM ao vivo (08/08):
1. O card É `<a href=".../collection/<cid>">` — dá pra mirar DIRETO pelo cid.
2. MAS o `<a>` tem **w=0 h=0** (`display:inline`, conteúdo absoluto):
   `click()` do Playwright NUNCA passa no actionability (timeout) e clique por
   coordenada cai num DIV. **Só o `.click()` NATIVO (JS) dispara o router da SPA.**
3. A varredura às cegas antiga (8 primeiros cards) morria na pilha de 9 órfãs.
**Fix em `_entrar_por_cid`:** localizar por `a[href*=cid]` (paciência de 30s — o
grid demora a montar no Chrome frio) + clique nativo JS + confirmação por URL.
Rota por cid agora vem ANTES da varredura de rótulo no `abrir_colecao`.
**PROBE G3: PASS ao vivo** (entrou em 'teste 02 - serpentes' com 9 órfãs na frente).

## V7-4 · 🔴 DEFEITO NOVO: o driver escolhia o PERFIL sozinho (conta errada)
2 probes falharam porque `perfis.primeiro_livre()` escolheu `conta1` — outra conta
Google, projeto invisível, página "vazia" (GUIA §1 avisa LITERALMENTE isso e manda
guardar `canal → {projeto_id, perfil_path}`; nunca tínhamos implementado). Pode ter
contribuído para a morte da rodada 2 do v8 (10/37). **Fix:** campo `perfil` no
projetos.json por canal + `fd.abrir(perfil=...)` no ciclo e probes.

## V7-5 · 🔴 SEQUELA DO v8: o RENAME pegou o TÍTULO DO PROJETO
O projeto "DIRETOR VEO" amanheceu chamado "teste 02 - serpentes" (print do
operador) — uma tentativa de rename acertou o campo editável do TOPO. **Fixes:**
`_campo_de_renome` e `_label_da_colecao` agora ignoram y<100 (barra do app).
PENDENTE: renomear o projeto de volta na mão (2 cliques, mais seguro que automação).
Sobraram ~9 coleções órfãs "sem título" no projeto (conteúdo do v8 já colhido);
limpeza manual pelo 🗑 do card quando o operador quiser.

## V7-6 · Coleção ÚNICA de verdade (lei do operador)
`veo_gerar` criava coleções `-soc`/`-curN` por rodada (v7 PIRANHA: 4 coleções).
Removido: TODAS as rodadas geram na MESMA coleção. Seguro porque o espelho é
incremental (_srcs.json) e o curador agora mata as 3 cópias do reprovado.

## V7-7 · Paralelo blindado
- Rede de emergência do `veo_gerar` agora respeita `fonte_alvo` (no paralelo as
  cenas de banco estão vazias DE PROPÓSITO no workspace do VEO).
- `_preparar` do runner WIPEIA o workspace antes de copiar (anti-E18).
- Pós-merge lista buracos explicitamente.

## V7-8 · TESTE SECO (08/08 12:19-12:29) — ✅ PASS COMPLETO
3 cenas do timeline v8 (0=banco, 2/5=veo_imagem), fluxo 100% real:
- Coleção "teste 03 - dry v9" criada UMA vez, renomeada, cid 58f7f6da registrado;
  re-entrada mid-ciclo pela rota nova do href (entrada zumbi detectada e curada).
- VEO: fiscal 3 rodadas → 2 prompts → rajada → espelho 2/2 → casamento 2/2 →
  entrega → socorro 0 órfãs (guarda V9-2 ignorou a cena de banco ✓) → curador 2
  limpos → **gate G4 2/2** → rc=0 em **3,6 min**.
- BANCO: resolver 51 candidatos julgados, ID-vid na cena 0, rc=0 em **10,4 min**
  (heartbeat batendo o tempo todo — lento é rede, não travamento).
- **Merge: veo 2 + banco 1 = 3/3 com mídia. Tempo real 10,4 min = o MAIS LENTO,
  não a soma (~14 min em série).** A lateralização paga.

## V7-9 · DOUTRINA DO TESTE v9 COMPLETO (ordem do operador, 08/08)
**Estreia virgem:** mesmo roteiro serpentes, VEO regenera TUDO — nada produzido
anteriormente pelo VEO entra (nem da coleção do Flow, nem do disco).
- Coleção NOVA dentro do projeto do canal (DIRETOR VEO).
- Pool: TODOS os `veo*` (v8 + dry) movidos para `_cache_stock/_v8_quarentena/`
  ANTES do teste → socorro acha 0 candidatos VEO → órfã vira GERAR.
- Reuso DENTRO da própria rodada v9 continua valendo (é produção do teste).
- Stock (Pexels/acervo) intacto — a ordem é sobre material do VEO.
- Paralelismo VEO×BANCO confirmado pelo operador como a arquitetura desejada
  ("agente VEO e BANCO está perfeito, é isso que eu quero").

---

# PARTE VIII — ESTEIRA DE PRODUÇÃO EM ESCALA (visão do operador, 08/08 — A AMADURECER, não implementar hoje)

Ordem do operador (adiantada para informar o desenho):

> "quero fazer a pasta de cache POR VÍDEO, como a pasta do Automator que é criada
> automaticamente com cada vídeo. Depois que o render for produzido e APROVADO,
> excluir a pasta daquele vídeo — e não um cache geral. Na fila de produção, com
> mais contas logadas no VEO rodando produções em contas diferentes, enquanto um
> vídeo é produzido os funcionários do PRÓXIMO já preparam os assets dele; quando
> o vídeo 1 termina o render, os assets do vídeo 2 já estão prontos, e os agentes
> já pegam os do vídeo 3. Se tiver tudo junto, os assets de um vídeo contaminam
> o do próximo."

## O desenho que isso pede (2 camadas, não 1)

O `_cache_stock` de hoje mistura DUAS coisas de vida útil diferente:

| material | vida útil | destino no desenho |
|---|---|---|
| **Takes VEO** (`veo*`) | UM vídeo (gerados p/ cenas de um roteiro; reuso cruzado = contaminação) | **pool POR VÍDEO** (`_pool/<video_id>/`), excluído após render APROVADO (zelador) |
| **Stock** (Pexels/downloads) | PERMANENTE (paisagem de rio serve em 50 vídeos) | **ACERVO compartilhado** (o Drive da ESPEC-ACERVO) + cópia de trabalho por vídeo |

É o MESMO desenho que o operador já tinha esboçado para o acervo Drive ("o agente
vai no drive, o que servir ele trás para o _cache_stock POR VÍDEO") — as duas
ideias se encaixam: acervo global = banco permanente; pasta por vídeo = bancada
de trabalho descartável.

## Pré-requisitos já mapeados
- **B1 (workspace por job)** — já especificado; a pasta-por-vídeo é a extensão
  natural dele (workspace + pool nascem e morrem JUNTOS com o vídeo).
- **Lock por CONTA, não global**: hoje o `ciclo.lock` é um-ciclo-por-máquina;
  com N contas VEO, vira um-ciclo-por-PERFIL (produções paralelas em contas
  diferentes não se bloqueiam, mas 2 ciclos na MESMA conta sim).
- **Zelador**: exclui a pasta do vídeo APÓS aprovação do render (gatilho =
  aprovado, não = renderizado).

## Estado de 08/08
- Quarentena v8 (392 arquivos, 1 GB) EXCLUÍDA — v9 estreia com pool sem nenhum
  take VEO pré-existente.

## V7-10 · DEFEITOS ACHADOS PELO PRÓPRIO TESTE v9 (08/08 tarde) — os 3 estágios do lote grande
O teste seco (3 cenas) validou arquitetura mas NÃO TEM TAMANHO para expor defeito
de varredura. O run real (102 cenas) expôs três, todos consertados no ato:
1. **Rename → barra de prompt** (13:26): a barra é textbox SEM atributo placeholder;
   o rename digitou nela e GEROU imagem com o nome da coleção. Fix: campo de rename
   só por identificação POSITIVA (pré-preenchido com o nome atual) + criação
   cid-first (diff de hrefs antes/depois; rótulo fora do caminho crítico de vez).
2. **Espelho desiste antes do fim da lista** (13:46 — operador: "estavam PRONTAS
   e o espelho achou 0"): 8 roladas secas × ~1s não atravessam a zona de itens já
   baixados nem esperam a linha virtual MONTAR. 13 imagens prontas invisíveis →
   reenvio → gêmeas na coleção. Fix: varrer do TOPO, orçamento 20, pausa 1.4-2.0s.
3. **Espera lê zero falso** : badge de % fora da viewport não conta (grid
   virtualizado — GUIA §7 aplicado ao envio mas violado na espera). Fix: o zero só
   vale se sobreviver a reload + recontagem.
Custo dos defeitos: gêmeas de imagem na coleção (tempo do servidor, zero dinheiro);
nenhuma imagem perdida (24/24, zero desistidos). Doutrina reafirmada: teste de
escala é OBRIGATÓRIO antes de declarar fluxo pronto — 3 cenas não compram 102.

---

# PARTE IX — RESULTADO DO TESTE v9 (run 7, 08/08 14:59) ✅

**GATE G4: 61/61 gerados próprios · merge 100/102 · exit 0 · 33,7 min em
paralelo real** (~50% do tempo em série). Curador 54/54 limpos (isenção de
anatomia preservada pelo motivo no prompt-store). Socorro 0. Rede 0.

Buracos (2, ambos do BANCO, ambos honestos): s071 (arquivo evaporou do cache +
tema sensível) e s095 (hard-miss de identidade). Não são defeito do paralelismo.

O caminho até aqui custou 7 runs e pagou 6 defeitos de escala (PARTE VII) + a
lei do PROMPT-STORE (prompt é propriedade da cena, escrito 1x por job, autor
pulado em re-run — runs idempotentes de verdade: o run 7 colheu 10 imagens e 1
vídeo de graça das rodadas anteriores).

Mapa de tempos medido: `_workspace/_mapa_tempos_v9.md` (formato-padrão da
esteira). Calibração: ~37 s/vídeo e ~36 s/imagem efetivos no VEO Lower Priority.
