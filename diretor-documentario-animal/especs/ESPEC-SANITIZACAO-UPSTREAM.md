# ESPEC — SANITIZAÇÃO ANTI-CONTAMINAÇÃO NA IMPORTAÇÃO DO UPSTREAM

> **Status:** 🟡 **AGUARDANDO GO.** Processo desenhado sobre MEDIÇÃO do estado real dos
> dois repos (não sobre suposição). A ferramenta da etapa 2 já existe e roda:
> `director/_inventario_upstream.py`.
>
> Ordem do operador (06/08): *"temos que trazer, ESTUDAR, FAZER UMA DECUPAGEM, VER DAS
> ATUALIZAÇÕES O QUE DÁ PARA TRABALHAR E COMO APLICAR NO NOSSO, não trazer e sair
> enfiando... o que formos implementar deverá se moldar À NOSSA REALIDADE, e não ao
> contrário... deverá entender o idioma que falamos aqui, do jeito que falamos aqui...
> trabalhar estilo container, onde um container não pode infectar/danificar os outros."*

---

# PARTE 0 — 🔴 O FATO FUNDADOR (medido antes de desenhar qualquer coisa)

```
commits em comum entre o nosso repo e o vidmator-director:  0
```

**Não somos um fork dele.** Nosso repo nasceu em 03/08 de um PORT (`checkpoint: VidMator
nosso — port v4`), com árvore própria. Sem ancestral comum, `git merge`, `git rebase` e
`git cherry-pick` entre os dois **são impossíveis por construção** — o git nem tem como
calcular a base de três vias.

Isso não é um detalhe burocrático: é o que torna "trazer e enfiar" **inexequível**, não só
arriscado. Toda importação daqui para a frente é uma **TRADUÇÃO** — alguém lê o que ele
escreveu, entende o que aquilo assume, e reescreve na nossa língua. O processo abaixo é
só a forma disciplinada de fazer isso sempre igual.

**Consequência prática:** o `git pull` acontece SEMPRE nas cópias vendorizadas
(`piter-repo/`, `director-repo/`), NUNCA no nosso repo. Elas são a *janela* para o
trabalho dele, não uma fonte de patches.

---

# PARTE 1 — 📊 O TERRITÓRIO, MEDIDO (`_inventario_upstream.py`)

```
upstream: 123 arquivos | ⚪ NOVO 102 · 🟢 ESPELHO 11 · 🟡 DIVERGENTE 9 · 🔴 NOSSO 1
```

## As quatro zonas — e o que se pode fazer em cada uma

| Zona | Critério (medido) | O que é PERMITIDO |
|---|---|---|
| 🟢 **ESPELHO** | diff ≤ 12 linhas | **Sincronizar.** A tradução é mecânica (ver 1.1) |
| 🟡 **DIVERGENTE** | 12 < diff < 300 | **Colher IDEIA**, linha a linha. Copiar arquivo: proibido |
| 🔴 **NOSSO** | diff ≥ 300 | O arquivo dele é outro programa. **Nem para copiar trecho** |
| ⚪ **NOVO** | não existe aqui | **Único caso de importação de verdade** — e ainda assim traduzida |

## 1.1 · Descoberta útil: o verde é mecânico

Os 11 arquivos 🟢 têm diff de **2 a 10 linhas** — e o gap de dialeto deles é quase sempre
só `path cravado`. Ou seja: **o port daqueles arquivos foi só a troca de caminho**; a
lógica está intocada desde 03/08.

```
🟢 preset.py           29/29   diff 0     (idêntico)
🟢 coldopen_quote.py   50/50   diff 2
🟢 datas.py            68/68   diff 2
🟢 fontes.py           56/56   diff 2
🟢 satelite_fetch.py   64/64   diff 2
🟢 epoca.py            76/76   diff 4
🟢 gemini_api.py     144/144   diff 4
🟢 pessoas.py        183/183   diff 4
🟢 pexels_api.py       71/71   diff 4
🟢 topicos.py        102/102   diff 4
🟢 ilustrar.py       170/168   diff 10
```

