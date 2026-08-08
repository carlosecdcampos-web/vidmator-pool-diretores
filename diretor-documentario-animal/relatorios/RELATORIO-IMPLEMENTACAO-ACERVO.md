# RELATÓRIO DE IMPLEMENTAÇÃO — ESPEC-ACERVO (40 variantes + Chapter + Mapas)

> **Para:** quem for executar a `ESPEC-ACERVO-40-VARIANTES.md` (eu mesmo numa sessão futura,
> ou outro agente). **Escrito em 04/08/2026, ANTES do GO.**
>
> 🔴 **Nada aqui é executado sem GO explícito do operador.** Ele revisa este documento antes.
>
> **Âncora:** este relatório não inventa regra. Tudo o que ele manda fazer (e proibir) vem de
> documentos e decisões REAIS do projeto:
> `director/CONTRATO-EDITORIAL.md` · `ESPEC-VIDMATOR.md` · `ESPEC-CONTRATO-VISUAL-EXECUCAO-REPRODUTIVEL.md` ·
> `ESPEC-MAPAS.md` (obsoleta, mas o artefato existe) · `ESPEC-DINAMISMO-TEXTO.md` ·
> `RUNBOOK-SERVIDOR.md` · o `RELATORIO-IMPLEMENTACAO-EDICAO.md` (irmão, já executado) · e as
> lições pagas em produção.

---

## 0. ONDE ESTAMOS

A espec irmã (`ESPEC-EDICAO-ADITIVOS`) **já foi implementada** — passos 0-5, na branch
`edicao-aditivos`, validada com render da Austrália e auditoria Vision sem regressão. Esta
espec é o bloco seguinte, e é **maior**: 40 variantes de acervo + `ChapterTitle` + o
subsistema de mapas.

**Diferença crucial em relação à espec irmã** — e o motivo deste relatório ser mais cauteloso:

| | Espec irmã (já feita) | Esta espec |
|---|---|---|
| Natureza | ajustar **valores** de componentes **já plugados** | **PLUGAR** componentes que nunca rodaram |
| Risco | baixo (número trocado) | **alto** (código novo no caminho do vídeo) |
| Reversão | trocar o número de volta | remover o gatilho inteiro |
| O que valida | comparação visual | comparação visual **+ o plano do report** |

> A espec irmã mexeu na **aparência** do que já existia. Esta mexe em **o que aparece**.

---

## 1. INVARIANTES — quebrar qualquer um destes é regressão

Herdados do relatório irmão (I1-I12), mais os que nascem deste bloco. Cada um veio de um erro real.

| # | Invariante | Onde nasceu |
|---|---|---|
| I1 | **Gated por preset.** Recurso novo tem knob; preset sem o knob = comportamento atual **byte a byte**. Os nichos do Piter não podem mudar sozinhos. | `ESPEC-VIDMATOR` (D3) |
| I2 | **Teto de SFX 0.20**; calibrados a 0.15 são imutáveis. | política do operador (03/08) |
| I3 | **Transição única — nunca empilhar.** | Contrato Editorial, grupo E |
| I5 | **Estado mínimo ~1s** — nada aparece por menos que isso. | Contrato Editorial, grupo E |
| I6 | **O word-pop é dono da tela** — nenhum overlay entra na janela dele. A guarda roda no ÚLTIMO pass. | N4 + N8 |
| I8 | **Mudou regra que o vet aplica → bumpar `VALIDATOR_VERSION`.** | `ESPEC-CONTRATO-VISUAL` (B1-5) |
| I9 | **Render é materialização, não validação** — nada entra sem PRE-RENDER REPORT + lock. | auditoria 3 |
| I11 | **NÃO bumpar `VALIDATOR_VERSION` à toa** — invalida decisões cacheadas e força re-resolver tudo. | converso do I8 |
| I12 | **Asset novo entra pela convenção do `preparar_render.py`**: copia → `*_rel` no dict `render` → ausente = `None` + AVISO (degrada limpo). | `preparar_render.py:17-45, 227-259` |
| **I13** | **Variante sem os dados exigidos NÃO PODE SER ESCOLHIDA.** A checagem roda **antes** da escolha. Escolher e depois receber `null` = cena sem elemento, em silêncio. | decisão do operador 04/08 (§7.2.4) |
| **I14** | **Dado que não está no roteiro não existe.** Gráficos e mapas recusam sem dado real (`R-32`); o Contrato Editorial (grupo B) proíbe inventar. O LLM **não preenche número não falado**. | `R-32` + Contrato Editorial |
| **I15** | **"OK" na espec = default intocado.** Nenhuma regra global se aplica por cima do que foi marcado OK. | ambiguidade do `dim`, resolvida em 04/08 |

