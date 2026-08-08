# ESPEC DE AJUSTES v3 — avaliações do operador sobre `urso_polar_v2` e `predadores_v2`

> **Status:** 🟢 **PRONTA PARA GO** (06/08) — coleta encerrada, revisada contra o código
> real, com plano de execução ordenado e âncoras `arquivo:linha` na PARTE 6.
>
> Veredito do urso: *"NO GERAL FICOU MUITO, MUITO BOM... ESTÁ DE PARABÉNS"*.
> Substitui e absorve o `AJUSTES-POS-URSO-V2.md`.

---

# PARTE 0 — 🗂️ MAPA DE STATUS: o que JÁ ESTÁ NO CÓDIGO × o que é SÓ ESPEC

> Pergunta do operador (06/08): *"essa questão dos overlays, aumento do tamanho dos
> elementos do card de capítulos, os overlays que vamos utilizar, já estão implementados
> ou deverão entrar na espec?"* — auditoria feita item a item no código.

## ✅ JÁ NO CÓDIGO (vale no próximo render, sem fazer mais nada)

| Item | Onde | Efeito imediato |
|---|---|---|
| **Card de capítulo 45% MAIOR** | `ChapterTitle.tsx` — `escala = 1.45` é o DEFAULT | ✅ o próximo vídeo já sai com o card grande |
| Props `overlayRel` / `overlayBlend` / `overlayOpacidade` | `ChapterTitle.tsx` | mecanismo pronto (mas ninguém alimenta — ver abaixo) |
| Trilha com variedade por roteiro | `trilha.py` | ✅ validado no predadores (7 faixas distintas) |
| Heartbeat de trabalho + zelador | `resolver_cascata.py`, `producao.py`, `zelador.py` | ✅ validado (2h26 sem intervenção) |
| Cache de veredicto persistente (E12) | `resolver_cascata.py` | ✅ 54 veredictos reaproveitados |
| Cache canônico por `pexels_id` (E8) | `resolver_cascata.py` | ✅ sem re-download |

## ⏳ SÓ NA ESPEC — **nada disso acontece no próximo render**

| Item | Seção | O que falta |
|---|---|---|
| **Overlay NO card de capítulo** | 1B.3 · 1B.10 | ⚠️ a prop existe, mas `BrollTest.tsx:913` chama o `ChapterTitle` **sem passar `overlayRel`** → sai SEM overlay. Falta: `preparar_render` escolher 1 do pool por roteiro (hash) e passar pela marcação |
| **Overlays de TAKE** (Barulho · Quadro · Filmadora) | 1B.1 · 1B.2 · 1B.9 | ⚠️ **zero linha de código.** Só existem os testes em ffmpeg. Falta: escolha, prop na cena, render dentro da `Sequence` da cena (1B.7) |
| Transições pareadas (Brilho 2 / glitch) + SFX | 1B.1 · 1B.2 | nada implementado |
| **Números de dinamismo** (3-4/min no hook, 2-3/min miolo) | 1.1 | nada |
| Respiro por domínio (overlay 3-4s) | 1.3 | nada |
| Rajada de famílias diferentes permitida | 1.4 | nada |
| KineticText como candidato | 1.7 | nada |
| Mood vermelho com teto por duração | 1C | nada |
| Timing (texto entrando cedo) | 1C.1 | nada |
| **P1** capítulos 2 e 1 faltando + badge errado | P1 | nada |
| **P2** rodízio na ordem cronológica | P2 | nada |
| **P3** atmosfera do pool sem vet (queimada no oceano) | P3 | nada |
| **P4** título do capítulo em CAIXA ALTA | P4 | ⚠️ o teste foi renderizado com o texto em caixa alta **na mão**; o `preparar_render` ainda grava minúsculo |
| **P5** guarda anti-loop da narração | P5 | nada |
| U1-U8 (canto sup. dir., peso após tamanho, 6 SFX) | Parte 3 | nada |

## 🎯 Resumo em uma frase

**Do que você pediu hoje, só o TAMANHO do card está de fato ligado.** Todo o resto —
overlays de capítulo, overlays de take, transições, SFX — está especificado, testado e
decidido, mas **não plugado no pipeline**. Se rodássemos um vídeo agora, o card sairia
grande e sem overlay nenhum.

---

# PARTE 1 — 🔴 DINAMISMO DE TEXTO (a prioridade nº 1 do operador, nos DOIS vídeos)

## 1.1 Os números FIXADOS para o próximo teste (ordem direta)

> *"a primeira entrada de texto entrou no segundo 36, poderia/deveria ter aparecido pelo
> menos 2 antes, nos primeiros 2 minutos, podemos fazer um teste fixando em **3-4 por
> minuto**, vai deixar muito dinâmico, depois pode variar entre **2-3 por minuto**."*

