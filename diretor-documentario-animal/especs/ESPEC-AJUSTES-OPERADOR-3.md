# ESPEC — Ajustes do operador pós-prova de fogo Austrália (03/08, madrugada)

**Status:** ESCRITA, aguardando GO. **NÃO EXECUTAR AINDA.**
**Princípio (operador, literal):** *"NÃO ALTERE NADA DO QUE O DIRETOR JÁ CONHECE, PORQUE
ESSES ERROS SÃO ERROS NOVOS, ENTÃO O QUE TEMOS QUE FAZER É INSERIR ESSE CONHECIMENTO NELE."*
Toda mudança aqui é **aditiva** e cirúrgica: nenhuma regra já validada (Amazônia) é tocada.
Confirmado pelo operador: **nenhum erro da Amazônia reapareceu na Austrália** — as correções
antigas se mantiveram. Os 6 erros abaixo são inéditos, expostos pelo cenário adverso.

---

## Sumário dos erros novos

| # | Erro | Gravidade | Causa raiz (confirmada com evidência) |
|---|---|---|---|
| N1 | **Fauna de outro continente** (leoa africana e gato doméstico em doc da Austrália; lontra como "hunter") | 🔴 CRÍTICA (risco de hate) | Query de `class_member` é genérica + vet sem gate REGIONAL |
| N2 | **Mesmo clip em cenas adjacentes, com transição entre elas** (15→16, 18→19) | 🔴 ALTA | Cache do `resolve()` é por QUERY: passes que dão a MESMA query a várias cenas devolvem o MESMO arquivo |
| N3 | **Word-pop com corte/transição no meio** (janela atravessa 3 cenas) | 🔴 ALTA | Mesma raiz de N2 + janela do pop não força fundo contínuo |
| N4 | **AMBUSH sobreposto a outro texto** ("...ands of Species" por baixo) | 🟠 MÉDIA | `legibilidade` (R1) move `aparece_em` p/ `inicio+0.3` DEPOIS da supressão do `enumeracoes` → texto entra dentro da janela do pop |
| N5 | **Take fora de contexto** (estacionamento com carro, aéreo de estrada) | 🟠 MÉDIA | Resolução "pai-neutra" e cenas sem contrato passam SEM vet (só o `vet_relevancia` leniente) |
| N6 | **Repetição do mesmo take ao longo do vídeo** (mesmo croc várias vezes) | 🟠 MÉDIA | Sem orçamento global de repetição por arquivo |

Pendência anotada (NÃO nesta espec): **hook magnético** — regras específicas de
introdução (o trecho de 1:30-2:00 vira intro). Espec própria depois.

---

## N1 — Fauna de outro continente / predador fora de escala

### Sintomas (prints do operador)
- `00:01:24` "Ruled by Apex Predator" → **leoa africana na savana** (doc sobre Austrália).
- `00:01:31` "Hidden Death Lurks" → **gato doméstico preto e branco** na grama.
- `00:00:06` "Rivers Hide Hunters" → **lontra** (predador, mas não à altura do tema).

### Evidência (manifesto + timeline da Austrália)
```
cena 20  query='Apex predator dominating landscape'
         contrato=[('apex predator', class_member, ['saltwater crocodile','dingo',
                    'white-bellied sea eagle','bull shark'])]
         id_log: id695ac21c349e.mp4 -> PASS na 1ª tentativa, violacoes=[]
cena 21  query='Hidden predator stalking prey'   contrato=[]  (sem sujeito -> sem vet)
cena  2  query='Mangrove river landscape wild'
         contrato=[('hunter', class_member, ['saltwater crocodile','sea eagle',...])]
```

### Causa raiz (dupla)
1. **A query não nomeia membro regional.** O contrato tem o vocabulário certo
   (`biomas.json → australia`), mas a query enviada ao Pexels é o termo GENÉRICO
   ("apex predator", "hidden predator") — que qualquer predador do planeta satisfaz.
   `variantes_query()` preserva o *canônico*, e o canônico aqui É genérico. A resolução
   por MEMBRO existe hoje só para takes fundidos, não para cenas
   ([resolver_cascata.py](director/resolver_cascata.py) `variantes_query`).
