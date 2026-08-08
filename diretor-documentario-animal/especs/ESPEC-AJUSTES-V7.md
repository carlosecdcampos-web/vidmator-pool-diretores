# ESPEC DE AJUSTES v7 — bloco guardado da revisão do `urso_polar_v6b` (06/08)

> **Status:** 📦 **ARQUIVADO POR ORDEM DO OPERADOR — não mexer agora.**
> *"Não vamos mexer mais nele agora... não vamos rodar o v6c, o v7 será rodado já com o
> funcionário do veo contratado, o foco agora está 100% em deixar o funcionário do veo
> otimizado para o trabalho."*
>
> Veredito dele do v6b: *"no geral, já melhorou muito."*
>
> Cada item abaixo já tem o CULPADO identificado no render json (`_render_urso_polar_v6b
> .json`) — quando houver GO, ninguém precisa recaçar nada.

## A1 · 0:13 — cinza ilegível no elemento

*"o cinza some na tela, o amarelo fica muito legível, mas o cinza sumiu — editar esse
elemento NA RAIZ e trocar o cinza por branco."*
Culpado provável: janela de **word-pop** (~13s) ou kinetic — não há acervo nesse instante
no json. Fix na RAIZ do componente (cor do texto secundário: cinza → **branco**), não no
uso. Confirmar o componente exato com 1 frame quando reabrir.

## A2 · 0:24 — duas medidas no MESMO elemento

FATO do json: `Texto05_BoxedKicker` @23,4s com **"Three metres tall. Over 600 kg."**
*"deixar só o 'three metres tall'; para o 'over 600 kg' usar a MESMA animação de número
dinâmica de 4:19."*
Fix: o detector de medidas (R7/lei dos números) deve **quebrar frase com 2 dimensões em 2
elementos** — a 1ª fica no elemento de texto, a 2ª vira animação NUMÉRICA (counter), nunca
as duas em texto corrido.

## A3 · 2:38 — DE NOVO a animação descartada, com o SFX descartado

FATO do json: **`Ovl06_CenterPunch` @157,4s com `impact_hit_02`** — é este o par que ele
já descartou (era o print 2 do v5, "no natural enemy", SFX alto). Eu tinha banido a
EditorialSerif achando que era ela — **o alvo real é o CenterPunch+impact_hit_02**.
Fix: banir `Ovl06_CenterPunch` do repertório (mesma cirurgia da R2: tabela + pools) ou, se
o operador quiser manter o visual, trocar o SFX — decisão dele no GO.

## A4 · 3:01 — mapa sem destaque do Arctic

FATO do json: mapa com **`pais: null`**, coord `[0, 90]` (polo norte), legenda "Arctic".
O R9 salvou o RENDER do crash, mas o `MapAnimation` sem país não tem shape para destacar —
zoom em nada. *"Primeira vez que isso acontece"* porque antes o passe inteiro morria.
Fix: lugar SEM país usa **variante própria** (satélite por coordenada, ou ponto+legenda),
nunca o MapAnimation de país. Elegibilidade no `detectar_mapas`.

## A5 · Elementos de texto desalinhados com o áudio

Recorrente e ainda aberto. Investigar com medição: comparar `aparece_em` gravado vs
instante FALADO da palavra-âncora (words.json) nos elementos do v6b — a hipótese antiga
(`antecipa_max_s`) já foi zerada, então a causa é outra (âncora errada do momento? deriva
da legibilidade?). Método Sherlock antes de mexer.

## A6 · (meu, para decisão dele) V4 herdou o vizinho em 14 cenas do predadores

O embargo por take, à risca, considera TEASER do hook como spoiler — 14 cenas herdaram o
vizinho (risco de repetição visual). Decidir: teaser nos primeiros ~30s é desejável?
Se sim, o V4 ganha janela de exceção no hook.

## A7 · F2 (cortes/min) — lembrete estrutural

Só fecha com timeline re-cortado (`montar_timeline` fresh — o corte no marcador e a cena
máx 6s já estão lá). O **v7 será produção FRESH com o VEO**, então o F2 será testado de
verdade nele — nada a fazer antes.

*Arquivado em 06/08/2026. Reabrir junto com o v7 (produção fresh + funcionário VEO).*
