# ESPEC-V13 — MODELO OFICIAL DO DIRETOR (ordens do operador, 10/08)

Este documento é o **MODELO OFICIAL**. Ordem literal do operador:
> *"isso que estamos alinhando e ajustando deve ser o MODELO OFICIAL DESSE DIRETOR,
> PORQUE DAQUI A POUCO IREMOS IMPLEMENTAR NO AUTOMATOR, E QUANDO ELE ESTIVER LÁ,
> NÃO DEVERÁ SEGUIR UM CAMINHO DIFERENTE DESSE QUE ESTAMOS REFINANDO AQUI. NÃO
> QUERO TER PASSADO DIAS REFINANDO A ROTA 'A', E NA HORA DO RENDER NO AUTOMATOR
> ELE IR PELO CAMINHO 'B'."*

## A1 🔴 ROTA ÚNICA — o Automator NÃO reimplementa nada
O Automator **invoca o mesmo pipeline e o mesmo pacote de diretor** que estamos
refinando; não existe segunda implementação da inteligência de edição.
Blindagem obrigatória:
- a inteligência mora no **pacote do diretor** (versionado) + no pipeline do
  Director; o Automator é só o disparador/orquestrador;
- o `leis.py` roda **dentro do pipeline**, então roda igual aqui e lá;
- **teste de rota**: o registro de leis grava no relatório final o `id` do pacote
  de diretor + o commit do pipeline usados. Render que não declarar os dois é
  reprovado. Assim, "rota B" fica impossível de passar despercebida.

## A2 🔴 O MODELO DE LEIS: HERANÇA COM OVERRIDE POR DIRETOR
Correção do operador ao meu desenho anterior (eu havia proposto leis globais):
> *"NÃO, AS LEIS ACOMPANHAM CADA DIRETOR, ELAS DEVEM SER A BASE, O DEFAULT, MAS EU
> QUERO PODER MUDAR A LEI DE CADA EDITOR PONTUALMENTE... 'FAÇA UMA CÓPIA DO
> DIRETOR DE DOCUMENTÁRIO ANIMAL, POIS VAMOS CRIAR O DIRETOR DE DOCUMENTÁRIO
> HISTÓRICO' — TODA ESSA INTELIGÊNCIA SERÁ COPIADA... MAS SE EU QUISER MUDAR ALGO
> NA EDIÇÃO DELE EU TENHO QUE PODER, TIPO BANIR UM ELEMENTO OU UM SOM
> E-S-P-E-C-I-F-I-C-A-M-E-N-T-E DELE, SEM ALTERAR A INTELIGÊNCIA DOS DEMAIS."*

Arquitetura:
```
leis_base.json          ← A INTELIGÊNCIA CANÔNICA (todas as leis, com verificador)
   └── herdada SEMPRE por todo diretor, sem knob, sem opt-in
diretores/<nome>/
   leis.json            ← SÓ OS OVERRIDES desse diretor (banir som X, banir
                          elemento Y, mudar parâmetro Z) — cada um com data,
                          motivo e quem autorizou
   estilo.json          ← molduras, SFX, fontes, cores (o que já é preset hoje)
   assets/              ← acervo próprio do nicho
```
Regras duras:
1. **Lei nunca chega "desligada" por esquecimento** — a base é herdada inteira. O
   estado atual (8 dos 10 presets sem NENHUMA lei, nem o maestro) fica proibido.
2. **Override é EXPLÍCITO, datado e local** — muda um diretor, nunca os outros.
3. **Clonar diretor = copiar `leis.json` + `estilo.json` + assets**; o clone nasce
   com a inteligência inteira do original, e diverge só onde o operador mandar.
4. O `leis.py` verifica **as leis efetivas daquele diretor** (base + overrides) e
   nomeia no relatório qual override estava ativo.

## A3 🔴 DECISÃO TOMADA NÃO SE DESFAZ
> *"o que é feito e decidido, não pode ser DESFEITO depois, pq senão as correções
> são aplicadas e descartadas depois por algo que não deveria descartar elas."*
- Toda decisão temporal (janela de elemento, fronteira de cena, alinhamento com a
  fala) é **congelada** no fim da fase que a decidiu e marcada com `_dono` e
  `_congelado_em`.
- Passe posterior que tentar reescrever campo congelado: **erro explícito**, não
  sobrescrita silenciosa. (Hoje o V7 move `acervo_texto.inicio` depois do R2b e o
  `legibilidade` apagava o alinhamento do B1 — os dois viram violação.)
- O `leis.py` pré-lock confere que nada congelado mudou.

## A4 🔴 DURAÇÃO É SEMPRE DINÂMICA — ZERO TEMPO FIXO
> *"não quero tempo fixo, tudo deve ser dinâmico e preciso, com o que está sendo falado."*
Alvos a eliminar: `dur: 4.0` / `4.5` (acervo), `dur_card 3.0` (capítulo), `2.6`
(tease), `DUR_PESSOA 3.6`, `DUR_MAPA 5.5`.
Regra única: janela = **da 1ª palavra falada do conteúdo (−0,45s) ao fim da última
(+0,8s de leitura)**, com piso de legibilidade; depois casada na fronteira (A5).
Para card de capítulo, o conteúdo falado é o marcador + o nome do item.

