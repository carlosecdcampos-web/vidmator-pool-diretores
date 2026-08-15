# ESPEC-V13 — "O ELEMENTO É UMA CENA DE VERDADE" (veredito do operador sobre o v11c)

Status: **ESPEC EM REVISÃO (Codex + contraditório). NADA DE RENDER até GO explícito**
(o Automator está usando a máquina). Saída do próximo render: `serpentes_v12.mp4`
— **proibido sobrescrever `serpentes_v11.mp4`**, que é o padrão de comparação.

## Princípio único desta rodada
O operador: *"na hora que começar a tratar esses elementos como CENAS verdadeiras,
vai resolver em todos os lugares."*
Hoje o elemento tela-cheia (capítulo, curiosidade, texto/gráfico full) é um adesivo
com duração FIXA colado por cima de uma linha de cenas que continua correndo por
baixo — daí, sempre nos mesmos lugares: fragmento de take antes, SFX de transição
no meio, e o take seguinte já pela metade quando o elemento sai.

## O que JÁ foi codado (commit `V12b`, aguardando o mesmo GO)
Registrado aqui para o Codex revisar junto e para não recontar como pendente:
- **SFX**: `ui_beep_01` (o apito) eliminado dos presets + trava permanente no
  preparar; `counter_digital` (escolha do operador) restaurado nas 11 variantes
  numéricas e no `sfx_counter`; revertido o gate que silenciava ring/%.
- **R2b**: janela do elemento tela-cheia casada nas fronteiras de cena.
- **Relógio-palavra** (legibilidade): entra na 1ª palavra falada (−0,45s), sai no
  fim da última (+0,8s). Antes o passe sobrescrevia com `início_da_cena + 0,3`.
- **LetterCascade**: a palavra não se parte ("WE LI"/"VE").

## V13-1 🔴 CAPÍTULO/CURIOSIDADE TAMBÉM SÃO CENAS
O R2b tratou os elementos de acervo; os **cards** continuam sendo esticados por
tolerância (1,2s / 0,6s) em vez de virarem cena. LEI: card de capítulo e tease de
curiosidade ocupam **uma fronteira-a-fronteira inteira**, iguais aos demais:
nada de take nascendo, morrendo ou piscando dentro da janela deles.
Consequência obrigatória: **nenhuma transição, SFX de transição, projetor de
`film` ou shutter pode soar dentro da janela** — hoje isso vale só para
`transicao` de moldura; tem que valer para o par SFX↔apresentação também.

## V13-2 🔴 DURAÇÃO É O TEMPO NECESSÁRIO, NUNCA FIXA
Operador: *"eles não precisam ter duração fixa, a duração deve ser exatamente o
tempo necessário, não mais, não menos — duração fixa congela e engessa o elemento
por tempo desnecessário."*
- Elemento com TEXTO: janela = da 1ª palavra falada ao fim da última (+0,8s de
  leitura), como já vale para `texto_impacto`. Vale também para `acervo_texto`
  (hoje `dur` nasce 4,0/4,5 fixo) e para o card de capítulo (hoje `dur` fixo 3,0).
- Elemento numérico/contador: o tempo da contagem + respiro, não um bloco fixo.
- Piso de legibilidade continua valendo (o texto não pode ficar ilegível de curto).

## V13-3 🔴 FRONTEIRA DE ENTRADA **E** DE SAÍDA
A pergunta do operador: *"respeita fronteira de entrada e de saída???"*
A janela do elemento tem que **começar exatamente numa fronteira e terminar
exatamente numa fronteira**. Se a duração necessária (V13-2) não coincide com uma
fronteira, quem cede é a **divisão de cenas**, não o elemento: a fronteira é
movida para o instante certo (dentro dos limites de V13-4). Elemento cortado no
meio de uma cena volta a produzir exatamente o defeito que esta ESPEC mata.

## V13-4 🔴 TETO DE 7,5s NA CENA (lei nova, física do VEO)
Operador: *"os vídeos do VEO têm 8 segundos; se as cenas/takes que anteceder a
animação de capítulo tiver 8s ou mais, vai travar e ficar repetindo no último
segundo — dá sensação de micro-take, mas é o loop do próprio vídeo."*
- Cena com take VEO: **máximo 7,5s** (o antigo teto de 6s do Codex#1 vira o alvo;
  7,5s é o limite duro para absorção).
- A cena que **antecede um card** pode ser esticada para absorver o fragmento
  (mata o micro-take de 0,5-1s antes do capítulo) **até 7,5s**. Passou disso: usa
  material de banco/imagem (que não loopa) ou divide de outro jeito.

## V13-5 FRAGMENTOS QUE SOBRARAM NO v11c (casos de teste desta rodada)
Cada um tem que sumir e virar caso do portão:
1. frame de take entre **2:02–2:04** — o operador quer o infográfico de 2:04
   ESTICADO para trás até 2:02 (é a aplicação direta de V13-3);