| Zona | Alvo FIXADO | O que conta |
|---|---|---|
| **Hook — primeiros 2 minutos** | **3-4 por minuto** | entradas de texto (KineticText) + acervo TEXTO + acervo OVERLAY |
| **Miolo — do minuto 3 ao fim** | **2-3 por minuto** | idem |
| **Primeira entrada de texto** | **antes de ~0:20** (hoje: 0:36) | pelo menos 2 entradas antes do segundo 36 |

Isto SOMA aos elementos que já existem (apresentações, ilustrações, gráficos, capítulos) —
não substitui. É o *ritmo de TEXTO* especificamente, que é o que ele sentiu faltar.

**Medido no predadores v2 (o que temos hoje):** 11 elementos de acervo em 7:36 = **1,4/min**,
e o primeiro só aos 33,4s. Precisa **mais que dobrar** no hook.

## 1.2 O TETO está estrangulando (revisão explícita da regra §3.5)

> *"o teto para usar esses elementos de texto, todos eles os que citei aqui e os outros das
> suas famílias, estão **puxando muito para baixo**... ficar repetindo só o mesmo, mais de
> 2/3x pode pesar, **mas pode usar os outros**."*

**Releitura da lei:** o teto de 2× por variante permanece como proteção contra *monotonia
da MESMA animação* — mas ele não pode virar teto do CONJUNTO. Com 10 variantes de Texto +
14 de Overlay, o repertório comporta 20-40 entradas sem repetir nada 3×.
**O gargalo real não era o teto** (ver 1.3) — mas a leitura fica registrada: **variedade
alta é o objetivo; teto é anti-repetição, não anti-quantidade.**

## 1.3 ✅ CAUSA MEDIDA — é o RESPIRO, não a cota nem o teto

O coordenador do predadores registrou o motivo de cada recusa:

```
respiro ................. 22   <<< maior bloqueador, isolado
rodízio moldura ..........  9
cota Pop .................  8
cota Apresentacao ........  7
tela é do word-pop .......  5
teto variante ............  4
```

Eu introduzi a combinação que trava: **F5 subiu a duração para 4-6s** e o **respiro virou
FIM→INÍCIO (4s)**. Cada elemento ocupa **8-10s de trilho** → teto físico de ~30 elementos
em 456s. Entraram 30 de 33 de orçamento: **o trilho acabou, não a cota.**

**Proposta (a aprovar):** respiro DIFERENCIADO por domínio —
**✅ DECIDIDO PELO OPERADOR (06/08): overlay 3-4s · tela-cheia 4s.**
(Minha proposta era 1,5-2s para overlay; ele fixou 3-4s — mais conservador, preserva
legibilidade. O ganho de trilho vem menor, então os números da §1.1 passam a depender
TAMBÉM de reduzir a duração de elemento de overlay, não só o respiro.)

## 1.4 A sequência que ele QUER ver (regra nova de composição)

> *"SE SERVIR A NARRATIVA, NUM MOMENTO QUE FAÇA SENTIDO, pode usar sequencialmente uma
> entrada de texto, logo na próxima cena um overlay, depois uma do acervo de texto, é legal
> esse dinamismo, uma, depois outra, depois outra, aí volta pra cena, tendo um overlay no
> meio para quebrar um pouco o fundo fixo."*

**Isto CONTRADIZ a regra de rotação de família atual** (que penaliza famílias diferentes em
sequência e recusou 2 candidatos por "rotação"). A nova leitura: **rajada de 3 elementos
seguidos de FAMÍLIAS DIFERENTES é DESEJÁVEL** quando a narrativa comporta — texto → overlay
→ acervo → volta pro footage. O que continua proibido é a mesma variante colada.

## 1.5 Sem obrigatoriedade — variedade é o valor

> *"não queria colocar obrigatoriedade e usar sempre uma ou outra, pq senão fica muito
> engessado e toda edição iria parecer a mesma edição e eu não quero isso."*

Nada de whitelist fixa. As variantes que ele **gosta muito** (usar como PESO na escolha,
nunca como regra):

**Texto (tela cheia):** `Texto01_Typewriter` · `Texto03_WordPop` · `Texto06_SplitBar` ·
`Texto07_StampImpact` · `Texto09_UnderlineDraw`
**Overlay:** `Ovl01_ChapterBig` · `Ovl14_PillVerdict` · `Ovl03_LowerThird` ·
`Ovl09_TickerCaption`

## 1.6 `Ovl11_SpecBadge` sem a bolinha laranja

> *"achei que no overlay usou o Ovl11_SpecBadge e sem a bolinha laranja e perdeu a chance
> de usar muitos outros"*

Usado 2× no predadores (33,4s e 145,4s) e 2× no urso. Duas coisas: (a) verificar por que
renderizou **sem a bolinha laranja** (elemento visual faltando no componente ou prop
ausente); (b) é sintoma da concentração — mesma variante repetida enquanto 38 outras não
apareceram.

## 1.7 KineticText não aparece no inventário

