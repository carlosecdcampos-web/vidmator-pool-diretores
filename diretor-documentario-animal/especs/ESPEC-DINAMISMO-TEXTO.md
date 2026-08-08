# ESPEC — Inteligência de legibilidade + enumeração dinâmica (Director)

**Criada:** 03/08/2026, após a PoC G0 (`poc_amazonia.mp4`) · **Status:** aguardando GO
**Pedido do operador:** duas inteligências novas, PERMANENTES no Director (não um retoque
no vídeo da PoC), estritamente aditivas — *"tudo que colocarmos deverá corrigir/ajustar
somente o que eu disse; todas as outras coisas estão PERFEITAS"*.

---

## 1. Decupagem da PoC — os dados que motivam a espec

### 1.1 Inserts de texto (`texto_impacto`) — o problema da legibilidade

7 inserts na PoC, todos de 4 palavras. Tempo REAL de tela (de `aparece_em` até o corte):

| Cena | Janela | Exibição real | Regra 2s/palavra pediria | Frase |
|---|---|---|---|---|
| 2 | 5,4–8,6s | **1,2s** ⚠️ | 8s | "Jungle hides unseen predators" ← o do screenshot |
| 7 | 20,6–23,2s | 2,6s | 8s | "Absurd serpent from nightmare" |
| 10 | 29,1–32,0s | 2,3s | 8s | "More unsettling than legend" |
| 14 | 40,7–47,3s | **1,2s** ⚠️ | 8s | "Amazon: An entire system" |
| 18 | 67,2–72,8s | 3,2s | 8s | "Living organism continental scale" |
| 20 | 79,3–85,7s | 4,5s | 8s | "Holds much hidden life" |
| 21 | 85,7–93,4s | 1,8s | 8s | "You are being watched" |

**Causa raiz (estrutural, não bug):** o insert nasce ancorado à palavra FALADA
(`aparece_em = word.start − 0,45s`, decidido no `montar_timeline`) e **morre no corte da
cena** ([BrollTest.tsx:745-750](remotion/src/compositions/BrollTest.tsx): `ovFrames =
sceneFrames − delay`). A duração de exibição é um SUBPRODUTO de onde a palavra caiu dentro
da cena. Palavra no fim da cena → 1,2s de texto. Nenhum código decide "dá para ler?".

### 1.2 A enumeração — a oportunidade do dinamismo

*"Water, canopy, shadow, mud, timber, silence and noise"* — timestamps reais do words.json:

```
47,52 Water · 48,24 canopy · 48,94 shadow · 49,68 mud · 50,24 timber · 50,92 silence · 51,50 noise
```

Cadência de ~0,6–0,7s por item — perfeita para word-pop sincronizado. Na PoC esse trecho
passou como cena comum (b-roll + narração), zero ênfase. Padrão recorrente em QUALQUER
roteiro: série de itens curtos separados por vírgula = momento de intensidade que pede
tratamento visual próprio.

---

## 2. REGRA A — Legibilidade do insert (duração ∝ palavras)

**Princípio:** exibição = `seg_por_palavra × nº de palavras` (knob, default **2,0s/palavra**),
não importa onde o corte da cena caia.

### Onde mora a inteligência — pass novo `legibilidade.py` (Director)

Pass ADITIVO (mesmo padrão de `datas`/`pessoas`: lê timeline.json, escreve campo novo).
Roda após `apresentar`, antes do `preparar_render`. Para cada cena com `texto_impacto`:

1. `alvo = seg_por_palavra × len(frase.split())` (contando só palavras reais; knobs do
   preset: `texto_seg_por_palavra: 2.0`, `texto_min_s: 3.0`).
2. Escreve **`texto_ate = aparece_em + alvo`** na cena — pode ultrapassar `fim` da cena
   (o texto ATRAVESSA cortes; é padrão de documentário e a Camada 3 é independente do vídeo).
3. **Conflitos, na ordem:**
   a. Se `texto_ate` invade o `aparece_em` do PRÓXIMO overlay (texto/infográfico):
      antecipa o próprio `aparece_em` (mostra o texto um pouco antes da palavra falada)
      até caber; âncora pode recuar no máx. `antecipa_max_s` (knob, default 2,5s).
   b. Se ainda não cabe: clampa em `próximo.aparece_em − 0,5s` e **loga** a cena
      (`legibilidade: cena N clampada — X,Xs para Y palavras`). Sem silêncio.
   c. Nunca ultrapassa `duracao` do vídeo.
4. `infografico`: knob separado `infografico_min_s` (default 3,0 — o da PoC teve 3,1s e
   leu bem). Sem regra por palavra (número + label ≠ frase). Decisão aberta ao operador.

### Mudança no renderer — 3 linhas, gated

Camada 3 do `BrollTest.tsx`: se `c.texto_ate` existir, `dur` e `ovFrames` derivam dele em
vez do fim da cena. **Campo ausente → código atual byte a byte** (os nichos do Piter nunca
geram o campo; os vídeos dele não mudam em NADA).

### Por que isso não quebra o que funciona

