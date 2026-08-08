# ESPEC-V7-FINAL — o arremate completo até disparar o v7

> Consolidação pedida pelo operador (06/08): autópsia da quarentena v6b + ESPEC-AJUSTES-V7
> (A1-A7) + adoção do funcionário VEO no pipeline real. **Quando tudo aqui estiver
> executado, o v7 dispara** (animal novo — o operador revela no GO).

## 0 · O que JÁ está pronto (não refazer)

Funcionário VEO completo no container `_import/2026-08-06-veo-flow/`:
- **Lei do Take** (contemplativo, 1 sujeito, câmera footage, 0 trilha positiva) — `86802cd`
- **Lei da Ilustração** (cutaway semi-realista, anatomia only, NB Pro travado no chip) — `53026d7`
- **Fronteira anatomia×medida** (`escopo_ilustracao`, medida é da lei dos números) — `b8681e3`
- **M1 rótulos ditados** + triagem take×ilustração DENTRO do veo_pedido — `01d900c`
- **Ciclo `--tipo ambos`** (lote misto, imagem→vídeo, pacing upstream corrigido) — `350d876`
- **Lei da Divisão** (40/36/24 intercalado, anatomia pinada) + **Lei do Socorro**
  (reuso-que-serve → gerar; régua de espécie mútua) — `bd55c22`
- Sessão viva (attach CDP, conta1), rota de download robusta (lista/coleção), 0 créditos.

---

## PARTE A — Autópsia da quarentena v6b (evidência, não palpite)

### A-F2 · "minutos abaixo de 12 cortes" (os dois vídeos)
**FATO**: v6b re-renderizou TIMELINES do v5 (cenas longas + heranças). A máquina fresh
já existe no `montar_timeline` (F1: teto 6s, alvo 4,5s → 10-13 cortes/min; elementos e
transições completam). **Só produção FRESH testa o F2 de verdade** — e o VEO elimina a
herança (cena sem take vira take fabricado, não clone do vizinho).
**E1 (novo, barato)**: checagem de cadência PRÉ-RENDER — depois do montar, contar cortes
projetados por minuto; minuto <12 → apertar alvo local para 4s e re-particionar aquele
trecho. Pega o problema em segundos, não depois de 40min de render.

### A-D5 · "1s de tela preta em 181s" (urso) — **É O A4. MESMO BUG.**
**PROVA** (frames extraídos do mp4 quarentenado, 180.4-182.8s): MapAnimation com
`pais: null` dá zoom no polo norte → oceano azul-escuro vazio + pontinho + "ARCTIC".
O D5 lê YAVG/YMAX baixíssimos = "preto". **Fix do A4 mata o D5@181 junto.**

