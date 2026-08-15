# ESPEC-V11 — PRECISÃO DE EDIÇÃO + NOVAS INTELIGÊNCIAS (veredito do operador sobre o v10, 09/08)

Contexto: "o SERPENTES V10 foi O MELHOR VÍDEO PRODUZIDO ATÉ AGORA... FINALMENTE
CONSEGUIMOS O DINAMISMO QUE EU QUERIA". Os ajustes abaixo são REFINO — "sem correr
risco de estragar o que já está muito bom". DESTINO: a inteligência do DIRETOR DE
DOCUMENTÁRIO ANIMAL (todos os vídeos), preparada para CLONAGEM de diretores novos
(clone + particularidades próprias SEM interferir no original).

## A. VERIFICAR ANTES DE MEXER (ordem explícita)
A1. ASMR dos takes VEO: conferir se a normalização + redução para -30dB (JÁ ordenada
    antes) está aplicada. Se JÁ está em -30 e ainda soou alto → baixar mais. Só
    alterar DEPOIS de verificar o estado real.
A2. Tamanho dos elementos (0:52, 3:28): o operador JÁ aumentou os tamanhos nos HTMLs
    de visualização que recebeu — verificar se foi implementado; aparentemente NÃO.
    Recuperar os valores dos HTMLs dele e aplicar.

## B. PRECISÃO (a guerra da v11)
B1. 🔴 TIMING DOS 3 PRIMEIROS MINUTOS: o alinhamento palavra-a-palavra funciona
    (min 4+ = "PERFEITOS, timing exato"), mas os elementos ADITIVOS da exceção dos
    3min entram SEM o mesmo casamento com o áudio (animação de coisa ainda não
    falada). LEI: "agora ou depois, SEMPRE deve seguir o mesmo direcionamento de
    casamento de elementos com áudio." (Também houve desencontros no FINAL do vídeo.)
B2. 🔴 TAKE×FALA SEMÂNTICO (1:22): elefante na tela ANTES da fala do elefante; na
    fala, o elefante já saiu. Precisão semântica take-narração.
B3. 🔴 FRAGMENTOS CURTÍSSIMOS (2:01, 5:46): sobras de milissegundos de take entre
    animações — proibir fragmento ultra-curto (incomoda a visão).
B4. Overlay "The deadliest thing..." (P5) apareceu bem antes da fala — classe B1.

## C. CAPÍTULOS
C1. 🔴 GATILHO ≠ CAPÍTULO: "By the end, the number one will explain..." é gatilho de
    retenção típico de lista — o maestro marcou CAPÍTULO nele (apareceu cap 1 e logo
    depois cap 5!). Reconhecer gatilho e usar ELEMENTO DE CURIOSIDADE novo: imagem
    com BLUR + "?" GIGANTE (quase tela toda). Nunca capítulo em gatilho.
C2. 🔴 TODO CAPÍTULO TEM NOME em CAIXA ALTA (lei JÁ ditada antes, violada de novo):
    "KING COBRA" e "RETICULATED PYTHON" saíram sem nome.

## D. NOVAS FEATURES/INTELIGÊNCIAS
D1. POLAROID CASCATA/COLLAGE (0:09 "sharks, wolves, lions, and crocodiles"):
    polaroids entrando um sobre o outro (cada um em posição diferente, pedacinho do
    anterior visível — sensação de collage), 1 por item enumerado, SFX de clique por
    entrada, fundo = grid. Referência de posições: Downloads/Fundo da Vinheta.jpg
    (ignorar o chromakey verde do exemplo).
D2. CIENTISTAS: fala de pesquisas/estudos/medições → mostrar cientistas, biólogos,
    laboratório (ex.: becker com veneno). Quebra padrão + realismo documental.
D3. BIOLOGIA VISUAL: fala de sistema nervoso/corrente sanguínea → takes que
    exemplifiquem (glóbulos etc.), SEM sangue explícito (políticas de monetização).
D4. ENCADEAMENTO overlay→número (2:12): overlay que termina em sentença de medida
    pode emendar EM CASCATA a animação de número (ex.: 200kg). Oportunidade, não
    obrigação.