2. **O vet não tem gate regional.** O prompt de `class_member` diz *"Accept ANY other
   clearly identifiable real member of the class that is plausible for the biome"*
   (hardening B1-2 da auditoria 2: lista é seed, não whitelist). O Vision leu "leão = apex
   predator real" e deu PASS — **nunca foi perguntado se a espécie ocorre na região**.

### Correção proposta (aditiva)
**(a) Query por membro nas cenas de identidade** — em `resolver_identidade`, quando
`match_mode=class_member` e o contrato tem `allowed_members`, as PRIMEIRAS variantes passam
a ser por membro concreto (`saltwater crocodile river ambush`, `dingo hunting outback`...),
antes de tentar a query genérica. É exatamente o mecanismo que já salvou o take
birds+butterflies (variantes por membro primeiro) — estendido para cenas.

**(b) Gate REGIONAL no vet** — nova violação `wrong_region`, com o bioma injetado
explicitamente:
```
REGION: this documentary is about {BIOMA}. The species shown must plausibly OCCUR
in that region. A species native to a different continent (e.g. an African lion in an
Australian documentary) is a FAIL with violation wrong_region — no matter how well it
fits the class. Domestic/pet animals (house cat, dog, farm animals) are ALWAYS a FAIL.
```
Mantém o hardening B1-2 (aceita membro fora da lista), mas **só se for da região**.

**(c) Escala do predador** — acrescentar ao `broll_diretrizes` do preset: quando a fala usa
termo genérico de predador e o documentário tem um protagonista declarado, o take deve
mostrar **o protagonista ou um predador de porte equivalente da mesma região** — nunca um
predador menor/incidental (lontra) quando o tema é um predador de topo.

