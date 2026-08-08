# ESPEC — RECALIBRAÇÃO: "Diretor de Documentário Animal"

> **Decisão do operador (05/08):** *"caso o diretor de documentário não consiga conciliar
> documentário animal de documentário arqueológico/histórico, nós vamos focar esse diretor
> para documentário animal / fauna, e eu vou duplicar ele depois e fazer um diretor para
> documentário arqueológico/histórico… esse diretor que estamos fazendo vai ser o diretor
> de documentário animal, pronto."*

## 1. A decisão tem respaldo na medição

A auditoria do acervo (399 casos, 7 corpora) mediu o buraco que motivou o corte:

| Grupo de corpora | Recall |
|---|---|
| animais (amazônia, austrália, áfrica, ártico) | **93,9%** |
| lugar histórico/arqueológico (egito, china, roma) | **82,9%** |

Onze pontos. As regras de desempate do detector assumem sujeito vivo; forçar os dois
domínios num diretor só estava custando acerto **nos dois**. Separar é a decisão certa —
e agora é decisão com número atrás, não intuição.

## 2. O que muda AGORA (feito)

| Item | Onde |
|---|---|
| `nome_diretor: "Diretor de Documentario Animal"` | `presets.json` |
| Escopo declarado: fauna, comportamento, predação, biomas vivos | `descricao` do preset |
| **A CHAVE do preset segue `doc_realista`** | ver §2.1 |

### 2.1 ⚠️ Por que a chave NÃO foi renomeada

`nicho: "doc_realista"` está gravado dentro dos timelines, manifestos e do `render_lock`
dos 3 vídeos já validados. Renomear a chave quebraria replay, lock e reprodução do que já
foi aprovado — trocaria um ganho cosmético por perda de rastreabilidade. **A identidade
mora em `nome_diretor`; a chave é endereço técnico.** O diretor de arqueologia nascerá com
chave própria (`doc_arqueo`), sem herdar esse histórico.

## 3. Os 2 ajustes de edição do render (nota 8,9 do operador)

> *"dois biomas ficaram MUITO BONS… o da austrália repetiu bastante… no da áfrica, teve só
> uma transição no meio de uma cena, quando usou o polaroid."*

### 3.1 ✅ Transição SEMPRE no corte — bug PROVADO e corrigido

**Evidência (timeline entregue da África):**

```
transicao apagao em 78,60s | cena dona 73,88-82,08s (presentacao=polaroid)
  -> estourou 3,48s DENTRO da cena; o polaroid seguiu depois
```

**Causa raiz:** a marcação nascia em `topicos[i].inicio`, e **fronteira de tópico ≠
fronteira de cena**. O código antigo até sabia disso — calculava a sobra até o fim da cena
— mas só exigia 2s de espaço: garantia que **coubesse**, não que caísse no corte.

**Correção** (`preparar_render.py`): a marcação **encaixa na fronteira de cena mais
próxima** dentro de `snap_max_s` (2,5s); sem corte na janela, **não marca** (o glitch de
tópico segue). Comprovação de que o mecanismo certo já existia: a marcação `origem=film`
(A3/H2), que nasce no início da CENA, caiu com distância **0,00s** do corte no mesmo vídeo.

### 3.2 ✅ Regra de repetição endurecida — com uma ressalva honesta

Três mudanças:

1. **`max_usos_por_fonte: 1`** — um vídeo-fonte serve **uma** cena no vídeo inteiro
   (identidade por `pexels_id`, não por arquivo). Respaldo: a medição de oferta encontrou
   **zero** queries com menos de 4 fontes distintas.
2. **Tentativas de substituto 2 → 4** — a regra existia mas **se rendia**: caía em
   *"pool esgotado, mantendo repetição"*. Provado na Amazônia: **3 usos do mesmo
   `pexels_id` 32339980** em arquivos de nomes diferentes.
3. **Fundos de word-pop deduplicados entre si** — refino do próprio D6. Como o D6 saiu,
   o fundo ficava isento de **qualquer** controle; com 3 janelas de word-pop (o caso da
   Austrália) nada impedia duas caírem no mesmo arquivo, cada uma segurando a tela por
   vários segundos. Agora não contam no orçamento de takes, mas **não repetem entre si**.

#### ⚠️ O que eu NÃO consegui comprovar

Medi os 3 vídeos renderizados por hash perceptual (amostra a cada 2s) e **as métricas não
reproduzem a percepção**:

