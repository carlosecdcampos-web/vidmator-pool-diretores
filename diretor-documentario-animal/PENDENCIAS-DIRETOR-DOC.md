# PENDÊNCIAS DO DIRETOR DE DOCUMENTÁRIO — índice mestre de execução

> **O que é este arquivo:** o **mapa único** de tudo que está aberto no Diretor de
> Documentário, em 04/08/2026. Nasceu do pedido do operador: *"pegue tudo que falei aqui, e
> o que ainda está aberto de implementação… organize numa espec detalhada, apontando para
> algum documento de fora que achar necessário, vamos começar a documentar todos os ajustes
> pendentes, para que possamos encaminhar para uma execução da espec de fato."*
>
> **Como usar:** este índice **não repete** o conteúdo das especs — ele aponta. Cada item
> tem: estado, onde está documentado, e o que falta para poder ser executado.
>
> **Regra-mãe de todo item:** aditivo e gated (I1). Nada altera o que já está validado
> (tag `diretor-doc-v1-validado`).

---

## 🗺️ MAPA DOS DOCUMENTOS

| Documento | Cobre |
|---|---|
| [ESPEC-ABERTURA-E-TAKES.md](ESPEC-ABERTURA-E-TAKES.md) | abertura com nome do país · mapa na introdução · takes do mesmo vídeo-fonte |
| [ESPEC-INATURALIST.md](ESPEC-INATURALIST.md) | nova fonte de imagem (tier 2/3) · licença · tratamento em moldura |
| [ESPEC-REGRA-DE-USO-ACERVO.md](ESPEC-REGRA-DE-USO-ACERVO.md) | quando cada uma das 40 variantes entra · detector · smoke tests |
| [ESPEC-ACERVO-40-VARIANTES.md](ESPEC-ACERVO-40-VARIANTES.md) | valores das 40 · ChapterTitle · mapas (cor, seleção, filtro) — **implementada** |
| [ESPEC-EDICAO-ADITIVOS.md](ESPEC-EDICAO-ADITIVOS.md) | apresentações · ilustrações · tema do canal · transições — **implementada** |
| [RELATORIO-IMPLEMENTACAO-ACERVO.md](RELATORIO-IMPLEMENTACAO-ACERVO.md) | invariantes I1-I15 · ordem de risco · erros a evitar |
| `director/CONTRATO-EDITORIAL.md` | as 8 leis editoriais (grupo B: não inventar dado) |

---

## 🔴 BLOCO A — PRONTOS PARA EXECUTAR (decisões fechadas, só falta GO)

### A1 · Abertura com o nome do país — ✅ **IMPLEMENTADA, APROVADA E AUDITADA**
**Doc:** [ESPEC-ABERTURA-E-TAKES §1](ESPEC-ABERTURA-E-TAKES.md) (**versão final 04/08**) ·
**Código:** `remotion/src/compositions/Abertura.tsx` · **Provas:** `out/abertura_AMAZON.mp4` ·
`abertura_CONGO.mp4` · **provas do acento** `abertura_ac_AMAZONIA/AFRICA/AMAZON_regressao.mp4`
Veredito (04/08): *"AGORA FICOU PERFEITO, REALMENTE MUITO BOM."*
✅ Fonte 23% + vão 1,4% da largura, **regra dos 3 degraus** · gradiente 38% · sombra 0.20 ·
zoom 1.06 · fade 1,5s · métricas do `.ttf` embutidas (larguras `ANTON_W` **+ topos
`ANTON_TOP`** — bug E2 do acento decepado encontrado na auditoria e **corrigido+provado**) ·
idioma do roteiro · nome curto (CONGO). Casos de nome **decididos** (D1 continente / D2
lista fechada, explícito vence).
**Falta para entrar no pipeline** (ordem na própria espec): pass de extração do nome ·
take fixo + `dur_seg` sorteada no `preparar_render` · plugagem no render final.
*(Não depende mais de B3 — o hook foi definido e só o Item 2/mapa usa a janela.)*

