# RUNBOOK — montar o VidMator numa máquina nova (servidor)

> **Para que serve:** o servidor de render chega ~fim de semana. Este arquivo é o checklist
> **testado** da instalação — cada passo aqui foi executado de verdade na máquina atual, com
> os erros já resolvidos. Montar o servidor não deve ser redescoberta, deve ser leitura.
>
> Regra (D6 da ESPEC): **toda dependência instalada entra aqui na hora**, com a pegadinha junto.

**Última atualização:** 03/08/2026 · Fase 0 (itens 0.1–0.4 concluídos)

---

## 0. Pré-requisitos da máquina

| Item | Versão validada aqui | Nota |
|---|---|---|
| Windows | 11 Pro | — |
| GPU NVIDIA | RTX 4050 Laptop (6 GB) driver 596.36 | Whisper usa CUDA; render Remotion usa Chrome headless |
| Node.js | v24.15.0 · npm 11.12.1 | Remotion exige Node ≥ 18 |
| Python (passes) | **3.12.10** | ⚠️ ver §2 — NÃO precisa do 3.14 |
| Python (whisper) | 3.13 (env conda `automator`) | tem `faster-whisper` |
| FFmpeg | 8.1.1 (winget Gyan.FFmpeg) | no PATH |
| Disco | ~10 GB livres p/ deps + cache | acervo do repo já são ~1,8 GB |

## 1. Clonar

```bash
git clone <nosso repo vidmator> "D:/AC Creator HUB/Vidmator"
git clone https://github.com/violipte/video-automator.git   "D:/AC Creator HUB/Vidmator/piter-repo"
git clone https://github.com/violipte/vidmator-director.git "D:/AC Creator HUB/Vidmator/director-repo"
```
Os dois clones do Piter são **material-fonte** (gitignored). O código que roda é `director/` e
`remotion/`, que são o **nosso port** — não os clones.

## 2. Python dos passes — 3.12, não 3.14

⚠️ **O Piter usa Python 3.14; nós NÃO precisamos.** Auditei os 16 arquivos do caminho v4:
nenhum usa sintaxe exclusiva de 3.13/3.14 (nem `match/case`, nem PEP 695). O 3.14 era escolha
dele. No 3.12 as wheels de `rembg`/`onnxruntime` instalam sem briga — que era o risco nº 1 da ESPEC.

```bash
python -m venv "D:/AC Creator HUB/Vidmator/_venv-director"
"D:/AC Creator HUB/Vidmator/_venv-director/Scripts/python.exe" -m pip install -r requirements-director.txt
```

**Pegadinha:** o **primeiro** `import rembg` leva ~2 minutos (JIT do numba compilando cache).
Não é travamento. Depois é instantâneo. Rode uma vez logo após instalar, para tirar esse custo
do caminho do primeiro job.

**Nota:** `onnxruntime` fica em **CPU** (providers: Azure, CPU). O `rembg` só recorta retratos
(poucos por vídeo), então CPU basta. GPU aqui exigiria `onnxruntime-gpu` + CUDA/cuDNN casados —
não vale a briga agora.

## 3. Node / Remotion

```bash
cd "D:/AC Creator HUB/Vidmator/remotion"
npm install --no-audit --no-fund      # ~358 pacotes, ~50s
npx remotion compositions             # smoke test
```

No primeiro render o Remotion baixa o **Chrome Headless Shell** (~113 MB) automaticamente.
Deixe acontecer uma vez antes de cronometrar qualquer coisa.

**Pegadinha resolvida:** `npx remotion compositions` quebrava com
`durationInFrames ... must be an integer, but got NaN`. Causa: as composições `Montagem`/`Montagem5`
(era **v5** do Piter) fazem `fetch` de `public/jobs/hilux_mont/montagem.json`, que o repo dele
gitignora. Solução: existe um **stub** versionado em `remotion/public/jobs/hilux_mont/montagem.json`.
Não apague. O caminho v4 (`BrollTest`) não dependia disso — usa `selectComposition` por id — mas
sem o stub a listagem inteira morre e você perde a ferramenta de diagnóstico.

## 4. Configuração — o arquivo único

**Migrar de máquina = editar `vidmator.config.json` e mais nada** (decisão D5).

