# RELATÓRIO DE IMPLEMENTAÇÃO — ESPEC-EDICAO-ADITIVOS

> **Para:** quem for executar a `ESPEC-EDICAO-ADITIVOS.md` (eu mesmo, numa sessão futura, ou
> outro agente). **Escrito em 04/08/2026, ANTES do GO.**
>
> 🔴 **Nada aqui é executado sem GO explícito do operador.** Ele revisa este documento antes.
>
> **Âncora:** este relatório não inventa regra nova. Tudo o que ele manda fazer (e proibir)
> vem de documentos e decisões REAIS do projeto:
> `director/CONTRATO-EDITORIAL.md` · `ESPEC-VIDMATOR.md` · `ESPEC-IDENTIDADE-DOS-TAKES.md` ·
> `ESPEC-CONTRATO-VISUAL-EXECUCAO-REPRODUTIVEL.md` · `ESPEC-DINAMISMO-TEXTO.md` ·
> `RUNBOOK-SERVIDOR.md` · e as lições pagas em produção (registradas nos commits).

---

## 0. O CONTEXTO QUE NÃO PODE SER ESQUECIDO

O diretor está **validado em 3 biomas** (Amazônia, Austrália, África) e o operador aprovou os
três renders. O estado congelado é a tag **`diretor-doc-v1-validado`**. A espec inteira é de
**aditivos de EDIÇÃO** — ela existe para ampliar o repertório visual, **não** para mexer na
inteligência de escolha de takes, que é o que custou 9 erros e 12 correções para ficar de pé.

**A frase que governa esta implementação** (regra do repositório, `feedback_aditivo_apenas`):
> produção rodando diariamente; nada de refator oportunista, tudo novo isolado.

---

## 1. INVARIANTES DO PROJETO — quebrar qualquer um destes é regressão

Estes não são conselhos: são regras já validadas em produção. Cada uma nasceu de um erro real.

| # | Invariante | Onde nasceu |
|---|---|---|
| I1 | **Gated por preset.** Todo recurso novo tem knob; preset sem o knob = comportamento atual **byte a byte**. Os nichos do Piter (`estoicismo`, `documentario`, `true_crime`, `ttm`…) não podem mudar. | `ESPEC-VIDMATOR` (decisão D3) |
| I2 | **Teto de SFX 0.20**, e os calibrados a 0.15 são **imutáveis**. | política permanente do operador (03/08) |
| I3 | **Transição única — nunca empilhar.** Entrada/saída de overlay É transição; a transição própria da cena vira corte seco. | Contrato Editorial, grupo E |
| I4 | **Camada que se MOVE tem que ser MAIOR que o quadro** (folga ≥ 3× o deslocamento). | N9 — falha nas bordas do parallax |
| I5 | **Estado mínimo ~1s**: nada aparece por menos de ~1s (parece erro, não dinamismo). | Contrato Editorial, grupo E |
| I6 | **O word-pop é dono da tela**: nenhum overlay pode entrar dentro da janela dele. A guarda roda no ÚLTIMO pass, sobre os valores finais. | N4 + N8 |
| I7 | **Cena com sujeito obrigatório re-resolve SEMPRE por `resolver_identidade`** — nunca por `resolve()` cru, em nenhum caminho. | o bug do leão |
| I8 | **Mudou regra que o vet aplica → bumpar `VALIDATOR_VERSION`** (senão decisões antigas ficam válidas indevidamente no modo `production`). | `ESPEC-CONTRATO-VISUAL` (B1-5) |
| I9 | **O render é materialização, não validação**: nada entra sem passar pelo PRE-RENDER REPORT + lock. | auditoria 3 |
| I10 | **JS dentro de string Python** (no AC-Automator): `\n` vira `\\n`. Não se aplica ao Vidmator (React de verdade), mas vale para qualquer patch no `app.py`. | `CLAUDE.md` |
| I11 | **NÃO bumpar `VALIDATOR_VERSION` nesta espec.** É o converso do I8: nenhum dos 6 passos muda regra que o vet aplica (são todos de RENDER, não de escolha de mídia). Bumpar sem motivo invalida todas as decisões cacheadas e força re-resolver os 3 biomas do zero — caro e sem ganho. | `ESPEC-CONTRATO-VISUAL` (B1-5), lido ao contrário |
| I12 | **Asset novo entra pela convenção do `preparar_render.py`**: `_copy_asset`/`shutil.copy2` para `PUBLIC_TEST` + chave `*_rel` no dict `render`, e **arquivo ausente ⇒ `_rel = None` + AVISO no log, degradando limpo** (é assim que o `enum_click`, o `mood_semtexto` e o `sfx_counter` já fazem). É o que materializa o I1 na prática. | `preparar_render.py:17-45, 227-259` |

