# ESPEC-INCIDENTE — ciclos VEO concorrentes: prompts no projeto ERRADO (07/08 20:40)

> Sintoma visto pelo operador AO VIVO: o ciclo do "teste 01 - piranha" estava gerando
> na coleção certa; *"a página atualizou e foi pra"* a raiz do projeto ANTIGO
> (teste-angulo), onde os prompts novos (sem carimbo) começaram a nascer misturados
> aos cards velhos. Depois *"carregou outra vez e voltou"*. A aba pulava sozinha.

---

## 1 · O QUE ACONTECEU (fatos, com evidência)

| hora | fato | evidência |
|---|---|---|
| 20:22 | Ciclo **A** lançado: `T15ANGULO/teste-angulo` (15 imagens, era do carimbo) | log `bt9wznqc3` |
| 20:40 | Ciclo **B** lançado: `DIRETOR-VEO/teste 01 - piranha` (15 imagens, sem carimbo) | log `brq4olg1y`: modo na raiz ✓ → coleção criada ✓ |
| 20:43 | Prompts do **B** gerando na RAIZ do projeto do **A** (7ab2d4a2), misturados aos cards "CENA 027" | print do operador |
| 20:44 | **QUATRO** processos `veo_ciclo` vivos: 2 desta sessão (_veo_venv) + **2 ZUMBIS em Python312 do sistema**, de execuções antigas | `Get-CimInstance Win32_Process` |
| 20:44 | Aba volta à coleção certa (o guard `_na_colecao()` novo re-entrou) e pula de novo | prints + mensagem do operador |
| 20:45 | Kill POR ÁRVORE dos 4 → "ZERO veo_ciclo vivo" → aba estabiliza; coleção certa com 5 imagens | PowerShell + print |

## 2 · CAUSA RAIZ (mecânica, confirmada pelo comportamento)

**Todos os ciclos usam o MESMO Chrome/perfil** (`chrome_profile_conta1` nos dois
logs). Quando dois ciclos vivem ao mesmo tempo, **cada `goto`/`reload` de um navega
a aba que o outro está usando** — e o envio de prompt digita na barra da PÁGINA
ATUAL, seja ela qual for. O ciclo A (em fase de espera, com reloads periódicos, e
rodadas novas com `goto` no projeto dele) puxava a aba para o projeto A; o ciclo B,
entre um guard e outro, digitava os prompts dele lá. Resultado: mídia do B na raiz
do projeto A.

**Por que havia ciclo A vivo**: (a) o timeout do lançamento em background NÃO matou
a árvore (a mesma lição da memória `feedback-taskstop-nao-mata-filhos` — o kill pega
o wrapper, o python filho segue); (b) além do A, havia **2 zumbis de execuções
ANTERIORES** (interpretador do sistema, não o _veo_venv) que ninguém matou — rodando
há horas, invisíveis, esperando a vez de estragar.

**Erro operacional do agente (meu)**: lancei o B sem CONFIRMAR a morte do A — a
notificação de conclusão do A nunca tinha chegado, o que era o próprio aviso de que
ele ainda rodava. A memória já dizia: *"NUNCA rodar sonda/ciclo em paralelo com
ciclo vivo"*. Regra existia; a checagem não era mecânica. Agora é (§3.1).

## 3 · BLINDAGENS (todas implementadas em 07/08)

1. **LOCK DE CICLO** (`_veo_flow/ciclo.lock`, `{pid, canal, ts}`): o ciclo
   reivindica no arranque; se há outro dono com **pid VIVO**, aborta ALTO antes de
   tocar o browser, com a instrução de kill por árvore na mensagem. Liberação no
   `finally`. Mesma doutrina do `workspace_lock` do producao, no domínio do browser.
2. **VARREDURA DE ALHEIOS no arranque**: o lock só enxerga quem o escreve — zumbi
   de CÓDIGO VELHO não escreve lock. Antes de reivindicar, o ciclo varre a lista de
   processos por `veo_ciclo` alheios e aborta se encontrar (não mata — matar processo
   que não conhece é decisão do operador/agente; abortar é sempre seguro).
3. **Guard `_na_colecao()`** antes de cada rajada, após cada reload e antes do
   download (guia do Piter §1) — foi ele que limitou o dano re-entrando na coleção.
   Contra colisão ele MITIGA (janela entre guards ainda vaza); quem CURA é o lock.
4. **Kill por árvore como doutrina** (`taskkill /T /F /PID`) + conferência de
   "zero vivo" ANTES de relançar — memória `feedback-taskstop-nao-mata-filhos`
   atualizada com este caso.

## 4 · O QUE É MITIGAÇÃO vs CURA (honestidade da barreira)

- **CURA da colisão**: lock (§3.1) + varredura (§3.2). Dois ciclos nunca mais
  coexistem — o segundo morre no arranque com mensagem clara.
- **MITIGAÇÃO**: o guard de coleção (§3.3) — protege contra QUALQUER expulsão
  (popup, redirect do Flow), não só colisão. Fica pelos outros motivos.
- **EM ABERTO** (aceito): a origem exata dos 2 zumbis Python312 não foi
  reconstruída (execução de teste antiga, anterior ao _veo_venv nos lançamentos).
  Irrelevante para a defesa: a varredura §3.2 pega qualquer um, de qualquer época.