D5. MAPA POR CAPÍTULO: roteiro multi-lugar (espécies de continentes diferentes) →
    mapa no INÍCIO de cada capítulo SE servir à narrativa; mesmo lugar → 1x só; e
    quando 1x, aparecer CEDO (antes do min 4 — retenção; 6:28 foi tarde demais).

## E. IDENTIDADE HUMANA (prints AKBAR / WA TIBA)
E1. 🔴 MENÇÃO DE HUMANO = IMAGEM DE HUMANO (lei do Roosevelt no v7, violada): os
    cards de pessoa de Akbar e Wa Tiba mostraram A COBRA como retrato. Com o VEO
    automatizado: GERAR foto representativa (ex.: fazendeiro indonésio), recortar
    e usar como retrato. Nunca animal representando pessoa.

## F. SFX (leis já existentes violadas + ajustes)
F1. 🔴 Animação de % (5:29): usa SFX JÁ DESCARTADO pelo operador (% subindo = SFX
    de número) e está ALTO (não respeita o volume default já definido).
F2. SFX de digitação (6:00): typing SÓ em animação typewriter; essa animação leva
    woosh baixinho (~0.13).

## G. VISUAL
G1. Overlays de TEXTO em CAIXA ALTA (0:56 saiu minúsculo) — mais bonito.
G2. QUADRO: overlay/textura aplicada SÓ NO CONTEÚDO DENTRO do quadro, não na tela
    toda — investigar viabilidade no BrollTest.

## H. MÉTODO
H1. Decupagem do serpentes_v10.mp4 na fonte para confirmar cada apontamento.
H2. Rodada ÚNICA: estes ajustes + os 9 achados reais do Codex + 2 arbitragens
    (infográfico vence word-pop; quota 3min = A esforço máximo).
H3. 🔴 ARQUITETURA DE CLONAGEM: ajustes moram na inteligência do Diretor de
    Documentário Animal; diretores novos = clone + particularidades PRÓPRIAS
    (abertura/personagem/etc.) sem tocar no original.

## DECISÕES DO OPERADOR (09/08, 2ª rodada de arbitragem)
- E1 REFINADA: retrato humano pode ser GENÉRICO, mas DEVE corresponder em **SEXO,
  IDADE e ETNIA** à pessoa mencionada ("não pode colocar uma mulher de 27 anos
  caucasiana no lugar de um fazendeiro da Indonésia de 50"). Vale para TODO humano.
- A2 DITADO: animação de número counter fontSize 150 → **200**; número com sufixo
  (compacta) 84 → **150**. (BrollTest.tsx:446/459)
- BLOCO 11 — MAESTRO LLM: direção criativa hoje roda em Gemini 2.5 FLASH (modelo
  de bolso). Novo desenho em camadas: chamadas de PENSAR (divisão de cenas,
  contrato visual, autor de prompts, briefs, direção) → **GPT 5.6 LUNA via CLI do
  operador** (`codex exec -m gpt-5.6-luna`, testado OK, custo $0/assinatura);
  chamadas de CONFERIR (fiscal mecânico, extrações) → Flash. Adaptador `gpt_cli`
  no director, knob por preset. Motivo: vídeos de 25-30 min vêm aí; a divisão
  precisa de perspicácia (cientista com o becker, elefante na hora certa) que
  regra nenhuma compra.

## CONTRATO DE FABRICAÇÃO DA V11 (ordem do operador, 09/08 — pré-GO)
"A v11 deverá começar como se fosse um vídeo novo — contaminação NENHUMA da v10"
(a dor: o 1º render 'v10' saiu com vídeo e áudio das piranhas).
CHECKLIST DE CERTIFICAÇÃO (roda ANTES do disparo, com prova impressa):
1. Workspace NOVO (só insumos verdadeiros: roteiro_en.txt + narracao.mp3 + words).
2. Timeline construído do ZERO pelo maestro novo (Luna divide as cenas).
3. Prompt-store ZERADO (v11 = job novo; prompts nascem do Luna).
4. Pool: TODOS os takes veo* da v10 em quarentena ANTES (socorro acha 0 herdados).
5. Coleção NOVA no Flow ("teste 05 - serpentes v11"), vazia, cid registrado.
6. render_lock/render json/espelho/gerar: inexistentes no workspace novo.
7. Prova: pré-voo imprime cada item verificado (como o pré-voo da v9).