2. frame de take entre **5:58–6:00**;
3. **transição de 6:05** disparando ~1s antes do corte real;
4. fragmento de **1,09s** imediatamente antes do capítulo 1 (359,87s);
5. micro-takes antes dos cards de capítulo (V13-4).

## V13-6 PORTÃO: os fixadores que faltam
O portão só é descartável quando enxerga o que o olho do operador viu:
- **F17** — take nascendo, morrendo ou piscando DENTRO da janela de qualquer
  elemento tela-cheia/card (hoje não é medido);
- **F18** — cena com take VEO acima de 7,5s (risco de loop);
- **F19** — elemento cuja janela não começa E termina em fronteira de cena;
- **F20** — SFX de transição/projetor/shutter soando dentro de janela de elemento.

## V13-0 🔴 ARQUITETURA: A FASE DE RECONCILIAÇÃO (o que o Codex provou)
O Codex derrubou o desenho original desta ESPEC com evidência, e ele tem razão:

- **O preparar_render NÃO pode decidir isto.** O lock assina o `timeline.json` antes
  (`preparar_render.py:147-177`); o R2/R2b mexem no plano DEPOIS da assinatura —
  o MP4 materializa um plano que o report nunca viu. Viola "render é
  materialização, não decisão".
- **O R2b nem sequer roda em vídeo sem capítulo**: R1/R2/R2b/R3 estão indentados
  sob `if _caps:` (`preparar_render.py:1540`). CONFIRMADO por mim no arquivo.
- **O R2b não faz o que meu commit disse**: ele desloca `el.inicio/dur`, não cria
  cena nenhuma. O nome "elemento é a cena" era maior que o comportamento.
- **O passe V7 (janela morta da abertura) desfaz o R2b depois** (`:1874-1944`).
- Mover fronteira DEPOIS do resolver alonga cena com take já escolhido, e o hash
  do manifesto ignora duração (`resolver_cascata.py:315-321`) — o take antigo é
  reusado para uma janela maior. **E o loop é real**: `SceneClip` envolve o vídeo
  em `<Loop durationInFrames={clipDurFrames}>` (`BrollTest.tsx:244-279`), que é
  exatamente a prova em código da lei dos 7,5s do operador.

**DECISÃO (minha, contra a opção 2c do Codex):** não vamos antecipar a decisão de
elementos para antes da divisão de cenas (o que exigiria o edicao inteiro rodar
antes do contrato/VEO — reescrita do pipeline). Adotamos a opção 2d dele:

> **Novo passe `reconciliar_cenas.py` no Director — roda DEPOIS do `legibilidade`
> e ANTES do `pre_render_report`.** É o único ponto onde os elementos já têm
> conteúdo e tempo definitivos e o plano ainda NÃO foi assinado.

Contrato do reconciliador (1 passada, sem loop — mata a circularidade do item 1h):
1. calcula a janela NECESSÁRIA de cada elemento full-screen (relógio-palavra);
2. move a fronteira de cena para casar entrada E saída (V13-3);
3. **re-valida localmente o que a mudança afeta** — teto de 7,5s da cena com take
   VEO (troca por banco/imagem se estourar), cadência ≥12 inícios/min, cotas de
   moldura do minuto tocado, janelas de word-pop, e âncoras absolutas
   (mapa/pessoa/data NUNCA se movem — só são re-clampadas);
4. grava tudo no `timeline.json`, e só então report+lock assinam o plano final.
Com isso, R2/R2b saem do preparar (voltam a ser só materialização) e o V7 deixa
de ter o que desfazer.

## V13-7 DEFEITOS DO V12 A CORRIGIR JUNTO (achados do Codex, aceitos)
- `if _caps:` engolindo R1/R2/R2b/R3 → **desindentar** (vídeo sem capítulo hoje
  fica sem NENHUMA correção).
- R2b encurtando elemento até 1,2s sem consultar piso de legibilidade nem o texto.
- R2b atravessando fronteira interna quando a 1ª fronteira dá <1,2s.
- R2b puxando a entrada para trás sem reconsultar janela de word-pop.
- **Projetor do `film` não é silenciado** por R2 (só `transicao` é) — é a causa
  literal do "SFX do elemento film no meio" que o operador ouviu em 0:17.
- `janela_falada` casa a ocorrência ERRADA quando a frase se repete na cena
  (busca do início, subsequência sem limite de distância): passar a âncora
  original e limitar o salto entre palavras.

## V13-8 CRITÉRIOS DOS FIXADORES (redação do Codex, adotada)
- Universo: `marcacoes.tipo ∈ {capitulo, curiosidade}` + `acervo_texto` com
  `dominio=="tela_cheia"` ou `familia=="Texto"`. **Pergunta ao operador: mapa e
  card de pessoa entram nessa lei também?** (ambos ocupam a tela inteira).