Nenhuma **entrada de texto (KineticText)** consta no cronograma do coordenador.
**Verificar se ele sequer é candidato no Cronograma Absoluto** — se não for, é o mesmo
defeito estrutural dos 6 sistemas: quem não é candidato não entra, por melhor que seja.

---

# PARTE 1B — 🎞️ OVERLAYS NOVOS NO DIRETOR (ordem do operador, 06/08)

Todos os 6 assets foram **verificados em disco** (1920×1080, 13-15s; o particles é 1280×720).

## 1B.1 · `Filmadora - CapCut.mp4` — o overlay de câmera

> *"olha que overlay legal para inserirmos nesse diretor de documentário também, para
> aumentar ainda mais o dinamismo da edição... **NUNCA DEVERÁ TER INSERÇÃO NENHUMA DE
> TEXTO** quando utilizar esse overlay, a transição de abertura deverá ser um **flash
> branco**, com o **sfx de clique**, sempre que esse overlay entrar."*

| Regra | Valor |
|---|---|
| Asset | `acervo/overlays/CapCut/Filmadora - CapCut.mp4` |
| **Texto** | 🚫 **PROIBIDO** — a janela inteira do overlay é zona livre de texto (vira janela exclusiva, como o word-pop) |
| Transição de entrada | **flash branco** — SEMPRE |
| SFX | **clique** — SEMPRE, pareado à transição (R4) |

**✅ DECIDIDO (operador 06/08):** o flash branco NÃO existe no acervo — *"vamos usar o
**Brilho 2** até segunda ordem"*. Asset: `transitions/CapCut/Brilho 2 - CapCut.mp4`.
SFX de clique: `acervo/sfx/ui/ui_click_01.mp3` (já existe).

## 1B.2 · `Barulho 1` + `Quadro de Filme` — efeito TV/câmera

> *"olha esses outros overlays que legal para documentário com efeito de tv/câmera...
> sempre que eles entrarem devem entrar com **transição de glitch e o sfx de glitch** também."*

| Regra | Valor |
|---|---|
| Assets | `overlays/CapCut/Barulho 1 - CapCut.mp4` · `overlays/CapCut/Quadro de Filme - CapCut.mp4` |
| Transição de entrada | **glitch** — SEMPRE |
| SFX | **glitch** — SEMPRE (`sfx/glitch/glitch_digital_01.mp3` ou `glitch_static_01.mp3`) |
| **Blend** | **`multiply` @ 100%** — decidido no teste 1B.9 |
| Ancoragem | filho da `Sequence` do TAKE (1B.7) |

## 1B.3 · 🔴 OVERLAY OBRIGATÓRIO NA ANIMAÇÃO DE CAPÍTULO

> *"SEMPRE NA ANIMAÇÃO DE CAPÍTULO, DEVERÁ USAR UM DESSES OVERLAYS... **NUM MESMO ROTEIRO
> NÃO DEVE MISTURAR** os overlays de fundo do capítulo, se um roteiro optar por um, deverá
> **USAR SEMPRE O MESMO OVERLAY EM TODOS OS CAPÍTULOS DO ROTEIRO**, no outro vídeo se usar
> o outro overlay, deverá usar somente esse outro."*

| Regra | Valor |
|---|---|
| Pool (escolher UM por roteiro) | `Vapor - CapCut.mp4` · `Fogo - Capcut.mp4` · `particles_gold_03.mp4` — **os 3 aprovados pelo operador (06/08 — pool é SÓ estes 3; 'sem overlay' NÃO entra no rodízio)**, cada um com o blend/opacidade da tabela 1B.8 |
| Escolha | **1 por ROTEIRO**, usado em **TODOS** os capítulos dele. Nunca misturar no mesmo vídeo. Vídeo seguinte pode usar outro (mesma mecânica determinística da trilha: partida por hash do roteiro) |
| **Camada** | **ACIMA do fundo, ABAIXO das letras** |
| **Opacidade** | **60%** |

## 1B.4 · 🧪 TESTE PEDIDO (executar): capítulo 5 dos predadores, 3 versões

> *"para fim de teste vamos renderizar o capítulo 5 o vídeo dos predadores, uma vez com
> cada um desses overlays, somente o trecho onde mostra capítulo 5 e o título escrito em
> maiúscula como eu determinei."*

Trecho: `35,40s` (capítulo 5, `numero=5`, `titulo='the tiger shark'` → **THE TIGER SHARK**).

**✅ EXECUTADO (06/08).** 4 clipes em `_saida/_teste_capitulo/` (3 overlays + 1 controle
sem overlay). Implementação mínima usada no teste, já no código:
- `ChapterTitle.tsx` ganhou `overlayRel` + `overlayOpacidade` (default 0.6). Camada
  ACIMA do fundo e ABAIXO das letras, com `mixBlendMode: screen` — sem o screen, o
  overlay de fundo escuro apagava o gradiente do card.
