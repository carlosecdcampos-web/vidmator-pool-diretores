# ESPEC — resolver_cascata de 76,8 min para 5–10 min

Co-autorada com o Codex em 11/08/2026. Meta do operador: **resolver entre 5 e 10 min**.
Fora de escopo: `veo_gerar` (45,7 min) e `render` (21,2 min) — já têm solução decidida
(paralelismo e motor Skia) e serão atacados depois. Um gargalo por vez.

## O que foi medido (3 vídeos de ~8 min, produção real)

| etapa | scarface | lobo | orca |
|---|---:|---:|---:|
| veo_gerar | 52,4 | 44,7 | 45,7 |
| **resolver_cascata** | **41,0** | **30,5** | **76,8** |
| render | 16,9 | 21,2 | 28,4* |
| todo o resto somado | ~31 | ~22 | ~28 |

Custo por cena: **banco 121s com 21% de hard miss · VEO 49s com 0% de falha**.
Mediana entre veredictos: **6,3s** — o sistema não é lento, ele empaca.
100% dos clipes entregues vieram do **Pexels**; Commons e SearXNG entregaram zero;
Archive.org está desligado no código (`USE_ARCHIVE_L1 = False`, "instável").

## 1 · Mais chaves do Pexels — ✅ FEITO 11/08 23:34

**FONTE.** Remove a parede antes dela existir: a orca fez ~130 buscas em 76 min
(~103 req/h); comprimidas em 5 min, as mesmas buscas equivalem a ~1.560 req/h.

Feito: 6 chaves válidas em `vidmator.secrets.json → pexels_api_keys`; campo legado
`pexels_api_key` REMOVIDO (L10 — um só dono). Independência de cota **testada**:
gastar 6 requisições na chave A derrubou o contador de A em 6 e o de B em 1.
Resultado: **1.200 req/h · 150.000 req/mês** (era 200/h · 25.000/mês).

⚠ Correção de um número que eu havia dado: o header do Pexels expõe o teto **mensal**
(25.000/chave), não o horário. O "200 req/h" vem do comentário do código, não de
medição. A chave original já consumiu 3.684 requisições neste mês.

Aceite: `pexels_api.KEYS` enxerga 6 chaves distintas, todas respondendo 200. ✅

## 2 · Não formar a consulta impossível — ✅ FEITO 11/08 23:5x

**FONTE.** `variantes_query()` fabrica `f"{canon} {bioma}"` para QUALQUER sujeito, e
`ancorar_regional()` prepende o adjetivo regional. Num doc de fiordes noruegueses isso
produziu `"great white shark norwegian fjords"` e `"Norwegian great white shark"` —
o roteiro diz que o tubarão está na **África do Sul**. 92 das 130 consultas da orca
levavam o bioma colado; 19 eram fisicamente impossíveis.

O dado para decidir **já está carregado**: o glossário regional do job lista
`Orca, White-tailed Eagle, Atlantic Herring, Grey Seal…` e `great white shark` não
está lá. A função usa esse glossário só no sentido contrário (se a espécie É da
região, deixa a query em paz) — nunca pergunta o inverso.

Mudança: um predicado ÚNICO `_da_regiao(nome)` consultado pelas duas funções.
Aceite: 0 das 19 consultas impossíveis reaparece no replay da orca; espécies do
glossário continuam sendo regionalizadas; total de consultas cai de 130 para ≤111.

## 3 · Estouro do teto vai para o VEO, não para hard miss — ✅ FEITO 12/08 00:0x

**REMÉDIO declarado** (limita o dano; o item 2 é que reduz a incidência).
Toda chamada de rede/ffmpeg/LLM tem timeout (15–180s); a composição não tem nenhum.
Houve janela de 24 min com zero veredicto entre 8 workers.

Mudança: prazo monotônico por cena em `_resolve_beat`, checado antes de cada nível;
ao estourar, a cena é enfileirada UMA vez no `veo_socorro` em vez de virar hard miss.
Hoje ela paga 121s e sai sem clipe.
Aceite: p95 por cena ≤60s; zero cena sem clipe E sem pedido VEO; zero pedido duplicado.

## 4 · PAR_WORKERS de 8 para 16 — ✅ FEITO (falta MEDIR)

**REMÉDIO.** Só depois do item 1 verde — senão troca lentidão por 429.
Aceite: três execuções com o resolver entre 5 e 10 min, sem 429 relevante e sem
perda de cobertura contra o baseline.

## 5 · Segunda fonte real — projeto separado

**FONTE**, mas depende de existir fonte que entregue. Hoje Commons e SearXNG estão
ativos e entregam zero, então repartir workers entre eles é repartir o nada.
Significa religar o Archive.org (desligado por instabilidade) ou plugar um banco de
vídeo de verdade. Decisão do operador, depois dos itens 1–4 medidos.

## Ordem e portões

