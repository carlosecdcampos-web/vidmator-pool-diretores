# ESPEC-V14 — A ÂNCORA DE FALA É DADO DE PRIMEIRA CLASSE

Ordem do operador (10/08), aprovando a regra-mãe desta rodada:
> *"a âncora de fala é um dado de primeira classe, que nasce no detector e viaja
> intacto até o render — nunca reconstruído por adivinhação a jusante."* — **CONCORDO.**

Origem: veredito do v12 (o operador assistindo) + investigação minha (frame a frame
no MP4) + investigação do Codex (código), com contraditório cruzado. As duas
investigações convergiram para **uma causa-raiz única** que explica capítulo, tease,
acervo, pessoa e mapa de uma vez.

---

## A CAUSA-RAIZ (confirmada pelos dois lados)

Todo detector JÁ localiza seu elemento por um **trecho verbatim** validado contra o
roteiro — e o **write-back joga esse campo fora**:

```python
tl["acervo_texto"] = [{"inicio":…, "variante":…, "dados":…, "dur":…} …]   # 7 campos à mão
```
`director/edicao.py:1394` · mesmo padrão em `pessoas.py:190` e `detectar_mapas.py:204`.

Não é otimização: é lista branca escrita à mão, feita quando ninguém precisava do
trecho depois. Resultado: o reconciliador tem que **re-adivinhar** a âncora a partir
de um texto editorial que nunca foi falado assim — e quando erra, cai na duração da
**cena inteira**. Daí os 14 elementos com `_dur_de="cena"` (todos os reclamados)
contra 4 com `_dur_de="fala"` (nenhum reclamado).

Prova numérica (capítulos, fala do título × tempo de tela):
| cap | card fica | ideal | erro |
|---|---|---|---|
| 5 KING COBRA | 5,41s | 3,21s | **+2,20s** |
| 4 GREEN ANACONDA | 6,38s | 3,72s | **+2,66s** |
| 3 RETICULATED PYTHON | 2,99s | 3,99s | **−1,00s** |
| 2 BLACK MAMBA | 6,72s | 3,83s | **+2,89s** |
| 1 RUSSELL'S VIPER | 5,59s | 3,21s | **+2,38s** |

Cinco de cinco batem com o que o operador apontou de olho, inclusive o único
negativo (cap 3 saindo antes de terminar o nome).

---

## ORDEM DE REFINAMENTO (executar nesta sequência)

### R1 🔴 A ÂNCORA NASCE E VIAJA — fim da lista branca
- Todo elemento passa a carregar `ancora = {trecho, w_ini, w_fim, t_ini, t_fim, metodo}`,
  gravada **no detector** (que já a calculou) e **preservada em todo write-back**.
- Write-back deixa de ser lista branca: campos que começam com `ancora`/`_` viajam.
- Fontes por tipo: acervo → `momento.trecho`; pessoa → `trecho` do detector;
  mapa → `trecho` do evento; **capítulo → marcador + título extraído das words**
  (hoje o título só nasce no preparar: a extração sobe para o `edicao`);
  tease → a cláusula do gatilho até a pontuação final.
- Mata: capítulos longos/curtos, pessoa e mapa fora de hora, acervo desalinhado.

### R2 🔴 MATCH NÃO FICA PRESO À CENA
`janela_falada` só procura dentro da cena atribuída — "India" é falado 0,26s **depois**
do fim da cena, "Past five meters" 2s **antes** do início. Com a âncora de R1 isso
deixa de ser busca (os índices já vêm prontos); onde ainda houver busca, a janela é
±6s em volta da âncora, não a cena.

### R3 🔴 NENHUM DONO COBRE OUTRO
Eu puxei o `Graf05_VersusBars` de 123,82s para 119,44s — **em cima do capítulo 4**.
Foi a minha correção que criou a oclusão, e nenhuma lei detecta dois donos na mesma
janela. Passa a valer: antes do write-back, interseção owner×owner é resolvida por
prioridade (capítulo > número/S10 > pessoa/mapa > texto), e o perdedor é **deslocado
ou devolvido**, nunca sobreposto. Lei nova **L10**.

### R4 🔴 SAÍDA NÃO DEIXA RESÍDUO
Mover a entrada incorpora o trecho anterior; mover a saída **corta a cena seguinte** e
cria resíduo (mapa: 1,41s — acima do meu limiar de 1,2s, então nem candidato a
absorção; e protegido pela minha própria regra de fronteira). Correções:
- limiar de fragmento sobe para **< 2,0s** (o operador reclamou de "0,5 a 1s" e de
  "milissegundos", mas o resíduo real medido foi 1,41s);
- resíduo criado PELO reconciliador é absorvido pela cena **seguinte** (a fronteira
  protegida é a de entrada do elemento, não a de saída);
- o take que fica **escondido** sob um elemento durante toda a vida dele não pode
  reaparecer no crossfade da próxima cena (provado frame a frame em 49,55–49,75s):
  cena totalmente coberta por dono de tela termina **junto** com ele e a próxima
  entra em **corte seco**, sem fade que revele o que estava por baixo.

### R5 🔴 WORD-POP É ENUMERAÇÃO, NÃO FRASE
Medido: 9,86s para 4 palavras, com intervalos de 3,86s e 3,52s — fundo travado, que é
o oposto do dinamismo que o word-pop existe para dar. Passa a exigir **cadência de
lista**: gap máximo entre itens ~1,2s e janela total ≤ ~4s; fora disso a série é
recusada (vira texto normal). Lei nova **L11**.

### R6 🔴 AS LEIS DEIXAM DE ACEITAR CARIMBO
- **L08 recomputa**: com `ancora.w_ini/w_fim` na mão, a lei recalcula o esperado
  (`t_ini − pré-roll`, `t_fim + folga`) e compara com tolerância de **1 frame**.
  Verificar a string `_dur_de` é aceitar carimbo — e o meu carimbo já mentiu duas vezes.
- Distinguir `fala` de `fala+piso`: o piso de legibilidade (2,5s) infla a janela e
  hoje se disfarça de "fala".
- **Guard de congelamento passa a cobrir as janelas dos donos**, não só `cenas`: hoje
  o R2/R2b mudam elementos depois do lock e o log anuncia "congelamento respeitado".
  Aquele ✓ era falso.

### R7 TAMANHO (pedido direto do operador)
`Graf01_CounterGlow`: número **190 → 250**, título **40 → 55** (letterSpacing 5, caixa
alta mantidos).

---

## MÉTODO
1. Implementar na ordem R1 → R2 → R3 → R4 → R5 → R6 → R7, **com o Codex revisando
   cada item** (contraditório antes de commit).
2. `_regressao_leis.py` roda a cada item: lei validada que virar ✗ é regressão e
   barra o avanço.
3. Bancada offline no timeline real; **render só com GO explícito**, saída em
   `serpentes_v13.mp4` (o v11 e o v12 ficam intactos para comparação).
