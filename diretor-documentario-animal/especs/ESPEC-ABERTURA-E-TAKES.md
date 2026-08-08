# ESPEC — ABERTURA DO DOCUMENTÁRIO + HOOK + IDENTIDADE DE TAKES POR VÍDEO-FONTE

> **Status:** 🟢 **VERSÃO FINAL (04/08, pós-auditoria)** — erros E1-E4 corrigidos, gaps
> G1-G6 respondidos pelas decisões D1-D6 do operador, e o **tamanho do hook DEFINIDO**
> (era a pendência B3). Aguardando **GO** para os itens ainda não implementados.
>
> **Já implementado e provado:** Item 1 (componente da abertura, com o fix do acento) —
> vereditos do operador: *"AGORA FICOU PERFEITO, REALMENTE MUITO BOM"* (render AMAZON/CONGO)
> e prova do acento pós-fix (AMAZÔNIA/ÁFRICA). Itens 2, 3 e 4: especificados, não codados.
>
> **Regra-mãe:** aditivo e gated. Nada altera o que está validado (tag `diretor-doc-v1-validado`).

---

## ITEM 1 — ABERTURA COM O NOME DO LUGAR ✅ componente pronto · ⏳ falta plugar no pipeline

**Onde está:** `remotion/src/compositions/Abertura.tsx` · composição `Abertura` no
`Root.tsx` · `remotion/_render_abertura.mjs` (prova original) ·
`remotion/_render_abertura_acentos.mjs` (prova do acento).
**Provas aprovadas:** `out/abertura_AMAZON.mp4` · `abertura_CONGO.mp4`.
**Provas do fix do acento (04/08):** `out/abertura_ac_AMAZONIA.mp4` · `abertura_ac_AFRICA.mp4`
· `abertura_ac_AMAZON_regressao.mp4` (regressão: idêntico ao aprovado, Δ3px).

**Referências do operador:** `Downloads/África.png` · `Amazônia.png` · `Egito.png` · `0728(1).png`
(feitas por ele no CapCut).

### 1.1 O efeito

> *"eu usei a fonte anton, obrigatoriamente caixa grande, aí no capcut usei no texto um
> efeito de mascarar, adicionei a máscara horizontal, deixei na metade do texto e puxei a
> peninha pra parte de baixo ficar com essa transparência e a parte de cima mais 100%"*

| Elemento | Especificação |
|---|---|
| Fonte | **Anton** (`fontes.ts`, tema `impact`) |
| Caixa | **ALTA obrigatória** |
| Cor | branco |
| Máscara | **gradiente vertical**: topo **100% opaco** → base **transparente** |
| Sombra no texto | **nenhuma** |
| Posição | topo do quadro |
| Quebra de linha | 🚨 **NUNCA.** *"SOMENTE EM UMA LINHA, SEMPRE EM UMA LINHA…… SOMENTE EM UMA LINHA."* |

#### ✅ VALORES FECHADOS — implementados e aprovados em render

| Grupo | Parâmetro | Valor |
|---|---|---|
| **Texto** | tamanho | **23% da largura** *(ver os 3 degraus abaixo)* |
| | distância do topo | **2%** — medida até o **topo da TINTA** (acento incluso; ver §1.1.1) |
| | espaço entre letras | **20** = **1,4% da largura** (27px em 1920) |
| | margem lateral | **4%** |
| **Máscara** | opacidade no topo | **1.00** |
| | opacidade na base | **0.00** |
| | onde começa a esmaecer | **38%** |
| **Footage** | sombra | **0.20** |
| | zoom lento do take | **1.06** |
| **Tempo** | fade in | **1.50 s** |

#### 🔬 TAMANHO E ESPAÇAMENTO — a regra, em 3 degraus (única e definitiva)

> Correção do operador que define a regra: *"O texto NÃO deve ocupar X% da tela. Eu
> organizei os prints de um jeito que ele ficava com 20 de espaçamento entre as letras,
> para ocupar MAIS do que ocupava com 0 — não obrigatoriamente ocupar X% da linha. (…)
> As letras devem ser espaçadas, mas até certo ponto; passar disso é exagero."*

| Degrau | Quando | O que faz |
|---|---|---|
| **1. padrão** | sempre | **fonte 23% da largura** + **vão 1,4% da largura**. Quanto ocupa da linha é **consequência**, não meta — nome curto ocupa menos, e está certo |
| **2. o vão cede** | não coube em 1 linha | reduz o vão até **0**, mantendo a fonte grande |
| **3. a fonte cede** | ainda não coube | fonte encolhe até caber, aí sim **ocupando o máximo da linha** com vão 0 |

