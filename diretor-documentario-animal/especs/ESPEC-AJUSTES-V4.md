# ESPEC DE AJUSTES v4 — refinos do operador sobre `urso_polar_v3` e `predadores_v3`

> **Status:** ✅ **EXECUTADA (06/08).** Os 10 passos implementados e postos à prova em
> render real. O que a execução EXPÔS está registrado abaixo em §8 — sete defeitos que só
> apareceram rodando, todos corrigidos e catalogados (E19-E23 na blindagem).
>
> Veredito dele: *"NO GERAL JÁ ESTÁ MUITO BOM, MUITO BOM, SEM ERROS GROTESCOS, agora
> realmente é só ajuste... AS ANIMAÇÕES DOS CAPÍTULOS FICARAM MUITO BOAS... a v3 é a que
> MAIS ESTÁ PERTO DA PERFEIÇÃO."*

---

# PARTE 0 — 🔍 AS DUAS INVESTIGAÇÕES (causa provada, não palpite)

## 0.1 · Por que a animação de PESO não entrou depois da de ALTURA (0:26)

> *"temos que entender pq a animação numérica de peso não entrou depois do da animação que
> usou pra falar que ele pode chegar a 3m depois do 0:26s"*

**Narração (20-45s):** *"A big male stands **three meters** tall when he rises and weighs
more than **600 kilograms**."*

**CAUSA: os dois números estão na MESMA FRASE, logo na MESMA CENA — e o
`montar_timeline` aceita NO MÁXIMO UM overlay por cena.** O prompt diz literalmente
*"choose AT MOST ONE overlay: A) INFOGRAPHIC ... B) TEXTO_IMPACTO"*. Altura entrou,
peso foi descartado na origem — nunca chegou a ser candidato, então nenhuma regra do
coordenador teve culpa.

**Fix:** permitir MAIS DE UM elemento numérico por cena quando a frase traz medidas de
**DIMENSÕES DIFERENTES** (altura ≠ peso ≠ distância) — exatamente a exceção do R5 que o
operador já havia concedido. Alterar o prompt do `montar_timeline` para devolver uma LISTA
de infográficos, e o coordenador distribui os dois dentro da cena (com respiro curto).

## 0.2 · Por que entrou animação POR CIMA do card de capítulo

> *"no 4º print, entrou uma animação por cima da animação do capítulo, animação de capítulo
> é soberana, não deve dividir a tela com nenhuma outra animação de texto."*

**Medido no `predadores_v3`: 3 dos 5 capítulos foram invadidos.**
```
cap 5 [35,4-38,4s] THE TIGER SHARK        <<< ilustracao 'card' em 35,4s
cap 4 [109,8-112,8s] THE GREAT WHITE SHARK <<< ilustracao 'card' em 109,8s
cap 3 [180,7-183,7s] THE KILLER WHALE      <<< acervo Ovl06_CenterPunch em 177,3s
```

**CAUSA (verificada):** o `edicao.py` protege a janela do capítulo lendo
`tl["marcacoes"]` — **mas as marcações só são criadas no `preparar_render`, que roda
DEPOIS.** Conferido: o timeline que o coordenador recebe **não tem a chave `marcacoes`**.
Ou seja, a proteção existe no código e nunca teve o que proteger. (É o achado 7.5-A4,
catalogado em 05/08 e nunca corrigido.)

**Fix:** o detector de roteiro-lista (o mesmo do B5) roda ANTES do `edicao`, grava as
janelas de capítulo no timeline, e o `preparar_render` passa a apenas MATERIALIZAR o que
já foi decidido — em vez de decidir por último. Assim a janela vira exclusiva de verdade.

---

# PARTE 1 — 🔴 REGRA DE EXCEÇÃO DOS 3 PRIMEIROS MINUTOS (aditiva)

> *"vamos inserir agora uma exceção à regra... deverá ser ALÉM, pegar o que já tem e SOMAR
> ESSE ADITIVO... essa exceção NÃO ENTRA NO LIMITE DE ANIMAÇÃO POR VÍDEO, ELAS SÃO A
> PARTE... o diretor de edição vai receber as inserções definidas e ele vai acrescentar
> esse aditivo em cima do que já chegou para ele."*