### A-F3 · "minuto 2 com 4-5 elementos (piso 6 no hook)" — a pergunta do KineticText
**Resposta: SIM, o garimpo de punchline é míope hoje.** Diagnóstico no código
([edicao.py:714](director/edicao.py#L714)):
1. `_frase_digna` só aceita **CLÁUSULA INTEIRA de 2-7 palavras** (o R6 antisserrote está
   correto — mas minuto de frases longas rende ZERO candidatos);
2. âncoras ocupadas (cena com infográfico/texto próprio) saem do jogo → sobram poucos
   slots justamente nos minutos densos de conteúdo.
**E2 (fix)**: passe de **GARIMPO DE PUNCHLINE** — LLM lê o minuto inteiro e devolve, por
cena-âncora, a punchline **ENCURTADA SEMANTICAMENTE** para 3-6 palavras (encurtar com
semântica ≠ serrote sintático; "he leaves the muscle and the organs" → "HE LEAVES THE
ORGANS"). Determinístico atual vira fallback. O PISO vira obrigação: minuto abaixo →
re-garimpo com régua mais frouxa até bater (ou log alto do motivo).

---

## PARTE B — ESPEC-AJUSTES-V7 (A1-A7), plano fino

| # | O quê | Plano | Decisão do operador? |
|---|---|---|---|
| A1 | 0:13 cinza ilegível | 1 frame confirma o componente → cor secundária cinza→**branco NA RAIZ** | não |
| A2 | 0:24 duas medidas num elemento | detector R7 quebra frase com 2 dimensões: 1ª fica texto, 2ª vira **counter** (igual 4:19) | não |
| A3 | 2:38 animação+SFX descartados de novo | ✅ **DECIDIDO (06/08): BANIR o visual `Ovl06_CenterPunch` DE VEZ** (tabela+pools, cirurgia R2) | decidido |
| A4 | mapa sem país (Arctic) | lugar sem shape → **variante ponto+legenda/satélite por coordenada**, elegibilidade no `detectar_mapas`; **mata o D5@181** | não |
| A5 | texto desalinhado do áudio | Sherlock com medição: `aparece_em` gravado × instante FALADO da âncora (words.json) nos elementos do v6b; só mexer com causa provada | não (ainda) |
| A6 | V4 herdou 14 cenas (teaser hook) | ✅ **DECIDIDO (06/08): TEASER LIBERADO** — V4 ganha janela de exceção SÓ no hook (~30s iniciais): flashes dos animais que vêm pela frente valem; corpo do vídeo segue 100% sob embargo | decidido |
| A7 | F2 estrutural | coberto pela PARTE A (fresh + E1) | não |

---

## PARTE C — ADOÇÃO do funcionário VEO no pipeline real (o "apontar diferente")

- **C1** `veo_alocacao` entra na ordem do `_rodar_e2e`: **depois do contrato_visual,
  antes do resolver_cascata** (grava `fonte_alvo`/`fonte_motivo` em toda cena).
- **C2** `resolver_cascata` ganha knob **`VM_VEO=1`** (aditivo, OFF default): busca
  stock SÓ para `fonte_alvo=="banco"`. Sem knob, comportamento de hoje intacto.
- **C3** fase de GERAÇÃO no pipeline (entre resolver e época/edição):
  `veo_pedido` (cenas veo_*) → `veo_ciclo --tipo ambos` → **curador** (gate visual,
  keep/) → `veo_entrega` (pool + timeline por ÍNDICE) → `veo_socorro` (reuso/gerar)
  → re-ciclo do `_gerar_socorro.json` se houver. Barreira V1-V6 continua soberana.
- **C4** ÁUDIO dos clipes VEO — ✅ **IMPLEMENTADO E BENCHADO (06/08)**. Decisão em
  duas rodadas: "-16 martelo" → ao ver a conta, o operador escolheu MINHA opção:
  *"calibre tudo do seu jeito, -30 dB; depois de rodar um teste real eu só valido ou
  não — melhor meio baixo (passa) do que meio alto (incomoda)."* Implementação:
  `veo_entrega` normaliza CADA clipe a **Mmax -30** (ebur128, doutrina SFX) na porta
  do pool; `preparar_render` marca `take_audio_vol=1.0` só para fonte veo*/vídeo;
  `SceneClip` toca o áudio do take com fade-in 0.4s (stock segue MUDO). Bench real:
  s029 veio a Mmax **-11.7** (mais alto que SFX de impacto!) → -18.3 dB → -30.0.
  ⏳ validação de ouvido no teste real do v7.
  🔴 Descoberta da bancada: o download da vista em lista do Flow embrulha TUDO em
  ZIP — os 6 "mp4" do ensaio eram PK disfarçado (o operador assistiu no
  Flow, os arquivos em disco eram bombas). Desembrulhados; a PORTA DO POOL agora
  detecta magic PK e extrai a mídia sozinha (defesa definitiva).
- **C5** adoção física: container → `director/` (passo 8 da sanitização), knob OFF
  até o GO; prova de idempotência ganha OWNED do `veo_alocacao` (`fonte_alvo`).
- **C6** infra: registrar o CANAL REAL em `_veo_flow/projetos.json` (hoje só ENSAIO);
  coleção por vídeo (nome = data/animal); conta1 basta para 1 vídeo/dia.

## PARTE D — Ordem de execução até o GO

1. **Fase 1 — cirurgias** (A1+A2+A3+A4): componentes/repertório; mata D5@181 junto.
2. **Fase 2 — densidade** (E2 garimpo de punchline + piso obrigatório; E1 cadência).
3. **Fase 3 — adoção VEO** (C1→C6, knob ON em bancada; medir e calibrar C4).
4. **Fase 4 — A5** (medição do desalinhamento na bancada do v7; fix só com causa).
5. **Fase 5 — prova geral**: idempotência + bancada F2 sintética + e2e curto com VEO.
6. **GO**: operador revela o animal → **v7 fresh** com tudo ligado.

**Decisões — TODAS BATIDAS (06/08)**: A3 ✅ banir `Ovl06_CenterPunch` de vez ·
C4 ✅ ASMR abaixo da TRILHA (composição, não elemento) · A6 ✅ teaser liberado no hook
(janela de exceção do V4 nos ~30s iniciais; corpo 100% sob embargo).
**Zero decisões pendentes — as 5 fases estão liberadas para execução.**

*Criada em 06/08/2026 — consolida ESPEC-AJUSTES-V7.md (arquivada) + autópsia da
quarentena + adoção VEO. Executar nesta ordem deixa o v7 livre.*

---

## 📋 EXECUÇÃO (06/08, sessão autônoma — "pode ir trabalhando até a hora do animal")

| Fase | Estado | Prova |
|---|---|---|
| 1 · A1+A2+A3+A4 | ✅ `2d71bf7` | frame 0:13 identificou o BeforeAfter cinza; A2 benchado no caso real; A4 força PinCallout (mata D5@181) |
| A6 teaser | ✅ `bcbad34` | janela <30s no V4 |
| C4 ASMR | ✅ `c5b9dfd` | s029 real: Mmax -11.7 → -30.0; 🔴 zips disfarçados de mp4 pegos e porta do pool blindada |
| 2 · E1 cadência | ✅ `6813804` | narração real: +12 cortes → nenhum minuto <12 |
| 2 · E2 garimpo | ✅ `896e726` | Gemini real: 6/6 punchlines válidas (subsequência) |
| 3 · C1+C2+C3 | ✅ `15b6b8b` | shims na cadeia PASSES (no-op sem VM_VEO); resolver filtra banco; alocação idempotente (2 rodadas = byte igual); pedido misto respeita cota (foto_cota com fórmula própria, espécie primária) |
| 4 · A5 | ✅ `c795917` | MEDIDO: kinetics adiantados 2-6s (âncora no início da cena); fix = âncora falada; 6/6 infratores reais caem para -0.45s |
| 5 · prova geral | ✅ COMPLETA | cadeia REAL e2e: 9 itens mistos → ciclo ambos (5 img 1ª rodada + 4 vid c/ pacing de rajada visível) → vision_gate 4/4 APROVADOS → entrega 9/9 por índice c/ ASMR (-18.0/-28.2/-25.2/-26.4 → -30.0, gerações DESIGUAIS provadas de novo) → socorro 0 órfãs |

**No GO do v7 (checklist de 2 minutos):**
1. Operador fala o ANIMAL → roteiro (pipeline de roteiro) → narração.
2. `projetos.json`: registrar canal real (ou usar ENSAIO) + coleção = animal/data (C6).
3. Rodar `_rodar_e2e` com `VM_VEO=1` (e `VM_VEO_CANAL`/`VM_VEO_COLECAO`).
4. Validações de ouvido/olho do operador: ASMR -30 (C4), estilo das ilustrações no
   contexto, fita da divisão. O que reprovar vira ajuste de constante, não de estrutura.

### Prova geral — o que ela pegou e consertou no caminho (3 tentativas)
1. `criar_colecao` BLOQUEAVA no rename da UI (campo aceita, Enter não confirma —
   medido ao vivo) → rename virou COSMÉTICO; a identidade é o ID capturado ao entrar
   (`nome→id` no projetos.json; rodadas seguintes = rota direta, zero clique). `2ad9013`
2. `vision_gate` não tinha vindo na tradução da frota → traduzido (CREDS via
   vmconfig), validado: 4/4 vídeos aprovados no gate real. `dda14d9`
3. Colisão de automações simultâneas (TaskStop mata só o wrapper; ciclo filho segue
   vivo) → lição em memória; ciclo é idempotente, relançar é seguro.
⚠️ Buraco conhecido (anotado): arquivo já baixado em rodada anterior NÃO passa pelo
gate no re-run (faltam=0 pula a rodada) — gate coberto manualmente na prova; para o
v7 fresh não afeta (arquivos nascem e são gateados na mesma rodada).

**PRONTO PARA O ANIMAL (06/08, ~19:45).** Checklist do GO no topo da PARTE D.