> 💡 *"ocupar o máximo possível da linha"* vale **só no degrau 3**.

**Comportamento verificado (métricas reais do Anton, fonte sempre partindo de 23%):**

| Nome | Fonte | Vão | Ocupa | Degrau |
|---|---|---|---|---|
| EGITO | 23% | 27px | 52% | 1 |
| CONGO | 23% | 27px | 61% | 1 |
| AMAZON | 23% | 27px | 79% | 1 |
| AMAZÔNIA | 23% | **11px** | 92% | **2** — o vão cedeu |
| AUSTRALIA | 23% | **6px** | 92% | **2** |
| SOUTH AFRICA | **18%** | 0 | 92% | **3** — a fonte cedeu |

Com a regra do nome curto (§1.3), o degrau 3 é rede de segurança, não rotina.

> 🐞 **Tentativas erradas, registradas no componente para não repetir:** (v1) fonte 15%
> fixa — o `vw` do HTML era relativo à **janela**, não ao card; (v2) `canvas.measureText`
> mede com a fonte errada (`ctx.font` não aceita família com fallback múltiplo); (v3/v5)
> `textLength` preenche **a qualquer custo** e espalha as letras. A saída: **métricas do
> `.ttf` embutidas** (65 glifos, larguras `ANTON_W`) + a regra dos 3 degraus.
>
> ⚠️ A antiga seção "ESCALA PROGRESSIVA" com fonte 15% era **fóssil da v1** e foi
> **removida desta espec** (auditoria 04/08, erro E1). A regra acima é a única.

#### 1.1.1 🐞 ACENTO — bug encontrado na auditoria (E2) e CORRIGIDO

O componente posicionava a baseline por fator fixo (`fs × 0.86`), calibrado para
caixa-alta **sem** acento. Fato medido no `.ttf`: caixa-alta tem topo de tinta em
**0.867em**, mas acentuadas chegam a **1.10em** (`Ô Á É Í`) — com fonte 23% (442px), o
acento era **decepado 85px acima do frame**. Os renders aprovados (AMAZON, CONGO) não
tinham acento, por isso nunca apareceu.

**Fix (implementado):** tabela `ANTON_TOP` com o **yMax real de cada um dos 65 glifos**
(extraída do `Anton-Regular.ttf`, mesmo método das larguras) e baseline calculada pela
**tinta mais alta do nome**: `yBase = topo 2% + fs × max(yMax dos glifos do nome)`.
- Nome **sem** acento: idêntico ao aprovado (Δ3px — prova de regressão AMAZON ✓).
- Nome **com** acento: o conjunto desce o necessário para o **topo da tinta** (o acento)
  ficar exatamente a 2% do topo — nada é cortado (provas AMAZÔNIA ✓ ÁFRICA ✓).
- O parâmetro "altura da linha 1,18" do HTML de refino **não existe no componente** — era
  o mecanismo do HTML para o mesmo fim; no SVG o mecanismo é a baseline pela tinta.

#### 🌍 IDIOMA DO ROTEIRO

> *"o que for escrito deve estar obrigatoriamente na língua do roteiro: se for inglês,
> inglês; se for espanhol, espanhol; se for alemão, alemão, e assim por diante."*

O nome sai **no idioma do roteiro** — `AMAZON` (EN), `AMAZONÍA` (ES), `AMAZONAS` (DE),
`AMAZÔNIA` (PT). A tradução é responsabilidade do **pass de extração** (§1.3) — o nome
chega **pronto** no timeline; o player não traduz nada.

### 1.2 A cena de abertura

- **Um único take fixo** no lugar das duas cenas curtas de hoje.
- **OBRIGATORIAMENTE VÍDEO** (nunca imagem).
- Duração **randômica entre 5 e 7 s** — 🎲 **sorteada no `preparar_render.py` e GRAVADA
  no timeline** (`abertura.dur_seg`). O render apenas materializa (E4): re-renderizar o
  mesmo timeline dá SEMPRE o mesmo vídeo. `Math.random()` em script de render é proibido.
- **Fade in de 1,5 s** englobando **texto + footage juntos**.
- **O texto vive colado ao take**: mesma composição, mesmo fade, mesma saída.
- **Saída (D3): corte seco** junto com o take — sem fade de saída. Dali em diante valem
  as transições normais do sistema.