**(d) `biomas.json`** — acrescentar por bioma a chave `_regiao` (nome legível: "northern
Australia", "African savanna") usada no prompt do gate regional. Aditivo; biomas sem a
chave usam o `documentary_setting`.

### Gate no PRE-RENDER REPORT
Nova checagem determinística: se a decisão de uma cena/take de identidade tem
`class_member` com `allowed_members` e a **query final não contém nenhum dos membros nem o
termo regional** → **AVISO** "query genérica de classe (risco de fauna fora da região)".
`wrong_region` no id_log → **ERRO**.

---

## N2 + N3 — Mesmo clip em cenas adjacentes e word-pop cortado

> Os dois têm a **mesma causa raiz** e por isso vêm juntos.

### Sintomas
- Take aéreo do crocodilo aparece, **transição**, e volta o **mesmo take** (prints 2 e 3).
- Word-pop "RIVERS": tela dividida no meio — **transição acontecendo durante o word-pop**
  (print 5). Regra do operador: *"sempre que tiver word-pop, deve ser um único take de
  fundo, do início ao fim, sem transições no meio, sem mudar o take de fundo."*

### Evidência
```
cenas ADJACENTES com MESMO arquivo:
  cena 15 -> 16: MESMO clip (v8ab877600dc4.mp4), transicao=slide_left
  cena 18 -> 19: MESMO clip (v2f7c45ca8674.mp4), transicao=whip

janela do word-pop 52.9-59.98s atravessa 3 cenas:
  cena 15 (47.57-53.53) · cena 16 (53.53-59.08) · cena 17 (59.08-67.46)
  -> transições slide_left @53.53 e zoom @59.08 DENTRO/na borda da janela
```

### Causa raiz
O `resolve()` cacheia por **hash da query** (`index[qhash(query)]`). Dois passes atribuem
deliberadamente a **mesma query a várias cenas**:
- `enumeracoes` → `stock_query_override = fundo_query` para TODAS as cenas da janela do pop
  (cenas 15, 16, 17);
- `resolver_cascata` → cena-pai-neutra usa `f"{BIOMA} canopy atmosphere"` para cada cena
  coberta (cenas 18 e 19).

Resultado: mesma query → **mesmo arquivo do cache** → cenas vizinhas idênticas, com a
transição da timeline no meio. Não é aleatoriedade; é determinístico.

### Correção proposta (aditiva)
**(a) Fundo ÚNICO e contínuo no word-pop (regra do operador).** A janela do pop passa a ter
**um só clip de fundo, sem cortes**: as cenas cobertas pela janela viram um bloco contínuo
— o clip da primeira cena da janela é estendido até o fim da janela e as transições internas
viram `none`. Implementação preferida (menor superfície): marcar as cenas da janela com
`pop_bg_group: <id>` no `enumeracoes`; no `BrollTest`, o grupo renderiza **um único
`SceneClip`** cobrindo a janela (as demais cenas do grupo não montam camada 1), e as
transições internas somem. Alternativa (se a fusão no player der trabalho): fundir as cenas
no próprio timeline antes do resolver.

**(b) Anti-adjacência dura.** Invariante novo no resolver: **duas cenas consecutivas nunca
podem apontar o mesmo arquivo**. Ao detectar, re-resolve a segunda com
`evitar={arquivo_da_anterior}`. Vale também para pai-neutra (passar `evitar` com os
arquivos já usados nas cenas vizinhas).

**(c) Cache com desempate.** Para queries repetidas de propósito (fundo do pop,
pai-neutra), o `resolve()` deve aceitar um parâmetro `variar=True` que pula o hit de cache
e pede o **próximo** candidato da lista do Pexels, em vez de devolver o mesmo arquivo.

### Gate no PRE-RENDER REPORT
- **ERRO**: duas cenas consecutivas com o mesmo `clip_path`.
- **ERRO**: janela de word-pop cobrindo mais de um clip de fundo (ou com transição interna).

---

## N4 — Word-pop sobreposto por outro texto

### Sintoma
`00:00:59`: "AMBUSH" (word-pop) na tela e, por baixo, "**...ands of Species**" — dois textos
simultâneos, ilegíveis. Regra do operador: *"o AMBUSH deveria ter saído na troca de take,
pois o take deveria ir até o final da palavra AMBUSH, e as palavras dinâmicas que estão
aparecendo ao fundo deveriam aparecer sem nada por cima delas."*

### Evidência (a mais bonita da rodada)
```
cena 17: aparece_em=59.38   aparece_em_orig=62.59   texto='Shelters Thousands of Species'
janela do word-pop: 52.9 -> 59.98
```
O `enumeracoes` roda ANTES e suprime overlays cujo `aparece_em` cai na janela — na hora
dele, o texto da cena 17 estava em **62.59** (fora da janela, corretamente preservado).
Depois, o `legibilidade` (regra R1, validada na Amazônia) reescreve
`aparece_em = inicio + 0.3` = **59.38** — que cai **dentro** da janela do pop. A supressão
foi feita sobre um valor que deixou de valer.

**Ordem dos passes:** `enumeracoes` → `legibilidade`. O `legibilidade` já respeita as
janelas do pop no TETO (`texto_ate`), mas **não** na entrada (`aparece_em`).

### Correção proposta (aditiva, 1 regra em 1 lugar)
Em `legibilidade.py`, ao aplicar R1: se o novo `aparece_em` cair dentro de uma janela de
enumeração, **empurrar a entrada para o fim da janela** (+0,3s de folga). Se com isso a
exibição ficar menor que o mínimo legível (1,2s), **suprimir o overlay** (o word-pop é o
dono da tela — a regra já existente). O texto do word-pop nunca divide a tela com outro
texto, e a palavra final do pop chega inteira até o corte.

### Gate no PRE-RENDER REPORT
**ERRO**: `aparece_em` (final, pós-legibilidade) de qualquer overlay dentro de uma janela de
enumeração. Este check pega a classe inteira, não só o caso do AMBUSH.

---

## N5 — Take fora de contexto (civilização em doc de natureza)

### Sintoma
`00:01:09`: **vista aérea de estacionamento com carro e estrada asfaltada** — repetida em
cenas seguidas, num documentário sobre crocodilo. Também é o mesmo clip do N2 (cenas 18/19).

### Causa raiz
Duas portas sem vet:
1. **Cena "pai-neutra"** — a re-resolução neutra chama `resolve()` direto, **sem vet nenhum**
   (nem contrato, nem relevância). Foi por aí que o estacionamento entrou.
2. **Cenas sem `required_subjects`** — passam só pelo `vet_relevancia`, que é
   deliberadamente leniente ("rejeita só o que é claramente nada a ver").

### Correção proposta (aditiva)
**(a) Vet de atmosfera para o doc realista.** Nova checagem, ligada por preset
(`vet_atmosfera: true`), aplicada a cenas SEM sujeito obrigatório e à pai-neutra:
reprova frames com **sinais dominantes de civilização** — estrada asfaltada, carro,
estacionamento, prédio, poste, cerca, placa — quando o documentário é de natureza. Reusa a
infra do `vet_contrato` (mesmos frames do subclip, saída estruturada) com uma violação nova
`civilizacao`. Barato: 1 frame por cena de atmosfera.

**(b) Pai-neutra passa a ser validada** — a resolução neutra usa o mesmo gate acima e, se
reprovar, tenta a próxima variante da atmosfera do bioma.

**(c) `broll_diretrizes`** ganha a linha explícita: nenhuma estrada/carro/estacionamento/
construção em documentário de natureza, salvo se a fala mencionar presença humana.

### Gate no PRE-RENDER REPORT
Violação `civilizacao` no manifesto → **ERRO**. (O report já mostra a thumb — o operador vê
o estacionamento antes do render.)

---

## N6 — Repetição do mesmo take ao longo do vídeo

### Sintoma / regra do operador (literal)
*"nenhum take deve repetir por mais de 2x… alguns, POUCOS, podem repetir SE NÃO ACHAR TAKE
DIFERENTE para colocar no lugar, no máximo 2x, com diferença de pelo menos 7-9 cenas um do
outro."* E: *"preferível take do mesmo animal mas take diferente."*

### Correção proposta (aditiva)
**Orçamento de repetição** no resolver, com esta ordem de tentativa:
1. arquivo **inédito** para a cena (preferência absoluta);
2. se o pool se esgotar: permite **2ª e última** aparição de um arquivo já usado, desde que
   a distância seja **≥ 7 cenas**;
3. nunca uma 3ª aparição; nunca 2 aparições a menos de 7 cenas.

Implementação: contador `_USOS_ARQUIVO {arquivo: [idx_cenas]}` alimentado à medida que as
cenas resolvem; `evitar` recebe automaticamente os arquivos que violariam a regra. Para
identidade, isso combina com N1(a): variar o MEMBRO da classe rende take diferente do mesmo
animal (crocodilo nadando / crocodilo na margem / olhos na água) em vez do mesmo clip.

### Gate no PRE-RENDER REPORT
- **ERRO**: arquivo usado 3+ vezes.
- **ERRO**: mesmo arquivo com distância < 7 cenas.
- Tag por cena: `reuso 2/2 (cena N)` para o operador ver onde o pool se esgotou.

---

## Ordem de execução proposta

| Bloco | Conteúdo | Risco de regressão |
|---|---|---|
| **1** | N4 (janela do pop no `legibilidade`) + N2(b) anti-adjacência + N6 (orçamento de repetição) | Baixo — regras locais, sem tocar em resolução |
| **2** | N1 (query por membro + gate regional + escala do predador) | Médio — mexe no vet: exige bump de `VALIDATOR_VERSION` |
| **3** | N3 (fundo único do word-pop) — envolve `BrollTest` | Médio — camada 1 do player |
| **4** | N5 (vet de atmosfera + pai-neutra validada) | Baixo — gate novo, gated por preset |
| **5** | Gates novos no PRE-RENDER REPORT (todos os acima) | Nenhum — só detecção |

**Validação:** re-rodar a **Austrália** em modo `surgical` nas cenas afetadas (2, 15-22) e
conferir no report; só então render. A **Amazônia final validada** deve ser re-rodada em
modo `production` como **teste de não-regressão** — o plano tem de continuar limpo, provando
que nenhuma regra antiga foi quebrada.

## O que NÃO mexer (validado, não tocar)

Word-pop (formato/escurecimento 30%), micro-takes com flash+SFX, pacote só-filtro
(mood tenso), transição única, estado mínimo ~1s, hierarquia do mapa (legenda/local),
saída de elemento por cima da cena entrante, contrato visual/vet estruturado, modos
production/surgical, teto de SFX 0,20, SFX de contador, degradação digna (hard miss).

---

# PARTE 2 — Erros da África (`africa_lion.mp4`)

**Veredito do operador:** *"mesmos erros da Austrália, uns mais pesados, outros mais leves"*
— N1..N6 **confirmados** num segundo bioma (não são acaso da Austrália):
- **N2/N3** — take do leão cortado no meio por uma transição para entrar o "8 km";
- **N4** — overlay entrando por trás durante o word-pop (agora com causa raiz DIFERENTE,
  ver N8);
- **N5/N6** — take de contexto pobre e repetição.

Mais **três erros inéditos**:

## N7 — Conteúdo de tragédia sem a fala pedir 🔴

### Sintoma
`00:01:02`: **queimada/incêndio** (fumaça e linhas de fogo em plano aéreo) enquanto a
narração fala de vida e abundância. Regra do operador: *"nem está falando de queimada,
tragédias devem estar em textos de tragédias."*

### Causa raiz
Mesma porta do **N5**: cena de atmosfera (sem `required_subjects`) resolve com o
`vet_relevancia` leniente, que só reprova o que é "claramente nada a ver". Fogo numa savana
é visualmente plausível como "savana" — passa. Não existe nenhuma regra que trate
**conteúdo de tragédia** como categoria própria.

### Correção proposta (aditiva)
Ampliar o **vet de atmosfera** do N5 com uma segunda categoria de veto — **conteúdo
trágico/catastrófico**: incêndio, queimada, desmatamento, poluição, carcaça, animal ferido
ou morto, seca extrema, enchente, destruição. Só entra se a fala **explicitamente** tratar
do tema (violação nova: `tragedia_sem_fala`). Vale para cenas de atmosfera E para o
material do fundo do word-pop.
Acrescentar ao `broll_diretrizes`: *"Tragedy footage (fire, deforestation, pollution,
carcasses, dead or injured animals, disaster) ONLY when the narration explicitly addresses
it. Never as generic mood."*