Campos que mudam no servidor:
- `raiz`, `workspace`, `remotion`, `director`, `videos_out` → caminhos novos
- `python_passes` → `<raiz>/_venv-director/Scripts/python.exe`
- `python_whisper` → env que tenha `faster-whisper`
- `credentials` → `credentials.json` do AC-Automator (**leitura**; provedores gemini/gpt)
- `config_pexels` → `vidmator.secrets.json` (ver §5)
- `musica_pasta` → banco de trilhas do acervo

O leitor é `director/vmconfig.py`. Ele aceita `VIDMATOR_CONFIG` no ambiente para apontar outro
arquivo (útil p/ rodar dois perfis na mesma máquina).

**Pegadinha:** o config é lido com `utf-8-sig`. O `Set-Content` do PowerShell 5.1 grava **BOM
e faz double-encode** de acentos — se editar por PS, use `Out-File -Encoding utf8` ou um editor
de verdade. Sintoma do double-encode: `"ÃšNICA"` no lugar de `"ÚNICA"`.

## 5. Segredos

`vidmator.secrets.json` (**gitignored**, criar à mão no servidor):
```json
{ "pexels_api_key": "<key>", "pexels_api_keys": ["<key>"] }
```

Por que arquivo próprio: o Director lê o "config.json" **só** para as chaves do Pexels
(auditado — `pexels_api.py` é o único consumidor). Apontando pra cá, o `config.json` de
**produção do AC-Automator não é tocado** (decisão D8: produção só na Fase 3).

As chaves de LLM (Gemini/GPT) são lidas **do** `credentials.json` do AC-Automator, em modo
leitura. Não duplicar chave.

## 6. Verificação — rode nesta ordem

```bash
cd "D:/AC Creator HUB/Vidmator/director"
<venv>/python.exe -c "import vmconfig as v; print(v.TESTE, v.CREDS.exists(), v.MUSICA_PASTA.exists())"
<venv>/python.exe -c "import pexels_api as p; print(len(p.KEYS), len(p.search('ocean waves', kind='videos', n=2)))"
<venv>/python.exe -c "import gemini_api as g; print(g.gemini_text('Responda apenas: OK'))"
```
Esperado: paths resolvidos + `True True` · `1 2` · `OK`.

## 7. O que NÃO precisa instalar

- **Python 3.14** — ver §2.
- **Gemini CLI** — os passes preferem CLI e caem na API; a API sozinha responde (testado).
- **Bancos de mídia do Piter** (`catalogo_stock`, `catalogo_epoca`, `index_*_imagens`) — são
  privados dele. Ficam vazios no config; o código degrada com `if exists()`.

## 8. Pendente (ainda não instalado/validado)