- **Fronteira de saída:** o corte cai na **fronteira de cena mais próxima** do valor
  sorteado (dentro de 5-7s; se nenhuma fronteira cair na janela, a mais próxima de 6s).
  A abertura **não adiciona tempo** ao vídeo: ela **cobre** as cenas existentes até a
  fronteira escolhida (as cenas cobertas não renderizam clip próprio; a narração corre
  normal por cima, começando junto do fade).

**De onde vem o take (G1 + D4), em ordem de degradação:**
1. **Establishing próprio** — busca dedicada usando o `documentary_setting` (o
   `contrato_visual.py` já usa o setting "no establishing da abertura"; o take da
   abertura usa a mesma âncora, com `prefer_video` obrigatório);
2. senão → o **primeiro clip de VÍDEO já resolvido** do timeline;
3. senão → **sem abertura** (degradação limpa, gated — nunca imagem, nunca abortar).

### 1.3 De onde vem o nome — 🚨 SEMPRE O NOME DO PAÍS (com as 2 exceções decididas)

> **Regra dura:** *"NUNCA 'COSTA NORTE DA AUSTRÁLIA'. Se for 'costa norte da Austrália',
> o país é AUSTRÁLIA. Sempre, SEMPRE o nome do país, SEMPRE."*
> **Nome curto sempre:** `DEMOCRATIC REPUBLIC OF CONGO` → **`CONGO`**. Vale o nome
> **comum/curto**, nunca o oficial.

**Decisões do operador (04/08) que fecham os casos abertos:**

| # | Caso | Regra decidida |
|---|---|---|
| **D1** | Região supranacional ("savana africana") | **país quando existir um; senão o CONTINENTE** (ÁFRICA — como na referência dele) |
| **D2** | Bioma × país (Amazônia) | **lista fechada de exceções NO CÓDIGO** — bioma vence o país **só se estiver na lista**; o LLM **nunca** decide. Regra de desempate do operador: *"se falar explicitamente BRASIL, deverá ser BRASIL; mas se falar AMAZON, esse termo sempre vencerá o país."* → o que o roteiro **menciona explicitamente** ganha; lista inicial: `AMAZÔNIA` (crescer só por ordem do operador) |

**O pass de extração (G5) — contrato:**
- **Onde roda:** junto do pass do `contrato_visual.py` que já produz o
  `documentary_setting` (mesma chamada; campo novo na resposta).
- **Saída no timeline:** bloco `abertura = { nome, dur_seg, take }` — `nome` já no
  **idioma do roteiro**, caixa-alta, nome curto, regras D1/D2 aplicadas.
- **Validação (bump de `VALIDATOR_VERSION`):** `nome` não-vazio · 1 linha (garantido pelos
  3 degraus) · **cobertura de glifos**: todo caractere fora das tabelas `ANTON_W`/`ANTON_TOP`
  gera **warning** no `pre_render_report` (largura vira aproximação 0.5em — não quebra,
  mas o operador precisa ver).
- I14 vale: o nome deriva do que o roteiro/setting **diz** — nada inventado.

### 1.4 ✅ TAMANHO DO HOOK — DEFINIDO (era a pendência B3)

> Decisão do operador (04/08): *"os textos dos 3 biomas deram média de ~1:30-1:40; um hook
> saudável deve ser ~2:20 a 2:30; faça uma regra de 3 e mensure pela QUANTIDADE DE PALAVRAS."*

**Medição real (3 timelines validados):**

| Bioma | Palavras | Duração narrada |
|---|---|---|
| Amazônia | 238 | 93,4s (1:33) |
| Austrália | 240 | 102,4s (1:42) |
| África | 240 | 96,2s (1:36) |
| **Total** | **718** | **292s** → **2,46 palavras/s (147,5 wpm)** |

**Regra de 3:** 2:20 (140s) → 344 palavras · 2:25 (145s) → 356 · 2:30 (150s) → 369.

**REGRA DO HOOK (fechada):**
- `HOOK_PALAVRAS = 355` (alvo 2:25; banda saudável 345-370 = 2:20-2:30).
- O hook são **as primeiras 355 palavras do roteiro**. Contagem determinística: tokens
  separados por espaço que contenham letra ou dígito.
- **Toda instrução "de hook"** (dinamismo, mapa obrigatório do Item 2, o que mais for
  definido) vale **somente dentro dessas 355 palavras**. *"Passou daí, não vale mais."*
