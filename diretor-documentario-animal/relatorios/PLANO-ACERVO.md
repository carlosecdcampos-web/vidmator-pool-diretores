# PLANO DE AÇÃO — Acervo Vidmator

> Atualizado: 19/07/2026 · Documento-guia pra não nos perdermos.
> Regra de manutenção: toda sessão que mexer no acervo ATUALIZA este arquivo
> (estado + fila). O detalhe fino de cada frente mora nas skills
> (`sonoplasta-trilhas`, `sonoplasta-sfx`, `acervo-overlays`).

## ESTADO ATUAL

| Frente | Status | Volume | Skill |
|---|---|---|---|
| **SFX** | ✅ operacional (3 rodadas) | **126** catalogados, 16 famílias | sonoplasta-sfx |
| **Overlays (vídeo)** | ✅ operacional (2 rodadas + curadoria do operador) | **32** catalogados, 12 categorias, 100% 16:9 | acervo-overlays |
| **Trilhas** | ✅ operacional (doc + espiritual + geral via Suno) | **272** catalogadas (doc 96 + espiritual 90 + geral 86) | sonoplasta-trilhas |
| Transitions | não iniciada | — | (futura) |
| LUTs | aprovada, não iniciada | — | (dentro da acervo-overlays por ora) |
| Mapas dinâmicos | ✅ V1 APROVADO 20/07 (12 receitas, 8 modos) | 12 | manifesto.json + ESPEC-MAPAS §11 |

Infra pronta: Pexels API key ✓ (`.env` do toolkit; exige User-Agent de navegador) ·
Mixkit via curl ✓ · manifestos com proveniência ✓ · contact sheets ✓ ·
auditoria de integridade obrigatória ✓ (lição do fantasma OVL-0020).
Chaves pendentes (opcionais): ACE-Step (trilhas) · ElevenLabs (SFX geração) ·
Pixabay API (alternativa ao navegador).

## FILA DE EXECUÇÃO (ordem acordada com o operador)

### PASSO 1 — Overlays de IMAGEM (molduras + filmadoras) — ✅ PARCIAL 19/07
1.1. ✅ **Molduras: 6 catalogadas** (OVL-0036..0041, Pixabay API): 2 grunge
     (incl. a de traço = exemplo do operador), papel envelhecido, frame 35mm
     com sprockets (★), 2 filmstrips. 5 com ALPHA REAL, 1 multiply.
     Campo `tipo: imagem` no manifesto; teste de alpha por alphaextract+YAVG.
1.2. ⚠️ **Viewfinder REC: NÃO EXISTE em stock gratuito** (Pixabay/Pexels só
     têm ícones de câmera; os exemplos do operador eram Freepik/Getty = licença
     paga). DECISÃO RECOMENDADA: fazer PROCEDURAL no Remotion (lado Piter —
     brackets + REC piscando + timestamp; customizável) → levar pro checkpoint
     com ele. Alternativa futura: openclipart/SVG CC0 (validar acervo).
1.3. Pixabay API key ATIVA (`.env`; 100 req/min; largeImageURL = 1280px máx
     no tier free; pedem uso gentil — sem downloads em massa).
1.4. Aberto: mais molduras grunge variadas (quando quiser engordar).

### PASSO 2 — Overlays vídeo: fechar buracos — ✅ CONCLUÍDO 19/07
2.1. ✅ +12 catalogados (OVL-0048..0059): fog mist+roll (categoria nova),
     rays beam★+burst (nova), vhs static+scanlines+glitch (nova),
     lightning clouds (nova), grain escuro screen, scratches fios,
     bokeh soft + gold minimal. Banco: **48 overlays, 16 categorias**.
2.2. ⚠️ **Fireflies: 2 tentativas, ZERO em stock** (só cenas de floresta;
     vagalume real filmado não existe grátis). DECISÃO: geração procedural
     (nosso script de partículas pulsantes OU ParticlesDrift do Remotion
     adaptado — checkpoint Piter). Pendura junto com viewfinder REC animado.
2.3. Buracos menores restantes (engordar quando quiser): lightning bolt
     de verdade em fundo preto, rain/snow na lente, water caustics,
     clouds timelapse, paper/ink.

### PASSO 3 — Pixabay: explorar (navegador integrado OU API própria)
3.1. Decidir a via: navegador integrado (zero cadastro) vs API oficial
     Pixabay (key gratuita, mais robusta — recomendada se o garimpo por lá
     virar rotina).
3.2. Usar pra: imagens do Passo 1, buracos do Passo 2, e avaliar o acervo
     deles de vídeo overlay em geral.

