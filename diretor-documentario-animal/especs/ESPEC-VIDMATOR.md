# ESPEC — VidMator nosso (port do Director + integração AC-Automator)

**Criada:** 03/08/2026 · **Status:** aguardando GO da Fase 0
**Contexto:** servidor de render chega ~fim de semana (comprado). Tudo que for validado
aqui na máquina atual migra pra lá — por isso portabilidade e runbook são LEI, não detalhe.

---

## 0. Decisões fixadas (não rediscutir sem fato novo)

| # | Decisão | Por quê |
|---|---|---|
| D1 | **Repo único**: tudo neste repo (`D:\AC Creator HUB\Vidmator`) — director, remotion, bridge, acervo, docs | Carlos 03/08. Sem o split do Piter. Push só quando validar (GitHub ainda não criado) |
| D2 | **Alvo = caminho v4** do `director-repo` (o que RODA em produção nos canais do Piter desde 02/07) | v5 é canteiro (último commit: "pendência de render"). v5 = opt-in futuro |
| D3 | **PoC com trecho CURTO** (1-2 parágrafos, ~60-90s de áudio) antes de roteiro completo | Carlos 03/08. Ciclo de validação de minutos, não de horas |
| D4 | **Janelas ociosas** da produção pra testes pesados (GPU compartilhada com produção diária) | Whisper do Director + render Remotion disputam a RTX 4050 com o pipeline atual |
| D5 | **Paths e interpretadores SÓ em `vidmator.config.json`** (raiz do repo) — zero hard-code | O erro do Piter (`F:/Canal Dark/` em 15 arquivos) tornou o código dele imóvel. Migração = editar 1 arquivo |
| D6 | **`RUNBOOK-SERVIDOR.md`** atualizado a CADA dependência instalada | A montagem do servidor vira checklist testado, não redescoberta |
| D7 | **SearXNG entra**: standup na Fase 0, integração no resolver na Fase 2 | Pedido Carlos 03/08. v4 não tem; v5 já tem o padrão (`fontes5.searxng_img`, env `SEARXNG_URL`) — reaproveitar |
| D8 | AC-Automator recebe SÓ o gancho fino (campo `motor` + chamada ao bridge), na Fase 3, default `simples` | Regra do aditivo — produção diária intocada. 16 canais sem o campo = zero mudança |

## 1. Fontes e inventário

| O quê | Onde | Papel |
|---|---|---|
| Director do Piter (vivo) | `director-repo\` ← github violipte/vidmator-director | fonte do port (caminho v4: `producao.py` + 14 passes + `remotion/`) |
| Automator do Piter (vivo) | `piter-repo\` ← github violipte/video-automator | referência do gancho (`render_worker.py:557,673` + `vidmator_render.py`) |
| Snapshot 18/07 | `_congelado-2026-07-18\` | exemplos de artefatos de run (`timeline.*.json`, `words.*.json`) que os repos gitignoram — úteis pra entender o schema |
| Nosso acervo | `acervo\` | 272 trilhas (−19 LUFS, `catalogo_trilhas.json` = formato que `trilha.py` lê DIRETO) · 126 SFX · 48 overlays · 35 transições · 9 LUTs · mapas v1 |
| Insumos de teste | produção real do AC-Automator | roteiro + MP3 já narrados, à vontade |

## 2. Arquitetura alvo

```
Vidmator\  (este repo)
├── vidmator.config.json      ← ÚNICA fonte de paths/interpretadores/keys-ref (D5)
├── director\                 ← port dos passes (v4) com paths via config
├── remotion\                 ← port das composições + renderers (npm install local)
├── bridge\                   ← CLI: roteiro+mp3 → MP4 (Fase 1)
├── acervo\                   ← já pronto
├── RUNBOOK-SERVIDOR.md       ← nasce na Fase 0
└── (clones do Piter, gitignored — material-fonte)

AC-Automator (Fase 3, aditivo):
  template.motor = "simples" (default) | "vidmator"
  render path → if motor=="vidmator": chama Vidmator\bridge\ (subprocess)