1 verde (feito) → 2 verde (0 consultas impossíveis, cobertura mantida) → 3 verde
(teto comprovado, encaminhamento idempotente) → 4 verde (3 execuções em 5–10 min) → 5.

## O que NÃO fazer

- Não subir workers antes das chaves — antecipa a parede de cota.
- Não corrigir as consultas com duas listas paralelas: um predicado, dois chamadores.
- `future.cancel()` não cancela thread já iniciada — não usar como garantia.
- Não esperar o VEO gerar dentro do resolver; só enfileirar no `veo_socorro`.
- Não declarar sucesso pela média: medir total, p95, cobertura, 429 e duplicidade.

---

# EXECUÇÃO — o que ficou no código (11–12/08)

| item | estado | arquivos | teste |
|---|---|---|---|
| 1 chaves | ✅ verde | `vidmator.secrets.json` | independência de cota medida |
| 2 consulta impossível | ✅ verde | `resolver_cascata.py` | `test_ancora_regional.py` (15 asserções) |
| 3 prazo + VEO | ✅ codado | `resolver_cascata.py`, `veo_recolher_perdidas.py` | `test_prazo_cena.py` |
| 4 workers | ✅ codado | `resolver_cascata.py` | falta job real |
| 5 2ª fonte | ⏸ decisão do operador | — | — |

## Item 2 — o que foi realmente feito
Predicado único `da_regiao(nome)` + `estrangeiros(contrato)` + `cita_estrangeiro(q, nomes)`.
`variantes_query` só fabrica `f"{canon} {bioma}"` se o sujeito for da região; a
ancoragem é decidida **por consulta**, não por cena.
Codex REPROVOU duas vezes antes de aprovar, e os três furos que ele pegou viraram teste:
cena com dois sujeitos (`[great white shark, orca]`), substring (`"ray"` dentro de
`"gray whale"`) e a direção `e.endswith(" "+n)` que declararia a foca ANTÁRTICA do
roteiro como norueguesa.

## Item 3 — desenho do operador, não o meu
Eu propus **reordenar** (resolver antes do veo_gerar, como a docstring do veo_gerar
manda). O operador propôs **chamar o VEO num segundo momento** só para as cenas
perdidas. Fui na dele: é aditivo, não mexe numa ordem que hoje funciona, e é no-op
quando não há perda. `ORCAMENTO_CENA=60s` (env `VM_ORCAMENTO_CENA`) em
`threading.local` — os 8/16 workers são threads do mesmo processo, um global daria o
prazo de um worker aos outros. As três funções bloqueantes (`_baixar_url`, `_vis_cli`,
`_api_text`) recebem `min(timeout_próprio, restante)`, atendendo a condição do Codex:
teto que só é checado depois não é teto.

## O QUE AINDA NÃO ESTÁ PROVADO (não vender como pronto)
- **Os 5–10 min não foram medidos.** Itens 3 e 4 só se validam num job real.
- **A cota está apertada:** 6 chaves = 1.200 req/h; a estimativa para 130 consultas em
  5 min era ~1.560. O item 2 derruba a contagem de consultas, o que deve fechar a
  conta — mas 2 chaves a mais dariam margem em vez de aposta.
- **Nenhum driver chama `veo_recolher_perdidas.py` ainda.** O próximo runner precisa
  incluí-lo depois do `resolver_cascata`, senão a realocação do item 3 não vira take.

---

## Medição Leopardo — 14/08/2026 — velocidade aprovada, cobertura reprovada

O replay válido terminou em **2,4 min**, sem `operational_exhaustion`, sem reuso
forçado e com no máximo um arquivo final materializado por cena. Porém resolveu no
banco somente **13/32 (40,6%)** e realocou **19/32 (59,4%)** ao VEO. O portão técnico
passou, mas essa cobertura não satisfaz a intenção híbrida do produto.

### Novo contrato para o próximo E2E

- Cobertura do banco >= **90%** das cenas alocadas a ele. Em 32 cenas: >=29 no banco
  e <=3 realocadas ao VEO.
- Tempo alvo **8–9 min**, teto duro abaixo de 10 min.
- Onda 1 preserva o caminho rápido atual (~2–3 min).
- Onda 2 trabalha somente os misses: reformula consultas, amplia profundidade e usa
  fontes alternativas; nunca refaz cenas já aprovadas.
- Onda 3 trabalha somente o resíduo até o teto total; depois disso, o VEO recebe
  apenas o hard miss legítimo.
- Continua valendo **um único arquivo persistente por cena**. Pré-visualizações e
  candidatos intermediários vivem em área temporária limitada e são expurgados ao
  fim de cada cena; não podem voltar a materializar centenas de downloads/GB.
- Métricas obrigatórias: cobertura por onda, motivos de rejeição, consultas, Vision,
  downloads de candidato, arquivos persistidos, bytes temporários residuais e taxa
  final de realocação ao VEO.

O portão de cobertura deve acionar as ondas seguintes; não deve declarar sucesso
definitivo aos 2,4 min enquanto a realocação superar 10%.