### A2 · Takes do mesmo vídeo-fonte = takes iguais
**Doc:** [ESPEC-ABERTURA-E-TAKES §3](ESPEC-ABERTURA-E-TAKES.md) — 🟢 **operador aprovou a
abordagem** (*"GOSTEI"*) · mecanismo completo especificado na auditoria (E3): `pexels_id`
persistido **no índice + no timeline**, `_ident()` com lookup reverso path→id; fallback
hash p/ cache legado (ganho real só em `fresh`). Exceção **decidida** (D6): fundos
word-pop/pai-neutra **fora** do orçamento.
**Falta:** medir o custo de `MAX_USOS_POR_FONTE = 1` — 3 biomas em `fresh` vs baseline
(`hard_miss`/tempo/downloads) **antes** de cravar o número (D5: o número decide).

### A3 · Herdados aprovados das transições
**Doc:** [ESPEC-REGRA-DE-USO-ACERVO §0](ESPEC-REGRA-DE-USO-ACERVO.md)
- **H1** shutter das transições **0.15 → 0.13** (ouvido no teste)
- **H2** apresentação `film` **sempre** com Apagão de **entrada** + sfx (saída não)
- **H3** `dim = 0.15` da família Overlay — ⚠️ mora no caller, que **nasce em B1** → NÃO é
  executável agora; vai junto com B1 (corrigido na auditoria 04/08)
**Falta:** só o GO. São **2 edições** executáveis já (H1, H2); H3 fica anotado para B1.

---

## 🟡 BLOCO B — PRECISAM DE DECISÃO SUA

### B1 · Regra de uso das 40 variantes (o "passo 8")
**Doc:** [ESPEC-REGRA-DE-USO-ACERVO.md](ESPEC-REGRA-DE-USO-ACERVO.md) · **Protótipo:** `director/acervo_texto.py`
✅ Vocabulário **completo das 40** · ✅ smoke de cobertura **19/19** · ✅ teste do silêncio
passou em 4 rodadas · ✅ estratégia definida (v2).
**🔒 Sequência travada (04/08):** smoke do B1 roda **POR ÚLTIMO** — *"mas vamos fazer
jaja, não esqueci disso"* — na **Fase 3** do roadmap, após aprovação das aberturas; em
seguida o **teste do B3** (hook em tamanho real). C2 (variância N=10) entra junto do smoke.
**Ainda com o operador:** revisar o gabarito no HTML (`Desktop/revisao_acervo_hooks.html`);
Fase 2 (roteiro-lista) segue adiada.

### B2 · iNaturalist como fonte tier 2/3 — ✅ **DECIDIDO (04/08), PRONTO P/ IMPLEMENTAR**
**Doc:** [ESPEC-INATURALIST.md](ESPEC-INATURALIST.md) — API verificada ao vivo · fluxo da
busca §4.1 · frequência do `quadro` §3.1 (≥7 cenas no hook, ≥14 pós-hook)
**Decisões travadas:** só **CC0 + CC-BY** (*"as duas mais safes"*) · **fotos bastam, para
começar** (sons fora) · crédito = **descrição do YouTube** (default a vetar no GO) ·
`quadro` com valores default, calibrar no teste das aberturas.
**⚠️ Subiu de prioridade: precisa estar OPERANTE antes do teste das aberturas** (Fase 1,
passo 5 do roadmap) — o teste também puxa material daqui.

### B3 · Tamanho do hook — ✅ **DEFINIDO (04/08)**
**Doc:** [ESPEC-ABERTURA-E-TAKES §1.4](ESPEC-ABERTURA-E-TAKES.md)
Regra de 3 sobre os 3 timelines validados (718 palavras / 292s = **2,46 palavras/s**):
**`HOOK_PALAVRAS = 355`** (alvo 2:25; banda 345-370 = 2:20-2:30). Hook = as primeiras 355
palavras do roteiro; instruções "de hook" valem **só** nesse trecho; a janela em segundos
sai do alinhamento da narração (a palavra manda, não o relógio). **Destrava A1 e B4.**
**Falta:** só implementar junto do GO da espec (constante no director + janela materializada).