### Gate no PRE-RENDER REPORT
Violação `tragedia_sem_fala` → **ERRO**. (A thumb no report mostra a queimada antes do render.)

---

## N8 — Overlay/número entrando dentro do word-pop 🟠

### Sintoma
`00:00:54`: "**1Million / Animals Sheltered**" sobreposto ao word-pop "**THUNDER**" — dois
elementos disputando a tela. Aos `00:00:58` (print 2) o mesmo elemento aparece sozinho e
**faz sentido**. Regra do operador: *"pode continuar essa animação, mas ela deve entrar na
hora certa."*

### Evidência (causa raiz DIFERENTE do N4 — atenção)
```
janela do word-pop:  50.9 -> 57.82   (thunder@54.46)
log do pass ilustrar: "53.4s  grafico  stat  pos=center"
ordem executada:  ... enumeracoes ... -> resolver -> ... -> ilustrar -> apresentar -> legibilidade
```
O `enumeracoes` **suprime** overlays que colidem com a janela — mas ele roda **antes** do
`ilustrar`. A ilustração de 53,4 s **ainda não existia** quando a supressão passou. Não é o
mesmo bug do N4 (lá o `legibilidade` movia um overlay já existente); aqui um pass posterior
**cria** um overlay dentro da janela.

### Correção proposta (aditiva) — resolve N4 e N8 de uma vez
**Guarda final das janelas de enumeração**, no ÚLTIMO pass da cadeia (`legibilidade`):
varredura de TODOS os overlays (`texto_impacto`, `infografico`, `ilustracao`) contra as
janelas do pop, já com os valores FINAIS de `aparece_em`:
1. overlay inteiramente dentro da janela → **suprimido** (o word-pop é dono da tela — regra
   já existente, agora aplicada no momento certo);