---

## 2. ORDEM DE EXECUÇÃO (do menor risco para o maior)

> **Commit por passo.** Se algo quebrar, o `git revert` tem que ser cirúrgico.
>
> A ordem não é arbitrária: os passos 1-4 mexem **só em aparência de coisa já plugada**
> (risco baixo, igual à espec irmã). Do 5 em diante começa o **código novo no caminho do
> vídeo** — é outra categoria de risco.

### PASSO 0 — Preparo *(obrigatório)*

Branch a partir do estado atual (`edicao-aditivos` já mesclada, ou a partir dela). Medir a
baseline **antes de tocar em nada**: `npx tsc --noEmit | grep -c "error TS"` → esperado
**47**. Se der outro número, é esse o novo critério — não o 47 escrito aqui.

---

### PASSO 1 — Cor dos mapas: vermelho → `#f59e0b` *(risco: baixo)*

**O quê:** §7.2.1. Trocar o accent do `MapAnimation.tsx`.

**Arquivo:** `remotion/src/compositions/MapAnimation.tsx`

**Cuidados:**
- 🚨 **São DOIS lugares, não um.** A constante `ACCENT` na linha 9 **e** o
  `rgba(224,69,46,…)` **escrito à mão** na linha 68. Trocar só a constante deixa o país
  **preenchido de vermelho com contorno âmbar**. O novo rgba é `rgba(245,158,11,…)`.
- Não tocar no `SatelliteZoom` (verde `#34d399` é deliberado, "verde scanner").
- Não tocar nos 6 componentes soltos: **já estão em `#f59e0b`** (por isso a troca *conserta a
  exceção*, não cria padrão).

**Validação:** render de uma cena com mapa; conferir que **nenhum** pixel de destaque saiu
vermelho (país, contorno, pin, anel, barra).

---

### PASSO 2 — Valores do `MapAnimation` *(risco: baixo)*

**O quê:** §7.2.1b — pin 7→**9**, anel 16→**30**, espessura do anel 2→**2.5**, linha
tracejada 2→**1.5**, card 380→**500**, foto 224→**280**.

**Cuidados:**
- ⚠️ **O componente tem DOIS layouts** — com `local` (380/224) e sem `local` (300/180). Os
  valores aprovados são do **com `local`**. O layout **sem** `local` é o dos **nichos do
  Piter** (I1): decidir com o operador se escala proporcional (≈395/230) ou fica intocado.
  **Na dúvida, não mexer no sem-`local`.**
- "clareza dos outros países" **não** está fechado (§7.2.1b, nota 2) — o controle do HTML não
  partia do valor do código. **Perguntar antes**, não aplicar por conta própria.

---

### PASSO 3 — Valores das 3 famílias do acervo *(risco: baixo)*

**O quê:** §4 (TEXTO, 7 variantes), §5 (OVERLAY, 11 variantes), §6 (GRÁFICO: **nada**).

**Arquivos:** `texto/AcervoTexto.tsx` · `texto/AcervoTextoOverlay.tsx`

**Cuidados:**
- **`AcervoGraficos.tsx` NÃO é tocado.** §6: aprovado sem alteração, 16/16. Mesma proteção do
  Item 4 da espec irmã (entradas de texto) — não "melhorar" o aprovado.
- **I15:** as variantes marcadas OK (`Texto05/07/08`, `Ovl09/12/13`) ficam **byte a byte**.
  O `dim` 0.15 **não** se aplica a elas.
- 🚫 **O `dim` 0.15 NÃO se implementa neste passo — nem neste GO.** Ele é aplicado pelo
  **caller** que compõe o overlay sobre o footage, e esse caller só nasce no passo 8 (fora
  do GO — resposta E). Não assar o dim dentro dos componentes: eles são transparentes, e
  isso quebraria as variantes OK (I15). A regra fica registrada no §5 esperando a espec do
  passo 8.