| Bioma | pares quase-idênticos distantes | maior cluster visual | dist. média |
|---|---|---|---|
| amazonia | 5 (13% dos frames) | 13% | 28,4 |
| **australia** | **2 (8%)** | 12% | 24,3 |
| africa | 0 | 15% | 26,2 |

Pela medição, a **Amazônia** teve mais reuso literal que a Austrália — e ela foi elogiada.
Duas diferenças de composição que a Austrália tinha e as outras não:

- **3 fundos de word-pop** (amazônia 1, áfrica 2) — cada um segura a tela por uma janela
  inteira, com um clipe só;
- **9 fotos** (`L4-foto`) contra 4-5 das outras — imagem se move menos que vídeo.

**Hipótese não confirmada:** o que foi percebido como "repetiu" pode ser **falta de
movimento** (mais stills + fundos longos), não reuso de material. Os 3 fixes acima atacam
reuso; se a sensação persistir no próximo render, o alvo é outro: **teto de fotos por
vídeo** e **teto de janelas de word-pop**. Registrado, não implementado — mexer nos dois
ao mesmo tempo impediria saber qual funcionou.

## 4. PRÓXIMOS PASSOS — o refino que o operador pediu

> *"o que precisamos fazer? rodar novos testes, fazer mais análises comprobatórias?
> ajustar com mais detalhe? quero refinar mais, para poder disparar um teste maior."*

### Trilha A — inteligência de texto (B1), barata e sem render

| # | Passo | Por quê |
|---|---|---|
| A1 | **Gabarito multi-label** | metade dos "erros" da última rodada é artefato do gabarito single-label (`lowerthird` + número de carreira). Sem isso, nenhum número novo é confiável |
| A2 | **Recalibrar corpora para FAUNA** | com o escopo estreitado, egito/china/roma saem do conjunto de avaliação deste diretor e viram semente do diretor de arqueologia. Sobram amazônia/austrália/áfrica (dev) + ártico (hold-out) — **falta 1 hold-out animal novo** |
| A3 | **Fix do `"7 million"`** | dígito + escala por extenso não compõe; grounding rejeita dado legítimo (bug aberto) |
| A4 | **Rodada de confirmação** | 4 corpora animais × 19 × 3, com multi-label. É o número que vale para decidir o plugue |

### Trilha B — inteligência de edição (render), cara

| # | Passo | Por quê |
|---|---|---|
| B1 | **Re-render dos 3 biomas** com os 2 fixes | única forma de saber se a transição-no-corte e a repetição endurecida resolveram. Nota atual 8,9 é a base de comparação |
| B2 | **Se a sensação de repetição persistir** | atacar movimento (teto de fotos / de janelas de word-pop), não reuso |
| B3 | **Take da abertura** | o establishing da Austrália veio de salinas com sinal de civilização — a busca usa `"{setting} aerial landscape"` e **não passa pelo gate Vision**. Corrigir amarrando à âncora regional |

### Ordem recomendada

```
1. A1 + A2 + A3   (barato, sem render, destrava a medição confiável)
2. B1             (re-render dos 3 — valida os 2 fixes de edição)
3. A4             (rodada de confirmação do texto, já em escopo fauna)
4. Passo 5 da ESPEC-B1 (gabarito cego do operador) -> aceite -> plugue
5. Duplicar para o Diretor de Documentário Arqueológico/Histórico
```

**Racional:** o barato e sem render primeiro (A1-A3), porque medir errado custa mais que
não medir. O re-render (B1) entra em seguida porque os 2 fixes já estão no código e ele é
a prova visual. A duplicação para arqueologia fica por último: clonar um diretor antes de
ele estar calibrado seria clonar os defeitos.

## 5. Duplicação futura — o que herda e o que muda

| Herda (estrutura invisível) | Muda (especificidade do nicho) |
|---|---|
| pipeline de 19 passes · orçamento de takes · marcações no corte · abertura · hook 355 palavras · grounding · silêncio | `broll_diretrizes` · `vet_regras` · vocabulário de biomas → sítios/períodos · regras de desempate do acervo (as que erraram em egito/china/roma) · lista de exceções de nome (D2) |

Vale a **regra-mãe** do projeto: diretor novo = clonar a estrutura validada e adaptar só as
especificações do nicho, nunca começar do zero.

---

*Criada 05/08/2026. Ajustes §3.1 e §3.2 já implementados; §4 aguarda GO.*