- **Materialização em segundos:** a janela temporal do hook termina onde a **palavra 355
  termina na narração alinhada** (timestamps por palavra). Esperado ≈2:20-2:30 se o
  narrador mantiver o ritmo medido; se variar, **as palavras mandam**, não o relógio.
- A constante mora no **director** (usada pelo roteirista nas instruções E pelo
  `preparar_render`/detectores para delimitar a janela).

---

## ITEM 2 — MAPA OBRIGATÓRIO NA INTRODUÇÃO *(agora destravado pelo §1.4)*

> *"obrigatoriedade de uma apresentação da localidade em mapa do lugar na introdução, se o
> lugar for EXPLICITAMENTE MENCIONADO."*

- **Gatilho:** o lugar ser **explicitamente mencionado** dentro da **janela do hook**
  (§1.4) — não inferido (I14).
- **Efeito:** mapa **obrigatório** na janela do hook — hoje o `detectar_mapas.py` decide
  por cooldown/teto e a introdução pode ficar sem mapa; a obrigatoriedade passa por cima
  do cooldown **dentro da janela**.
- **Interação com o filtro de elegibilidade (I13):** a variante continua sendo escolhida
  pelos **dados disponíveis**; o que muda é a obrigatoriedade de **haver** um mapa.
- A abertura (Item 1) e o mapa do hook são coisas distintas: o nome do país na abertura
  **não** conta como "apresentação em mapa".

---

## ITEM 3 — 🚨 TAKES DO MESMO VÍDEO SÃO O MESMO TAKE

> *"são do MESMO VÍDEO, então ele deve classificar isso como TAKES IGUAIS."*

### 3.1 Diagnóstico com evidência (render aprovado da Austrália, 04/08)

Orçamento hoje conta por **hash do arquivo** (`_ident` → `_file_hash`,
`resolver_cascata.py:1420`), com `MAX_USOS_ARQUIVO = 2` e `DIST_MIN_REUSO = 7`:

| Fato medido | Número |
|---|---|
| Cenas com clip | 23 |
| Arquivos usados **2×** | **3** (cenas 2+9, 7+20, 15+17) |
| Cenas cujo vídeo-fonte tem **15 s ou mais** | **5** |
| Exemplo extremo | cena 0: fonte de **35 s**, exibe **2,6 s** = 7% |
| Caso do operador | cenas 2 e 9: mesmo arquivo de **27 s**, ~3 s cada |

Como o `<Loop>` reinicia do frame 0 (`BrollTest.tsx:227`, sem `startFrom`), dois usos
mostram **a mesma imagem** — item C3 do índice de pendências, separado deste.

### 3.2 O que o hash de arquivo NÃO pega

1. mesmo vídeo Pexels em **resoluções diferentes** (bytes distintos, imagem idêntica);
2. mesmo vídeo achado por **queries diferentes** — fato do código (E3): o arquivo é
   nomeado por **hash da QUERY** (`v{h}.mp4`), então o mesmo vídeo-fonte pode existir em
   **dois paths**; o `_file_hash` até pega quando os bytes coincidem, mas não por
   identidade de origem.

### 3.3 Correção — identidade por VÍDEO-FONTE (mecanismo completo, E3)

```
identidade_do_take = pexels_id  (se houver)  →  senão  _file_hash(arquivo)  →  senão path
```

**Onde cada coisa se materializa (obrigatório — sem isso o dedup continua furado):**
1. **No download** (`pexels_download` já recebe o `item` com `item["id"]`): gravar
   `pexels_id` **na entrada do índice** (`index[h]`) junto de `file/media_tipo/dur/nivel`;
2. **No timeline:** registro da cena ganha `pexels_id` ao lado de `arquivo_hash`;
3. **No orçamento:** `_ident(path)` faz **lookup reverso** path→`pexels_id` no índice;
   só cai no `_file_hash` se o path não tiver id (cache legado, Commons, archive).
- ⚠️ **Cache legado não tem id** → em modo cache o ganho é ≈zero na primeira rodada
  (tudo cai no fallback, igual hoje). O ganho real aparece em `fresh`. A medição (D5)
  precisa ser em `fresh`.