Para esses, sincronizar é barato e o risco é baixo — **desde que o diff continue pequeno**.
No dia em que um deles passar de 12 linhas, ele MUDA DE ZONA automaticamente e volta para
a fila da decupagem. A classificação é medida a cada importação, nunca herdada.

## 1.2 · O vermelho, para não haver dúvida

```
🔴 resolver_cascata.py   2325 nossas / 701 dele   diff 1692
```

Triplicamos o arquivo (pool de takes, poster-gate com cache persistido, embargo, vet de
sujeito, heartbeat de trabalho). Um `git checkout` do dele aqui apagaria semanas. Está na
zona vermelha por medição, não por opinião.

## 1.3 · Os 102 ⚪ NOVO não são 102 candidatos

A maioria é infraestrutura do canal dele (`app.py` com 11 mil linhas, workers de CTA,
Drive, painel do YouTube) — nada a ver com o Diretor. Os que interessam ao nosso
documentário, hoje:

| Arquivo | Linhas | Por que interessa |
|---|---|---|
| `veo_driver.py` · `veo_lote.py` · `veo_ingest.py` | 321 + … | **É o E16** — geração VEO para cobrir hard-miss de espécie exata e exaustão de repetição, adiada por você |
| `vision_gate.py` | 171 | Gate de conteúdo por Vision. Temos o nosso (poster-gate); serve para COMPARAR critérios |
| `curador_footage.py` | 249 | "Funcionário 2" na arquitetura dele |
| `tabela_decisoes.py` | 92 | Trecho falado × o que aparece na tela — parente do nosso Cronograma Absoluto |
| `foley.py` · `ambiencia.py` | 84 + 71 | Camadas de áudio que não temos |
| `parallax5.py` | 96 | Parallax 2.5D — apresentação nova |
| `stock_ondemand.py` | 159 | Resolvedor alternativo de stock |

---

# PARTE 2 — 🗣️ O DIALETO: o que é "falar a nossa língua"

Esta é a parte que responde ao *"deverá entender o idioma que falamos aqui, do jeito que
falamos aqui"*. Não é estilo — é **contrato verificável**. Um candidato só entra depois de
falar as 10.

| # | Regra nossa | Como é no sistema DELE | Medido no upstream |
|---|---|---|---|
| 1 | **Zero path cravado** — tudo por `vmconfig` | `F:/Canal Dark/...` no topo do arquivo | **56 arquivos** com path cravado |
| 2 | **Cliente compartilhado** — `gemini_api`/`pexels_api`, com rotação de chave | cada arquivo lê `credentials.json` e faz a própria rotação | vários, incl. `vision_gate.py` |
| 3 | **Gating por knob** (I1) — feature nasce DESLIGADA, ligada pelo preset do nicho | roda sempre | **0 arquivos** dele usam `preset`/knob |
| 4 | **Degradar, não abortar** (I12) — passe que falha não derruba o job | `sys.exit` no meio | vários |
| 5 | **Idempotência** (S14) — passe limpa a PRÓPRIA saída antes de decidir | não é um contrato lá | a prova roda em cima do candidato |
| 6 | **Vocabulário do timeline** — `contrato`, `documentary_setting`, `required_subjects`, `acervo_texto`, `overlay_take`, `pop_bg_*`, `_capitulos_previstos` | campos dele, outro conjunto | mapa por candidato |
| 7 | **Barreira de validação** — a lei é reconferida no `preparar_render`, que roda em 100% dos pipelines | lei mora onde decide | — |
| 8 | **Heartbeat** — passe longo pulsa via `producao.run` (o zelador mata mudo) | sem heartbeat | — |
| 9 | **Campo de LLM nunca cru em f-string** (E23) | `{m['pais']:<14}` | detectado por regex |
| 10 | **Docstring em PT com o PORQUÊ e a MEDIÇÃO** — *"por que existe"*, não *"o que faz"* | docstring curto, descritivo | — |