- `Loop layout="none"` (lição do `quadro`): sem isso o Loop embrulha em Sequence absoluta
  e colapsa o vídeo.
- Script: `remotion/_render_teste_capitulo.mjs`; assets em `remotion/public/_teste_cap/`.
**Pendente da decisão do operador:** qual dos 3 vira o pool padrão (ou se os 3 ficam no
pool com escolha determinística por roteiro, como a trilha).

## 1B.5 · 📏 TAMANHO DO CARD DE CAPÍTULO (operador 06/08)

> *"eu acho esses elementos do capítulo pequenos, precisamos aumentar o tamanho default"*

**Medidas ATUAIS** (px em 1920×1080), respondendo à pergunta dele:

| Elemento | Hoje | ×1,25 | ×1,45 |
|---|---|---|---|
| Label "CHAPTER" | 26 (espaço 14) | 33 | 38 |
| **Número** | **210** | 263 | 305 |
| **Título** | **104** | 130 | 151 |
| Subtítulo | 34 | 43 | 49 |
| Réguas (cada lado) | 230 × 2 | 288 | 334 |

Implementado como prop **`escala`** (multiplica TUDO de uma vez, mantendo a proporção).
**✅ DECIDIDO PELO OPERADOR (06/08) após comparativo renderizado: `escala = 1.45` é o
DEFAULT do componente.** Já aplicado no `ChapterTitle.tsx`.

## 1B.6 · ⚠️ O VAPOR COM `screen` FALHOU — medido e corrigido

O 1º teste do vapor saiu como **lavagem cinza com o título quase ilegível** (print do
operador). Causa medida: **brilho médio do asset**:

| Overlay | YAVG |
|---|---|
| `Vapor` | **150** (claro, preenche o quadro) |
| `Fogo` | 37 |
| `particles_gold_03` | 45 |

Com `mixBlendMode: screen`, um overlay CLARO soma luminância em toda a tela → o gradiente
escuro do card morre e o texto branco perde contraste. Os escuros (fogo/gold) ficam ótimos
no screen. **Lição:** o blend não pode ser fixo — depende do asset.
Prop **`overlayBlend`** (`screen` | `normal`) criada.
**✅ DECIDIDO PELO OPERADOR (06/08): `cap5_vapor_screen25` — o vapor fica em `screen` com
opacidade 25%.**

### Tabela FINAL de configuração por overlay de capítulo

| Asset | blend | opacidade | por quê |
|---|---|---|---|
| `Vapor - CapCut.mp4` | `screen` | **0,25** | YAVG 150 (claro); a 60% vira lavagem cinza |
| `Fogo - Capcut.mp4` | `screen` | **0,60** | YAVG 37 (escuro) — 60% é a regra geral |
| `particles_gold_03.mp4` | `screen` | **0,60** | YAVG 45 (escuro) |

Ou seja: **a opacidade de 60% do operador vale como REGRA GERAL; asset claro é a exceção
calibrada por medição** (YAVG ≳ 80 → reduzir). Ao entrar overlay novo no pool, medir o
YAVG antes de assumir 60%.

## 1B.7 · 🔒 REGRA DE ANCORAGEM — overlay ATRELADO ao elemento (operador 06/08)

> **Capítulo:** *"ele deverá estar ATRELADO A ANIMAÇÃO DO CARD DE CAPÍTULO, entra com ele,
> sai com ele, **nenhum milissegundo antes, nenhum milissegundo depois**."*
> **Demais overlays (take):** *"os outros overlays que eu mandei, para entrar nos takes dos
> vídeos, estilo documentário, seguem a mesma lógica, são **atrelados ao take**, não entram
> nem antes, nem depois. JUNTO, entram junto e saem juntos."*

**Lei única:** overlay NÃO é elemento de linha do tempo próprio. Ele é **filho** do
elemento que o hospeda — nasce e morre com a `Sequence` do hospedeiro, com precisão de
frame.

| Overlay | Hospedeiro | Ancoragem |
|---|---|---|
| Vapor · Fogo · particles_gold | **card de capítulo** | dentro do `ChapterTitle` |
| Filmadora · Barulho 1 · Quadro de Filme | **o TAKE (cena)** | dentro da `Sequence` da cena |

✅ **Capítulo: JÁ GARANTIDO.** O overlay é renderizado dentro do próprio `ChapterTitle`
([BrollTest.tsx:912](remotion/src/compositions/BrollTest.tsx) o envolve em
`<Sequence from durationInFrames>`), então é impossível vazar — não existe caminho no
código para ele começar antes ou terminar depois.
⏳ **Take: A IMPLEMENTAR com a mesma arquitetura** — o overlay entra como PROP da cena e é
renderizado dentro da `Sequence` dela. **Nunca** como entrada solta em `timeline.overlays`
com `inicio`/`dur` próprios: seria a porta para exatamente a deriva que ele proibiu.

