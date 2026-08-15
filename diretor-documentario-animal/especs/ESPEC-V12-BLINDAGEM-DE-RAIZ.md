# ESPEC-V12 — BLINDAGEM DE RAIZ (veredito do operador sobre o v11, 09/08 noite)

Contexto: "a edição agora está muito melhor... se não tivesse sido isso, esse teria
sido o primeiro vídeo 100% perfeito". Erros CHATOS e RECORRENTES, já proibidos em
especs passadas, voltaram. Perícia feita com o render json + 4 gravações decupadas +
log do driver — CADA erro tem réu identificado com evidência. Meta: blindar na raiz
para o portão virar descartável ("esse tempo [34min do re-render] é jogado fora").

## R1 🔴 MESMO TAKE EM CENAS CONSECUTIVAS — os RÉUS são as HERANÇAS
LEI (ditada): "NUNCA takes consecutivos — por mais que estejam dentro de quadro,
film, polaroid, overlay de tela. PROIBIDO."
Medido no render json: 11 pares consecutivos + 1 TRIPLA (79.4/84.6/91.3s, cru →
polaroid → film — o print das 22:13).
RÉU A — herança de hard-miss (`preparar` linha ~1281 do log): "4 cenas SEM MÍDIA →
herdaram o clipe VIZINHO dentro de moldura {3: quadro, 23/24: polaroid, 33: quadro}"
— duplicata por CONSTRUÇÃO, a moldura era o disfarce.
RÉU B — barreira V4 (`preparar`): "8 takes do sujeito do capítulo ANTES do marcador
→ herdaram o vizinho" — protege o capítulo criando duplicata colada (30.4+33.4,
36.4+39.3, 42.2+44.2, 116.4+119.4, 278.1+281.3).
RÉU C — zip/socorro VEO: "s084: MESMO material da cena 81 (zip casou o mesmo card
2×)" + socorro "1 reuso".
FIX: toda HERANÇA/reuso escolhe doador NÃO-adjacente (±1 proibido; mesma espécie,
distância ≥2) + GUARD FINAL anti-adjacência no preparar (chokepoint da ordem final):
cena i e i+1 com o mesmo clip → troca o herdeiro por doador legal; sem doador →
AVISO explícito (nunca silêncio).

## R2 🔴 ELEMENTO TELA-CHEIA É A CENA, NÃO ADESIVO POR CIMA
LEI (ditada): capítulo, curiosidade "?", counters tela-cheia e afins "DEVEM SER A
CENA, no tempo exato, sem takes de fundo — a transição/corte deve ser feito NESSA
animação".
Sintomas confirmados nas gravações: fragmento de take cru entre a polaroid e o
counter (~0.3s); SFX de transição de cena tocando NO MEIO do counter; pós-capítulo 4
(anaconda) vazou take de OUTRA espécie; capítulo com fundo visível.
RÉU — arquitetura em camadas: a linha de cenas continua rodando (cortes + transições
+ SFX) por baixo do elemento dono da tela.
FIX (no render, sem redesenhar a divisão):
  a) fundo 100% OPACO nos donos de tela cheia (capítulo, curiosidade, counter/stat
     tela-cheia);
  b) janela do elemento ESTICA até as fronteiras de cena vizinhas quando a fresta
     for <0.6s (mata o fragmento pré/pós);
  c) SUPRIMIR transições de moldura (R1/R2), SFX de transição e whoosh de corte que
     caiam DENTRO da janela do elemento (o corte embaixo fica mudo e invisível).

## R3 🔴 CONTEÚDO DUPLICADO ENTRE PASSES (mesma coisa, 2 elementos)
Sintomas: "FIVE METERS" gigante + kicker "METERS" simultâneos (1:02); DateStamp
"MARCH 2017" + ticker "MARCH OF TWO THOUSAND SEVENTEEN" simultâneos (3:39); textos
consecutivos repetindo o mesmo assunto (2:12).
RÉU — datas.py × acervo_texto × R7/lei dos números decidem sobre a MESMA fala sem
se enxergarem ("QUEIMAMOS um elemento que poderia ser usado para outro texto").
LEI: elementos consecutivos/simultâneos SÓ com assuntos DIFERENTES.
FIX: dedup semântico no chokepoint (preparar, que enxerga datas+acervo+cenas+R7):
janelas sobrepostas OU consecutivas (<8s) com conteúdo normalizado sobreposto (≥60%
das palavras) → fica o de maior prioridade (capítulo > número/S10 > data > texto),
o outro é DEVOLVIDO (não gasto).

## R4 TÍTULO DO CAPÍTULO 1 ERRADO ("NUMBER ONE")
RÉU — meu dedup no capitulos_previstos (edicao): ficou com a 1ª ocorrência
(358.78s, "is still not number one" — REFERÊNCIA) em vez do MARCADOR real (360.26s,
"Number one, the Russell's Viper"). O título foi extraído do ponto errado.
FIX: no dedup por número, ocorrência _abre_frase (pós-pontuação final) VENCE a
referência; título extraído do marcador vencedor.

## R5 SFX DE NÚMERO VETADO VOLTOU (5:30)
RÉU — meu F1 estreito demais: cortei counter_digital só de ring/%; o counter "45
minutes" (332.5s) tocou o som vetado.
LEI: o SOM de contador (counter_digital) está DESCARTADO — não é o formato, é o som.
FIX: eliminar counter_digital.mp3 de TUDO (sfx_counter + tabela sfx_por_variante
Graf01/02/10/14/15, Ovl10/12/13) — número anima em silêncio ou com o SFX da família.

## R6 FRAGMENTOS DE CENA REAIS (1.70s + 1.09s no capítulo 1)
RÉU — merge do montar_timeline: cena curta ANTES de cena _abre_item não pode fundir
pra frente (comeria o marcador) e o código NÃO tenta pra trás → fica o toco.
FIX: nesse caso, fundir com a ANTERIOR (marcador intocado).

## R7 PRECISÃO DE SAÍDA (0:26 — elemento saiu cedo)
Texto do hook saiu antes da hora. FIX: exibição mínima = fim da fala do trecho
+0.8s (clamp no texto_ate/dur).

## MÉTODO
- Bancada com o timeline do v11 (redump) prova R1/R2/R3 sem re-render completo.
- Portão ganha fixadores novos: adjacência de take idêntico (F15) e conteúdo
  duplicado simultâneo (F16) — para nunca mais depender do olho do operador.
- Codex revisa o diff (lei do 2º fiscal) antes da bancada final.