- Os valores marcados *(igual)* **não geram edição**. Só os que a espec marca em negrito.
- ✅ **Confiabilidade das tabelas conferida (04/08):** ao contrário do `ChapterTitle` (onde a
  réplica divergia do código), as tabelas de TEXTO/OVERLAY foram validadas contra os `.tsx`
  (`Texto01/02`, `Ovl02/03/08` linha a linha) — os "(era X)" **são** os valores do código.
  Pode aplicar direto.

**Validação:** `tsc` na baseline + stills das variantes alteradas comparados com o catálogo.

---

### PASSO 4 — `ChapterTitle` + barrinha do `CountryCharacterMap` *(risco: baixo)*

**O quê:** §6.1 (15 dos 19 parâmetros mudam) e §7.2.5 (barrinha 150/8/12/4).

**Cuidados (as 3 pegadinhas do §6.1):**
1. **`paddingLeft: 16` no label "CHAPTER"** compensa opticamente o `letterSpacing`. Ao mudar
   o espaçamento para 14, **recalcular ou manter proporcional** — removê-lo descentraliza.
2. **Margem das réguas é assimétrica** hoje (`34px 0 30px`) → vira `38px 0`.
3. **Subtítulo é itálico peso 300** — o refino não mexeu; **manter**.
4. No `CountryCharacterMap`, o nome hoje é um `div` solto posicionado por `left/top`:
   **envolver num container** antes de pendurar a barra embaixo.

> ⚠️ O `ChapterTitle` **continua não plugado** depois deste passo — aqui só se ajusta a
> aparência. O gatilho é o passo 7.

---

### PASSO 5 — Filtro de elegibilidade dos mapas *(risco: MÉDIO — código novo)*

> **A partir daqui é código que decide o que aparece no vídeo.**

**O quê:** §7.2.4, pendência 3 — **I13**. Nenhuma variante pode ser escolhida sem os dados
que ela exige.

**Arquivo:** `director/detectar_mapas.py`

**Cuidados:**
- 💡 **Não escrever do zero.** `validarGeo()` já é exportada pelo `AcervoMapas.tsx`
  *"p/ o registry/Diretor decidir ANTES de escolher a variação"* — é o comentário do próprio
  arquivo. A tabela do §7.2.4 mapeia variante → condição.
- **I14:** o "valor ancorado" do `Map08` tem que **vir da fala**. Sem número dito, `Map08`
  não entra. Não deixar o LLM inventar para "preencher".
- **Fallback obrigatório:** nenhuma variante elegível → **`MapAnimation`** (o de hoje), que
  exige o mínimo. Nunca deixar a cena sem mapa em silêncio.
- **Knob (I1):** todo o passo 5+6 fica atrás de um knob no preset (sugestão:
  `mapas_variantes: true` só no `doc_realista`). Preset sem o knob → `detectar_mapas.py`
  não escreve `variante` → comportamento atual byte a byte.
- Logar a decisão (variante escolhida + por quê) — sem isso, depurar "por que veio esse mapa"
  vira arqueologia.

**Validação:** rodar o pass nos 3 biomas validados e conferir o log; nenhum deve perder mapa
que hoje tem.

---

### PASSO 6 — Plugar as 14 variantes de mapa *(risco: ALTO)*

**O quê:** §7.2.4 — 12 variantes novas entram no repertório.

**Arquivos:** `director/detectar_mapas.py` (escolha) · `remotion/preparar_render.py` (assets)
· `remotion/src/compositions/BrollTest.tsx` (render da camada 1.5).

**Cuidados:**
- 🔒 **O gating é o campo `variante?` no `MapaSeg`** (§7.2.4, pendência 5): entrada **sem** o
  campo = comportamento de hoje byte a byte (`satelite` → `SatelliteZoom`, senão →
  `MapAnimation`). O `detectar_mapas.py` só escreve `variante` com o knob do preset ligado.
  É o que garante o I1 para timeline antigo, preset sem knob e replay de manifesto.