- `montar_timeline` (o pass que acerta) **não é tocado** — âncora na fala continua a dele.
- O pass novo só ADICIONA um campo; quem não roda o pass, nada muda.
- Knob no PRESET: `legibilidade: true` só nos presets NOSSOS (`natureza`, futuro
  `espiritual`). `documentario`/`ttm`/`estoicismo` do Piter ficam sem o pass agir.

---

## 3. REGRA B — Enumerações viram word-pop sincronizado

**Princípio:** detectar séries de itens curtos separados por vírgula no roteiro; na janela
da fala, cada item vira um insert central, fonte grande, pop + SFX de clique, na batida
exata do áudio.

### Pass novo `enumeracoes.py` (Director)

1. **Detecção determinística** (v1 sem LLM — regex primeiro, previsível e testável):
   série de **≥ 4 itens**, cada item com **1–2 palavras**, separados por `,` (aceitando
   `and/e` no último). Knobs: `enum_min_itens: 4`, `enum_max_palavras_item: 2`.
   (v2 opcional: Gemini valida "é enumeração dramática?" — só se a regex gerar falso
   positivo em produção; decisão adiada de propósito.)
2. **Casamento com words.json**: item → timestamp da 1ª palavra dele (match tolerante a
   pontuação/caixa, janela sequencial — mesmo método do `produto_cta`, que já faz isso
   com frases). Item sem match confiável → enumeração inteira descartada + log (meio
   word-pop é pior que nenhum).
3. Escreve no timeline:
   ```json
   "enumeracoes": [{"ini": 47.5, "fim": 52.4,
     "itens": [{"texto": "Water", "t": 47.52}, {"texto": "canopy", "t": 48.24}, ...]}]
   ```
4. **Supressão de conflito**: dentro da janela, zera `texto_impacto`/`infografico` de
   cenas sobrepostas (word-pop é o dono da tela; dois overlays juntos = poluição).

### `preparar_render.py` — 1 bloco novo

Copia `enumeracoes` pro render json + resolve o SFX de clique do NOSSO acervo
(`sfx.enum_click` no `vidmator.config.json`; vazio → word-pop SEM som, degrada limpo).

### Componente novo `EnumPop` (Remotion) — camada própria

- Para cada item: `Sequence` de `t` até `t + enum_item_dur` (knob, default 0,9s; o último
  item persiste `enum_ultimo_dur`, default 1,6s — fechamento).
- Palavra centralizada, fonte do `theme` GRANDE (~2× o KineticText), animação
  scale-pop (spring 1,15→1,0) + fade-out; `<Audio>` do clique na entrada de cada item.
- **Camada nova entre a 3 e o CTA** — não toca nas camadas existentes.
- Gated: `timeline.enumeracoes` ausente/vazio → componente nem monta.

### SFX do clique — decisão do operador (ouvido)

Banco tem 13 SFX da família `ui` + 10 typewriter. Rodada 3 do acervo marcou "click UI
limpo" como buraco — candidatos atuais em `acervo/sfx/ui/`; se nenhum agradar, gerar/
garimpar 1 clique seco. **Escolha é do Carlos ouvindo** (gate desta espec).

---

## 4. Garantias de não-regressão (o "SEM QUEBRAR")

| Proteção | Como |
|---|---|
| Passes que acertam ficam intactos | `montar_timeline`, `resolver`, `efeitos`, `apresentar`... não são editados; as 2 inteligências são ARQUIVOS NOVOS no fim da fila |
| Nichos do Piter imunes | knobs `legibilidade`/`enumeracoes` só nos presets NOSSOS; sem knob → passes viram no-op |
| Renderer gated | `texto_ate`/`enumeracoes` ausentes → BrollTest byte a byte igual ao atual |
| Teste de regressão objetivo | re-rodar a PoC com preset `documentario` (sem knobs) e diffar o `timeline_render.json` → deve ser IDÊNTICO |
| Teste de efeito | re-rodar com preset `natureza` + knobs → tabela §1.1 tem que virar 8s/insert e a enumeração §1.2 virar 7 pops |

## 5. Ordem de execução proposta (quando vier o GO)

1. `legibilidade.py` + campo `texto_ate` no BrollTest → re-render PoC → conferir os 7 inserts.
2. `enumeracoes.py` + `EnumPop` + SFX → re-render → conferir o trecho 47–52s.
3. Regressão: preset documentario sem knobs → diff idêntico.
4. Registrar os 2 passes na lista `PASSES` do futuro bridge (Fase 1) — e NOTA de re-sync:
   quando o Piter atualizar o Director, nossos passes continuam por serem arquivos novos.

## 6. Registro para o futuro (pedido do operador, NÃO é desta espec)

**Catalogação de canais × estilo visual por nicho:** quando implementarmos o cadastro de
canais (o Painel/Cadastro estilo Piter, com campo motor simples|vidmator), haverá campo
**nicho** — e cada nicho terá seu "BrollTest look" (ex.: natureza = cor viva; true crime =
escuro/dessaturado; história = vintage P&B como o `grayscale(0.9) sepia(0.18)` atual do
[BrollTest.tsx:600](remotion/src/compositions/BrollTest.tsx)). Alinhar look ↔ nicho ↔ canal
nesse momento. (Hoje o look sai do preset via `periodo`/`vintage` — o preset `natureza`
criado em 03/08 já é o primeiro exemplo.)