| Item | Estado |
|---|---|
| **SearXNG** (item 0.5 / D7) | Docker Desktop instalado; engine TRAVADO em "Virtualization support not detected". **Diagnóstico feito (03/08): BIOS OK (VT habilitado no firmware) — faltam as FEATURES do Windows** (WSL2/VirtualMachinePlatform; `wsl --status` dá rc=50). Fix: `wsl --install --no-distribution` como ADMIN + **reboot**. Config pronta em `_searxng-config\` (settings.yml com `search.formats: [html,json]` — sem isso a API dá 403 — + `subir_searxng.ps1`, porta 127.0.0.1:8090). ⚠️ Nativo Python foi TENTADO e abortado: o repo tem arquivos com `:` no nome (templates uwsgi/nginx), inválidos no NTFS — clone quebra; contornável com sparse-checkout, mas operador decidiu Docker. No SERVIDOR novo: habilitar WSL2 logo na montagem (mesmo procedimento) |
| Rodada completa (roteiro 28min) | G1 — a PoC de 93s passou inteira (ver §10) |
| SFX/overlays nossos plugados | Fase 2 (blocos `sfx` e `bancos` do config preparados, vazios). Trilhas JÁ plugadas (§10) |

## 10. PoC G0 executada (03/08/2026) — o pipeline RODA nesta máquina

Insumo: roteiro documentário (Amazônia) 1.345 chars EN + narração Chatterbox voz Ashtar.

| Etapa | Tempo | Nota |
|---|---|---|
| Narração (Chatterbox, 5 chunks) | 1,2 min | ⚠️ chunking OBRIGATÓRIO: limite real = `CHATTERBOX_CHUNK_LIMIT=400` chars (~1000 tokens AR ≈ 40s de áudio). Texto inteiro num chunk = TRUNCA SILENCIOSO (1º teste saiu com 35s dos 93s) |
| Whisper words (base, cuda/int8_float16) | ~30 s | sem segfault; 239 palavras |
| 14 passes (NICHO=documentario) | ~4,3 min | resolver 110s (22 cenas Pexels) · mapas 40s · ilustrar 40s · restantes <25s cada |
| preparar_render | ~10 s | copiou 22 clips + 3 trilhas |
| Render Remotion (2 chunks, GPU angle) | ~4,9 min | 93s @1080p30 CRF20 → 189 MB · ~3,2× a duração do vídeo |
| **Total ponta-a-ponta** | **~11 min** | para 93s de vídeo · custo API: $0 (Gemini free ~40 chamadas + Pexels ~30 req) |

**Fixes obrigatórios descobertos na rodada (já commitados):**
1. `narracao_joanne.mp3` → `narracao.mp3` (nome da voz do Piter cravado em `montar_timeline`/`producao`).
2. **Mapa de moods EN→PT no `trilha.py`**: o Gemini rotula tópicos em EN (tense/dark/neutral);
   nosso banco usa moods PT (tenso/sombrio/frio...). Sem o mapa = 0 segmentos de trilha,
   silenciosamente. `MOOD_MAP` resolve; fallback = frio.
3. Chunking da narração (item da tabela acima).

**QA visual da PoC**: footage casou com a fala (floresta/rio no hook, macaco na copa, tarântula
com wash vermelho no tópico "tense"); look vintage P&B do preset documentário aplicado; 1 mapa.
**Curadoria a refinar (Fase 2)**: 1 cena usou estátua de dinossauro de parque para "serpente
pré-histórica" — o vet de relevância deixou passar; é o tipo de coisa que o preset nosso + gate5
(v5) aperta. Legendas: caminho v4 só põe karaokê no hook se `hook_ate>0` no preset.

## 8.5 ⚠️ Episódio térmico 03/08 (máquina atual) — lições que valem pro servidor

- **Desligamento sujo** (Kernel-Power 41) com render Remotion (concurrency 14) + Docker/WSL2
  + produção juntos. O WSL2 **default reserva METADE da RAM** (8 GB aqui!) — sempre criar
  `%USERPROFILE%\.wslconfig` com `[wsl2] memory=2GB processors=2 swap=1GB` ANTES de instalar
  o Docker (o SearXNG não precisa de mais).
- **Nesta máquina, render com `RENDER_CONCURRENCY=8`** (não os 14 do benchmark do Piter) —
  custa ~5% de tempo e alivia a térmica. No servidor (desktop, refrigeração real), voltar a 14.
- **Docker pós-crash-sujo entra em loop de erro** ("unexpected error... remove <socket>:
  Não é possível o acesso"): sockets unix ÓRFÃOS ficam indeletáveis (WinError 1920, nem
  unlink resolve). Fix que funciona: **RENOMEAR as pastas-mãe** (`AppData\Local\Docker\run`
  e `AppData\Local\docker-secrets-engine` → `*_orfa`), o Docker recria limpas. As `_orfa`
  somem num reboot futuro. Cada crash pode espalhar sockets em MAIS de uma pasta — varrer
  as duas de uma vez, com TODOS os processos docker mortos antes.

## 9. Arquivos auxiliares do Piter que vieram no port e NÃO usamos

Em `remotion/`: `_render_acervo_*.mjs`, `_render_previews.mjs`, `gerar_sfx.py`,
`upload_catalog.py` — ferramentas dele, com paths `F:/Canal Dark/...` ainda cravados.
**Não estão no caminho v4** e não foram portadas de propósito. Se um dia forem úteis
(ex.: gerar catálogo do nosso acervo), portar na hora, com config.
Idem `director/producao.py:27` (mapa de vozes dele) — o nosso driver é o bridge da Fase 1.