2. overlay que só *começa* dentro → **empurrado** para o fim da janela +0,3 s; se sobrar
   menos que o mínimo legível, suprime.

Assim a supressão passa a ser feita sobre o estado final da timeline, e nenhum pass futuro
consegue furar a janela sem ser pego.

**Regra irmã (âncora de número):** para `infografico`/`counter`, a entrada deve continuar
ancorada na **palavra falada** (`aparece_em_orig`), não no início da cena — número entra
quando o número é dito. A R1 (texto amarrado à cena) continua valendo para `texto_impacto`.

### Gate no PRE-RENDER REPORT
Já coberto pelo gate do N4 (qualquer `aparece_em` final dentro de janela de enumeração =
ERRO) — este caso teria sido pego lá.

### Observação (menor, sem ação nesta espec)
A fala diz *"shelters **millions** of animals"* e o gráfico diz *"**1 Million**"*. O número
foi criado pelo `ilustrar`, não está na narração. Anotado para a espec de **fidelidade
numérica** (número em tela deve sair da fala, nunca ser inventado).

---

## N9 — Falha do overlay nas bordas em cenas com movimento 🟠

### Sintoma (achado do operador, com solução própria)
Faixa vertical **sem escurecimento** na borda: *"começa com essa falha no lado esquerdo e
termina com a falha no lado direito"*. Sugestão dele: *"deixar o mood 'maior' do que o 16:9,
pq nesses efeitos de movimento ele nunca vai ser 'menor' do que o take de fundo."*