| Minuto | ADITIVO obrigatório (SOMA ao que já existe) |
|---|---|
| **1** | **+3 entradas de texto (KineticText)**, cada uma de: `Texto01_Typewriter` · `Texto03_WordPop` · `Texto06_SplitBar` · `Texto08_GradientGlow` · `Texto09_UnderlineDraw` |
| **2** | **+2 KineticText** e **+2 acervo de TEXTO** do pool B (abaixo) |
| **3** | **+2 KineticText** e **+1 acervo de TEXTO** do pool B |
| **4 em diante** | **pelo menos 2 KineticText por minuto** |

**Pool B (acervo de texto da exceção):** `Texto10_LetterCascade` · `Ovl01_ChapterBig` ·
`Ovl09_TickerCaption` · `Ovl14_PillVerdict` · `Texto06_SplitBar` · `Texto04_EditorialSerif`
· `Texto09_UnderlineDraw`

**Regras da exceção:**
- ⚠️ **FORA do teto de 2× por variante.** *"NÃO TEM PROBLEMA PQ ESSE DETALHE DOS 3
  PRIMEIROS MINUTOS ESTÃO FORA DA REGRA, SÃO EXCEÇÃO."*
- ✅ **A proibição de duas animações IGUAIS SUBSEQUENTES CONTINUA VALENDO.**
  *"lembre-se não deve ter duas animações iguais subsequentes, isso o diretor ainda deve
  respeitar."*
- É **aditivo**: o coordenador recebe o que já decidiu e ACRESCENTA por cima, para
  garantir a meta.

## 1.1 · Evitar a animação do print 2 (urso)

> *"vamos evitar usar a animação do print 2, aqui nas regras da exceção tem animações
> dinâmicas de texto MUITO MAIS BONITAS"*

Print 2 do urso = card serifado tela-cheia (`World Arriving Late.`) — provável
`Texto04_EditorialSerif` ou `Texto02_HighlightSweep`. **Rebaixar o peso dessa variante**
(não banir: ela segue no pool B para o minuto 2-3 por escolha dele).

---

# PARTE 2 — 🎞️ MAIS OVERLAYS DE VÍDEO (documentário)

> *"GOSTEI MUITO DELE TER USADO O OVERLAY DE TELA: Barulho 1 - CapCut, ficou muito legal,
> muito mesmo... podemos aumentar o uso desses overlays de vídeo... daria para usar pelo
> menos 2x cada um, deixando em lugares diferentes."*

| Regra | Valor |
|---|---|
| Pool de TAKE | `Barulho 1` · `Quadro de Filme` · `Filmadora` (os de CAPÍTULO são **exclusivos** do capítulo — nunca no take) |
| Alvo | **pelo menos 2× CADA UM** por vídeo (hoje: barulho 3, quadro 2, filmadora 0) |
| Distribuição | em lugares DIFERENTES do vídeo |
| Rodízio | nunca o MESMO overlay subsequente; **um overlay não repete antes de os outros aparecerem** |
| Overlays DIFERENTES em sequência | **permitido**, mas *"SE E SOMENTE SE servir à narrativa — não devem ser colocados soltos para encher buraco"* |

⏳ A `Filmadora` continua sem calibração de opacidade sobre footage (brilho 4 = base
escura → `screen`). Fazer o teste de 5s antes de liberá-la.

---

# PARTE 3 — 🎵 TRILHA: cobertura e cross-fade

> *"teve locais sem trilha sonora... momentos sem trilha são importantes para o suspense,
> pode ter momento sem trilha, mas achei um pouco demais nesses vídeos."*

| Ajuste | Regra |
|---|---|
| **Cobertura** | trilha em praticamente todo o vídeo; silêncio é RECURSO pontual, não buraco |
| **Fade-in** | **3 s** até o volume normalizado que o diretor já aplica |
| **Buraco entre faixas** | proibido ficar 1-2 min sem trilha. Se a faixa acabar antes, **repete em LOOP** |
| **Cross-fade** | as faixas se **SOBREPÕEM**: a que sai em fade-out e a que entra em fade-in, terminando JUNTAS — *"para que quando o fade-out acabe o fade-in da próxima também acabe, mantendo a imersão"* |
| Corte de tópico | o corte seco atual por tópico dá lugar ao cross-fade |