## 1B.8 · 📊 BRILHO MEDIDO DOS 6 OVERLAYS — o blend não pode ser adivinhado

Medição própria (média de 3 frames, escala 96×54, luma 0-255):

| Overlay | Brilho | Blend correto | Observação |
|---|---|---|---|
| **Filmadora** | **4** | `screen` | quase preto — o screen mostra só o efeito |
| **Fogo** | 26 | `screen` 60% | ✅ validado no render |
| **particles_gold_03** | 31 | `screen` 60% | ✅ validado no render |
| **Vapor** | **177** | `screen` **25%** | ✅ decidido pelo operador (a 60% lavava a tela) |
| **Barulho 1** | **197** | ⚠️ claro — **calibrar antes** | mesmo risco do vapor |
| **Quadro de Filme** | **190** | ⚠️ claro — **calibrar antes** | mesmo risco do vapor |

🔴 **AVISO IMPORTANTE:** `Barulho 1` (197) e `Quadro de Filme` (190) são ainda MAIS claros
que o Vapor, que já falhou. Se entrarem com `screen 60%` vão **lavar o take inteiro**.
Precisam de teste de calibração igual ao do vapor ANTES de irem para produção — e sobre
FOOTAGE, não sobre card escuro (o comportamento é outro).

> **Procedimento que fica valendo:** todo overlay novo tem o brilho medido antes de entrar.
> ≳80 → não assumir 60%; renderizar amostra e calibrar.

## 1B.9 · 🧪 TESTE DOS OVERLAYS DE TAKE — `multiply`, não `screen` (06/08)

> *"vamos testar os overlays acima, para funcionar, deve utilizar o efeito **clarear**,
> tipo as máscaras do capcut, quando selecionamos o clarear e **tira o branco** do
> overlay"*

**O que os assets SÃO (conferido por frame):**
- `Barulho 1` → grão/riscos ESCUROS sobre campo CINZA-CLARO
- `Quadro de Filme` → moldura ESCURA sobre campo CINZA-CLARO

**Correção de nomenclatura (importa para não errar de novo):** em CapCut,
**Clarear = Screen → remove o PRETO**. Quem **remove o BRANCO é Escurecer = Multiply**.
Como os dois assets são de base CLARA com efeito ESCURO, o modo que entrega o que o
operador descreveu ("tirar o branco") é **MULTIPLY**.

**Resultado do teste (take de 5s, `_saida/_teste_overlay_take/`, 9 arquivos):**

| Modo | Barulho 1 | Quadro de Filme |
|---|---|---|
| **`multiply` 100%** | ✅ grão e riscos aparecem, take preservado | ✅ moldura de filme perfeita, take intacto no centro |
| `multiply` 60% | ✅ mais sutil | ✅ moldura mais suave |
| `screen` 100% | 🔴 **tela branca** — take some | 🔴 **tela branca** — take some |
| `screen` 60% | 🔴 lava | 🔴 lava |

**Confirma o alerta da 1B.8:** ambos são MAIS claros que o vapor (197 e 190 vs 177).
Com screen, o take desaparece por completo.
**Regra que emerge, agora com base física e não por tentativa:**
> **base ESCURA → `screen` (remove o preto) · base CLARA → `multiply` (remove o branco).**
> O brilho médio (1B.8) prevê qual é qual: ≳80 = base clara = multiply.

**✅ DECIDIDO PELO OPERADOR (06/08): `barulho_multiply_1.0` e `quadro_multiply_1.0` —
os dois em `multiply` a 100%.**

## 1B.10 · ✅ TABELA CONSOLIDADA — configuração final de TODOS os overlays

| Overlay | Hospedeiro | Blend | Opac. | Transição de entrada | SFX | Regra especial |
|---|---|---|---|---|---|---|
| `Vapor - CapCut` | card de capítulo | `screen` | **0,25** | (a do card) | — | 1 por roteiro, o mesmo em todos os capítulos |
| `Fogo - Capcut` | card de capítulo | `screen` | 0,60 | (a do card) | — | idem |
| `particles_gold_03` | card de capítulo | `screen` | 0,60 | (a do card) | — | idem |
| `Filmadora - CapCut` | **TAKE** | `screen` | a calibrar | **Brilho 2** | **clique** (`ui_click_01`) | 🚫 **PROIBIDO texto na janela** |
| `Barulho 1 - CapCut` | **TAKE** | **`multiply`** | **1,00** | **glitch** | **glitch** | — |
| `Quadro de Filme` | **TAKE** | **`multiply`** | **1,00** | **glitch** | **glitch** | — |

**Invariante que vale para os 6 (1B.7):** o overlay é FILHO da `Sequence` do hospedeiro —
entra e sai com ele, sem um frame de folga. Nunca entrada solta na linha do tempo.

⏳ **Único pendente:** a `Filmadora` (brilho 4 = base escura → `screen`) ainda não teve
opacidade validada sobre footage. Fazer o mesmo teste de 5s antes de ir a produção.

