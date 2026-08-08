# ESPEC — IMPORTAÇÃO DA AUTOMAÇÃO VEO/FLOW DO PITER (1º candidato da sanitização)

> **Status:** 🟡 **DECUPAGEM PRONTA — aguardando (a) infraestrutura provisionada e
> (b) GO para a tradução.** Etapas 1-4 do processo de sanitização executadas: coleta
> (pull nos dois repos), inventário, triagem e esta ficha. NADA foi copiado para
> `director/` — regra do container.
>
> Ordem do operador (06/08): *"quero incorporar TODA A INTELIGÊNCIA DA AUTOMAÇÃO DO
> VEO... fará a gente ganhar muito tempo, ele já está refinando isso há dias, então
> vamos fazer isso com inteligência e cautela."*

---

# 1 · COLETA (etapa 1) — o que chegou no pull de hoje

- `director-repo` (vidmator-director): **79 commits novos** desde a nossa cópia de 31/07
  — praticamente todos na frota Flow/VEO. O push das ~23h de ontem está incluído
  (`d82752a`, "grafo atualizado: colecoes, ciclo v2, avatar, planejador de takes").
- `piter-repo` (video-automator): 6 commits novos, todos de COLLAB de canal (RPA de
  convite/aceite) — não é VEO, fica para outra rodada.

# 2 · INVENTÁRIO (etapa 2) — a frota, medida

**15 arquivos, ~3.100 linhas, TODOS ⚪ NOVO** (não existem aqui = importação legítima).
Gap de dialeto baixo: `path cravado` em 4, `aborta o pipeline` em 1, resto limpo.

## A linha de produção dele (medida no código, não no discurso)

```
plano.json (beats)                        ← story-engine DELE (dialeto dele)
   └─ veo_lote.py       gera o LOTE de prompts (modo video/imagem/misto por canal)
        └─ veo_prompt.py    "DIRETOR DE FOTOGRAFIA": beat -> direção real (sujeito+ação+
                            ambiente+luz+câmera+look). NÃO manda query de stock crua —
                            e explica por quê (keywords/negação quebram o VEO)
   └─ veo_driver.py / veo_ciclo.py   dirigem o Google Flow (labs.google) via Playwright
        · flow_driver.py    sessão logada em perfil Chrome DEDICADO (login manual 1x)
        · veo_colecao.py    PROJETO = CANAL · COLEÇÃO = VÍDEO (preserva personagens,
                            e dá o "Baixar coleção" — 1 zip por vídeo)
        · veo_supervisor.py ENCARREGADO: decide travamento por ARQUIVO NO DISCO (não
                            por log), mata driver+Chrome, resgata com --so-baixar,
                            retoma só o que falta (nasceu do lote de 98 que ficou 3h30
                            pendurado)
        · veo_paralelo.py   N perfis Chrome = N contas = N fatias do lote em paralelo
        · perfis.py         monitor: qual conta está LIVRE/logada
   └─ veo_zip.py        abre o zip da coleção e casa clipe->prompt por título
   └─ curador.py        GATE: manda o VÍDEO INTEIRO pro Gemini (vê MOVIMENTO —
                        flicker/morphing/física, que frame parado não pega);
                        keep/reject/revisar + curadoria.json
   └─ veo_ingest.py     só o que está em keep/ entra no job (formato do executor DELE)
```

## Inteligência EMPÍRICA embutida (o que custou dias para ele descobrir)

| Descoberta | Commit/doc |
|---|---|
| `Veo 3.1 Lite [Lower Priority]` = **0 créditos** (fila lenta); Fast = 10 créd/clipe 8s; imagem Nano Banana = 0 | GUIA_VEO_FLOW_GRATIS |
| Teto empírico da fala de um take: **80-90 chars / ~16 palavras** (menos fala LENTO, mais corta) | commits 31/07-05/08 |
| Negação no prompt QUEBRA o gerador ("no X" às vezes produz X; áudio "Falha ao gerar") — enquadramento sempre POSITIVO | `2dab9c4` |
| Nome de personagem SÓ no chip `@` — no texto vira pronome (política de "pessoa famosa") | `acd7ee1` |
| Variação automática de prompt no reenvio quando cai na política | `9947dc8` |
| Menção `@` por digitação NÃO funciona (3 falhas) — o fluxo real foi sondado na UI | `92bf746`→`5c3cd57` |
| "Restaurar páginas?" do Chrome rouba clique — limpar `exit_type` nas Preferences ANTES de abrir | flow_driver `_limpar_saida_suja` |
| Travamento se mede por ARQUIVO PRONTO no disco; log de driver de browser mente | veo_supervisor |
| reCAPTCHA Enterprise ATIVO no Flow — nunca automatizar login; sessão persistente é o caminho | FLOW_MAP |

