# ESPEC — ACERVO EM BOXES + DISCIPLINA DE TEMPO DOS ELEMENTOS

Investigação 12/08 (vídeos lobo + orca) · co-autorada com o Codex (3 rodadas).
Estado: **AGUARDANDO GO DO OPERADOR** — nada implementado.

## O que o dado provou (orca, 19 elementos)

1. **3 elementos <1s** (0,09s · 0,25s · 0,70s); dois TERMINAM antes da própria âncora
   de fala COMEÇAR. Causa: `reconciliar_cenas.py` re-carimba `dur = janela da cena
   dona` após mover fronteiras — cena espremida ⇒ elemento herda ~zero. NÃO existe
   piso de duração de elemento nem regra de desistência. (Hipótese "cortado pelo
   seguinte" REFUTADA: próximo elemento a 8–20s nos 3 casos.)
2. **Texto05_BoxedKicker 10,2s na tela**: `_dur_de: fala` com âncora da FRASE INTEIRA
   (25 palavras); a tela exibia só 8. Duração segue o falado, não o exibido.
3. **Vício de repertório**: 25 de ~30 chaves da TABELA têm UMA variante. Rodízio só
   dentro do vídeo. `Texto04`/`Ovl06` banidas; `punchline overlay` vazio; variantes de
   intenção rara (preco, local_tag) nunca saem.
4. **Duas verdades**: manifesto `quando:` no TSX (ninguém lê) × TABELA no python (o
   roteador real). Elemento novo hoje = tocar 4-6 lugares.

## Decisões do operador (não re-litigar)

- Piso com DESISTÊNCIA: não cabe o mínimo ⇒ o elemento NÃO ENTRA. Nunca encolhe.
- Teto por leitura: `dur = min(janela_falada, leitura do exibido)`.
- **Teto de 4 palavras: SÓ no Texto05_BoxedKicker** (sugestão dele; campo por
  elemento no JSON, sem default global — Codex vetou 12 preventivo: "regra
  editorial não aprovada").
- Boxes: manifesto ÚNICO por família, JSON fora do código, lido pelos dois lados.
  Elemento novo = 1 componente + 1 entrada JSON. Sem duplicação (L10).
- Rotação entre vídeos: melhor para a parte, sem viciar.

## Desenho aprovado pelo Codex

### 1 · Portão temporal final (depois do reconciliar)
Remove elemento que viole: `dur >= piso` · `interseção(elemento, âncora) > 0` ·
início <= t_ini da âncora < fim. Piso por carga cognitiva (ELABORADA):
decorativo 1,2s · overlay com texto 2,5s · tela_cheia 3,0s — defaults no box,
exceção por elemento. Motivo da desistência registrado (`abaixo_do_piso`,
`antes_da_ancora`, `sem_intersecao`). Vira INVARIANTE DE LEI (portão reprova).
Aceite: fixture da orca ⇒ 0 elementos abaixo do piso, 0 terminando antes da âncora.

### 2 · anchor_text ≠ display_text
`anchor_text` = trecho verbatim longo (localização/grounding — NUNCA truncado).
`display_text` = subsequência contígua exibida; com `max_palavras` no elemento,
reduzido; sem o campo, igual ao anchor. I14/I16 validam o display (verbatim,
contíguo, dentro do anchor). Fórmula: `leitura_s = palavras_exibidas/2,0 + 1,0`;
`dur = min(janela_falada, max(texto_min_s, leitura_s))`; abaixo do piso ⇒ desiste.
⚠ renomear semântica: `texto_seg_por_palavra=2.0` está sendo lido como s/palavra
(4 palavras = 8s!) — virar `texto_palavras_por_segundo`.
Aceite: Texto05 nunca >4 palavras nem >teto de leitura; demais intactos.

### 3 · Box JSON (1 arquivo por família, schema versionado)
Campos: id, component, familia, quando (humano), intencoes[], dominios[], tons[],
max_palavras?, dur_min/max?, habilitada, ban_reason?, prioridade numérica (nunca
"preferida" booleana — vira outro vício). Codegen no preparo gera
`registry.generated.ts` (imports ESTÁTICOS — import dinâmico quebra bundling e
typecheck). Migração: leitor JSON em MODO SOMBRA compara com a TABELA ⇒ corrige
divergências ⇒ CORTE único (TABELA morre; sem fallback permanente). Manifesto
inválido/componente ausente/ID duplicado = falha FECHADA no preparo.
Aceite: elemento novo = componente + entrada JSON; tsc passa; TABELA com zero
leituras; manifesto inválido para o preparo.

### 4 · Rotação entre vídeos (ledger por canal)
`state/element_usage.json` por CANAL, fora dos jobs (cruzar jobs é o PROPÓSITO —
não viola L11, que isola assets de produção, não memória editorial).
Ordem: adequação semântica PRIMEIRO; empate ⇒ last_used_at mais antigo ⇒ menor
count ⇒ hash estável. Ledger só atualiza quando o elemento SOBREVIVE aos portões
(contar descartado falsifica o histórico). Idempotente por video_id+elemento.
Aceite: 3 variantes equivalentes em 10 vídeos ⇒ nenhuma em 2 vídeos consecutivos
havendo alternativa; diferença de uso <= 1; inferior semântica nunca vence por
estar "fria".

### Ordem de execução
1. Portão temporal + desistência + invariante de lei  ← resolve o dano visível
2. anchor/display + teto de leitura + max_palavras(Texto05)
3. Box JSON + codegen + sombra + corte
4. Ledger de rotação por canal
Fixture de regressão: os 19 elementos da orca congelados ANTES de qualquer mudança.
Tudo sem mexer no LLM (intenções continuam as mesmas).

### O que NÃO fazer (Codex)
Não corrigir só no TSX · não esticar curto até o piso (viola a desistência) · não
usar duração de cena como válida por si · não fazer python interpretar o `quando`
livre · não manter TABELA como fallback permanente · não importar TSX dinâmico ·
não sortear antes de avaliar adequação · não atualizar ledger antes dos portões ·
não reusar `2.0 s/palavra` sem corrigir a semântica.


---

# EXECUÇÃO (12/08) — 3 itens feitos, o 4º SUBSTITUÍDO

| item | estado |
|---|---|
| 1 · piso + desistência + L12 | ✅ e maior que o previsto: eram **9 de 19** elementos ilegíveis, não 3 |
| 2 · teto de leitura + max_palavras | ✅ BoxedKicker 10,21s → 3,00s, 4 palavras, âncora intacta |
| 3 · Box JSON + codegen + sombra + corte | ✅ 40 variantes, sombra 32/32 e 238/238, tsc no baseline |
| 4 · ledger de rotação por canal | ❌ **DESCARTADO** — ver abaixo |

## Por que o item 4 morreu

O operador questionou se era necessário. A medição deu razão a ele: **25 das 31
rotas têm UMA variante** — a rotação não teria o que rotacionar. As 6 rotas com
alternativa são todas de gráfico; nenhuma toca texto ou overlay, que é onde ele
reclamou (o BoxedKicker abrindo todos os vídeos). Construir lock, contador
monotônico e idempotência para 6 rotas de gráfico é o que a diretriz do "problema
não existir" condena.

E o dado apontou a causa real: **24 das 38 variantes nunca saíram** na orca — não
por falha de rotação, mas porque a ROTA delas nunca é pedida.

## O que entrou no lugar: a variante SERVE À CENA

Ordem do operador (12/08): *"cada variante serviria melhor a um tipo específico de
spec, e a seleção seria determinada pela ADEQUAÇÃO AO MOMENTO, não pela posição na
lista. Trabalhando para servir à cena, não criando um padrão fixo."*

Campo `serve` no box, FILTRO RÍGIDO (Codex vetou pontuação parcial: variante venceria
violando limite essencial). Operadores: `palavras_min/max`, `caracteres_min/max`
(largura visual ≠ nº de palavras), `valores_min/max`, `trecho_palavras_min/max`,
`duracao_min/max`, `tem`/`nao_tem`, `posicao`. Sem `serve` = coringa.
Ordem: adequação → especificidade → ordem_base → peso_operador → id.
`usadas` (rodízio intra-vídeo) entra por ÚLTIMO, nunca acima da adequação.

**A variação entre vídeos vira consequência, não mecanismo**: cenas diferentes
escolhem elementos diferentes sozinhas. Sem arquivo de estado, sem lock, sem
idempotência, sem problema com produção paralela ou 200 canais. Nenhum campo novo:
todos os atributos já existem no plano na hora da escolha.

Provado em `test_serve_a_cena.py`: o mesmo `spec tela_cheia` com unidade dá
BoxedKicker; sem unidade dá outro elemento. Determinístico.

## O que NÃO está provado

Nada disto rodou num job real. Portão temporal, teto de leitura e catálogo foram
provados contra fixture e testes — nunca contra uma produção de ponta a ponta.

## Trabalho de CURADORIA que sobrou para o operador (não é engenharia)

O vício só morre quando as rotas que o LLM realmente pede ganharem irmãos, cada um
com seu `serve`. Hoje `spec tela_cheia` = só BoxedKicker. Acrescentar irmão agora é
uma entrada de JSON + `python director/boxes_codegen.py`.