---

# PARTE 4 — DEFEITOS DO URSO v3

| # | Defeito | Ação |
|---|---|---|
| **U-1** | SFX do **print 1** (`A week before hunting`, canto sup. dir.) | trocar por **clique** |
| **U-2** | 🔴 **Áudio ainda com a frase duplicada em 3:54** | ver §4.1 |
| **U-3** | **6:02** deveria ter tido animação de NÚMERO e não teve | investigar (provável mesma causa da §0.1 ou grounding) |
| **U-4** | SFX da animação do **print 2** | trocar por **whoosh** |

## 4.1 · Por que o áudio duplicado NÃO foi corrigido — explicação honesta

A guarda anti-loop (P5 da v3) **nunca foi implementada** — ficou na espec como pendência.
E mesmo se estivesse pronta, **ela age na GERAÇÃO**: este render **REUSOU** o
`narracao_urso_polar.mp3`, o mesmo arquivo defeituoso. Reusar áudio preserva o defeito
do áudio, por definição.
**Duas ações necessárias:** (a) implementar a guarda no `narrator.py`; (b) **RE-GERAR** as
duas narrações — sem isso, o defeito acompanha qualquer render futuro desses roteiros.

---

# PARTE 5 — DEFEITOS DO PREDADORES v3

| # | Defeito | Gravidade / causa provável |
|---|---|---|
| **P-1** | 🔴 Falando de **tubarão** e apareceu take de **jacaré** | *"inadmissível"*. Take do POOL entra com ZERO Vision e sem checar o SUJEITO da cena atual — o pool casa por rótulo, e `saltwater crocodile` estava no pool |
| **P-2** | 🔴 Animação **por cima do card de capítulo** | ✅ causa provada na §0.2 (3 de 5 invadidos) |
| **P-3** | Falando de **baleia**, take que não faz sentido | mesma família do P-1 |
| **P-4** | Áudio com o mesmo erro de repetição | §4.1 |
| **P-5** | 🔴 Take de **QUEIMADA** em doc de oceano (de novo) | atmosfera do POOL pula o `vet_atmosfera` (era o P3 da v3, não implementado) |
| **P-6** | 🔴 **Saltwater crocodile revelado ANTES do capítulo 1** | *"não pode revelar o conteúdo do capítulo antes de citar o capítulo"*. Precisa de **janela de embargo**: o sujeito de um capítulo não aparece em cena ANTES do card dele |

---

# PARTE 6 — ✅ O QUE ELE ELOGIOU (não mexer)

- **Animações dos capítulos** — *"FICARAM MUITO BOAS"*
- **Overlay `Barulho 1`** — *"ficou muito legal, muito mesmo"*
- Impressão geral — *"MUITO BOM, SEM ERROS GROTESCOS"*, *"a v3 é a que MAIS ESTÁ PERTO DA
  PERFEIÇÃO"*

---

# PARTE 7 — ORDEM DE EXECUÇÃO PROPOSTA

```
A. Capítulo soberano  — janelas de capítulo ANTES do edicao (§0.2)  [causa provada]
B. Exceção dos 3 min  — aditivo de KineticText/acervo (§1)          [pedido nº1]
C. Trilha             — cobertura + fade-in 3s + cross-fade (§3)
D. Overlays de take   — 2× cada, rodízio, filmadora calibrada (§2)
E. Pool com vet       — P-1/P-3/P-5: sujeito da cena e atmosfera conferidos
F. Embargo de spoiler — P-6: sujeito do capítulo não aparece antes do card
G. Multi-número/cena  — §0.1 (peso + altura na mesma frase)
H. Narração           — guarda anti-loop + RE-GERAR os dois áudios (§4.1)
I. SFX                — U-1 clique · U-4 whoosh
J. Re-render + portão
```