---

# PARTE 1C — MOOD VERMELHO: teto por duração (ordem do operador, 06/08)

> *"é legal o uso de mood, traz dinamismo também, o mood vermelho com o overlay dele, mas
> vamos **limitar o uso desse**... pois temos outros overlays para serem utilizados também."*

| Duração do vídeo | Máximo de usos do mood vermelho |
|---|---|
| até 10 min | **2×** |
| até 15 min | **3×** |
| até 20 min | **4×** |

## 1C.1 · Timing: texto entrando ANTES da hora

> *"eu percebi que algumas inserções de texto dinâmicos entraram um pouco antes do que
> deveriam entrar"* — observado nos DOIS vídeos.

Mesma família do D8 (capítulo que anunciava antes da fala, corrigido com encaixe de 1,5s
que ATRASA e nunca adianta). **Aplicar o mesmo princípio a TODO elemento de texto:** na
dúvida, entra DEPOIS da palavra-âncora, nunca antes. Investigar o `aparece_em` /
`antecipa_max_s` (hoje 2,5s no preset — provável causa: antecipa até 2,5s).

---

# PARTE 2 — DEFEITOS CONFIRMADOS NO TIMELINE (predadores v2)

## P1 🔴 Capítulos: só 3 de 5 entraram, e o acervo preencheu os outros 2 com número ERRADO

**Evidência:** `marcacoes` do render tem só 3 capítulos —
`5 → the tiger shark (35,4s)` · `4 → the great white shark (109,8s)` · `3 → the killer whale (180,7s)`.
**Capítulos 2 e 1 não foram inseridos.**

Nos lugares deles, o ACERVO colocou `Ovl10_NumberBadge`:
- `259,3s` → texto "Number Two: The Sperm Whale" **com badge #1**
- `336,6s` → texto "Number One: The Saltwater Crocodile" **com badge #1**

É exatamente o que ele printou: *"aqui era chapter 02, marcou errado"* e *"mesmo erro do
capítulo 2, não marcou o capítulo 1"*. O `Ovl10_NumberBadge` **não carrega o número do
capítulo** — mostra "#1" fixo.
**✅ CAUSA (a) MEDIDA (06/08):** não havia **corte de cena** na janela de encaixe. Marcadores
falados vs cortes disponíveis em `[-0,25s, +1,5s]`:
```
cap 5 @ 35,2s  -> corte em 35,40  ✅        cap 2 @ 259,3s -> NENHUM  ❌ pulado
cap 4 @108,9s  -> corte em 109,76 ✅        cap 1 @ 336,6s -> NENHUM  ❌ pulado
cap 3 @179,8s  -> corte em 180,65 ✅
```
O encaixe de 1,5s (que eu apertei no D8 para o card não anunciar antes da fala) é rígido
demais quando não há corte por perto. **Fix: capítulo é OBRIGATÓRIO — sem corte na janela,
insere no MARCADOR FALADO mesmo assim.** É seguro: o card é TELA CHEIA, cobre o que estiver
embaixo, então não precisa de corte para ficar limpo. A regra "atrasa, nunca adianta"
continua valendo (nunca antes do marcador).

**Causa (b):** o filtro que tira `capitulo` do acervo em roteiro-lista
([edicao.py:200](director/edicao.py)) **não pega `item_lista`**, que resolve para
`Ovl10_NumberBadge` — a porta lateral por onde entraram os cards com badge "#1" errado.

## P2 🔴 Dois POLAROID consecutivos (192,86s e 199,54s = 3:13-3:19)

Confirmado no timeline. **Sequência real de molduras no vídeo:**
```
film · film · polaroid · film · quadro · polaroid · POLAROID · film · quadro · polaroid · quadro
```
Mas o ledger que o `edicao` gravou tem só 6 e está PERFEITO (`film,quadro,polaroid,film,quadro,polaroid`).

**Causa raiz identificada:** o rodízio é aplicado na **ordem de ACEITAÇÃO** do coordenador,
não na **ordem CRONOLÓGICA** do vídeo — e a herança de hard-miss (12 no predadores!)
acrescenta molduras depois, fora dessa ordem. Resultado: 11 molduras no vídeo contra 6 no
ledger.
**Fix:** o rodízio tem de ser validado/reordenado sobre a linha do tempo FINAL, com TODAS as
molduras (cronograma + herança), no `preparar_render` — que é quem enxerga tudo por último.

## P3 🔴 Take de QUEIMADA (savana em chamas) em documentário de OCEANO (~5:29)

Print do operador: *"take de queimada no lugar errado, roteiro de oceano, ter foto de
queimada é foda."* O `vet_atmosfera` existe justamente para vetar `tragedia_sem_fala` e
`wrong_region` — **investigar por que não pegou**: cena de atmosfera vinda do POOL entra com
**zero Vision** (pula o vet), e o pool dos predadores tem 35 atmosferas herdadas de rodadas
antigas — provável origem.
**Fix candidato:** atmosfera do pool precisa passar pelo `vet_atmosfera` OU o pool só aceita
atmosfera cujo `documentary_setting` de origem case com o do vídeo atual.