# 3 · TRIAGEM (etapa 3) — o que entra, o que fica, o que espera

## 🟩 NÚCLEO agnóstico → TRADUZIR E IMPORTAR (a "inteligência" pedida)
`flow_driver` · `veo_driver` · `veo_ciclo` · `veo_colecao` · `veo_supervisor` ·
`veo_paralelo` · `perfis` · `veo_zip` · `curador` · `veo_prompt` · `projetos.json`
— dirigem o Flow e não sabem nada do pipeline de ninguém. Tradução = dialeto
(vmconfig, sem path F:/, knob, degradar, heartbeat).

## 🟨 PONTAS acopladas ao sistema DELE → substituir por ADAPTADORES nossos
- `veo_lote.py` lê `plano.json`/beats/style_card — **dialeto do story-engine dele**.
  Nosso adaptador (`veo_pedido.py`) lê o que JÁ produzimos: **hard-miss do resolver +
  espécie exata do contrato + `documentary_setting`** → `_gerar.json` no formato que o
  driver espera. É o E16 exatamente onde ele doía.
- `veo_ingest.py` escreve o formato do executor dele (`bNNN__T0__veo`) — o nosso escreve
  no **pool** (`_cache_stock`, com sidecar de origem `veo`) e no `clip_path` da cena,
  passando pela MESMA barreira de validação do preparar (nada de porta lateral).

## 🟦 AVATAR → CATALOGADO, NÃO IMPORTA AGORA (decisão do operador)
`veo_personagem` · `veo_avatar_plan` · `veo_voz` · `avatar_prompts` ·
`personagens.json` · doutrina de identidade — *"esse do avatar usaremos em OUTROS
diretores"*. Fica mapeado nesta espec para a hora certa.

# 4 · 🧰 LISTA DE INFRAESTRUTURA (o que o operador pediu)

## Obrigatória antes do 1º clipe
| # | Item | Quem faz | Observação |
|---|---|---|---|
| 1 | **Conta Google com acesso ao Flow** (plano AI Pro ou Ultra) | operador | dá para começar 100% GRÁTIS: `Veo 3.1 Lite [Lower Priority]` = 0 créditos (fila mais lenta); Nano Banana imagem = 0 |
| 2 | **Login manual 1× no perfil dedicado** | operador (5 min) | eu abro o Chrome no perfil novo, você loga; a sessão persiste. **Nunca** login automatizado (reCAPTCHA Enterprise ativo; risco de conta) |
| 3 | **Chrome do sistema instalado** | ✅ já temos | o driver usa `channel="chrome"` — e nesta máquina isso é OBRIGATÓRIO: o Chromium do Playwright é quebrado para janela visível (lição do onboarding do YouTube) |
| 4 | **venv com Playwright** (`_veo_venv`) | eu | espelho do `veo_venv` dele (Playwright 1.61); nosso `_venv-director` não leva Playwright — isolado é mais seguro |
| 5 | **Estrutura de pastas** `_veo_flow/{perfis/<conta>, downloads, zips}` | eu | fora do git (gitignore), como cache |
| 6 | **1 PROJETO no Flow por canal + 1 COLEÇÃO por vídeo** | eu crio via driver, você confere | é a estrutura que preserva personagens e dá o "Baixar coleção"; registrada em `projetos.json` nosso |
| 7 | **Chaves Gemini para o curador** | ✅ já temos | rotação já existe no nosso `gemini_api` |
| 8 | **Smoke de 1 clipe** nesta máquina | eu, com você olhando | valida o driver headed + Chrome do sistema AQUI antes de qualquer lote |

## Para escalar depois (não bloqueia o início)
| # | Item | Nota |
|---|---|---|
| 9 | **N contas Google = N perfis = vazão N×** (`veo_paralelo`) | cada perfil é um Chrome separado, créditos separados |
| 10 | **Créditos/plano pago** se o Lite-fila-lenta não der vazão | Fast: 10 créd/clipe de 8s |
| 11 | **Disco** para zips de coleção + clipes | ~50-150 MB por vídeo gerado |

## Riscos assumidos (com os olhos abertos)
- **ToS/bot-detection**: reCAPTCHA Enterprise visto na rede do Flow. Mitigação do
  Piter (que adotamos): sessão humana persistente, login manual, ritmo de UI real,
  variação de prompt no reenvio. Risco existe e fica dito.