## A5 🔴 ELEMENTO É CENA DE VERDADE (inclui mapa e pessoa)
Janela do elemento começa **e** termina em fronteira de cena; nenhum take nasce,
morre ou pisca dentro dela; nenhuma transição, SFX de transição, **projetor do
`film`** ou shutter soa dentro dela. Universo: capítulo, curiosidade, texto/gráfico
tela-cheia, **mapa e card de pessoa** (decisão do operador de hoje).

## A6 🔴 TAKE NUNCA LOOPA — DIVIDIR, JAMAIS REPETIR
> *"se não couber 7,5, divida a cena em dois takes, um de 3,5s e outro de 4, mas
> nunca, NUNCA reinicie o take para parecer um loop e dar impressão de microtake."*
- Cena com take VEO > 7,5s é **dividida** em N cenas ≤7,5s (alvo 6s), cada uma com
  **take PRÓPRIO e DIFERENTE** (a lei anti-take-repetido continua valendo — dividir
  e repetir o mesmo clipe seria trocar um defeito por outro).
- Sem material distinto no pool: usa banco/imagem (que não loopa) e **registra**.
- Verificação no render: `ceil(dur*fps) > floor(7,5*fps)` reprova; e nenhuma cena
  pode ter `dur > clip_dur` (é isso que faz o `<Loop>` repetir o take).

## A7 PROIBIDO DESISTIR EM SILÊNCIO
A curadoria B achou leis que simplesmente param quando fica difícil (o piso de
texto abandona sem candidato; o mapa some se `localizar()` falha; o anti-fragmento
desiste se há janela exclusiva no vão). Passa a valer: **toda desistência de lei
emite violação registrada** — o vídeo pode até sair (opção A do operador), mas
nunca calado.

## A8 🔴 O RENDER VÊ OS ASSETS COMO "RECÉM-SAÍDOS DO FORNO"
Ordem do operador (10/08), ao autorizar reaproveitar material num teste:
> *"o fato de pegarmos algo que já tem pronto agora é só para AGILIZAR o teste;
> numa produção real deve seguir todo o fluxo. Quando a fabricação for de ponta a
> ponta, deverá seguir o mesmo direcionamento como se tivéssemos produzido cada
> take, pq para o render ele deve ver os assets como 'acabaram de sair do forno' —
> para não viciarmos ele sempre com heranças pré-existentes."*

Consequências permanentes:
1. **Nenhum passe pode depender de estado de rodada anterior.** O que o pipeline
   produz (`_congelado`, `_janelas_curiosidade`, `_dur_de`, `_take_de`, `sfx_ate`)
   nasce DENTRO da mesma cadeia, na mesma ordem, em toda execução.
2. **IDEMPOTÊNCIA é requisito, não sorte**: rodar o mesmo passe duas vezes sobre o
   mesmo plano tem que dar o MESMO resultado. Verificado para o reconciliador em
   10/08 (2 execuções → 0 fronteiras diferentes). Todo passe estrutural novo tem
   que passar por esse teste antes de entrar na cadeia.
3. **Teste com material reaproveitado valida a EDIÇÃO, nunca a FABRICAÇÃO.** O
   veredito de ponta a ponta só vale com produção virgem (certificação de 7 itens
   da ESPEC-V11).

## PLANO DE IMPLEMENTAÇÃO (para aprovação — nada codado ainda)
| # | Item | Onde | Risco |
|---|---|---|---|
| 1 | `leis_base.json` + `leis.py` (registro + verificador, roda pré-lock e pós-render) | `director/` | baixo (aditivo) |
| 2 | Pacote de diretor (`leis.json`/`estilo.json` por diretor) + herança | `director/` + `diretores/` | médio (toca todos os presets) |
| 3 | `reconciliar_cenas.py` — elemento vira cena, fronteira casada, duração dinâmica | após `legibilidade`, antes do report | **alto** (2º escritor estrutural) |
| 4 | Divisão anti-loop (A6) + guard de `dur > clip_dur` | `reconciliar` + `resolver_cascata` | médio |
| 5 | Congelamento de decisão (A3) + morte do V7 sobre campos congelados | `preparar_render` | médio |
| 6 | Silenciar transição/projetor/shutter na janela (A5) | `preparar_render` + `BrollTest` | baixo |
| 7 | Defeitos do V12 (desindentar `if _caps`, piso do R2b, word-pop, `janela_falada` repetida) | vários | baixo |
| 8 | Fixadores F17–F20 no registro de leis | `leis.py` + `_decupagem_video` | baixo |
| 9 | Migrar as 40 leis condicionais da curadoria B para a base herdada | `presets.json` → pacote | **alto** |

Ordem sugerida de execução: **1 → 7 → 6 → 3 → 4 → 5 → 8 → 2 → 9**
(o registro de leis primeiro, para que cada item seguinte já nasça verificado).