### PASSO 4 — LUTs (.cube) — ✅ CONCLUÍDO 19/07 (via GERAÇÃO PRÓPRIA)
4.1. ✅ `acervo\luts\` criado: **9 LUTs P&B GERADOS POR NÓS** (LUT-0001..0009,
     zero licença): neutro, contraste, noir, suave/fade, filtro-vermelho,
     filtro-verde, prata, selênio (frio ★espiritual), sépia (quente ★western).
4.2. ✅ QA em frame REAL (Jack Rowen 22/07): contact sheet em
     `luts\_previews\pb_aplicados_jack_rowen.jpg`. Gerador editável em
     `luts\_geradores\gerar_luts_pb.py` (mix de canais + curva S + tint).
4.3. Garimpo de packs (RocketStock/IWLTBAP) ficou DESNECESSÁRIO pro P&B;
     reabrir só pra looks coloridos de marca (licença um a um). Próxima leva
     possível pelo MESMO script: vintage warm, teal dark espiritual.
4.4. Uso: `ffmpeg -vf lut3d=arquivo.cube` (Remotion/Director aplicam igual).

### PASSO 5 — SFX rodada 3 (geração) — ⏭️ PULADO (operador 19/07: sem key EL)
5.1. Buracos de SFX de design ficam pra garimpo CC0 futuro (Pixabay/Freesound)
     quando fizer sentido: drone_dark, whoosh_reverse, ka-ching, boom cine.

### PASSO 6 — Transitions (4ª skill) — ✅ INAUGURADO 19/07
6.1. ✅ Skill `acervo-transitions` criada (regras X1-X4: arco obrigatório
     constrói→cobre→libera verificado em frames 20/50/80%, modo
     luma/screen/insert, pico_s, SFX PAR obrigatório validado contra o banco).
6.2. ✅ Rodada 1: **4 catalogadas** (TRS-0001..0004): 3 ink luma mattes
     (bloom/multi/drop — coberturas 95-97%) + glitchburst insert. Cada uma
     com sfx_par + sfx_alt apontando pro banco de SFX.
6.3. ✅ **FÁBRICA DE RECEITAS PROCEDURAIS (19/07 noite)**: decupagem das 33
     transições favoritas do operador (CapCut) + 4 rodadas de implementação
     com demos aprovadas uma a uma. **PLACAR: 26 de 33 no catálogo**
     (4 assets + 22 receitas FFmpeg com SFX par). Descartadas por ele:
     clarão, brilho_2, fita_scotch. Doc: `transitions\DECUPAGEM-CAPCUT-
     FAVORITAS.md` (com 3 regras técnicas: format=yuv420p sempre; blend de
     luz em RGB/gbrp senão magenta; PTS+X/TB pra clipe atrasado).
6.4. ✅ **gl-transitions FECHADO POR NÓS (20/07)** — sem batata quente pro
     Piter. Renderizador WebGL próprio em `transitions\_geradores\gl_render\`
     (Playwright headless + ~125 shaders GLSL MIT/BSD; auto-retry pro
     não-determinismo do SwiftShader). 9 aprovadas cobrindo os 6 slots
     (TRS-0027..0035). **TODAS AS 33 favoritas do CapCut cobertas.**
     Ferramenta reutilizável: `python render_gl.py <shader> A.mp4 B.mp4 out.mp4`.
6.5. Catálogo de transições FECHADO: 35 entradas (22 receitas FFmpeg + 4
     assets + 9 gl). Fireflies procedurais (OVL-0060) também fechados.

### PASSO 7 — Trilhas — ✅ LOTE 1 (DOCUMENTÁRIO) CONCLUÍDO 26/07
7.0. Motor decidido pelo operador: **Suno** (via extensão de letras automáticas
     dele, formato LEI `LETRA N / LYRICS: / STYLES: / Song Title:` — adaptado p/
     instrumental: só tags em colchetes + "Instrumental, no vocals" no STYLES).
     ACE-Step ficou de reserva (key acemusic nunca colada — segue vazia no .env).
7.1. ✅ **96 faixas catalogadas** no banco `documentario` (lote de 48 prompts ×
     2 versões que o operador manteve — takes ligeiramente diferentes). Todas
     −19 LUFS (verificado −18.6/−19.9), naming `<prefixo>_NNN.mp3`, manifesto
     TRK-0001..0096 + catalogo_trilhas.json. Prompt-fonte em
     `acervo\trilhas\_lotes\documentario_lote1.txt`.
     Distribuição: tenso 16, frio 14, revelacao 14, nostalgia 12, sombrio 10,
     epico 8, urgente 6, build 6, stinger 6, hook 4.
7.2. ⚠️ 3 hooks vieram longos (119-155s vs alvo 30-60s) — funcionam como
     abertura (Director pega o começo) ou viram bed extra; trim opcional.
     QA sob narração (gates T1/T2 no ouvido) PENDENTE do operador — manifesto
     marca isso; dud → remover o `<prefixo>_NNN` + entrada.
7.3. Próximo: banco `espiritual` + `geral` (mesmo fluxo Suno) quando o operador
     pedir; aprofundar moods se algum ficar magro (pool principal já maduro).

### PASSO 8 — Mapas dinâmicos — ✅ V1 IMPLEMENTADO E APROVADO 20/07
(motor em acervo/mapas/_engine · catálogo manifesto.json MAP-0001..0012 ·
contrato/regras na ESPEC-MAPAS §11 · aprovações do operador: destaque hero,
rota A→B, multi-ponto, ida-volta, marítima mar-aberto, bateria dos 8 modos ·
pendências v2 no manifesto: GIBS, admin-1, atlas 50m, sfx navio)
8.1. ✅ Base do Piter estudada: MapAnimation (zoom+destaque país, d3-geo/
     Natural Earth), SatelliteZoom (pirâmide de tiles), detectar_mapas
     (LLM acha lugares+coords+timestamps no roteiro). Faltam: ROTAS,
     estados, capitais, estilo docu.
8.2. ✅ Brainstorm alinhado com o operador (4 decisões): protótipo NOSSO
     plug-and-play pro Remotion · tema dark universal com cor por canal ·
     rotas com TODOS os modos (avião/navio/trem/carro/bike/cavalo/exército/
     a pé) · 2D primeiro, satélite v2 via NASA GIBS (PD).
8.3. ✅ **`ESPEC-MAPAS.md`** — o MD rico: arquitetura (_engine com map_core.js
     puro = o plug pro Remotion), dados Natural Earth (PD: países+ESTADOS+
     capitais offline), 3 blocos (destaque/rota/ponto), tema.json, modos.json
     com SFX do banco por modo, contrato mapa_job.json, v2 satélite, plano
     de execução em 6 passos.
8.4. ⏳ Mão na massa: aguardando GO do operador.

## TRANSVERSAIS (sem dono de passo, fazer quando fizer sentido)

- **Sync pro Drive**: espelhar `acervo\` (layout já é Drive-friendly) —
  decisão de infra pendente.
- **Checkpoint com o Piter**: quando a decupagem dele fechar, a lista de
  animações vira lista de demanda (SFX pareados, overlays, transitions).
  Viewfinder/REC procedural em Remotion é candidato natural pro lado dele.
- **Regras permanentes do operador** (valem pra TODO o acervo):
  royalty-free comercial sem risco · 16:9 only (overlays) · arquivo único
  com dedup VISUAL · minimalista/orgânico, "nada grosseiro" · proveniência
  registrada de cada asset · auditoria de integridade no fim de toda rodada.

## LOG DE MARCOS

- 18/07: skills trilhas+sfx construídas · SFX rodadas 1-2 (116) · repos
  Piter curados em `_referencias\` · incidente OpenAI resolvido (key velha).
- 19/07: SFX typewriter (126) · skill overlays + rodadas 1, 2A, 2B (32) ·
  curadoria do operador (O7-O9) · Pexels key ativa · Jack Rowen 20 e 22/07
  agendados · proxies dos 3 musicais trocados e validados.
- 26/07: **Trilhas Passo 7 destravado via Suno** (não ACE-Step) — lote 1
  documentário: 48 prompts → operador gerou as 2 versões → intake de 96 faixas
  (−19 LUFS, naming, manifesto TRK-0001..0096, catálogo). Banco documentário
  operacional. Originais do operador em `Downloads\Trhinhas - primeira remessa`.
- 26/07: **Banco espiritual** (lote 1, 45 prompts × 2 versões) → intake de 90
  faixas (TRK-0097..0186), MOVIDAS (Downloads esvaziado, sem cópia). Manifesto
  global = 186. Hooks (doc 3 + esp 4) aparados p/ 45s. ⚠️ Suno estica material
  etéreo/ambiente: 4 "stingers" espirituais vieram longos (Light Shimmer 170/153s,
  Veil Toll 58/68s) — PENDENTE decisão do operador (reclassificar p/ bed ou
  aparar p/ acento). Stinger é o papel que o Suno faz pior — candidato a ACE-Step
  (--duration exata) numa próxima.
- 27/07: **stingers longos do espiritual RESOLVIDOS** — Light Shimmer (170/153s)
  reclassificados p/ beds etéreos (`etereo_017/018`, etéreo 16→18); Veil Toll
  aparados p/ acento de 12s. Espiritual: stinger fica com 4 próprios. **Banco
  geral (lote 1, 43 prompts)** montado em `_lotes\geral_lote1.txt` (coringa:
  neutral 12 + amostras de cada mood + build/hook/stinger) — aguardando operador
  gerar no Suno. Regra confirmada: Suno estica hook/stinger (aparar no intake);
  stinger curto de verdade só via ACE-Step (key acemusic ainda pendente).
- 27/07: **banco GERAL intake concluída** — 86 faixas (43 prompts × 2 versões;
  Tenso 03 e Revelacao 03 vieram com 4 versões; Tenso 02 e Revelacao 02 NÃO
  vieram do Suno). −19 LUFS, movidas (Downloads esvaziado), TRK-0187..0272. 3
  hooks aparados p/ 45s; stingers vieram curtos (ok). **Acervo trilhas = 272**
  (doc 96 · espiritual 90 · geral 86). Buracos: geral falta Tenso 02 + Revelacao
  02 (regenerar quando quiser); pool de stinger ainda fino em todos os bancos.
