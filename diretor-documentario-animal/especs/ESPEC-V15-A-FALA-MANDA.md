# ESPEC-V15 — A FALA MANDA (agendamento por palavra, não por relógio)

Decisões do operador, 10/08, verbatim:

> "a obrigatoriedade é DURAR ENQUANTO O TEXTO REFERENTE A ELA DURAR, SIMPLES ASSIM,
>  SE NÃO COUBER DOIS ELEMENTOS DE TEXTO CONSECUTIVOS PARA CORRESPONDER O QUE ELES
>  REPRESENTAM SEM CORTAR TEMPO DE ANIMAÇÃO DO TEMPO DA FALA ELES ENTRAM
>  SUBSEQUENTES, SE NÃO, NÃO."

> "temos a transcrição palavra a palavra, e ele consegue colocar cada elemento no
>  lugar EXATO DE CADA UM, então não tem porque isso acontecer."

> "eles só entrariam no tempo que eles deveriam entrar, entram, saem, pronto.
>  exatamente 100% no tempo preciso."

## O defeito que originou

Medido no plano de 10/08 (após liberar TEXTO consecutivo):

    Texto09_UnderlineDraw   fala 372,04 -> 376,68
    Texto01_Typewriter      fala 375,82 -> 382,20
                                  ^ 0,86s de fala COMPARTILHADA

Os dois foram agendados sobre as MESMAS PALAVRAS. O sistema colocou os dois e
TRUNCOU o primeiro em 375,37s — cortando 1,3s da fala dele. O operador proibiu:
nunca se corta.

Causa: o agendador raciocina em SEGUNDOS DE RELÓGIO (`respiro`), enquanto os
elementos vivem em PALAVRAS. O teste "estão a mais de 1,5s um do outro?" passava
(3,78s de distância) sem nunca perguntar se as FALAS se invadem.

## As três regras

**R1 — a duração É a fala.** A janela do elemento é `[t_ini - animação_entrada,
t_fim]` da âncora dele. Sem piso, sem teto, sem esticar para casar fronteira.
Punchline de 3 palavras fica ~1s na tela, e isso é correto.

**R2 — colisão é PALAVRA COMPARTILHADA, não proximidade.** Dois elementos
colidem quando as janelas ancoradas se sobrepõem. O espaço mínimo entre dois
elementos consecutivos deixa de ser um número arbitrário: é exatamente o tempo
de animação de entrada do segundo — o que ele precisa para nascer antes da
palavra dele.

**R3 — o PRIMEIRO fica.** Colidiu, o de fala mais cedo permanece e o outro é
DEVOLVIDO (sai do plano, com aviso no log). Nunca truncar, nunca sobrepor.
Segue o princípio já vigente nas molduras: a 1ª decisão prevalece.

## PRÉ-REQUISITO (achado do Codex, 10/08) — a âncora tem que CHEGAR ao agendador

`ancora.localizar()` chama `ancorar()` e **devolve só o `t_ini`**, jogando fora o
intervalo (`director/ancora.py:135-138`). `edicao.py:403` usa esse `localizar()`,
então no momento em que `_cabe()` decide, o candidato de acervo **não tem âncora**.
É a MESMA doença da V14 pela terceira vez: a âncora nasce completa e é descartada
no caminho.

Sem corrigir isso, a R2 consulta um campo vazio. Ordem de implementação:
1. o candidato passa a nascer COM a âncora completa (não só `t_ini`);
2. `janelas_exclusivas()` passa a usar a âncora de mapa/pessoa/capítulo, em vez
   de `inicio + dur` (`director/edicao.py:247-270`);
3. só então a R2 entra em `_cabe()`.

## MATRIZ DE COLISÃO — "sem âncora" NUNCA significa "sem colisão"

O Codex mostrou que matar o respiro sem isto deixaria Overlay, Gráfico,
Ilustração, infográfico e o próprio KineticText **sem teste de colisão nenhum**.
Três classes, três testes:

| Classe | Quem | Teste de colisão |
|---|---|---|
| **A — fala ancorada** | acervo Texto/Overlay/Gráfico, KineticText, mapa, pessoa, capítulo | janelas ancoradas não podem se sobrepor. O primeiro fica. |
| **B — janela própria** | word-pop, infográfico | já têm `ini/fim` medidos; tratados como janela exclusiva (comportamento atual, preservado) |
| **C — sem fala** | moldura/apresentação, ilustração | `respiro_s` CONTINUA VALENDO — é o único teste que eles têm |

Decisão explícita (desfaz a ambiguidade do texto anterior): **o knob `respiro_s`
NÃO é removido**. Ele deixa de ser aplicado entre dois elementos da classe A, e
continua íntegro para a classe C e para os cruzamentos com ela.

## O que MORRE nesta espec

- `respiro_s` **entre dois elementos da classe A** (o knob continua vivo p/ classe C)
- `PISO_ELEMENTO` (2,5s) para elementos cuja âncora é de método `exato`
- esticamento da janela para além do fim da fala

O que fica: corte seco entre elementos (dinamismo que o operador pediu), e o
respiro apenas para elementos SEM fala que os ancore (moldura/apresentação).

## O PISO NÃO MORRE INTEIRO (achado do Codex)

O piso de 2,5s existe porque um card de pessoa ficou 0,78s na tela. Mapa e pessoa
ancoram por método `forte` — cujo `t_fim` é ESTIMADO pelo tamanho do trecho, não
medido. Matar o piso ali trocaria uma duração medida por uma estimativa.

Regra final: o piso vale **apenas quando a âncora não é `exato`**. Com âncora
exata, a fala é medida palavra a palavra e manda sozinha — que é a ordem do
operador. Sem âncora exata, o piso protege contra o card-relâmpago.

## Consequência aceita

Elemento cuja fala é curta aparece pouco tempo. Decisão declarada do operador,
não efeito colateral.