---

## 2. ORDEM DE EXECUÇÃO (do menor risco para o maior)

> Cada passo tem: o que fazer · arquivos · como validar. **Commit por passo**, nunca um
> commit gigante — se algo quebrar, o `git revert` tem que ser cirúrgico.

### PASSO 0 — Preparo *(2 minutos, obrigatório)*

Branch a partir da tag validada, e **medir a baseline antes de tocar em nada**:
`npx tsc --noEmit | grep -c "error TS"` → tem que dar **47** (reconferido em 04/08). Se der
outro número, a baseline mudou e o critério de validação dos passos seguintes é esse novo
número — não o 47 escrito aqui.

---

### PASSO 1 — Valores dos templates de apresentação *(risco: baixo)*

**O quê:** aplicar em `Presentacao.tsx` os valores do **Item 3** (lupa, spotlight, polaroid,
reveal, split, grid). `kenburns` e `parallax` ficam **IGUAIS** — não tocar.

**Arquivo:** `remotion/src/compositions/Presentacao.tsx`

**➡️ Usar o `MAPA DE APLICAÇÃO` da espec** (rótulo → variável → linha). Ele existe porque os
valores foram anotados com os rótulos em português do HTML de refino, que **não** são os
nomes do código. Três são ambíguos e estão desambiguados lá: `amplitude` (é um controle para
duas oscilações — a vertical é derivada por ×0.64 na lupa e ×0.66 no spotlight),
`inclinação −3°` (é o fim do spring, que o código **já** atinge → não mexer na entrada do
polaroid) e `brilho da linha` (alimenta blur **e** spread do `boxShadow`).

**Cuidados:**
- **Escopo decidido (§4-A): os valores são BASE DE TODOS OS DIRETORES** — aplicar direto no
  componente, sem gating por preset. A mudança alcança os nichos do Piter por decisão
  explícita do operador.
- **Só sete valores realmente mudam.** O mapa marca em negrito quais; os demais já são
  iguais ao código. "Aplicar tudo" mexe à toa em coisa que está no render aprovado.