## 5 · Limpeza do rastro

- Projeto antigo `teste-angulo` (7ab2d4a2): tem os cards vazados do lote B (sem
  carimbo) misturados aos do teste de ângulos — **lixo inofensivo em projeto de
  teste**; o operador pode excluir o projeto inteiro quando quiser.
- Coleção `teste 01 - piranha` (projeto DIRETOR-VEO): intacta, 5 imagens certas no
  momento do kill; o relançamento reenvia o lote completo (duplicata do mesmo prompt
  não confunde o casamento por título — mesmo prompt → mesma cena).

*Escrita 07/08/2026, no calor do incidente, com os prints do operador como evidência.*

---

# PARTE 2 — o PAREAMENTO pós-carimbo (mesma noite, 20:53→21:20)

Com o carimbo CENA morto (ele queimava a plaquinha no quadro), o download precisou
de identidade nova. Três rotas medidas, uma sobreviveu:

| rota | resultado | por que |
|---|---|---|
| **Zip do projeto + título do arquivo** (guia do Piter §5) | **1/15** | o Flow resume o título pro SUJEITO ("Red-bellied_piranha_in_river") — num lote de sujeito ÚNICO os títulos ficam iguais; funciona no b-roll multi-sujeito do Piter, não no doc de UM animal |
| **Vision no conteúdo** (assinatura ângulo/direção da lei) | 8/15 (greedy) | funciona mas é FRÁGIL e caro — workaround, vetado pelo operador: *"os prompts aparecem na página, não consegue ver na hora do download?"* |
| **Texto do card na VISTA EM LISTA** (rota final, ditada) | casamento exato (sonda: 3/3; mesa: 15/15) | o prompt COMPLETO está pareado com cada imagem na página — é a identidade literal |

## A pegadinha que custou 3 execuções vazias: A VISTA

- Coleção recém-carregada abre em **GRADE**: sem texto de prompt, e o botão Baixar
  só existe em hover → o loop rodava no VAZIO (0 casamentos, 0 erros — silêncio).
- A sonda "funcionou" uma vez porque leu a ABA VIVA do operador, que tinha
  alternado pra lista manualmente; o script seguinte navegou de novo → grade → 0/15.
- **FATO ditado pelo operador (prints 21:10)**: na vista em lista, o download por
  mídia mora em **hover NA IMAGEM → ⋮ (canto da imagem) → "Baixar"** — mesmo
  caminho para imagem e vídeo. Hover no centro da LINHA cai na coluna de texto e
  não materializa nada (era o "muito tempo parado sem nada acontecer").

## Leis do download 1-a-1 (veo_download, era pós-carimbo)

1. `_garantir_vista_lista`: detecta a lista pela âncora universal "cinematic
   documentary" (todo prompt nosso a carrega — lei 8 do GUIA); em grade, tenta os
   ícones de alternância; sem achar, GRITA e espera o operador (1 clique) — parado
   e honesto > baixando errado.
2. Casamento: carimbo legado se houver; senão cobertura de tokens raros do texto
   do card × prompts do lote (IDF do lote, piso 0.45, margem 0.15 — ambíguo NÃO casa).
3. Download: hover na IMG → ⋮ no box da imagem → menuitem "Baixar" (fallback:
   botão fixo). PK unwrap na chegada.

**EM ABERTO**: o seletor/ícone exato do alternador de vista da coleção (o dump não
o revelou; a lista de candidatos view_list/view_agenda/table_rows está no código —
confirmar no próximo uso real e fixar).

## DESFECHO (22:05) — a rota vencedora: LINHA-A-LINHA, ordem literal do operador

A rota do hover→⋮→Baixar chegou a 15/15 mas com 3 pares de conteúdo DUPLICADO
(o clique materializado pega o card VIZINHO em ~20% dos hovers; a guarda anti-dup
por hash rejeitava e re-tentava, mas a heurística primeiro-chega-é-dono não é
prova de correção). O operador ditou a arquitetura final, a mesma da extensão
AC-Flow VEO3 Automator PRO: **"cada linha tem um prompt e uma imagem: baixa a
imagem, coloca o nome do prompt no arquivo, pula, baixa a próxima."**

`_dl_linha_a_linha.py` (rota canônica p/ generalizar no ciclo):
1. bloco de prompt = elemento MINIMAL com innerText >80c que não é metadado —
   **sem âncora de palavra**: o Flow trunca o FIM do prompt no DOM, "cinematic
   documentary" some (a causa do 0/15 da 1ª rota-src);
2. linha = primeiro ancestor do bloco que contém `<img>`; guard: linha com 2
   blocos = subiu até a lista → descarta;
3. baixa `img.src` (reescrito `=w###` → `=s0` p/ original) via
   `page.request.get` (cookies da sessão) — **ZERO cliques = zero vizinho**;
4. nome do arquivo = prompt sanitizado (porta do filename.js, 80c, uniquify _2);
5. rolagem incremental até 8 secas (virtualização); dedup por src.

**Resultado: 21 downloads em ~2 min, 20 mídias (1 lixo de UI filtrado a
posteriori), 20 conteúdos ÚNICOS por hash.** O casamento prompt→cena, quando
necessário (produção), é OFFLINE a partir do nome do arquivo (cobertura de
tokens, mesa 15/15) — nunca mais durante o download.