*Aberto em 06/08/2026. Aguardando GO.*


---

# PARTE 8 — 📋 O QUE A EXECUÇÃO EXPÔS (06/08)

Os 10 passos foram implementados e o render real derrubou **sete** defeitos que nenhuma
leitura de código tinha achado. Todos corrigidos; os de produção viraram E19-E23 no
`CATALOGO-BLINDAGEM-PRODUCAO.md`.

| # | Defeito | Como apareceu | Correção |
|---|---|---|---|
| 1 | **Patch do passo D nunca gravou** | o log do v5 ainda dizia `teto 4 = 0.5/min`; `grep` confirmou o código velho. A `filmadora` seguia entrando **0×** | reescrito: alvo 2× CADA + rodízio estrito |
| 2 | **`_titulo_item` usado antes de definido** | `NameError` no `preparar` — só quebra em roteiro COM capítulos, por isso o urso passou e o predadores morreu | definição movida para antes do uso |
| 3 | **E19 · `overlay_take` não era limpo** | timeline dos predadores trazia 4 do run v3 e o passe SOMOU: `quadro_filme` 3× e rodízio `b,q,q,f,b,q,f` | `pop()` na entrada do bloco |
| 4 | **Overlays amontoados no começo** | simulação mostrou os 6 nos primeiros 104s de 456s | espaçamento mínimo `dur/(teto+1)` ≈ 70s |
| 5 | **Exceção presa ao ORÇAMENTO** | minutos 4 e 6 do urso com **ZERO** texto: o orçamento (34) fechava no 4º aditivo | o orçamento É o "limite por vídeo" que o operador isentou — exceção passa por fora |
| 6 | **E20 · loop do áudio sobrevivia ao re-render** | `narracao_urso_polar.mp3` era de 05/08 12:01, a guarda anti-loop de 06/08 09:38 — o arquivo nunca foi tocado | `_deloop_audio.py`: cirurgia no MP3 + words + timeline, preservando os takes |
| 7 | **E21 · `detectar_v2` devolveu ZERO em silêncio** | mesmo roteiro: 28 momentos numa rodada, 0 na seguinte. Família **Overlay inteira** sumiu (8→0) e o Typewriter apareceu 6× tapando buraco | retry 3× + degradação registrada; e rodízio no pool da exceção |

| 8 | **D3 · overlay de take sobre card TELA CHEIA** | conferência visual do urso v5 final: o `quadro_filme` (multiply) caiu sobre um card de texto de fundo preto — **invisível**, e um dos 2 usos do overlay foi jogado fora | cena coberta por elemento tela-cheia deixa de ser candidata a qualquer overlay de take |

⚠️ O **D3 foi encontrado DEPOIS do render** — o `urso_polar_v5_final.mp4` entregue ainda tem
esse 1 overlay invisível em 106,7s. Corrigido no código; entra no próximo render.

**Também corrigidos no caminho:** E22 (o runner sobrescrevia `RENDER_CONCURRENCY` do `.bat`,
e o render morria em `delayRender timeout` a 118s de 120) e E23 (`pais=None` do LLM
derrubava o `detectar_mapas` num f-string, **depois** de o mapa estar pronto).

## 8.1 · O que ficou em aberto (não corrigido, e por quê)

- **F2 (cortes/min ≥ 12)** continua reprovando: medido `[8, 8, 15, 3, 3, 3, 5, 2]`. Não é
  ajuste de edição — 72 cenas em 449s dão ~9,6 cortes/min de teto. Fechar exige re-cortar o
  timeline no `montar_timeline`, o que **invalida os takes já aprovados**. Decisão do
  operador para uma v6.
- **Apresentação/ilustração também não se limpam entre runs** (mesma família do E19). Dano
  baixo (o `edicao` poda o que não aceita), mas o passe não é idempotente. Anotado.
- **`abertura` sorteia palavra diferente a cada render** do mesmo roteiro ('ARCTIC' → 'ICE'
  → 'ICE'). Não é defeito, mas é não-determinismo num componente que o operador congelou.
