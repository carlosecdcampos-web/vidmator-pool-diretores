# ⚠️ RASCUNHO ABSORVIDO — ver ESPEC-EDICAO-DINAMICA-FINAL.md

> Este documento foi consolidado na **ESPEC FINAL** em 05/08/2026, que soma a
> perícia print-a-print dos dois vídeos E2E e o cross-check com a decupagem do
> vídeo de referência. Mantido como histórico; NÃO implementar por aqui.

# ESPEC — DINAMISMO DA EDIÇÃO + CORREÇÃO DOS DEFEITOS DO E2E

> **Origem:** o operador assistiu 50s do render `urso_polar_e2e` e reprovou. Este documento
> separa **o que foi contaminação minha** do **que é defeito real**, registra as **4 regras
> novas** que ele estabeleceu, e traz a **decupagem medida** do vídeo de referência do Piter
> (`amazonia_top5_v5.mp4`) para fixar o que significa "mais dinâmico".

---

## 0. ⚠️ ANTES DE TUDO: o render julgado estava CONTAMINADO por erro meu

Dois erros meus, confirmados por medição, invalidam boa parte do que foi visto:

| # | Erro | Evidência | Consequência no vídeo |
|---|---|---|---|
| **A** | Áudio errado | hash do áudio no render = `97d1692f0019` = narração dos **PREDADORES** (456,4s). A do urso (`297208468cc2`) nunca entrou | vídeo do urso com voz do outro roteiro |
| **B** | Preset errado | minhas rodadas manuais de `preparar_render` **sem `NICHO` no ambiente** caíram no preset `default` | ver abaixo |

**O que o preset `default` causou:**

| Knob | `default` (usado) | `doc_realista` (correto) | Efeito |
|---|---|---|---|
| `glitch_topico` | **ligado** | **desligado** | **SFX solto sem nada acontecer** em toda fronteira de tópico |
| `sfx_vol_max` | ausente | 0.20 | **volume alto de novo** |
| `sfx_vols` | ausente | calibrados | 0.13/0.15 ignorados |

Causa raiz do erro A: restaurei o `timeline.json` do backup entre as tentativas, mas **não a
narração** — e o timeline aponta `narracao.mp3` por caminho, que o job 2 já havia
sobrescrito.

### 0.1 🔴 DECISÃO DO OPERADOR: o `default` do sistema está obsoleto

> *"os default do sistema estão obsoletos, já te falei, então o default real deveria ser os
> do doc_realista, pq isso será válido para TODOS OS diretores."*

O preset `default` ainda carrega valores da era do Piter. Enquanto ele for o fallback, todo
esquecimento de `NICHO` reintroduz glitch, volume alto e ausência de marcações. **Ação:**
promover os valores do `doc_realista` a `default` (mantendo os presets antigos do Piter
explícitos nos nichos deles). Isso transforma o erro B numa impossibilidade.

---

## 1. 🎬 DECUPAGEM MEDIDA — `amazonia_top5_v5.mp4` (Piter, 9:55)

### 1.1 O achado principal: o dinamismo é RITMO DE CORTE, não quantidade de elemento

| | **Piter** | **Nosso (urso)** |
|---|---|---|
| Cortes | **150** em 595s | **41** em 454s |
| 1 corte a cada | **4,0s** | **11,1s** |
| Mediana entre cortes | 4,0s | 3,2s |
| Cortes por minuto | 16·17·12·14·15·15·16·16·17·12 | 4·10·4·7·**1**·9·**2**·4 |

**Ele nunca desacelera.** 12 a 17 cortes por minuto do primeiro ao último. O nosso tem
minutos com **1 e 2 cortes** — parado. A mediana do nosso é até menor (3,2s), o que revela
o padrão: temos rajadas curtas seguidas de longos trechos mortos, enquanto ele mantém
cadência constante.

**Este é o fixador nº 1 e o mais barato de implementar: não existe cena longa.**

### 1.2 Densidade de elementos: FRONT-LOADED, não uniforme

Folhas de contato (1 frame a cada 4s):

| Trecho | Elementos na tela |
|---|---|
| **0-2 min** (hook) | ~**2/3 dos frames** têm algo: donut com legenda, ticker no rodapé, punchline tela cheia, **CHAPTER 01**, **CHAPTER 02**, lower-third com kicker, **split de 2 imagens**, corner tag, foto emoldurada |
| **5-7 min** (miolo) | ~**2 elementos em 30 frames**: um card de texto em fundo CLARO e o **CHAPTER 05** |

Ou seja: ele **carrega o hook** de elementos e depois **sustenta pelo corte**, reservando os
elementos para as fronteiras de capítulo. Não é "elemento o tempo todo" — é elemento onde
decide, corte em toda parte.

### 1.3 Vocabulário observado (o que ele usa, que nós temos e não usamos)