- 🎯 **11 das 14 vêm do `AcervoMapas.tsx`** — que compartilha `Base`, `resolverPais` e
  `projecaoFit`. Importar o `MAPAS_COMPS` e despachar por nome é **muito mais barato** que
  plugar 14 componentes independentes. **Não reimplementar o que o arquivo já resolve.**
- **`Map13_CineLocation` precisa de FOTO, não satélite:** reutilizar o `imagem_rel` que o
  pipeline **já busca** para o card do `MapAnimation` — não criar um segundo caminho de foto.
- **6 das 14 dependem de imagem** (satélite/foto). O `satelite_fetch.py` precisa cobri-las,
  senão a variante recusa em runtime. **I12:** ausência de imagem = degradar limpo com AVISO,
  nunca quebrar o render.
- **`Map10_SatPinApp` usa `#ea4335`** (vermelho Google, deliberado). O operador pediu âmbar
  nos mapas — **decisão pendente**, não assumir.
- A camada 1.5 hoje tem lógica de saída delicada (`exitFrames`, `mapaCorteEntra`): o mapa que
  termina num corte sai **por cima** da cena entrante. **Qualquer variante nova entra por
  esse mesmo caminho** — se for renderizada fora dele, as ruínas vazam no corte (erro já
  cometido e corrigido em 03/08).

**Validação:** render completo de 1 bioma + PRE-RENDER REPORT limpo + comparação com a versão
de ouro.

---

### PASSO 7 — Gatilho do `ChapterTitle` + transições Apagão/Brilho *(risco: ALTO)*

**O quê:** §6.1 (gatilho + mecânica fechada) e §7.1. **Este passo É a "rodada própria" das
transições prometida na espec irmã** — o Item 1 dela se implementa aqui, não em outro lugar.

✅ **A colisão está resolvida por decisão do operador (resposta D)** — cadeia de precedência
na fronteira de tópico, um elemento por emenda:

> **`ChapterTitle` > transição Apagão/Brilho > glitch**

**Mecânica do capítulo (fechada no §6.1 — seguir à risca):** duração **3,0 s** (90 frames;
a animação interna completa em 62) · **nunca no tópico 1** (o hook é sagrado) · `title` =
`topico.titulo`, `chapterNumber` = índice, `subtitle` **vazio** (sem fonte factual = não
inventa, Contrato Editorial grupo B) · é **elemento de tela cheia**: corte seco por baixo e
saída POR CIMA da cena entrante, o mesmo caminho do mapa · fronteira dentro de janela de
word-pop → capítulo não entra, cai para a transição (**I6**).

**Transições (o técnico já está resolvido na espec irmã, Item 1):**
- 🚨 blend por clipe **medido**: `Brilho 2` → `screen`; **`Apagão` → normal** (em `screen`
  fica invisível — é preto na maior parte dos frames);
- 24 fps → **30 e 21 frames** no timeline de 30;
- SFX `foley_camera_shutter_01.mp3` a **0.15** nas duas (I2);
- escolha pelo **mood do tópico entrante**: `tense`/`dark`/`somber` → Apagão;
  `mysterious`/`neutral` → Brilho;
- **I5**: cena entrante < ~2 s → pula a transição;
- a cena que entra e a que sai viram **corte seco** (mesmo predicado de `pkgScenes`/
  `mapaCorteEntra` — estender, não duplicar).

**Validação:** render completo; conferir **frame a frame** cada fronteira de tópico — só um
elemento por emenda, capítulo nunca sobre o hook, Apagão visível (não "brilho duplicado").

---

### PASSO 8 — Regra de uso de TEXTO/GRÁFICO *(risco: ALTO — fora deste GO?)*

**O quê:** §8.1 e §8.2 — quando cada uma das 40 entra e como o LLM preenche.

**Cuidados:**
- **I13/I14 valem igual:** o `R-32` do acervo recusa gráfico sem dado real.
- Os manifestos (`TEXTO_MANIFEST`, `OVERLAY_MANIFEST`, `GRAFICOS_MANIFEST`) já trazem o campo
  `quando` — é o insumo da regra, como o `MAPAS_MANIFEST` foi para os mapas.
- **I6:** overlay novo **não pode** entrar na janela do word-pop. Se um pass novo criar
  overlay, ele entra **antes** do `legibilidade` (último pass) ou a guarda não pega — erro já
  cometido com o "1 Million".