### B4 · Mapa obrigatório na introdução — 🟢 destravado por B3
**Doc:** [ESPEC-ABERTURA-E-TAKES §2](ESPEC-ABERTURA-E-TAKES.md)
Obrigatório quando o lugar é **explicitamente mencionado** dentro da **janela do hook**
(agora definida). Obrigatoriedade passa por cima do cooldown do `detectar_mapas.py` só
dentro da janela; escolha da variante continua pelo filtro de elegibilidade (I13).

### B5 · Detector de roteiro-lista → religa o ChapterTitle
**Doc:** [ESPEC-ACERVO §6.1](ESPEC-ACERVO-40-VARIANTES.md)
O `ChapterTitle` está **pronto e desligado** (`capitulo: false`). Sua regra: capítulo **só**
em roteiro-lista, cada item = um capítulo, **numeração completa 1..N**, nunca na intro.
**Falta:** o detector que reconhece a estrutura-lista do roteiro.

---

## 🔵 BLOCO C — DÍVIDA TÉCNICA CONHECIDA (não bloqueia, mas cobra juros)

### C1 · Grounding não entende número composto
`director/acervo_texto.py` → `_numeros_falados()` só reconhece número de **palavra única**
("six", "ninety"). **"forty thousand" falado seria descartado** como não-falado — o
validador rejeitaria um dado legítimo. **Resolver antes da produção do B1.**

### C2 · Variância do detector não medida a fundo
As médias (97% de cobertura, 100% no hook) vêm de **2 rodadas**, não de 10. Observei erros
diferentes na mesma configuração. O resíduo (`local_tag` ↔ `legenda_doc`) é de baixo
impacto visual, mas existe. **Rodar N=10 antes de produção.**

### C3 · O `<Loop>` sempre reinicia o vídeo no frame 0
**Doc:** [ESPEC-ABERTURA-E-TAKES §3.4](ESPEC-ABERTURA-E-TAKES.md)
Dois usos do mesmo material mostram **exatamente a mesma imagem**. Se o trecho exibido
variasse, o reuso deixaria de ser idêntico — **atenua A2**, mas é item próprio.

### C4 · Mapas guiados pelo tema do canal
**Doc:** [ESPEC-ACERVO §7.2.2](ESPEC-ACERVO-40-VARIANTES.md) — escopo futuro que você
anunciou. As cores do mapa seguindo claro/escuro do cadastro.

### C0 · ✅ VERIFICADO — a escala dos outros HTMLs de refino está CORRETA

Depois do erro de unidade da abertura, o operador perguntou (com razão) se **todos** os
valores que ele ajustou nos HTMLs sofreriam do mesmo problema. **Verificado arquivo por
arquivo em 04/08 — não sofrem.**

| HTML de refino | Como escala | Veredito |
|---|---|---|
| `acervo_TEXTO` · `acervo_OVERLAY` · `acervo_GRAFICO` | canvas virtual 1920×1080 + `transform:scale` | ✅ px reais |
| `ilustracoes_refino` | `clientWidth/1920` | ✅ |
| `apresentacoes_refino` · `textos_refino` · `polaroid_legenda` | `clientWidth/1920` | ✅ |
| `film_moldura_refino` | `clientWidth/1500` (espaço do filme; o componente converte por `1536/1500`) | ✅ desvio de 2,4%, desprezível |
| `mapa_chapter` · `mapas_21_catalogo` | canvas virtual 1920×1080 | ✅ |
| **`abertura_refino`** | **`vw` = relativo à JANELA** | 🔴 **era o único errado — já corrigido** |