- **Card de CAPÍTULO** tela cheia: rótulo pequeno "CHAPTER" + número grande em âmbar +
  título em uma linha. Aparece em 01, 02 e 05 → **numeração completa**, exatamente a regra
  que o operador já tinha dado.
- **Split de duas imagens** lado a lado com divisória — e usado em **frames consecutivos**
  (dura 4-8s).
- **Foto emoldurada** sobre fundo escuro (nosso `quadro`).
- **Card de texto em fundo CLARO** (creme) no meio de um vídeo escuro — quebra de padrão
  brutal, e nós só temos fundo escuro.
- **Ticker/caption de rodapé** com marcador âmbar.
- **Bloco kicker + parágrafo** (nosso `Ovl03_LowerThird` / `SideNote`).
- **Donut com legenda lateral** (nosso `Graf03_DonutPercent`).

**Todos esses existem no nosso acervo.** O problema nunca foi repertório — foi frequência e
coordenação.

### 1.4 Os elementos PERSISTEM 4-8s

Vários aparecem em dois frames consecutivos da folha (amostra de 4s), ou seja duram no
mínimo 4s e frequentemente 8s. Nosso default é 2,8-3,2s — **curto demais para ler**.

---

## 2. AS 4 REGRAS NOVAS (operador, 05/08)

| # | Regra | Estado |
|---|---|---|
| **R1** | `film` → **sempre** transição **Brilho 2** na entrada + o SFX fixo dele | a implementar |
| **R2** | `quadro` → **sempre** transição **Apagão de Filme** na entrada + o SFX fixo dele | a implementar |
| **R3** | **Nenhuma apresentação subsequente a outra.** Jamais duas seguidas — nem iguais, nem diferentes | a implementar |
| **R4** | SFX preso ao **evento de edição**, nunca ao tempo. Não existe SFX sem elemento visível | a implementar |

> R4 explica o pior sintoma: *"o SFX apareceu no fundo quando não acontecia nada na cena"*.
> Parte veio do `glitch_topico` (erro B), mas a regra vale como invariante permanente:
> **som sem imagem correspondente é defeito, sempre.**

---

## 3. OS 7 DEFEITOS REAIS (independem dos meus erros)

| # | Defeito | Causa provável | Correção proposta |
|---|---|---|---|
| **D1** | **Word-pop virou trecho-pop** — orações inteiras viram pop | detector de `enumeracoes` aceita item longo | regra dura: item de **1 palavra** (máx 2), em série separada por vírgula com 3+ itens. Fora disso, não é word-pop |
| **D2** | **Apresentações rarefeitas** — 1ª `film` em 1:57, 1º polaroid em 5:48 | `apresentacao_freq` + cota de 20% do coordenador | cota vira **piso e teto**; garantir apresentação nos primeiros 30s |
| **D3** | **Duas apresentações seguidas** (`quadro`→`film`) | **meu** código de herança de hard-miss pôs moldura em cenas adjacentes | R3 como guarda no coordenador, aplicada DEPOIS da herança |
| **D4** | **Overlay com texto estourando a caixa** ("Open Water (June" cortado) | componente sem clamp de largura/fonte | encolher fonte até caber, como na abertura |
| **D5** | **Fundo preto/escuro** em 3:10, 4:52, 7:28 | cena cujo material falhou ou apresentação sem mídia | investigar no timeline; nenhum frame pode ficar sem imagem |
| **D6** | **Mapa sem nada acontecendo** e **ilustração ruim** | falta de dado no mapa; ilustração escolhida sem conteúdo | filtro de elegibilidade (I13) também para mapa/ilustração |
| **D7** | **Acervo entregou 6 elementos** em 1.187 palavras (orçamento previa ~34) | o detector é conservador; o teto proporcional não foi respeitado por ele | o `teto` vira **meta**, não limite: se vier menos que o piso, roda 2ª passada |
| **D8** | **Só 2 dos 5 capítulos entraram** (roteiro-lista dos predadores) | **erro de precedência meu**: o bloco B5 roda DEPOIS do bloco de tópicos, então os 3 capítulos que caíram em fronteira com transição cederam a ela. A precedência documentada é **capítulo > transição > glitch** — quem devia sair é a transição | capítulo **EXPULSA** a transição da mesma emenda; e apertar o encaixe de 3,0s para ~1,5s (o cap. 3 entrou **2,72s adiantado** em relação à fala) |

**Medição do D8** (predadores, 456s):

| item | falado em | corte + próximo | desvio | veredito |
|---|---|---|---|---|
| 5 | 35,22s | 35,40s | 0,18s | ✅ entrou |
| 4 | 108,90s | 108,61s | 0,29s | ❌ emenda já tinha transição |
| 3 | 179,76s | 177,04s | **2,72s** | ⚠️ entrou adiantado |
| 2 | 259,34s | 261,34s | 2,00s | ❌ emenda já tinha transição |
| 1 | 336,60s | 336,38s | 0,22s | ❌ emenda já tinha transição |