- **Não transformar `spec.foco` em constante.** O HTML fixou o centro em `50/58` porque era
  demo; o código lê `fx/fy` do `foco` que o LLM escolhe por cena
  ([Presentacao.tsx:29](remotion/src/compositions/Presentacao.tsx#L29)). Trocar por 50/58
  mata o enquadramento e **não gera erro nenhum** — regressão silenciosa.
- O `film` **já está aplicado** — não reaplicar nem "melhorar".
- Estes templates são usados por **todos os nichos** (o `apresentar.py` não é gated por
  preset). Mudar os números muda os vídeos do Piter também. **Se o operador quiser preservar
  os nichos dele, os valores precisam vir do preset** — é uma decisão a confirmar no GO
  (ver §4, pergunta A).
- Ao mexer no `polaroid`, usar **margem de baixo 40** (valor testado COM a legenda), não o 90
  do item 3. A espec já traz o aviso.

**Validação:** `npx tsc --noEmit` (baseline: **47 erros pré-existentes** — nem um a mais) +
render de 5s pelo `FilmGridTest` adaptado, ou conferência no `apresentacoes_refino.html`.

---

### PASSO 2 — Legenda do `polaroid` *(risco: baixo)*

**O quê:** Item 3.1 — trocar a fonte manuscrita por **Courier New** (49/400/2.5, cor #2a2a2a,
distância 18) e fazer o LLM preencher `legenda`.

**Arquivos:** `remotion/src/compositions/Presentacao.tsx` (fonte) · `director/apresentar.py`
(prompt do LLM devolve `legenda`).

**Cuidados:**
- O campo `spec.legenda` **já existe e já é renderizado** — não criar campo novo.
- No prompt do `apresentar.py`, a legenda tem que ser **curta (1-3 palavras)** e **falada/
  factual**: o Contrato Editorial (grupo B) proíbe inventar informação que não está no
  roteiro. Se o LLM não souber o lugar, devolve vazio — melhor tarja vazia que legenda falsa.
- Sem legenda, a tarja mantém a altura (a moldura não pode "pular" entre cenas).

**Validação:** duas cenas de polaroid — uma com legenda, uma sem.

---

### PASSO 3 — Ilustrações: valores + correção do `stat` *(risco: baixo)*

**O quê:** Item 5 — aplicar os valores das 13 ilustrações e a **correção 5.1**: a linha do
`stat` cresce **do centro para os dois lados** — `x1 = 210 − 150·p`, `x2 = 210 + 150·p`
(cx = 210, largura total 300) — no lugar do `dashP(p)`.

**Arquivo:** `remotion/src/compositions/Illustration.tsx`

**Cuidados:**
- `Illustration.tsx` tem coerção defensiva (`txt()`, `num()`, `asList()`) porque **o LLM gera
  shapes imprevisíveis**. Ao mexer nos componentes, **não remover** essas guardas nem o
  `<Safe>` (ErrorBoundary) — sem eles, um spec ruim derruba o render inteiro.
- A escala vem de `baseBig` + `spec.tamanho`. Se os valores novos já embutem tamanho, **não
  somar escala duas vezes** (conferir no primeiro render).
- **`lw` (largura da linha) = 300 é a largura TOTAL**, não o `x2=300` de hoje. Mesma pegadinha
  no `bar`: `larg`(500) é o gráfico, `bw`(80) é a barra. Ler o mapa da espec antes de digitar.
- O mesmo padrão de revelação unidirecional existe no `steps`, `definition` e `line`. **Só o
  `stat` foi reclamado** — não "consertar" os outros por conta própria.

**Validação:** `ilustracoes_refino.html` lado a lado com um render real; conferir que uma
ilustração com dados malformados ainda não derruba o vídeo.

---

### PASSO 4 — SFX do projetor no `film` *(risco: médio — mexe em áudio)*

**O quê:** Item 2 — sempre que a apresentação `film` for usada, tocar
`machine_projetor_filme_01.mp3` a **0.20**.

**Arquivos:** `remotion/preparar_render.py` (copiar o SFX + expor o rel) ·
`remotion/src/compositions/BrollTest.tsx` (tocar junto da cena que tem `presentacao.tipo === "film"`).

**Cuidados:**
- **I12 — seguir a convenção**: copiar para `PUBLIC_TEST`, expor `sfx_projetor_rel` no dict
  `render`, e **se o arquivo não existir → `None` + AVISO no log** (igual ao `enum_click` em
  `preparar_render.py:227`). Sem esse degradê limpo, uma máquina sem o acervo quebra o render.
- **I2**: 0.20 é o teto. Passar disso viola a política permanente.
- O SFX tem **99,9 s** (medido) e a cena tem ~6 s: o `<Audio>` corta sozinho no fim da `<Sequence>` —
  não é preciso trim, mas **é preciso** que ele esteja dentro da Sequence da cena, senão toca
  o vídeo inteiro.
- Hoje isso só existe em `FilmGridTest.tsx` (composição de teste). **Não** plugar a
  composição de teste na produção — reimplementar no `BrollTest`.
- Conferir se o áudio realmente entrou: `ffprobe -show_entries stream=codec_type` tem que
  listar `audio`.

**Validação:** render de uma cena `film` e escuta; e um render **sem** `film` para garantir
que nada mudou.

---

### PASSO 5 — Tema do canal (grid claro/escuro) *(risco: médio)*

**O quê:** Item 2 — decisão fechada em **§4-B**: o tema mora no **cadastro do canal** (UI
futura, com a marca "edição simples | Vidmator"); até a UI existir, campo interino na config
do canal que o bridge injeta no timeline como `tema_visual`. Tema claro → **somente**
`grid_claro`; escuro → `grid_escuro`; ausente → `escuro`. (Mapas pelo tema = escopo futuro
anunciado, NÃO entra agora.)

**Arquivos:** config do canal/execução (interino, ver §4-B) ·
`director/montar_timeline.py` ou o bridge (gravar `tema_visual` no timeline) ·
`remotion/preparar_render.py` (copiar o grid + passar o rel) ·
`remotion/src/compositions/BrollTest.tsx` (passar `fundoRel` ao `<Presentacao>`).

**Cuidados:**
- A prop `fundoRel` **já existe** no `Presentacao` e é opcional — sem ela, fundo preto (o
  comportamento atual). Isso é o que garante **I1**.
- **Fallback:** sem tema definido → `escuro`.
- O grid é imagem **estática** cobrindo 100% — não cai no **I4** (não se move).
- Hoje só o `film` consome `fundoRel`, e o `<Presentacao>` é chamado **sem essa prop**
  ([BrollTest.tsx:204](remotion/src/compositions/BrollTest.tsx#L204)) — é aí que ela passa.
- **Alcance corrigido (conferido no código, ver tabela na espec):** estender para
  **`polaroid`** e **`grid`**. O **`split` NÃO entra** — são duas imagens cobrindo 100% da
  tela, não existe fundo visível ali. O relatório anterior dizia "split" por engano.
- Staging do grid pela convenção do **I12** (`_copy_asset` re-encoda imagem: os JPGs de
  2560×1440 do Canva passam por ele sem problema, estão abaixo do teto de 3200px).

**Validação:** um canal de cada tema, comparando o mesmo trecho.

---

### PASSO 6 — Transições Apagão de Filme + Brilho 2 *(⏳ FORA DESTE GO — §4-C)*

> O operador adiou a regra de frequência (*"vamos definir depois"*). **Este passo NÃO é
> executado nesta rodada** — fica documentado, com todos os cuidados prontos, para a rodada
> própria quando a regra existir.

**O quê:** Item 1 — os dois clipes entram **por cima** de um corte seco entre dois takes, com
`foley_camera_shutter_01.mp3` a **0.15**.

**Arquivos:** `remotion/preparar_render.py` (copiar os 2 mp4 + o sfx) ·
`remotion/src/compositions/BrollTest.tsx` (camada nova acima da Camada 1) ·
`director/efeitos.py` ou pass novo (decidir ONDE cada transição entra).

**Cuidados (o passo mais perigoso):**
- 🚨 **NÃO copiar o `mixBlendMode: "screen"` do overlay existente para as duas.** É o erro
  mais fácil de cometer aqui, porque é a convenção de toda a casa. Medição (`signalstats`):
  o **Brilho 2** é um estouro de luz sobre preto → `screen` ✅; o **Apagão de Filme** é preto
  na maior parte dos 24 frames → em `screen` o preto é neutro e **a transição fica
  invisível**, virando um segundo "brilho". Apagão pede composição **normal** (ou `multiply`).
- **Frames, não segundos:** os clipes são **24 fps** e o projeto é **30**. Apagão = 30 frames,
  Brilho = 21 (0,708 × 30 = 21,25 → arredondar). Não herdar a contagem do arquivo.
- **I3 é o risco central.** A Camada 1 já tem lógica de transição única para o pacote
  só-filtro e para a saída do mapa (`pkgScenes`, `mapaCorteEntra`). A transição nova **tem
  que entrar no mesmo predicado** — se ela aparecer numa cena que já tem glitch ou saída de
  mapa, empilha, e isso já foi reprovado pelo operador em vídeo.
- O operador disse **"não obrigatoriamente sempre"**: precisa de uma regra de frequência.
  **Não inventar** — ver §4, pergunta C.
- **I5**: os clipes têm 1,02 s e 0,72 s. Se a cena for curta, a transição come a cena inteira.
  Definir duração mínima de cena para poder receber a transição.
- Estes são os únicos itens da espec que tocam a **Camada 1** (o b-roll). Um erro aqui
  aparece em todas as cenas. **Deixar por último** e validar com render completo.

**Validação:** render completo de um dos 3 biomas validados + comparação frame a frame nos
cortes; o PRE-RENDER REPORT tem que continuar limpo.

---

## 3. ERROS A EVITAR (todos já aconteceram neste projeto)

| Erro | O que aconteceu | Como não repetir |
|---|---|---|
| **Mexer no que já está aprovado** | — | Item 4 (entradas de texto) está ✅ **aprovado**: não tocar. `kenburns` e `parallax`: **IGUAIS**. `film`: já aplicado. |
| **Caminho posterior atropela decisão validada** | o orçamento de repetição e o invariante de saída re-resolviam com `resolve()` cru e furavam o gate regional (o leão) | Qualquer código novo que **substitua mídia** tem que respeitar **I7**. |
| **Overlay criado por pass posterior fura a janela** | o `ilustrar` roda depois do `enumeracoes`; a supressão não via a ilustração (o "1 Million") | **I6**: a guarda final é no `legibilidade` (último pass). Se um pass novo criar overlay, ele entra **antes** do `legibilidade` ou a guarda não pega. |
| **Camada animada do tamanho exato do quadro** | faixa sem escurecimento na borda do parallax | **I4** |
| **Empilhar transição** | glitch + slide na mesma cena; take cortado ao meio | **I3** |
| **Achar que "está centralizado" olhando só o fim** | a linha do `stat` está centralizada no fim, mas descentralizada durante a animação inteira | Validar animação em **vários instantes**, não só no frame final. |
| **Assumir que o valor da espec é o último** | a margem de baixo do polaroid foi 90 e virou 40 quando a legenda entrou | Ler os **avisos ⚠️** da espec antes de copiar número. |
| **Render como forma de descobrir erro** | ciclo render→erro→render que o operador rejeitou | **I9**: report + lock antes de renderizar. |
| **Commit gigante** | — | Um commit por passo, com mensagem explicando **o porquê**, não só o quê. |
| **Deixar composição de teste virar produção** | — | `FilmGridTest` e `_render_filmgrid.mjs` são **teste**; a produção é o `BrollTest`. |
| **Copiar a convenção da casa sem olhar o asset** | (achado em 04/08, antes de custar render) o `mixBlendMode:"screen"` de todos os overlays CapCut anularia o Apagão, que é preto | Medir o clipe (`signalstats`) antes de escolher o blend. Preto = `screen` é no-op. |
| **Traduzir rótulo de HTML direto para código** | `amplitude` é 1 controle para 2 oscilações; `largura da linha` do `stat` é o total, não o `x2` | Usar o **MAPA DE APLICAÇÃO** da espec, nunca o número solto. |
| **Bumpar `VALIDATOR_VERSION` "por garantia"** | invalidaria as decisões cacheadas dos 3 biomas e forçaria re-resolver tudo | **I11**: nenhum passo desta espec muda regra de vet. |

---

## 4. PERGUNTAS — ✅ RESPONDIDAS PELO OPERADOR (04/08, pré-GO)

**A. Escopo dos valores → TODOS OS DIRETORES, atuais e futuros.** Palavras do operador:
*"os valores dos templates valem para TODOS OS DIRETORES... tem coisas que todos os
diretores terão como base"*. A base comum de todo diretor é: **estrutura invisível**
(ajusta-se só o direcionamento por nicho) + **a configuração destes assets** (apresentações,
entradas de texto, ilustrações do Sistema 1, e futuramente o acervo).
→ **Consequência prática:** aplicar direto em `Presentacao.tsx`/`Illustration.tsx`, SEM
gating por preset. Isso muda também os vídeos do Piter — **decisão explícita do operador**,
não é violação do I1 (o I1 protege contra mudança *acidental*; esta é deliberada e
registrada). Isso simplifica os passos 1-3.

**B. Onde mora o `tema` → no CADASTRO DO CANAL, na UI.** O cadastro de canais ganha a marca
de tipo de edição — **"edição simples"** (o fluxo de hoje) ou **"Vidmator"** — e, quando
Vidmator, o campo **`tema: claro | escuro`**. O diretor considera o tema ao montar o vídeo:
tema claro → **somente** variantes com `grid_claro` no fundo; tema escuro → `grid_escuro`.
→ **Escopo futuro já anunciado:** o tema vai guiar também a **representação de MAPAS**
(cores do mapa pela cor do tema). NÃO implementar agora — registrado para não perder.
→ **Interino até a UI existir:** o campo entra na config do canal/execução que o bridge já
lê, injetado no timeline como `tema_visual`; fallback ausente = `escuro`. Quando a UI de
cadastro nascer, ela passa a ser a fonte — o resto do encanamento não muda.

**C. Frequência das transições (Item 1) → ⏳ ADIADA pelo operador** (*"vamos definir a regra
de frequência depois"*). → **Consequência:** o **PASSO 6 fica FORA deste GO**. O GO cobre os
passos 0-5; as transições Apagão/Brilho entram numa rodada própria quando a regra existir
(os cuidados do passo 6 — blend por clipe, 24→30fps, I3, I5 — ficam prontos aqui esperando).

**D. Sistema 1 × acervo → CONVIVEM.** O acervo NÃO substitui as ilustrações: depois desta
espec executada, o acervo ganha espec nova (refino das 40 variantes) e **é inserido também**,
como adição. Investir nos 13 componentes do Sistema 1 vale a pena — resposta dada.

---

## 5. PLANO DE VALIDAÇÃO (obrigatório antes de dizer "pronto")

> ⚠️ **O que cada gate cobre — não confundir.** O PRE-RENDER REPORT valida o **PLANO** (qual
> mídia entra em qual cena, janelas, repetição). Ele **não enxerga** mudança em
> `Presentacao.tsx` / `Illustration.tsx` — passos 1 a 3 podem estar visualmente quebrados com
> o report 100% limpo. Para esses, o gate real é `tsc` + **comparação visual**. O report só é
> gate de verdade no **passo 6**, que mexe em transição (camada 1).

1. **`npx tsc --noEmit`** → baseline **47 erros pré-existentes** (reconferido 04/08). Nem um a mais.
2. **PRE-RENDER REPORT limpo** nos 3 biomas em modo `production` (reuso do manifesto):
   0 erro, 0 hard miss. É o teste de **não-regressão do plano** — vale sobretudo para o passo 6.
3. **Render completo de 1 bioma** (sugestão: **Austrália** — tem word-pop, takes, mapa,
   infográfico e apresentações; é o mais completo dos três).
4. **Comparação com a versão de ouro**: `_renders_aprovados/2026-08-04_australia_croc_FINAL/`.
   Diferenças esperadas: só as da espec. Qualquer outra = regressão.
5. **Auditoria pós-render** (`auditoria_composicao.py`) sem violação nova.
6. **Escuta** do áudio: SFX novos dentro do teto, sem competir com a narração.

---

## 6. ROLLBACK

- Ponto seguro: **`git checkout diretor-doc-v1-validado`** (estado aprovado nos 3 biomas).
- Como cada passo é um commit isolado, `git revert <sha>` desfaz um item sem derrubar os outros.
- Os renders aprovados estão em `_renders_aprovados/` (fora do git, na máquina do operador) —
  servem de referência visual para comparar. Conferidos: `2026-08-04_australia_croc_FINAL`,
  `2026-08-04_africa_lion_FINAL`, `2026-08-03_amazonia_FINAL-VALIDADA`.
- ⚠️ **Os HTMLs de refino também estão fora do git** (`Desktop/*_refino.html`, ~5 MB no
  total). Eles foram a origem de todos os valores — mas o que o código precisa já está
  transcrito no **MAPA DE APLICAÇÃO** da espec. Se os HTMLs sumirem, nada se perde; se
  divergirem da espec, **a espec vence** (foi ela que o operador revisou).

---

## 7. ORDEM RESUMIDA

```
0. preparo (branch + baseline tsc = 47)
1. valores dos templates          (baixo)   → tsc + refino HTML     [§4-A: base global, sem preset]
2. legenda do polaroid            (baixo)   → 2 cenas de teste
3. ilustrações + correção do stat (baixo)   → animação em vários instantes
4. SFX do projetor no film        (médio)   → ffprobe + escuta
5. tema do canal (grids)          (médio)   → 1 canal de cada tema  [§4-B: decidido]
6. transições Apagão/Brilho       ⏳ FORA DESTE GO                  [§4-C: regra adiada]
—— então: report limpo nos 3 biomas + render da Austrália + comparação com a versão de ouro
```

**Estimativa honesta:** com A e B decididos (§4), os passos 0-5 são diretos e este GO os
cobre por inteiro. O 6 já tem dono de rodada própria — não entra aqui por decisão do
operador, não por corte de escopo meu.