```

---

## FASE 0 — PoC: o Director roda aqui? (gate G0)

Zero linha do AC-Automator. Zero mudança nos repos do Piter. Tudo em sandbox.

**0.1 — Ambiente Python dos passes.** Tentar na ordem: (a) o `pip freeze` do Piter se
chegar; (b) Python 3.14 + `rembg onnxruntime pillow httpx faster-whisper`; (c) se wheel
faltar em 3.14 → 3.12 (os passes não usam sintaxe 3.14; o 3.14 foi escolha dele, não exigência).
Registrar CADA passo no runbook.

**0.2 — Node + Remotion.** `npm install` no nosso `remotion\` (port das composições do
director-repo). Remotion ~4.0.475 (versão do package.json dele). Smoke test:
`npx remotion compositions` lista as 40 composições sem erro.

**0.3 — Port mínimo com config.** Copiar do `director-repo` para `director\` os arquivos do
caminho v4 (14 passes + `producao.py` + `transcrever_words.py` + `preparar_render.py` +
`presets.json` + libs `gemini_api/pexels_api/preset`), trocando TODO path fixo por leitura
do `vidmator.config.json`. Diff documentado (o que mudou vs original — pra facilitar
re-sync quando o Piter atualizar).

**0.4 — Credenciais.** Formato = idêntico ao nosso `credentials.json`/`config.json`.
Adicionar entradas `pexels` (key já temos, do acervo). Gemini/GPT já existem.
⚠️ Vários passes usam "Gemini CLI primário, API fallback" — validar se o fallback API
segura sozinho (não temos Gemini CLI; se precisar, runbook).

**0.5 — SearXNG standup (D7).** Docker Desktop (se ausente, instalar — runbook) +
container searxng oficial + `SEARXNG_URL` no config. Validar: 1 query de imagem via
`format=json` devolvendo resultados. SÓ standup — integração é Fase 2.

**0.6 — Insumo curto (D3).** Escolher 1 roteiro real; cortar os 2 primeiros parágrafos
(`roteiro_curto.txt`) + cortar o MP3 correspondente no fim de frase (`ffmpeg -t ~75s`).
O Whisper transcreve o áudio cortado — roteiro e áudio precisam casar no trecho.

**0.7 — Rodada manual.** Na ordem: `transcrever_words` → 14 passes → `preparar_render` →
`render-broll.mjs`. Nicho do preset: `default` (o espiritual nosso nasce na Fase 2).
Rodar em janela ociosa (D4). Anotar cada erro e conserto no runbook.

**0.8 — Medições.** Tempo por etapa · pico de VRAM/RAM · nº de chamadas Gemini/GPT/Pexels
(custo real por minuto de vídeo) · tamanho do MP4.

**GATE G0 (todos obrigatórios):**
- [ ] MP4 do trecho curto ABRE e é assistível (footage casando com a fala, legenda, trilha)
- [ ] Nenhum path fora do `vidmator.config.json`
- [ ] SearXNG respondendo query JSON
- [ ] Runbook reproduz o ambiente do zero (teste mental: "daria pra seguir no servidor?")
- [ ] Custo por vídeo estimado e anotado
**Se G0 falhar** por dependência impossível nesta máquina → item vai pro runbook como
"validar no servidor" e a fase segue com o que rodar. Falha total = parar e reavaliar com Piter.

## FASE 1 — Bridge nosso (gate G1)

`bridge\render_vidmator.py`: CLI `python render_vidmator.py --roteiro X --mp3 Y --out Z
[--nicho N]`. Workspace DENTRO do repo (`bridge\_workspace\`), lock atômico (os.mkdir, igual
Piter), gate de negócio produto_cta preservado, retry de chunks preservado.
**Desenhar já com modo `JOB` isolado** (o `preparar_render.py` v4 já suporta via env — no
servidor isso destrava renders paralelos).

**GATE G1:** mesmo MP4 do G0 saindo por UM comando · dois jobs em sequência sem
contaminação de workspace · roteiro COMPLETO (28min) rendido com sucesso e cronometrado.

## FASE 2 — Tempero: nossa cara (gate G2)

- **2.1 Preset `espiritual`** no nosso `presets.json` (não existe no do Piter): fontes,
  efeitos permitidos, cold-open, video_frac, SEM produto_cta por ora (nosso CTA é o
  comentário fixado — decidir se takeover entra depois).
- **2.2 Trilhas nossas**: apontar `MUSICA_PASTA` pros nossos bancos (encaixe direto já
  verificado). ⚠️ QA de ouvido das 272 faixas (pendência do operador) acontece AQUI,
  ouvindo sob narração de verdade.
- **2.3 SFX/overlays/transições/LUTs nossos**: mapear o que o Remotion consome
  (`sfx_lib.json`, componentes) e plugar o acervo onde couber. O que não couber vira
  backlog com nota (não forçar).
- **2.4 SearXNG no resolver (D7)**: portar o padrão `searxng_img` da v5 como fonte
  adicional da cascata v4, atrás do gate de Vision (score/veto protege licença/qualidade).
- **2.5 Mapas**: conferir se o motor de mapas v1 nosso (ESPEC-MAPAS) conversa com o
  `detectar_mapas`/`MapAnimation` — só levantamento, integração é backlog próprio.

**GATE G2:** 1 vídeo completo do nicho espiritual com preset nosso + trilha nossa,
aprovado NO OLHO pelo operador.

## FASE 3 — Gancho no AC-Automator (gate G3)

Aditivo puro: campo `motor` no template (default `"simples"`), branch no render path
chamando o bridge por subprocess, progresso no monitor. Nenhum template existente ganha
o campo. Lock do bridge respeitado (render_queue já é 1 worker — dupla proteção).

**GATE G3:** produção normal de 1 canal atual passa INTACTA (regressão zero) · 1 job
`motor=vidmator` disparado pela UI sai MP4 no lugar certo com naming certo.

## FASE 4 — Piloto (gate G4)

Canal NOVO (nunca um que fatura). E2E: tema → roteiro → narração → VidMator → publicação.
**GATE G4:** primeiro vídeo publicado + custo/tempo por vídeo medidos em produção real.

---

## Transversais

**Migração (o motivo de tudo):** ao fim de cada fase, perguntar "o que disto depende DESTA
máquina?" — resposta vai pro runbook. Meta: servidor = clone do repo + `git clone` dos 2
repos do Piter + runbook passo-a-passo + editar `vidmator.config.json`.

**Riscos:**
| Risco | Mitigação |
|---|---|
| Wheels 3.14 (rembg/onnxruntime) | fallback 3.12; pip freeze do Piter se vier |
| GPU contention com produção | D4 (janelas ociosas); no servidor some |
| Gemini free tier 429 em massa | runbook Piter: Vision em lote = gpt-4o-mini detail:low (~$0.0005/img) |
| Sem Gemini CLI (passes preferem CLI) | validar fallback API na 0.7; instalar CLI se necessário |
| v5 instável contaminar o port | D2: só arquivos do caminho v4; v5 é leitura |
| Docker indisponível p/ SearXNG aqui | env `SEARXNG_URL` é opcional por design — degrada limpo; valida no servidor |
| Repo do Piter mudar embaixo de nós | port é CÓPIA com diff documentado (0.3); re-sync é decisão, nunca autômato |

**O que NÃO fazemos:** mexer nos clones do Piter · pushar antes do Carlos criar o repo no
GitHub · tocar template/canal existente antes da Fase 3 · adotar v5 antes do Piter fechar.

**Registro:** cada fase fechada = commit neste repo + entrada no runbook + medições na
tabela abaixo.

| Medição | Trecho curto (G0) | Completo (G1) | Piloto (G4) |
|---|---|---|---|
| Tempo total render | | | |
| Pico VRAM / RAM | | | |
| Chamadas LLM/Vision (custo) | | | |
| Downloads Pexels/Commons | | | |
