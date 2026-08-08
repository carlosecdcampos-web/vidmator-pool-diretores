# ESPEC — ZELADOR DE PRODUÇÃO (auto-correção sem operador e sem Claude de plantão)

> **Status:** 🟢 **APROVADA (operador, 05/08)** — implementação em curso.
>
> **Decisões travadas:** Z-a livro Z1-Z9 aprovado como está · Z-b **antes da App de
> Publicação** · Z-c última degradação = **modo pool-puro** (o vídeo SAI; report mostra o
> que degradou) · Z-d heartbeat **resolver 10 min · render 15 · narração 20** · Z-e
> Telegram no **bot do Automator (ESPEC 07)** · Z-f v2 com LLM aprovada em CONCEITO,
> implementar só após 2 semanas de `zelador_log` limpo.
>
> **Ordem que origina isto (05/08, verbatim):** *"quando rodar pelo automator, você não
> vai estar monitorando, você não vai estar de prontidão, então os erros tem que ser
> auto-corrigidos, podemos pensar em agentes zeladores, com programações específicas de
> ação... hoje o automator está totalmente blindado, tenho confiança de colocar para
> produzir, ir dormir e ter a certeza de que vou acordar amanhã e os vídeos estarão
> prontos."*
>
> A régua é essa: **a mesma confiança do Automator de hoje.** E o Automator chegou lá
> exatamente assim — watchdog, health monitor de thread, timeouts por etapa, reset de
> estado em erro, retry com causa raiz no log. O zelador replica essa arquitetura para o
> pipeline do Vidmator.

---

## 1. O que é (e o que NÃO é)

O **Zelador** é um processo supervisor DETERMINÍSTICO que roda junto do runner e age
sozinho segundo um **livro de regras** (o [CATALOGO-BLINDAGEM-PRODUCAO.md](CATALOGO-BLINDAGEM-PRODUCAO.md)
em forma executável). Não é um LLM de plantão: v1 é código puro, com gatilhos e ações
programadas — LLM só entra na v2, e para DIAGNOSTICAR, nunca para agir sem regra.

## 2. Arquitetura v1 (determinística — a que dá pra confiar dormindo)

```
runner (_rodar_e2e / futuro job do Automator)
  ├─ escreve: heartbeat.json  (a cada unidade de trabalho: cena resolvida, pass concluído)
  ├─ escreve: _live/<pass>.live.log  (VM_LIVE_LOG — já existe)
  └─ zelador.py (processo irmão, watchdog)
       ├─ SENTE:  heartbeat parado · padrões de erro no live.log · disco/RAM ·
       │          processo morto · pass além do orçamento de tempo
       ├─ AGE:    livro de regras (§3) — ação programada por sintoma
       ├─ AVISA:  Telegram (infra ESPEC 07 do Automator) a cada ação tomada
       └─ REGISTRA: zelador_log.jsonl (toda ação com evidência — auditável de manhã)
```

**Heartbeat** (E4): cada pass longo toca `heartbeat.json` `{pass, unidade, ts}` a cada
unidade de trabalho (cena resolvida, chunk narrado, % de render). Zelador: sem batida por
X min (por pass: resolver 10, render 15, narração 20) → incidente.

## 3. Livro de regras v1 (sintoma → ação, sem julgamento)

| # | Sintoma (detecção) | Ação programada (escada) |
|---|---|---|
| Z1 | Heartbeat parado no resolver | 1) dump de diagnóstico (tail do live.log + processos filhos) 2) mata SÓ o pass 3) re-lança com degradação: `VISION_POSTER_GATE=0` na 2ª tentativa 4) 3ª falha → pula para modo pool+atmosfera puro (0 Vision) e segue |
| Z2 | Heartbeat parado no render | taskkill do ffmpeg/node órfão → re-lança render (retry já existe no runner) → 2ª falha: reduz concorrência 8→6→4 |
| Z3 | Gate com corte <10% em 20 lotes (E1: otimização virou custo) | desliga o gate sozinho (`VISION_POSTER_GATE=0`) e loga |
| Z4 | 3 lotes 4/4 na mesma cena (E5: oferta exaurida) | já resolvido no resolver (teto de 8 lotes); zelador só confere e loga |
| Z5 | `FONTE INDISPONÍVEL`/429 em série no log | pausa o pass 5 min (backoff externo) antes do retry — não queima tentativas contra rate limit |
| Z6 | Disco < 10GB ou RAM > 90% | limpa `_cache_stock` órfão (LRU, preservando `pex/` canônico e `_ESCOLHIDOS`) · se persistir: reduz workers |
| Z7 | Pass excede orçamento de tempo (proporcional ao roteiro) | mesmo caminho do Z1 (o timeout deixa de ser a única linha de defesa) |
| Z8 | Processo do runner MORREU | re-lança o runner do job atual (estado idempotente: narração reusada, index/pool cobrem o refeito) — é o watchdog.bat do Automator aplicado ao job |
| Z9 | Job terminou | roda o portão de decupagem; PASSA → Telegram "pronto + métricas"; FALHA → Telegram com fixadores reprovados + NÃO entrega |

Regra de ouro da escada: **degradar > abortar**. Todo nível de degradação existe e é
honesto (pool puro, sem gate, menos workers, hard-miss + moldura) — o job TERMINA.

## 4. v2 (futura) — zelador com diagnóstico LLM

Para sintoma FORA do livro de regras: o zelador chama Claude CLI (mesma infra dos agentes
do Automator) com o tail do log + contexto, e recebe UMA de N ações permitidas (enum
fechado: retry, degradar-X, pular-cena, abortar-com-estado). LLM diagnostica e ESCOLHE
entre ações pré-aprovadas — nunca inventa ação. Todo caso v2 vira regra v1 no dia
seguinte (o catálogo cresce, o LLM encolhe).

## 5. Integração no Automator (quando o Vidmator virar produção diária)

- Zelador = thread irmã do orchestrator (mesmo padrão do health monitor de 60s que já
  existe), lendo os live.logs dos passes do diretor.
- Telegram: reusa ESPEC 07 (avisos de produção) — início, cada ação do zelador,
  conclusão com veredito do portão.
- `zelador_log.jsonl` entra no relatório diário: o operador acorda e lê O QUE o zelador
  fez de madrugada, com evidência — confiança vem de auditabilidade, não de fé.

## 6. Ordem de implementação proposta

```
1. heartbeat.json nos passes longos (resolver: por cena · render: por % · narração: por chunk)
2. zelador.py v1 com Z1-Z2-Z8-Z9 (os que devolvem o "durmo tranquilo")
3. Z3-Z7 (degradações finas)
4. Telegram plugado (ESPEC 07)
5. v2 (LLM) só depois de 2 semanas de zelador_log limpo
```

*Rascunho de 05/08/2026, escrito durante o render final v2. Aguarda decisão do operador
sobre: (a) aprovar o livro de regras Z1-Z9; (b) prioridade da implementação vs App de
Publicação.*