O `_inventario_upstream.py` já mede as regras 1, 2, 3, 4 e 9 automaticamente e imprime o
**gap de dialeto** por arquivo. Ele não reprova nada: ele **dimensiona o trabalho de
tradução antes de alguém prometer prazo**.

---

# PARTE 3 — 📦 O CONTAINER (isolamento concreto, não metáfora)

*"trabalhar estilo container, onde um container não pode infectar/danificar os outros."*

Quatro camadas, todas com mecanismo que já existe no repo:

### C1 · Quarentena de arquivo
O candidato traduzido nasce em **`_import/<AAAA-MM-DD>-<nome>/`**, nunca em `director/`.
Nada em `director/` é tocado enquanto o candidato não passar em tudo.

### C2 · Workspace isolado
Roda com **`VIDMATOR_CONFIG`** apontando para um workspace-cópia — o mesmo mecanismo que o
`_prova_idempotencia.py` já usa (`_workspace_prova/`). O candidato não enxerga, não lê e
não escreve no `_workspace/` de produção. Se corromper alguma coisa, corrompe a cópia.

### C3 · Gating desligado
Entra atrás de knob **OFF por padrão** (I1). Mesmo depois de adotado, um vídeo só passa a
usá-lo quando o preset do nicho ligar. Feature nova nunca muda o comportamento de um
render que já funcionava.

### C4 · Reversão de um comando
Cada adoção é **um commit só, de um candidato só**, com a mensagem citando a ficha de
decupagem. Deu ruim: `git revert` de um commit derruba a feature inteira, sem colateral.

> **A regra que amarra as quatro:** enquanto o candidato não passar, ele existe em disco
> mas **não é importável por nenhum passe do pipeline** — não está em `director/`, então
> nenhum `import` o alcança.

---

# PARTE 4 — 🔄 O PIPELINE DE IMPORTAÇÃO (8 etapas)

```
1 COLETA      git pull SÓ nas cópias vendorizadas (piter-repo/, director-repo/)
2 INVENTÁRIO  _inventario_upstream.py  -> zona + gap de dialeto de cada arquivo
3 TRIAGEM     operador escolhe os candidatos (⚪ e 🟢); 🟡 vira "colher ideia"; 🔴 nem lê
4 DECUPAGEM   ficha por candidato (PARTE 5) — o que faz, o que ASSUME, o que toca
5 TRADUÇÃO    reescrita na nossa língua (as 10 regras) dentro de _import/<data>-<nome>/
6 QUARENTENA  roda em workspace-cópia; saída DIFFADA contra o baseline do mesmo job
7 PROVA       idempotência + portão + invariantes + "o vídeo mudou pelo motivo certo?"
8 ADOÇÃO      1 commit, knob OFF, ficha anexada, entrada no CATÁLOGO
```

### Sobre a etapa 6 — o teste que realmente pega contaminação
Rodar o mesmo job **duas vezes**: uma sem o candidato (baseline), outra com. Diffar os
dois `timeline_render.json`. **Toda diferença tem de ter explicação na ficha.** Diferença
que ninguém previu = contaminação, e o candidato volta para a etapa 5.

Isso pega exatamente a classe de estrago que nos custou caro esta semana: campo que
sobrevive onde não devia (E19/E24), passe que escreve fora do seu domínio, valor cravado
sobrescrevendo configuração (E22).

### Sobre a etapa 7
Reusa o que já está construído: `_prova_idempotencia.py` (o candidato limpa a própria
saída?), `_decupagem_video.py` (o vídeo continua passando nos fixadores?), e a **barreira
de validação** do `preparar_render` (o candidato respeita as leis do operador?).

---