- **Sugestão honesta:** este passo é uma espec própria. Entregar 0-7 e deixar o 8 para uma
  rodada dedicada é a escolha certa se o tempo apertar.

---

## 3. ERROS A EVITAR (todos já aconteceram neste projeto)

| Erro | O que aconteceu | Como não repetir |
|---|---|---|
| **Trocar a constante e esquecer o valor hardcoded** | (achado em 04/08) o vermelho do mapa está na constante **e** num `rgba` na linha 68 | Passo 1: grep pelo valor **e** pela forma rgb antes de dar por feito |
| **Copiar a convenção da casa sem olhar o asset** | o `mixBlendMode:"screen"` de todos os overlays anularia o Apagão, que é preto | Medir o asset antes (`signalstats`) |
| **Escolher a variante e só depois descobrir que falta dado** | — | **I13**: checar antes. Fallback para `MapAnimation` |
| **Deixar o LLM inventar número para preencher** | — | **I14** + `R-32` + Contrato Editorial grupo B |
| **Mexer no que está aprovado** | — | §6 (16 gráficos) e as marcadas OK: **byte a byte** |
| **Empilhar elemento na mesma emenda** | glitch + slide na mesma cena; take cortado ao meio | **I3** — e é exatamente o risco do passo 7 |
| **Overlay criado por pass posterior fura a janela** | o `ilustrar` roda depois do `enumeracoes`; a supressão não via a ilustração | **I6**: guarda final no `legibilidade` |
| **Renderizar elemento de tela cheia fora do caminho da camada 1.5** | as ruínas vazavam ms no whip do mapa | Passo 6: usar o mesmo `exitFrames`/`mapaCorteEntra` |
| **Traduzir rótulo de HTML direto para código** | `amplitude` é 1 controle para 2 oscilações; `lw` do `stat` é o total | Usar as tabelas da espec, que marcam "hoje × aprovado" |
| **Aplicar delta de 1-2px** | — | §7.2.6: 5 valores marcados como **ruído de slider**, não implementar |
| **Bumpar `VALIDATOR_VERSION` "por garantia"** | invalidaria as decisões dos 3 biomas | **I11** — só o passo 5/6 mexe em regra de escolha; avaliar caso a caso |
| **Commit gigante** | — | Um commit por passo, explicando **o porquê** |

---

## 4. PERGUNTAS — ✅ TODAS RESPONDIDAS (operador, 04/08, pré-GO)

**A. `MapAnimation`, layout sem `local` → NÃO MEXER.** Fica **300/180** como está. É o layout
dos nichos do Piter, e o **I1** manda não alterá-lo sem decisão explícita — a decisão foi
justamente **não** alterar. Só o layout **com `local`** recebe os 500/280.
→ *Efeito no passo 2: os dois layouts passam a divergir bastante (500/280 vs 300/180). Isso é
intencional, não bug — não "harmonizar" depois por conta própria.*

**B. "Clareza dos outros países" → MANTER `#151b29`.** O controle do refino nasceu num valor
diferente do código, então o 0.10 era default do slider, não escolha. Os países não
destacados ficam exatamente como hoje.
→ *Efeito no passo 2: um parâmetro a menos. Não tocar no fill dos países não destacados.*

**C. `Map10_SatPinApp` → MANTER o vermelho `#ea4335`.** A imitação do app de mapas é o ponto
do componente; em âmbar ele perde a referência e vira mais um pin.
→ *Efeito no passo 6: **exceção deliberada e documentada** à regra do §7.2.1. Quem for
implementar não deve "corrigir" essa cor achando que passou batido. Anotar no código.*

**D. Passo 7 — `ChapterTitle` × transições → O CAPÍTULO SUBSTITUI A TRANSIÇÃO.** Onde entra
card de capítulo, ele **é** a marcação (toma a tela inteira, não precisa de transição junto).
Nas fronteiras sem capítulo, entra Apagão/Brilho normalmente.
→ *Efeito no passo 7: vira uma cadeia de precedência simples na fronteira de tópico —*
> **`ChapterTitle` > transição Apagão/Brilho > glitch**
>
> *Um elemento por emenda, sempre. O **I3** fica satisfeito por construção, e não por
> checagem posterior — que é a forma robusta de garantir.*