## P4 · 🔴 Título do capítulo SEMPRE em CAIXA ALTA (reforçado pelo operador 06/08)

> *"MAS DEVERÁ SER MAIÚSCULO O NOME DO CAPÍTULO NO CARD DA EDIÇÃO."*

Hoje o `preparar_render` grava minúsculo — `the tiger shark`, `the great white shark`,
`the killer whale`, como saem da narração. O teste do card saiu em caixa alta porque eu
escrevi `THE TIGER SHARK` **à mão** no script de teste; o pipeline NÃO faz isso.
**Fix:** `.upper()` no retorno do `_titulo_item()`
([preparar_render.py:927](remotion/preparar_render.py)). Uma linha.

## P5 · Narração com palavra repetida (4:26-4:30) — MESMO defeito do urso

Confirma o diagnóstico do urso: **loop do Chatterbox em fronteira de chunk**, não erro de
edição. Aparece nos DOIS vídeos → é sistemático, não acidente.
**Fix:** guarda no `narrator.py` — após transcrever, detectar n-grama longo repetido em
janela curta e RE-GERAR o chunk.

## P6 · Oportunidades de dinamismo perdidas

- **5:55** — poderia ter entrado **animação de ANO**, não entrou
- **6:10** — poderia ter entrado **dinamismo de LETRA**, não entrou
- **0:36** — primeira entrada de texto tarde demais (ver 1.1)

## ✅ P7 · ELOGIADO — não mexer

- **6:23** — animação de número: *"achei legal, ficou bem construída"*
- **Urso, seg 27** — elemento de tamanho: *"ficou ótimo"*

---

# PARTE 3 — do urso, que NÃO se repetiu no predadores

*(sinal de que podem ser específicos, não sistêmicos — reavaliar)*

- **U1** — elemento no canto superior direito ficou estranho (`Fat layer > 10 cm thick`, 0:07)
- **U2** — elemento de PESO não entrou logo após o de TAMANHO (~0:27). É a exceção do R5 que
  ele concedeu (dimensões diferentes PODEM ser consecutivas). Investigar qual regra barrou.
- **U3-U8 (SFX)** — cada família de animação precisa de um SFX FIXO próprio, catalogado:
  | # | Onde | Ajuste |
  |---|---|---|
  | U3 | card `600kilometres` (4:37) | faltou o SFX **já usado na animação de números** |
  | U4 | 5:02 | **SFX solto sem transição** — viola R4 |
  | U5 | `Cubs do not survive` (5:58) | **encontrar** SFX e fixar |
  | U6 | tópico anterior ao 7:03 | fixar o **mesmo** SFX do `Ice Cycle Change` |
  | U7 | `Ice Cycle Change` (7:03) | referência aprovada |
  | U8 | `The only question is when` (7:33) | **encontrar** SFX e fixar |
  > Candidato a virar tabela no preset: `sfx_por_variante`.

---

# PARTE 4 — o que o portão de decupagem mediu sozinho

| | urso v2 | predadores v2 |
|---|---|---|
| Fixadores reprovados | 5 | **2** |
| F2 cortes/min | `12·3·5·0·7·9·4·2` | `10·7·5·5·6·4·2·2` |
| Elementos/min | 3,8 | **4,6** |
| F3 hook / F4 miolo | ✗ / ✗ | ✓ / ✓ |
| Formatos distintos | 8 | **19** |
| Frames pretos | 0% | 0% |
| Elementos vazios | 0 | 0 |

**F2 é o que resiste nos dois**: cadência decaindo até 2 cortes/min no fim, contra 12-17
constantes do Piter. Ligado ao vazio de acervo (E16/VEO): pouco material distinto = poucos
cortes reais.

---

# PARTE 5 — já corrigido, aguardando validação no próximo render

- **Trilha** — todo vídeo usava `sombrio_001/frio_001/tenso_001/nostalgia_001` (4 de 96).
  Partida agora determinística por roteiro. **Predadores já saiu com 7 faixas distintas**
  (`sombrio_007, frio_006/007/008/009, sombrio_008, tenso_002`). ✅ validado.
- **Heartbeat/zelador** — 2h26 de produção, **zero intervenções**, encerrou sozinho. ✅

---

# PARTE 6 — 🚀 ORDEM DE EXECUÇÃO (revisada contra o código real, 06/08)

Cada item com a âncora onde o trabalho acontece. Ordem escolhida por **risco crescente**:
começa pelo que é uma linha e termina pelo que mexe na alocação.

## Bloco A — correções de 1 a 5 linhas (risco ~zero)