---

## 3.1 🔴 D9 — CONCORRÊNCIA DO RENDER vs. NÚMERO DE APRESENTAÇÕES

**Sintoma:** o render dos predadores morreu **duas vezes no frame 2399** com
`write EOF` + erro 500 do compositor, apontando `scene_20.mp4` — um arquivo
**comprovadamente íntegro** (h264, 30fps CFR, yuv420p, 315 frames, decodifica e busca sem
erro; a conferência do §0 aprovou corretamente).

**Causa raiz:** não é material ruim — é o **processo compositor morrendo por exaustão de
recurso**. É consequência direta do D3 de hoje: `quadro` e `film` passaram a aceitar
**vídeo dentro da moldura**, e este roteiro tem **15 apresentações** contra 5 do urso. Cada
moldura com vídeo soma um decodificador concorrente, e com `concurrency=14` o compositor
estourou. O urso, com 5 apresentações, passou na mesma concorrência.

**Confirmado:** com `RENDER_CONCURRENCY=6` o render passou dos 50% sem um único erro.

### Calibração progressiva (decisão do operador 05/08)

> *"no próximo vamos aumentar de 6 para 8, vamos testando o limite sem estourar, para não
> sacrificar muito o tempo do render também."*

O custo de baixar a concorrência é real: o urso rendeu em **12,6 min com 14 workers**; os
predadores com 6 devem levar ~25-30 min. Então o alvo não é "o mais baixo que funciona" e
sim **o mais alto que não estoura**.

| Rodada | `RENDER_CONCURRENCY` | Apresentações | Resultado |
|---|---|---|---|
| urso | 14 | 5 | ✅ passou · 12,6 min |
| predadores | 14 | 15 | 🔴 morreu no frame 2399 (2×) |
| predadores | **6** | 15 | ✅ em curso, 0 erro |
| **próxima** | **8** | — | **a testar** |

**Regra de trabalho:** subir de 2 em 2 a cada render bem-sucedido, registrando aqui o par
(concorrência, nº de apresentações). Quando estourar, o teto anterior vira o valor seguro
**para aquela faixa de apresentações** — a suspeita é que o limite dependa do número de
molduras com vídeo, não do vídeo em si.

**Automatizar depois (não agora):** derivar a concorrência do próprio timeline, algo como
`concurrency = clamp(14 - k × apresentações_com_vídeo)`, com `k` calibrado por estes
testes. Só faz sentido com 3-4 pontos medidos.

## 4. FIXADORES DE DINAMISMO (o que vira número no preset)

| Fixador | Valor proposto | Base |
|---|---|---|
| **F1 · duração máxima de cena** | **6s** (corta em 2 se passar) | Piter: 1 corte/4,0s; nosso: 11,1s |
| **F2 · cortes por minuto (piso)** | **≥ 10** | ele nunca desce de 12 |
| **F3 · elementos no hook** | **≥ 1 a cada 12s** nos primeiros 120s | 2/3 dos frames do hook dele têm elemento |
| **F4 · elementos no miolo** | 1 a cada 30-45s + **todos os capítulos** | ele rarefaz mas ancora nas fronteiras |
| **F5 · duração do elemento** | **4-6s** (era 2,8-3,2s) | os dele persistem 4-8s |
| **F6 · nenhuma apresentação adjacente** | R3 | regra do operador |
| **F7 · fundo claro** | liberar 1 variante de card claro por vídeo | ele usa como quebra de padrão |

---

## 5. O QUE **NÃO** MUDA

> *"NÃO IREMOS ALTERAR NADA NOSSA ABERTURA, A ABERTURA DEVE SE MANTER DO JEITO QUE ESTÁ."*

A abertura (nome do lugar em Anton, gradiente, take fixo, corte seco) está **congelada**.
E o vídeo do Piter é **referência de ritmo, não modelo a copiar** — decisão explícita do
operador: *"não vamos tornar a dele modelo FIXO E IMUTÁVEL"*.

---

## 6. ORDEM DE EXECUÇÃO PROPOSTA

```
1. Promover doc_realista a default          (§0.1 — impede a classe inteira do erro B)
2. R1..R4                                    (regras novas, baratas)
3. D3, D4, D5                                (defeitos visíveis, correção direta)
4. F1/F2 — ritmo de corte                    (o de MAIOR impacto percebido)
5. D1 — word-pop de palavra única
6. F3/F4/F5 + D2/D7 — densidade de elementos
7. D6 — elegibilidade de mapa/ilustração
8. Re-render do urso (áudio e preset certos) -> nova avaliação
```

*Criada em 05/08/2026 a partir da reprovação do E2E e da decupagem medida do vídeo de
referência. Nada aqui foi implementado ainda.*