# PARTE 5 — 📋 FICHA DE DECUPAGEM (uma por candidato, obrigatória)

```markdown
## <arquivo.py>  ·  zona <⚪/🟢/🟡>  ·  upstream <commit> <data>

### 1. O que faz (em 3 linhas, na minha língua)

### 2. O que ele ASSUME do sistema DELE
   - paths / credenciais:
   - campos do timeline que LÊ:
   - campos do timeline que ESCREVE:
   - quem o chama lá, e em que ordem:
   - specs dele que ele referencia:

### 3. Como isso se traduz AQUI
   - as 10 regras do dialeto, uma a uma: [ok] ou [o que muda]
   - nossos campos equivalentes (mapa DELE -> NOSSO):
   - onde entra na NOSSA ordem de passes, e por quê:
   - knob que o liga (nome + default OFF):

### 4. Conflito com o que já temos
   - duplica algo nosso? (ex.: vision_gate × nosso poster-gate)
   - se duplica: substitui, complementa, ou é descartado?

### 5. Prova
   - baseline vs com-candidato: o que MUDOU no render json, e por quê
   - idempotência: [passa/reprova]
   - portão: [antes] -> [depois]

### 6. Veredito
   ADOTAR / ADOTAR PARCIAL (só a ideia X) / DESCARTAR — com a razão.
```

Ficha sem a seção 5 preenchida **não vale**: é a diferença entre "eu li e achei bom" e "eu
medi".

---

# PARTE 6 — 🚫 AS REGRAS DURAS (o que NUNCA acontece)

1. **Nunca** `git pull`/`merge`/`cherry-pick` do repo dele para o nosso. Sem ancestral
   comum, isso nem é uma operação válida — e se alguém forçar com `--allow-unrelated`, o
   resultado é lixo.
2. **Nunca** copiar arquivo da zona 🔴 ou 🟡. Da 🟡 se colhe IDEIA; o código é reescrito.
3. **Nunca** adotar candidato sem knob. Feature nova nasce desligada, sempre.
4. **Nunca** adotar dois candidatos no mesmo commit. Um container por vez.
5. **Nunca** rodar candidato no `_workspace/` de produção. Só em cópia.
6. **Nunca** aceitar diferença no render json que não esteja explicada na ficha.
7. **Nunca** herdar a classificação de zona da importação anterior — ela é MEDIDA de novo
   a cada rodada (arquivo verde vira amarelo sozinho quando ele mexe).

---

# PARTE 7 — O QUE FALTA CONSTRUIR (se houver GO)

| Peça | Estado |
|---|---|
| `_inventario_upstream.py` (etapa 2) | ✅ **pronto e rodando** |
| Workspace isolado por `VIDMATOR_CONFIG` (C2) | ✅ pronto (usado pela prova de idempotência) |
| `_prova_idempotencia.py` (etapa 7) | ✅ pronto |
| `_decupagem_video.py` (etapa 7) | ✅ pronto |
| `_traduzir_dialeto.py` — troca mecânica de path cravado por `vmconfig` na zona 🟢 | ⏳ a fazer (~40 linhas, resolve os 11 verdes) |
| `_diff_baseline.py` — diffa dois render json e destaca o que mudou (etapa 6) | ⏳ a fazer (~60 linhas) |
| `_import/` + template da ficha | ⏳ a fazer (estrutura + markdown) |
| Entrada no `CATALOGO-BLINDAGEM-PRODUCAO.md` por adoção | ⏳ convenção |

**Estimativa:** as três peças pendentes são ~2h. Depois disso, cada importação vira
rotina — e o primeiro candidato natural é o **VEO** (`veo_driver`/`veo_lote`/`veo_ingest`),
que é o E16 que você adiou e que resolve o hard-miss de espécie exata.

---

*Aberta em 06/08/2026, sobre medição do estado real dos dois repos. AGUARDANDO GO.*