| # | O quê | Âncora |
|---|---|---|
| A1 | **P4** título do capítulo em CAIXA ALTA | `preparar_render.py:927` — `.upper()` no `_titulo_item()` |
| A2 | **P1b** `item_lista` fora do acervo em roteiro-lista | `edicao.py:200` — somar `item_lista` ao filtro |
| A3 | **P1a** capítulo obrigatório: sem corte na janela, insere no marcador | `preparar_render.py:912-917` — trocar o `continue` por fallback no `_t` |
| A4 | **1C.1** texto entrando cedo | preset `antecipa_max_s: 2.5` → **0** (nunca adianta) |
| A5 | **1.3** respiro por domínio (overlay 3-4s · tela-cheia 4s) | `edicao.py:307-317` `_conflito_respiro` |

## Bloco B — overlays (o que o operador validou hoje)

| # | O quê | Âncora |
|---|---|---|
| B1 | Overlay do CAPÍTULO: escolher 1 do pool por roteiro (hash, igual à trilha) e gravar na marcação | `preparar_render.py` bloco B5 + `BrollTest.tsx:913` (passar `overlayRel/Blend/Opacidade`) |
| B2 | Copiar os 3 assets para `public/` no preparar | `preparar_render.py` (padrão dos outros assets) |
| B3 | Overlays de TAKE (Barulho · Quadro @ `multiply` 100%) como **filho da Sequence da cena** (1B.7) | `BrollTest.tsx` camada de cena + prop no `render["cenas"]` |
| B4 | `Filmadora`: mesma mecânica + **janela proibida de texto** + Brilho 2 + SFX clique | idem + `edicao.py` (janela exclusiva, como o word-pop) |
| B5 | Transição/SFX pareados: glitch p/ Barulho e Quadro; Brilho 2 + clique p/ Filmadora | `preparar_render.py` (mesmo padrão do R1/R2) |
| B6 | ⏳ **Calibrar a `Filmadora` antes** (teste de 5s sobre footage, como 1B.9) | — |

## Bloco C — dinamismo de texto (o pedido nº 1)

| # | O quê | Âncora |
|---|---|---|
| C1 | **1.1** alvo 3-4 elem. de texto/min no hook · 2-3/min no miolo · 1º antes de 0:20 | `edicao.py` alocação (piso por minuto, não só orçamento global) |
| C2 | **1.4** permitir rajada de FAMÍLIAS DIFERENTES (remover a penalidade de rotação) | `edicao.py:_cabe` — ramo `rotação (família repetida)` |
| C3 | **1.5** peso (não regra) nas variantes preferidas do operador | `acervo_texto.escolher_variante` |
| C4 | **1.7** KineticText (`entrada_texto`: pop/blur/words/up/slam/cascade) como CANDIDATO do cronograma | `edicao.py` coleta + `montar_timeline.py:240` |
| C5 | **1.6** `Ovl11_SpecBadge` sem a bolinha laranja | `AcervoTextoOverlay.tsx` |
| C6 | **1C** teto do mood vermelho (2/3/4 por 10/15/20 min) | `efeitos.py` (mood `tenso` = red wash) |

## Bloco D — integridade (defeitos que sujam o vídeo)

| # | O quê | Âncora |
|---|---|---|
| D1 | **P2** rodízio de moldura na ordem CRONOLÓGICA, com heranças incluídas | `preparar_render.py` (validação final, depois da herança) |
| D2 | **P3** atmosfera do pool passa pelo `vet_atmosfera` (ou casa bioma de origem) | `resolver_cascata.pool_take_atmosfera` |
| D3 | **P5** guarda anti-loop da narração (n-grama repetido → re-gera chunk) | `narrator.py` |
| D4 | **U2** elemento de peso após tamanho (exceção do R5 já concedida) | `edicao.py` — investigar qual regra barrou |
| D5 | **U1** elemento do canto superior direito | `AcervoTextoOverlay.tsx` (`Ovl05_CornerTag`) |

## Bloco E — SFX por variante (U3-U8)

| # | O quê | Âncora |
|---|---|---|
| E1 | Tabela `sfx_por_variante` no preset — cada família de animação com SEU som fixo | `presets.json` + `BrollTest.tsx` |
| E2 | **U4** matar o SFX solto de 5:02 (viola R4) | rastrear origem no `BrollTest.tsx` |

## Bloco F — validação

| # | O quê |
|---|---|
| F1 | Re-render dos 2 roteiros (narração reusada, pools atualizados) |
| F2 | Portão de decupagem + **conferência dos números novos** (3-4/min hook, molduras, capítulos 5/5, caixa alta) |
| F3 | Avaliação do operador |

> **Fora deste escopo (decidido, mas é projeto próprio):** o **VEO** para cobrir
> repetição e hard-miss de espécie exata (E16 no `CATALOGO-BLINDAGEM-PRODUCAO.md`).


---

*Aberto em 06/08/2026. **Coleta encerrada e revisada contra o código real. PRONTA PARA GO.***