- Tolerância ε = 0,05s, e nunca reprovar por diferença de 1 frame.
- **F17**: falha se houver início/fim de take em `(E0+ε, E1−ε)`.
- **F18**: falha se `ceil((fim−inicio)*fps) > floor(7,5*fps)` (30fps → 225 frames);
  `7,51s` já viola, sem tolerância extra.
- **F19**: entrada E saída a ≤ε de uma fronteira, e ambas da MESMA cena.
- **F20**: SFX de transição / projetor(`film`) / shutter intersectando o interior
  da janela por mais de ε. O SFX próprio do elemento é isento.
- Os quatro rodam **duas vezes**: no timeline (antes do lock) e no render.json.

## V13-9 🔴 DECISÕES DO OPERADOR (10/08, fecham a ESPEC)
1. **Mapa e card de pessoa = CENA DE VERDADE** — entram na mesma lei dos demais
   elementos de tela cheia: janela casada em fronteira, nenhum take por baixo,
   nenhum SFX de transição/projetor/shutter dentro.
2. **A inteligência de edição é UNIVERSAL** — *"com capítulo e sem capítulo, essa
   inteligência será unânime em TODOS OS DIRETORES... edição PERFEITA é uma
   premissa de qualidade em QUALQUER vídeo"*. Consequências duras:
   - nada de lei de edição pendurada em `if _caps:` (o defeito atual) nem em
     qualquer condição de nicho/preset;
   - o reconciliador e os fixadores rodam SEMPRE, em qualquer diretor
     (documentário, histórico, psicologia, chosen one, estoicismo);
   - preset controla ESTILO (quais molduras, quais SFX, quais fontes), **nunca**
     se a lei de edição vale. Lei de edição não é opt-in.

## V13-10 🔴 BLINDAGEM ANTI-REGRESSÃO — O REGISTRO DE LEIS
Operador: *"prevenir é melhor que remediar... senão daqui a pouco estaremos na v30
com erros absurdos que já deveriam ter sido corrigidos, ou pior, que foram
corrigidos e apareceram depois."*
Diagnóstico honesto do porquê isso vem acontecendo: as leis moram espalhadas em
comentários e `if`s dentro dos passes. Quando um passe novo entra (ou eu insiro um
bloco no lugar errado), a lei antiga é sobrescrita **em silêncio** — foi assim que
(a) o D1 do preparar reescreveu o ledger V10-L2 do maestro rodando a lei v8 morta,
(b) o `legibilidade` apagou o alinhamento palavra-a-palavra do B1 com
`inicio+0,3`, (c) o meu bloco de curiosidade re-atou o `elif` dos capítulos.
Nenhum dos três foi pego por revisão — os três foram pegos pelo OPERADOR, no olho.

**Solução: `director/leis.py` — um REGISTRO ÚNICO e EXECUTÁVEL de invariantes.**
- Cada lei tem: `id` (L01…), data, quem pediu, texto na língua do operador, e uma
  função `verificar(tl_ou_render) -> lista de violações`.
- O registro roda em **dois pontos obrigatórios**: antes do lock (sobre o
  timeline) e depois do render (sobre o render.json + MP4). Mesma lei, dois
  momentos — o que o timeline promete e o que o vídeo entrega.
- **Lei nunca é apagada, só marcada como revogada com data e motivo** (a revogação
  é decisão do operador, não conveniência de código).
- O portão passa a ser a EXECUÇÃO do registro, não uma lista solta de checagens.
- Toda lei que o operador já ditou e que hoje só existe em comentário entra no
  registro nesta rodada — inclusive as antigas (V10-L2 adjacência, S10 lei dos
  números, teto de variante, ASMR −33, teto de 7,5s, relógio-palavra, elemento é
  cena, SFX banidos).

## V13-11 CURADORIA PONTO-A-PONTO (ordem do operador: eu + Codex, no código real)
Cada ponto desta ESPEC tem que ser ancorado no arquivo:linha que o implementa ou
que o quebra, com o modo de falha nomeado e a blindagem correspondente. O
resultado da curadoria vira a seção final desta ESPEC antes de qualquer código
novo. Fatias da curadoria:
- **A** — reconciliador: onde nasce, o que invalida, o que re-valida, e prova de
  que nenhum passe posterior o desfaz.
- **B** — universalidade: varredura de TODA lei de edição hoje presa a condição
  (`if _caps`, knobs de preset, nicho) que precisa virar incondicional.
- **C** — registro de leis: inventário das leis já ditadas, onde cada uma mora
  hoje, e quais estão sem nenhuma verificação automática.

## MÉTODO DESTA RODADA (ordem do operador)
1. ESPEC revisada por **Codex** (adversarial) + contraditório meu — **antes** de
   codar V13-1..V13-6.
2. Fixes implementados e validados em bancada offline (sem render).
3. **Só com o GO explícito**: render para `serpentes_v12.mp4`, preservando o v11
   como referência de comparação.
