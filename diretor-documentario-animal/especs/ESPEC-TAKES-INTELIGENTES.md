# ESPEC — Takes inteligentes: micro-takes por entidade + b-roll ciente de nicho

**Criada:** 03/08/2026 · **Status:** aguardando OK EXPLÍCITO do operador para implementar

## 0. ARQUITETURA DE DIRETORES (decisão estrutural do operador, 03/08)

O que esta espec constrói é o **DIRETOR DE DOCUMENTÁRIO** — o primeiro de VÁRIOS diretores:

- **Um diretor por nicho**: Documentário (este — serve animais E lugares: fatos reais,
  históricos) · Estoicismo/Psicologia (o do Piter, **já bom, usar como está**) · futuros.
- **Cadastro de canais** (futuro) aponta o diretor do canal, como aponta pipeline/template.
- **Core estrutural vs. otimização de viés**: funções estruturais (word-pop, legibilidade
  2s/palavra, abertura sempre vídeo, trilha mood-matched...) TODO diretor tem; o que muda
  por diretor é o VIÉS (diretrizes de b-roll, regras de vet, passes ligados/desligados,
  micro-takes, estética).
- **Implementação = a camada de preset que já existe**: um "diretor" é um preset completo
  (knobs + diretrizes de prompt + regras de vet + gates de passes). O preset `natureza`
  desta PoC é o embrião do Diretor de Documentário — **na execução desta espec ele é
  RENOMEADO para `doc_realista`** ("natureza" era estreito; o diretor cobre lugares e
  história factual também). O preset `documentario` do Piter (vintage/WWII) fica intocado
  e é outro diretor (histórico/época).
- Nada disto muda o desenho técnico das seções abaixo — muda o NOME e o destino: cada
  seção define ou o core (compartilhado) ou o viés do Diretor de Documentário.
**Origem:** ESTUDO-TAKES.md (3 culpados rastreados até a linha) + decisão do operador:
word-pop é ótimo mas não pode ser o único formato; entidades filmáveis pedem FOOTAGE
(sem word-pop), com flash + SFX entre takes; **o Director decide sozinho qual usar**.

---

## 1. A RÉGUA DE DECISÃO (o coração — como o Director escolhe sozinho)

Quando o detector acha uma enumeração na fala (série de itens separados por vírgula):

```
enumeração detectada
  └─ classificador (1 chamada Gemini, JSON):
       cada item → { filmavel: true/false, query?: "..." }
       ├─ TODOS os itens filmáveis ──────────► MICRO-TAKES (footage, flash+SFX, SEM word-pop)
       ├─ qualquer item abstrato/sensorial ──► WORD-POP (comportamento atual, aprovado)
       └─ Gemini falhou/ambíguo ─────────────► WORD-POP (fallback seguro)
```

**Filmável** = entidade com take óbvio em stock: animal, lugar, paisagem, fenômeno
natural, objeto físico concreto ("jaguars", "Sahara desert", "lightning storm").
**Não-filmável** = conceito, sensação, elemento abstrato ("silence", "shadow", "fear").

**A cadência NÃO decide o formato — ela agrupa**: item com gap < 0,9s do vizinho **funde
no mesmo take** com query combinada. Validado na PoC com timestamps reais:

| Fala | Cadência real | Régua aplica |
|---|---|---|
| "Water, canopy, shadow, mud..." | 0,70–0,74s/item | shadow/silence = abstratos → **WORD-POP** (o aprovado, intacto) |
| "jaguars, harpy eagles, and pink river dolphins, for birds and butterflies" | 0,96 / 1,82 / 1,18 / 0,70s | todos filmáveis → **MICRO-TAKES**: jaguar → harpy eagle → pink dolphin → birds+butterflies (últimos 2 fundem: gap 0,7s) |
| "from the Sahara, to the Amazon, to the Atlantic" (hipotético) | — | lugares = filmáveis → micro-takes de PAISAGEM — a régua é linguística, serve a qualquer tema do nicho |

## 2. Arquitetura (aditiva; ordem da fila muda)

**Ordem nova:** `montar_timeline` → **`enumeracoes`** (detecta + classifica + escreve
`modo` e queries) → **`resolver_cascata`** (resolve os micro-takes) → …demais… → `legibilidade`.
(Hoje enumeracoes roda no fim; precisa vir antes do resolver para os takes serem resolvidos.)