**Orçamento (D5 — decisão condicionada à medição):**
- Candidato: `MAX_USOS_POR_FONTE = 1` (um vídeo-fonte = uma cena no vídeo inteiro).
- **Medir antes de cravar:** rodar os 3 biomas em `fresh` com `MAX=1` e comparar
  `hard_miss`/tempo/downloads com o baseline. O número decide.
- Com `MAX=1`, `DIST_MIN_REUSO` vira letra morta — **só se aplica se ficar MAX=2**.

**Exceção decidida (D6):** fundos de **word-pop** e **pai-neutra** ficam **FORA do
orçamento de takes** — o design atual usa a mesma query→mesmo arquivo de propósito
(`resolver_cascata.py:633`); são fundos, não "takes" percebidos pelo espectador.

### 3.4 Ganho colateral (registrado, não é o pedido)

Trecho exibido variável dentro do vídeo-fonte (via `startFrom` no `<Loop>`) faria dois
usos deixarem de ser idênticos — é o item **C3** do índice, não entra neste GO.

---

## ✅ IMPLEMENTAÇÃO (executada em 04/08, pós-GO)

| # | Item | Onde ficou |
|---|---|---|
| 1 | Pass do nome + bloco `abertura` no timeline | `director/abertura.py` (novo) · knob `abertura` no preset |
| 2 | Take fixo + `dur_seg` sorteada **no director** | idem — sorteio único, **idempotente** (re-rodar não re-sorteia) |
| 3 | Materialização + plugagem no render | `remotion/preparar_render.py` · `BrollTest.tsx` (camada ABERTURA) |
| 4 | `HOOK_PALAVRAS = 355` + janela | `director/hook.py` (novo) |
| 5 | Mapa obrigatório no hook | `director/detectar_mapas.py` · knob `mapa_hook_obrigatorio` |
| 6 | Identidade por vídeo-fonte | `director/resolver_cascata.py` (mapa persistente `pexels_ids.json`) |

**Detalhes que a execução fixou:**
- O corte da abertura **cobre** as cenas até a fronteira: cena inteiramente dentro da
  janela **não monta clip próprio** (não é só invisível — é decode economizado).
- **Backdrop preto obrigatório** sob a abertura: o fade de 1,5s é opacidade; sem o preto,
  o fade revelaria a cena de baixo em vez de sair do preto.
- D2 olha o roteiro **e** o `documentary_setting` — o setting é derivado do próprio
  roteiro, então doc da Amazônia em PT dá `AMAZÔNIA`, não `AMÉRICA DO SUL` (bug pego no
  teste dos 6 casos, hoje 6/6).
- `MAX_USOS_POR_FONTE` virou knob (`max_usos_por_fonte` / env `MAX_USOS_FONTE`), default
  **2 = comportamento de hoje**. D5 é um flip depois do número.
- **D6 no código:** fundo de word-pop (`prefer_video_forcado` / `stock_query_override`)
  fica FORA do orçamento de takes.

## 🧪 PROTOCOLO DO TESTE (travado pelo operador, 04/08 — roda SÓ após o GO explícito)

> *"O render das introduções acontecerá SOMENTE DEPOIS da execução de TODAS as especs da
> lista, com meu GO explícito."*

1. **Executar TODAS as especs da lista** (A3 · esta espec passos 1-5 · C1 · iNaturalist
   operante · mecânica do Item 3) — **antes** de qualquer render de teste.
2. Operador dá o **GO explícito** do teste.
3. **Render das 3 aberturas** — fila pronta em `_workspace/producao_jobs_v2abertura.json`:

   ```bash
   cd director && VM_MODO=fresh python producao.py ../_workspace/producao_jobs_v2abertura.json
   ```

   com os áudios já prontos (Amazônia · Austrália · África):
   - os áudios atuais são **menores que o hook** (~1:30-1:40) — servem para o teste;
   - **takes TOTALMENTE NOVOS** — modo `fresh`, TODOS os assets novos, nada do cache das
     versões anteriores;
   - o **iNaturalist já operante** — o teste também puxa material de lá;
   - **NÃO sobrescrever** os renders antigos: saída com **nome distinto**
     (`<bioma>_v2abertura_<data>.mp4`) para sabermos que são elas.
4. **Se aprovadas** → operador dá GO para **B1** (smoke da regra das 40, por último mas
   não esquecido) → depois o **teste do B3** (hook em tamanho real, 355 palavras).

*Versão final 04/08/2026, pós-auditoria (E1-E4 corrigidos, G1-G6 fechados por D1-D6).
Substitui integralmente a versão anterior.*
