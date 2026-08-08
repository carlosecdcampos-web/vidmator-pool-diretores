# Vidmator — Curadoria dos repositórios do Piter (18/07/2026)

> ⚠️ **ATUALIZADO 03/08/2026** — as cópias curadas abaixo foram movidas para
> **`_congelado-2026-07-18\`** e estão **OBSOLETAS como fonte**: descobrimos que o Piter
> versiona os DOIS sistemas em repos públicos no GitHub, clonados vivos aqui na raiz:
> - `piter-repo\` ← https://github.com/violipte/video-automator
> - `director-repo\` ← https://github.com/violipte/vidmator-director (20 commits além da cópia; era v5)
>
> Atualizar = `git pull` nos dois. As congeladas ficam como registro histórico (têm artefatos
> de run — `timeline.*.json`, roteiros de exemplo — que os repos gitignoram).

Origem: `C:\Users\lenovo\Downloads\Vidmator - 1.0` (3 pastas baixadas dos repos do Piter).
Os originais foram MANTIDOS no Downloads como backup — aqui é CÓPIA, verificada por hash MD5
arquivo a arquivo (372/372 idênticos à origem, zero perda).

## O que está aqui

| Pasta | Origem | Conteúdo |
|---|---|---|
| `_congelado-2026-07-18\vidmator-director/` | `1 - vidmator-director-master` | O "Director": inteligência de edição. `banco-videos/teste/` = passes Python (timeline, resolver de cenas, mapas, pessoas, trilha, efeitos) + `remotion/` = composições React + 8 renderers (broll, map, sat, gallery, doodles, catalog, comp, broll-slice). 233 arquivos. |
| `_congelado-2026-07-18\video-automator-piter/` | `1,1 - video-automator-master` | Fork do video-automator do Piter: arquitetura VPS + workers locais, frontend React (`frontend2/`), Docker pods, bridge `vidmator_render.py`, docs (`VIDMATOR_INTEGRATION.md` = doc central da integração). 139 arquivos. |

## O que foi deduplicado (e a prova)

- `1.3 - vidmator-director-master` era **cópia byte-a-byte idêntica** de `1 - vidmator-director-master`:
  233 arquivos, mesmos paths, mesmos hashes MD5, zero diferença. Por isso só uma cópia está aqui.
  Nenhum conteúdo foi perdido.
- Os dois repos NÃO se sobrepõem (único path em comum: `.gitignore`, conteúdos diferentes) —
  são complementares, mantidos íntegros lado a lado.
- Duplicatas INTERNAS observadas e mantidas de propósito (são dados funcionais, não lixo):
  - director: `mapas/mapa_1.jpg = mapa_9.jpg`, `mapa_7 = mapa_8`, `mapa_13 = mapa_14`
  - fork: `rules/de|en|es|pt.json` idênticos entre si (defaults)

## Notas de leitura

- Comece por `video-automator-piter/VIDMATOR_INTEGRATION.md` — descreve o sistema em produção
  (motor por template simples|vidmator, bridge com lock, 14 passes do Director, cascata do
  resolver de cenas, lições de bugs reais).
- Os repos referenciam paths da máquina do Piter (ex.: `F:/Canal Dark/Aplicativo de Edição/...`) —
  não rodam aqui sem adaptação; isto é material-fonte/referência para o container Vidmator do
  AC-Automator.
