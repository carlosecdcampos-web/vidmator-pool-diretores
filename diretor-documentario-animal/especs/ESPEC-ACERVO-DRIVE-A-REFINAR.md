# ACERVO NO DRIVE + CACHE EFÊMERO POR JOB — ideia a refinar (NÃO IMPLEMENTAR AINDA)

> Status: **EM MATURAÇÃO**. Ideia do operador em 08/08/2026, nascida da investigação
> da noite do v8 (SERPENTES), quando a contaminação entre produções apareceu em três
> lugares diferentes na mesma madrugada. Nada aqui foi implementado. Refinar as
> decisões abertas (§4) ANTES de escrever qualquer linha de código.

---

## 1 · O PROBLEMA QUE ISTO RESOLVE (fatos medidos, não hipótese)

Hoje o estado é **compartilhado e mutável**: todo job escreve nos mesmos lugares, e a
única coisa que separa um vídeo do outro é uma **heurística de TEXTO**. Três vazamentos
reais numa única madrugada:

| # | estado global | o que aconteceu de fato |
|---|---|---|
| 1 | `_cache_stock/` (pool plano) | 133 assets de outros vídeos (124 amazon + 9 ártico) disponíveis ao socorro das serpentes; barrados só por régua de texto |
| 2 | `_workspace/decision_manifest.json` | manifesto do **v7 PIRANHA** vivo no workspace do v8 — o resolver reaproveitaria stock de piranha nas cenas de cobra (neutralizado à mão) |
| 3 | `veo_job/assets/` | **54 takes da piranha ocupavam os slots sNNN do v8**: 8 imagens de piranha entrariam no vídeo e 16 vídeos jamais seriam gerados (o ciclo os deu como "prontos") |

**Por que as réguas de texto não bastam** — a prova do v7: 74 sidecars diziam "piranha"
na `busca_original`, e **14 deles tinham vídeo de TUBARÃO/BALEIA**. O sidecar mentia; a
régua leu texto e aprovou; só o Vision olhando o CONTEÚDO descobriu. Uma barreira que um
campo de texto errado derruba não é barreira — é filtro de boa-fé.

**Diagnóstico do operador (literal)**: *"melhor prevenir do que remediar… numa produção
real, essa é uma fragilidade de sistema inaceitável e temos que estudar maneiras de
bloquear isso na fonte, sem precisar de remendos."*

---

## 2 · A ARQUITETURA PROPOSTA (palavras do operador, 08/08)

> *"Fazer o cache_stock no Drive… tudo que estiver lá já terá sido catalogado, recebido
> um id, um resumo — nosso próprio banco de dados no Drive. Antes de procurar qualquer
> coisa que não seja VEO, ele vai no Drive e verifica se tem o que precisa; se tiver,
> reutiliza; senão, busca de fora. Podemos sempre limpar o `_cache_stock` da máquina: o
> agente vai no Drive, o que servir ele traz para o `_cache_stock` (POR VÍDEO). Numa
> geração simultânea os agentes podem ir preparando o `_cache_stock` dos outros canais
> que estão na fila, de forma lateralizada, economizando tempo por job. Depois de
> terminar completamente o vídeo, ele pode excluir."*

**A inversão que isso representa** — e é ela que mata o problema na fonte:

| | hoje | proposto |
|---|---|---|
| acervo compartilhado | pasta local plana, **todos escrevem** | Drive **read-only curado** (escrita = ato deliberado) |
| cache do job | compartilhado entre produções | **efêmero, por vídeo**, descartável |
| isolamento | régua de texto (bioma/espécie) | **físico**: o que não está na pasta do job não existe para ele |
| réguas de espécie/bioma | única linha de defesa | segunda linha (defesa em profundidade) |

Ganho colateral: **preparação lateral** — o agente pode preencher o cache do próximo
canal da fila enquanto o atual ainda produz. (O paralelismo em si é assunto de fluxo:
está na **ESPEC-V9 §V9-1**; aqui interessa só que o acervo precisa suportar leitura
concorrente de vários jobs.)

---

## 3 · FLUXO DESENHADO

```
job novo → cria _cache_stock/<job>/ (vazio)
         → consulta ÍNDICE do acervo (espelho local do catálogo do Drive)
         → o que casa: baixa do Drive para o cache do job
         → o que falta: busca externa (Pexels/etc.) ou gera (VEO)
         → produz o vídeo usando SÓ o cache do job
         → curadoria: o que merece virar acervo é PROMOVIDO ao Drive (com verificação)
         → vídeo publicado → apaga _cache_stock/<job>/
```

---

## 4 · DECISÕES ABERTAS (refinar ANTES de implementar)

1. **O que impede o Drive de virar o novo pool contaminado?** Se qualquer job escrever
   lá automaticamente, o problema apenas muda de endereço. Proposta: promoção é ATO
   DELIBERADO com **verificação de conteúdo pelo Vision** (descreve o que ESTÁ no
   arquivo, nunca o que o prompt dizia) — foi exatamente a mentira do sidecar que
   causou os tubarões do v7.
2. **Como o agente acha o que precisa?** Listar o Drive a cada busca é lento. Proposta:
   **índice local espelhado** (id, tags, espécie, bioma, resumo, hash) sincronizado
   periodicamente; a busca roda no índice, o download só do que casou.
3. **VEO dentro ou fora da consulta?** O operador disse "antes de procurar qualquer
   coisa QUE NÃO SEJA VEO" — ou seja, take autoral sempre novo. A decidir: um take VEO
   excelente (que já custou crédito) merece entrar no acervo para reuso futuro, com
   critério mais rígido (espécie + ângulo + direção)?
4. **Quando limpar o cache do job.** Renderizado ≠ terminado: re-render dois dias depois
   precisa dos assets. Proposta: apagar quando o vídeo for **PUBLICADO**, não quando
   for renderizado.
5. **Custo/latência do Drive** (quota da API, tempo de download de 40 mídias por vídeo)
   — medir antes de adotar; se for pior que buscar de fora, a ordem inverte.
6. **Granularidade da partição do acervo**: por canal? por nicho? por espécie? (Um
   acervo "documentário animal" ainda misturaria piranha e cobra — o que só é problema
   se a busca for frouxa; com índice por espécie, não é.)
7. **Migração**: os 133 assets atuais entram no Drive? Precisam de auditoria de conteúdo
   antes (o expurgo V8-4 provou que ~10% mentiam).

---

## 5 · O QUE FICA VALENDO ATÉ LÁ (remendos conscientes, assumidos como tais)

- filtro de bioma (V8-4) + régua de espécie mútua no socorro — **sabidamente frágeis**
  (leem texto), mantidos como defesa em profundidade até o acervo existir
- isolamento de workspace por job é assunto de FLUXO: **ESPEC-V9 §V9-4**

*Escrito em 08/08/2026 durante a produção do v8, a pedido do operador: "salve essas
ideias em implementações a refinar antes da implementação".*