- **`enumeracoes.py` (estende o existente):** detector ampliado (itens de até **3 palavras**
  — "pink river dolphins" —, mínimo 3 itens) + classificador Gemini + escreve no timeline:
  `enumeracoes: [{modo: "pop"|"takes", itens: [{texto, t, query?, take_grupo}], ...}]`.
  Modo "pop" = fluxo atual intocado.
- **`resolver_cascata`:** para janelas `modo:"takes"`, resolve 1 clip POR grupo de take
  (query do classificador, `prefer_video=True`, MESMO vet content-aware do §4). Grava
  `clip_path` em cada grupo.
- **BrollTest — camada 2.5 (nova):** micro-takes são Sequences POR CIMA da cena pai, nos
  timestamps da fala (a cena pai continua embaixo — zero mudança estrutural nas camadas
  existentes; mesmo desenho do word-pop/texto). Componente **FlashCut** na fronteira de
  cada take: 3–4 frames de branco com fade + `<Audio>` do SFX.
- **SFX do flash:** `sfx.take_flash` no `vidmator.config.json` (degrada sem som).
  Candidatos do banco pro operador escolher DE OUVIDO na execução (como foi o clique):
  whoosh curto · camera shutter · riser-curto.
- **Supressões na janela de takes:** sem word-pop, sem texto_impacto, sem presentacao
  especial, sem ilustração — o footage é o dono da tela.

## 3. D1 — Perfil de b-roll por nicho (queries)

O prompt do `montar_timeline` ganha um bloco injetado do preset (`broll_diretrizes`).
No `natureza`:

> "This is a REALISTIC NATURE documentary. The spoken entities (species, places, natural
> phenomena) ARE the ideal b-roll — name them in the query (e.g. 'jaguar stalking prey
> night'). Real wildlife/landscape footage only. NO humans unless the line explicitly
> mentions people/tribes. NEVER objects, documents, books, studio shots."

A instrução atual *"Avoid named people/places — describe a GENERIC scene"* passa a valer
só quando o preset NÃO define diretrizes (nichos do Piter intocados).

## 4. D3 — Vet content-aware por nicho (vídeo E foto, unificado)

`vet_regras` no preset injeta vetos no gate Vision (o `vision_imagem_broll_ok` da rodada 3
e o vet de vídeo `#5b` passam a usar o MESMO prompt-base + regras do nicho). No `natureza`:

- humano visível e a fala não menciona gente/tribo → **reprova**
- objeto/livro/documento/interior/estúdio → **reprova**
- espécie/lugar citado na fala → o take tem que mostrar **aquela** espécie/lugar
- régua dura: só **"good"** passa no vet de vídeo (mata o "weak passa" leniente) — knob
  `vet_video_rigor: "good"` (default do Piter continua "none", intocado)

## 5. D4 — Pass `imagens` gated por preset

`usar_imagens_pd: false` no `natureza` (o pass do "recorte de jornal" é de história/crime —
foi ele que pôs a capa de livro em cima do jaguar). Sem o knob → roda como sempre.

## 6. D5 — Rodada de RECONSTRUÇÃO TOTAL (regra do operador)

Para a validação desta espec, o Director re-decide TUDO do zero — proibido reaproveitar:
1. Zerar `index_cascata.json` (backup) + **renomear `_cache_stock`** (mídia antiga fora
   do alcance — se ele baixar igual por mérito, ok; reaproveitar por cache, não).
2. Re-rodar a fila completa (montar re-decide queries com D1; enumeracoes classifica;
   resolver com vet D3; sem pass imagens).
3. Render completo novo.

## 7. Gates de validação (novo trecho)

- [ ] "jaguars → harpy eagles → dolphins → birds+butterflies" = **sequência de takes de
      ANIMAIS REAIS** com flash+SFX entre eles, sem word-pop
- [ ] "Water, canopy, shadow..." = **word-pop mantido** (formato aprovado, intocado)
- [ ] **Zero humanos** em cena (a fala não menciona gente)
- [ ] **Zero livro/objeto/documento** (pass imagens desligado)
- [ ] Reconstrução comprovada: index zerado, cache renomeado, queries novas no timeline
- [ ] Regressão: fila com `NICHO=documentario` → timeline idêntico (nichos Piter imunes)

## 8. O que NÃO muda (aprovado e blindado)

Word-pop caixa alta + SFX-0127 a 0.15 · legibilidade 2s/palavra · abertura sempre vídeo ·
cor do natureza · trilha mood-matched · SearXNG na cascata · tudo gated por preset.