- **UI do Flow muda sem aviso**: o FLOW_MAP é de 15/07-05/08; seletor quebrado =
  driver parado até recalibrar (o supervisor pelo menos NÃO trava: mata e reporta).

# 5 · PLANO DE TRADUÇÃO (etapas 5-8 — depois do GO + infra)

```
_import/2026-08-06-veo-flow/          ← container (nada em director/ até passar)
  nucleo/ (10 arquivos traduzidos: vmconfig, sem F:/, degrada, heartbeat)
  veo_pedido.py     ← NOSSO: timeline (hard-miss + espécie exata) -> _gerar.json
  veo_ingest.py     ← NOSSO: keep/ -> pool/_cache_stock + clip_path (via barreira)
  knob: preset.veo = {"ativo": false, "modo": "lite_gratis", "max_clipes": N}
```
1. Traduzir núcleo (dialeto §PARTE 2 da ESPEC-SANITIZACAO).
2. Smoke de 1 clipe (infra #8) — valida driver nesta máquina.
3. Quarentena: rodar o urso com `veo.ativo=true` num workspace-cópia; diff do render
   json contra baseline — toda diferença explicada.
4. Prova (idempotência + portão) → adoção em 1 commit, knob OFF por default.

**Ganho esperado:** os 8 hard-miss do urso (que hoje viram herança-com-moldura da MESMA
foto) e a espécie exata que o Pexels não tem (baleia-azul → 4/4 rejeitados) passam a ser
FABRICADOS sob medida — o buraco de acervo que trava o F2/F3 fecha na origem.

*Aberta em 06/08/2026. Decupagem verificada arquivo a arquivo no upstream atualizado.*


---

# 6 · ✅ SMOKE COMPLETO (06/08, 16:58) — o ciclo inteiro provado NESTA máquina

`_veo_flow/downloads/smoke_arctic.mp4` — 8s, 720p, **0 créditos**: configurado, enviado
com prova de vida, gerado (fila Lower Priority ~10 min) e **baixado pela automação**.

## A DOUTRINA DA SESSÃO (o que esta máquina exigiu — divergências do upstream)

O caminho até aqui queimou 5 tentativas. O que ficou provado:

| # | Lei | Por quê (medido) |
|---|---|---|
| 1 | **Chrome nasce como PROGRAMA NORMAL** (subprocess), nunca pelo Playwright | o Google rejeita login em navegador parido por automação ("pode não ser seguro") — o Chrome novo ignora o flag anti-detecção que o Piter usa |
| 2 | **Playwright CONECTA via CDP** (`connect_over_cdp`), nunca lança | mesma lição do onboarding do YouTube nesta máquina |
| 3 | **Login + autorização do app (AI Test Kitchen) 1× por perfil**, com 2FA "não perguntar novamente" | a autorização fica no perfil; não repete |
| 4 | **Fechar = só desconectar** — o Chrome fica vivo entre execuções | sessão quente; force-kill perde cookies não gravados |
| 5 | **Clonar sessão de outro ambiente NÃO serve para Google** | clone passa no 1º load e morre no 1º refresh de token (re-desafio) |
| 6 | **Prova de vida obrigatória no envio** (card de % no grid) | o fill() colava sem eventos e o driver dizia "enviado" com projeto vazio — 3× |
| 7 | **Imagem = Nano Banana Pro, SEMPRE** (ordem do operador; upstream usa Nano Banana 2) | — |

Divergências que o Piter vai querer saber (a inteligência correndo no sentido contrário):
o flag `--disable-blink-features=AutomationControlled` está MORTO no Chrome atual — quando
o Chrome dele atualizar, o setup dele quebra igual quebrou o nosso.

## Próximo (para o funcionário "otimizado para o trabalho")
1. Traduzir o restante da frota no container: `veo_driver` (lote+download), `veo_ciclo`
   (rodada por coleção — casa perfeito com a doutrina da sessão única), `veo_supervisor`,
   `veo_zip`, `curador`, `veo_prompt`.
2. Adaptadores nossos: `veo_pedido` (hard-miss + espécie exata do timeline → _gerar.json)
   e `veo_ingest` nosso (keep/ → pool + clip_path via barreira).
3. Ensaio geral: lote pequeno (5-10 takes) numa coleção, ciclo completo sem supervisão.
4. **v7 = produção fresh do documentário com o VEO cobrindo os hard-miss.**

## ⚖️ A LEI DO TAKE (operador, 06/08 — veredito sobre o ensaio de 6 real)

O ensaio de 6 provou empiricamente onde o VEO acerta e onde quebra. **Aprovados (4):**
os takes contemplativos de UM sujeito — a espreita paciente no buraco, o urso se
afastando, a morsa em repouso, o voo aéreo sobre o gelo. **Reprovados (2):** ação e
interação — "o urso batendo na foca ficou muito zoado"; urso+raposa "ficaram parecendo
amigos, deitados juntos". Não é erro nosso nem do prompt individual: é limitação do
próprio VEO com ação/muitos elementos/transições.

A lei (codificada em `veo_pedido.py`: GUIA reescrito + `_leis_do_take()` = enforcement
determinístico pós-LLM — prompt que viola é re-vestido, nunca enviado como veio):

1. **ESPECIFICIDADE ACIMA DE TUDO** — "se é uma cena de um urso polar, deve chegar até
   ele que é de um urso polar, não de um predador do ártico. A especificidade é o que
   vai fazer o trabalho ficar perfeito; sem tudo especificado ele gera lixo."
   (`_sujeito()`: required_subjects do contrato; canonical genérico "animal" não vale,
   cai na stock_query que nomeia a espécie.)
2. **Estilo SEMPRE**: "cinematic documentary look" + tranquil ("sense of tranquil
   solitude" / "serene natural ambiance").
3. **Câmera SEMPRE estilo footage**: "very slow zoom in" OU "very slow zoom out". Só.
4. **ZERO trilha sonora** (a trilha é nossa) — pedido na forma POSITIVA "ambient
   natural sound only" (negação gera o negado, lição 2dab9c4). ASMR ambiente é muito
   bem-vindo (construção de cena).
5. **AÇÃO É PROIBIDA** — o momento de ação vira o instante contemplativo da MESMA
   narrativa (a espreita, não o golpe; o vestígio, não o combate). `_ACAO_RX` detecta
   verbo de ação que escapou do LLM → re-veste com a fórmula contemplativa.
6. **UM sujeito por take** — interação entre espécies não vai para o VEO.
7. **Imagem**: mesmas leis, SEM movimento de câmera.

Bancada (mesmos 6 hard-miss, lei ativa): s029 "paw striking" → "seal quietly surfacing
to breathe" (1 sujeito, momento calmo); s035/s036 raposa+urso → raposa SOZINHA. 6/6.

### Pendência derivada (edição, não prompt): normalização do áudio dos clipes VEO
Os clipes saem com loudness DESIGUAL entre gerações (uns altos, outros baixos). O ASMR
é construção de cena e não pode roubar espaço da narração. Doutrina = a mesma dos SFX:
**normalizar cada clipe primeiro, teto depois** — teto único sem normalizar prévia
deixaria uns altos e outros inaudíveis. Implementar na ENTREGA (`veo_entrega`/engine),
medindo loudness real (M/ebur128) por clipe. AINDA NÃO IMPLEMENTADO.

## ⚖️ A LEI DA ILUSTRAÇÃO (operador, 06/08 — martelo batido no duelo de estilos)

**Escopo**: o veto às imagens de desenho/gráficas vale para o EDITOR (pegar de banco/
internet = lixo). O funcionário do VEO GERA — ilustração gerada é permitida, mas
**EXCLUSIVAMENTE para trechos de ANATOMIA**. Bônus estrutural: o banco não tem a
espécie exata (crocodilo comum passava por crocodilo-de-água-salgada); gerando, a
espécie é EXATA.

**FRONTEIRA DO ESCOPO** (precisão exigida pelo operador depois do martelo — *"anatomia
real; peso, altura, força da mordida NÃO entram nisso; pele, o que tem embaixo da pele,
dentes, órgãos, metabolismo, informações com esse teor sim"*):

| ✅ É anatomia → ILUSTRAÇÃO | ❌ É medida → LEI DOS NÚMEROS (S10/R7) |
|---|---|
| pele, o que há sob a pele, pelo, gordura | peso / quilos / toneladas |
| dentes, presas, mandíbula, ossos, crânio | altura, comprimento, metros |
| órgãos, cérebro, coração, pulmão, sangue | força da mordida (psi) |
| músculos, tendões, cartilagem, garras | velocidade, temperatura, profundidade |
| metabolismo, visão/olhos, guelras | qualquer número falado |

Motivo: medida já tem dono — a lei dos números fabrica elemento numérico animado (o
"600 kg" vira contador). Ilustrar medida duplicaria a informação em duas linguagens.
Codificado em `veo_pedido.escopo_ilustracao(texto)` (anatomia SIM; anatomia+medida na
mesma frase → a medida manda, vira número). 10/10 na bancada, PT e EN.
⚠️ Eu mesmo quase violei: o prompt do crocodilo do ensaio pedia a musculatura "that
powers its bite force" — músculo é anatomia, força da mordida é número.

**Processo**: ensaio de 5 anatomias (urso/tubarão/cachalote/crocodilo-salgado/coruja)
em flat 2D → operador: Nano Banana Pro muito melhor que Banana 2, mas o estilo flat 2D
não convenceu → duelo de 3 estilos na mesma mandíbula de tubarão (prancha científica ×
render 3D × cutaway semi-realista; raio-X adiado por ordem).

**VENCEDOR: CUTAWAY SEMI-REALISTA** — animal FOTORREALISTA (consistente com o footage
ao redor) e só a "janela" do corte é diagramática; é o que menos briga com o take real
antes/depois (não quebra a imersão). Fórmula codificada em
`veo_pedido.prompt_ilustracao_anatomia(especie, ponto_anatomico)`:
> Photorealistic {espécie} with a clean cutaway window revealing {ponto}, the reveal
> rendered as a subtle educational diagram, cinematic documentary lighting, dark
> background, purely visual wordless diagram, species-accurate anatomy

Herda da LEI DO TAKE: espécie exata, fundo escuro, wordless (texto REAL entra pelo
nosso overlay), SEM movimento de câmera, **Nano Banana Pro SEMPRE** (confirmado no chip
— configurar_imagem/garantir_modo agora falham alto em modelo errado; o dropdown do
painel tem o MESMO rótulo do chip e o .first clicava o chip → fallback silencioso
pro modelo errado, pego pelo operador em produção).

Integração pendente: detector de "trecho de anatomia" no pipeline (a cena chega ao
funcionário marcada como anatomia + espécie + ponto) — hoje a fórmula existe e o
gatilho é manual.

### Camada educacional: M1 vence (rótulos ESCRITOS na imagem)

Duelo de 3 maneiras de "dar nota de ensino" à ilustração vencedora: **M1** linhas-guia
com rótulos escritos na própria imagem · **M2** linhas + marcadores ①②③ e o texto real
entrando pelo nosso overlay · **M3** revelação progressiva (zoom macro em sequência).
**VENCEDOR: M1.**

⚠️ **LEI DO RÓTULO DITADO** — as palavras vão LITERAIS no prompt ("labeled in clean
small sans-serif capital letters exactly: FUNCTIONAL TEETH, REPLACEMENT ROWS, JAW
CARTILAGE"). Texto que o modelo INVENTA sai com grafia quebrada; texto que ele COPIA
sai limpo. NUNCA pedir "label the parts" — sempre entregar as palavras prontas.
Codificado: `prompt_ilustracao_anatomia(especie, ponto, rotulos)`; sem `rotulos` cai no
fallback wordless (overlay nosso escreve por cima).

Consequências assumidas do M1 (para decidir quando forem incomodar, não agora):
· o texto fica **queimado na imagem** → tipografia fora do nosso controle e idioma
  preso; canal em outro idioma exige regenerar (0 créditos, barato);
· os rótulos não animam nem entram sincronizados com a narração (isso o M2 dava).
Quem gera os rótulos: quem já decide o `ponto_anatomico` — o mesmo passe que classifica
a cena como anatomia (integração pendente abaixo).

## Lote misto no ciclo: `--tipo ambos` (06/08 — "já não pode ficar ponta solta")

O veo_pedido emite lote MISTO (takes + ilustrações de anatomia). O `veo_ciclo` agora
tem `--tipo ambos` (DEFAULT): roda o ciclo completo POR TIPO — **imagem primeiro**
(rápida, sai do caminho e sobrevive se a sessão longa de vídeo cair), **vídeo depois**
— cada passe com `garantir_modo` próprio (Nano Banana Pro confirmado no chip para
imagem; Veo para vídeo). Estados por tipo já eram separados
(`_gate_tentativas_{tipo}.json`, zips `_colecao_{tipo}_rN.zip`); `_envios.json` é
compartilhado sem colisão (chaves = sNNN.mp4 vs sNNN.png). Gate automático continua
só para vídeo; imagem é julgada pelo curador/operador.
Smoke validado: lote misto com arquivos prontos → dois passes fecham sem abrir Chrome.

### 🐛 Divergência pro Piter: bug de pacing no veo_ciclo upstream (d82752a)
No original, o log da rajada e a PAUSA entre rajadas estão presos dentro do
`if _reenv:` DEPOIS do loop de envio — as rajadas saem COLADAS (sem os ~50s que a
doutrina escrita três linhas acima promete) e o `i` referenciado é o do loop já
encerrado. Sobrevive porque o Lower Priority enfileira no servidor. Corrigido no
NOSSO container (log+pausa dentro do loop); o repo dele é read-only, não tocamos.

## ⚖️ A LEI DA DIVISÃO DAS CENAS (operador, 06/08 — ditada palavra a palavra)

**Cotas**: 40% banco de imagens (como hoje) · 60% VEO — e dentro do VEO 60% vídeos /
40% imagens NB Pro. Do total: **40% banco · 36% veo_video · 24% veo_imagem**.
**Anatomia vai para a cota de imagem AUTOMATICAMENTE** (pinned pelo conteúdo; se a
anatomia exceder os 24%, a cota se ajusta comendo de veo_video e depois banco).
**INTERCALADO, NUNCA em blocos** — "banco, veo, veo, banco, veo, banco, imagem NB...
bem dinâmico". Implementação: `veo_alocacao.py` com difusão de erro (a fonte mais em
déficit vence o slot; determinística — re-rodar dá o mesmo timeline) + anti-bloco
(3 iguais seguidas só quando as cotas não deixam alternativa). Campo OWNED:
`fonte_alvo`/`fonte_motivo`. Bancada no urso (72 cenas): cotas cravadas 29/26/17,
fita `B V I B V I B V B V I B V B I V...`, 4 anatomias pinadas.

**Endereçamento sem perda**: mídia do VEO entra por ÍNDICE (sNNN ↔ cena.idx), fora de
ordem (20, 29, 37, 42...) sem se perder — veo_entrega casa por idx, nunca por posição.

### A LEI DO SOCORRO (`veo_socorro.py`) — nenhuma cena órfã, nesta ordem:
- cena BANCO cujo stock não casou: **não procura substituto no banco**; 1º REUSO de
  asset já feito pelo VEO **que sirva à narrativa** (fim da herança do vizinho como
  resposta padrão); 2º nada serve → **GERAR sob medida** (fallback direto).
- cena VEO que não produziu/baixou: 1º re-gerar o take isolado (ciclo: rodada
  seguinte + variação anti-filtro); 2º ainda não → REUSO que sirva; 3º re-pedido.

**"Servir à narrativa" (régua determinística)**: espécie MÚTUA (bigrama da cena no
asset E bigrama do asset na cena — um lado só mentia nos dois sentidos: frase inteira
fazia urso reprovar urso, e um lado só fazia RAPOSA aceitar asset de URSO) + mood
igual (V5 soberana: apresentação nunca recebe tenso) + desempate por proximidade
temporal. Régua imperfeita ERRA PARA O GERAR — nunca põe mídia errada. Unit no urso:
urso serve urso (24/32/69), foca e raposa recusam e geram (29/35/36).

**Integração no v7 (pendente de adoção, passo 8 da sanitização)**: alocação roda após
contrato_visual e ANTES do resolver_cascata; o resolver passa a buscar stock SÓ para
fonte_alvo=="banco" (knob no director/, OFF por default até o GO); veo_pedido pede as
cenas veo_* pelo tipo alocado; socorro roda após ciclo+entrega.

## ⚖️ O FLUXO CANÔNICO DE CURADORIA DOS TAKES DO VEO (operador, 07/08 — DEFINITIVO)

> *"usa tudo que está pronto; regenera SÓ quem tem CENA+número na tela, uma vez;
> se vier queimada de novo, entra outra pronta que sirva; ponto.
> Zero geração para buracos — buraco enche com o que existe."*

Ordem executável (implementada em `_recuperar2.py`, vira o padrão do veo_gerar):
1. ENTREGA tudo que está pronto (por número de cena, download SEMPRE um a um).
2. Buraco → SOCORRO/reuso de pronta; sobrou → round-robin das prontas. NUNCA gera.
3. CURADOR: só o padrão "CENA + número" reprova (texto de livro/prancheta/rótulo
   didático é legítimo). Vision transcreve, regex decide.
4. Reprovada → regeneração ÚNICA.
5. Regen veio queimada de novo → entra pronta que sirva. Nunca há 2ª regeneração.