### Causa raiz (confirmada no código — a descrição do operador bateu exata)
[Presentacao.tsx:60](remotion/src/compositions/Presentacao.tsx#L60), apresentação `parallax`:
```jsx
const fg = interpolate(pp, [0, 1], [22, -22]);
...
<AbsoluteFill style={{ transform: `translateX(${fg}px)`,
                       background: "radial-gradient(... rgba(0,0,0,0.5) 100%)" }} />
```
A camada de escurecimento é um `AbsoluteFill` **exatamente do tamanho da tela** que recebe
`translateX` de **+22 px → −22 px**. No começo desloca 22 px para a direita (descobre 22 px
à esquerda); no fim desloca 22 px para a esquerda (descobre 22 px à direita). A imagem de
fundo não sofre disso porque tem `scale(1.12 → 1.28)`.

### Correção proposta (a do operador, generalizada)
**Invariante:** *toda camada de tratamento que se move deve ser MAIOR que o quadro.*
- `parallax`: a camada de gradiente ganha folga — `inset: -60px` (≈3× o deslocamento máximo
  de 22 px) ou `scale(1.15)`. Zero impacto visual no centro; some a faixa.
- **Auditoria irmã**: varrer as demais apresentações e camadas animadas
  (`lupa`, `spotlight`, `film`, `reveal`, e o overlay do pacote só-filtro) atrás do mesmo
  padrão *(transform em camada do tamanho exato do quadro)* e aplicar a mesma folga.

### Gate
Não é detectável no PRE-RENDER REPORT (é geometria do player, não decisão editorial).
Vira **item de checklist visual** da auditoria pós-render + um teste de frame no
`_testes/` comparando as colunas de borda (primeiro/último frame de cena com `parallax`).

---

## Ordem de execução (ATUALIZADA com a Parte 2)

| Bloco | Conteúdo | Risco |
|---|---|---|
| **1** | N4+N8 (guarda final das janelas no `legibilidade` + âncora de número) · N2(b) anti-adjacência · N6 (orçamento de repetição) | Baixo |
| **2** | N9 (folga nas camadas animadas — `parallax` primeiro, auditoria das demais) | Muito baixo |
| **3** | N1 (query por membro + gate regional + escala do predador) — bump de `VALIDATOR_VERSION` | Médio |
| **4** | N5+N7 (vet de atmosfera: civilização + tragédia; pai-neutra validada) | Baixo |
| **5** | N3 (fundo único e contínuo do word-pop — mexe no `BrollTest`) | Médio |
| **6** | Gates novos no PRE-RENDER REPORT (todos os acima) | Nenhum |

**Validação final:** Austrália e África re-rodadas em `surgical` nas cenas afetadas +
**Amazônia em `production` como teste de não-regressão** (tem de continuar limpa).