**Prova empírica** (não só leitura de código): renderizado um still de
`Ovl03_LowerThird`, cujo texto o operador mudou de 40 → **68**. Medição do PNG:
cap-height 50px → **fontSize ≈ 69px**. Bate com o aprovado.

→ **Os valores das 40 variantes, das ilustrações e das apresentações estão corretos.**

### C6 · ⏳ D5 — o número de `MAX_USOS_POR_FONTE` ainda não foi medido a fundo
**Feito:** o mecanismo (identidade por `pexels_id`) e o **knob** (`max_usos_por_fonte`,
env `MAX_USOS_FONTE`), com default **2 = comportamento de hoje**. Trocar o número é um flip.
**Evidência de OFERTA colhida em 04/08** (24 queries dos 3 timelines validados): **0
queries** com menos de 4 vídeos-fonte distintos — a mediana bateu no teto da amostra (8).
Oferta bruta **não** é o gargalo.
⚠️ **Isso NÃO é o número que o operador pediu.** Ele mediu oferta, não `hard_miss` (que
depende do gate Vision, não do estoque). O número definitivo (`hard_miss`/tempo/downloads
com `MAX=1` vs baseline) sai **das rodadas FRESH da Fase 2** — que já são fresh, então a
medição não custa uma segunda passada de cota.

### C5 · Branches não mescladas
`master` (docs) · `edicao-aditivos` (rodada 2) · **`acervo-mapas-chapter`** (atual, contém
tudo). Nada foi ao `master`. **Decidir quando mesclar** — quanto mais tempo separadas, mais
caro o merge.

---

## 📋 ORDEM DE EXECUÇÃO SUGERIDA

**🔒 ROADMAP TRAVADO PELO OPERADOR (04/08)** — *"executaremos todas as especs, depois eu
dou GO e rodaremos o render das 3 aberturas"*:

```
FASE 1 — ✅ EXECUTADA EM 04/08 (GO dado; tsc na linha de base 47, py compile ok):
  1. A3    ✅ shutter 0.13 (BrollTest+TransTest) · film SEMPRE com Apagão de entrada
  2. A1    ✅ director/abertura.py + camada ABERTURA no BrollTest (nome 6/6 nos testes)
  3. B3+B4 ✅ director/hook.py (355 palavras) + mapa obrigatório no hook
  4. C1    ✅ número composto no grounding (9/9; regressão preservada)
  5. B2    ✅ director/inaturalist.py + apresentação `quadro` (trava de espécie 11/11)
  6. A2    ✅ identidade por pexels_id + D6; knob max_usos_por_fonte (default 2)
           ⏳ D5: número definitivo sai das rodadas FRESH da Fase 2 (ver C6)

FASE 2 — GO EXPLÍCITO do operador -> TESTE:
  render das 3 aberturas (áudios prontos dos 3 biomas, menores que o hook — servem)
  · takes TOTALMENTE NOVOS (fresh, zero cache antigo)
  · iNaturalist entrando no material
  · saída com NOME DISTINTO (<bioma>_v2abertura_<data>.mp4), sem sobrescrever

FASE 3 — APROVADAS as aberturas:
  GO do operador para B1 (smoke da regra das 40 — por último, NÃO esquecido)
  -> depois o teste do B3 (hook em tamanho real, 355 palavras)

FORA DO ROADMAP (sem data): B5 (roteiro-lista -> ChapterTitle) · C2 (variância N=10,
entra junto do smoke do B1) · C3 (<Loop> startFrom) · C4 (mapas por tema) · C5 (merge).
```

**Racional:** barato e reversível primeiro (A3), impacto visual com decisão fechada (A1),
hook que destrava o mapa, dívida que destrava o B1 (C1), iNaturalist antes do teste
(o teste puxa de lá), e a medição do A2 fecha a fase. Testes só com GO explícito.

---

*Índice criado em 04/08/2026. Atualizar sempre que um item mudar de bloco — este arquivo é
a fonte da verdade sobre "o que falta".*