**E. Passo 8 → RODADA PRÓPRIA.** Este GO entrega os **passos 1-7**. A regra de uso das 40
variantes de texto/gráfico vira espec dedicada.
→ *Efeito: o escopo deste GO fecha em "aparência + mapas plugados + capítulo/transições". O
`ilustrar.py` **não é tocado** neste GO — o que remove um risco inteiro de regressão sobre o
Sistema 1, que está em produção.*

---

## 5. PLANO DE VALIDAÇÃO

> ⚠️ **O que cada gate cobre.** O PRE-RENDER REPORT valida o **PLANO** (que mídia entra em
> qual cena). Ele **enxerga** os passos 5-6 (escolha de variante) mas **não enxerga** os
> passos 1-4 (aparência de componente Remotion). Para esses, o gate é `tsc` + comparação
> visual.

1. **`npx tsc --noEmit`** → baseline **47**. Nem um a mais.
2. **PRE-RENDER REPORT limpo** nos 3 biomas em modo `production` — gate real dos passos 5-8.
3. **Render completo de 1 bioma** — sugestão: **Austrália** (tem word-pop, takes, mapa,
   infográfico e apresentações; o mais completo).
4. **Comparação com a versão de ouro** `_renders_aprovados/2026-08-04_australia_croc_FINAL/`.
   Diferenças esperadas: só as da espec. Qualquer outra = regressão.
5. **Auditoria Vision** (`auditoria_composicao.py`) — comparar a **contagem** com a do vídeo
   de ouro, não com zero: o baseline dele tem 4 achados pré-existentes.
6. **Escuta** — nenhum SFX novo neste bloco, mas confirmar que nada mudou.

---

## 6. ROLLBACK

- Ponto seguro: **`git checkout diretor-doc-v1-validado`**.
- Um commit por passo → `git revert <sha>` desfaz um item sem derrubar os outros.
- Renders aprovados em `_renders_aprovados/` (fora do git) servem de referência visual.
- Os HTMLs de refino estão no **Desktop, fora do git** — mas o que o código precisa já está
  transcrito nas tabelas da espec. **Se divergirem, a espec vence.**

---

## 7. ORDEM RESUMIDA

```
0. preparo (branch + baseline tsc = 47)
1. cor dos mapas #f59e0b              (baixo)  → ATENÇÃO: 2 lugares, não 1
2. valores do MapAnimation            (baixo)  → SÓ o layout com `local` (A) · fill intocado (B)
3. valores TEXTO + OVERLAY            (baixo)  → GRÁFICO não se toca
4. ChapterTitle + barrinha            (baixo)  → 3 pegadinhas do §6.1
—— daqui em diante é CÓDIGO NOVO no caminho do vídeo ——
5. filtro de elegibilidade            (médio)  → I13; validarGeo() já existe
6. plugar as 14 variantes de mapa     (ALTO)   → Map10 fica vermelho (C), exceção documentada
7. gatilho ChapterTitle + transições  (ALTO)   → precedência: capítulo > transição > glitch (D)
—— então: report limpo nos 3 biomas + render da Austrália + comparação com o ouro

8. regra de uso de TEXTO/GRÁFICO      → FORA DESTE GO (E): rodada e espec próprias
```

**Escopo deste GO: passos 0-7.** Nenhuma dependência ficou em aberto — as 5 decisões do §4
estão fechadas, então a implementação pode correr do início ao fim sem parar para perguntar.

**Estimativa honesta:** os passos 1-4 são diretos e de baixo risco — mesma natureza da espec
irmã, que correu limpa. Do 5 em diante muda a categoria: é código que decide o que aparece.
Se o tempo apertar, **entregar 1-5 e parar** é uma entrega coerente e segura: a aparência fica
toda no lugar novo e o filtro protege o que vier depois.

> 🛡️ **A resposta E removeu um risco inteiro:** com o passo 8 fora, o **`ilustrar.py` não é
> tocado** neste GO. Ele é o pass do Sistema 1, que está em produção e acabou de receber os
> valores novos das 13 ilustrações. Mexer nele agora seria empilhar mudança sobre mudança
> recém-feita — exatamente o tipo de coisa que o `feedback_aditivo_apenas` proíbe.
