# CATÁLOGO DE BLINDAGEM — erros reais da produção autônoma (para a integração no Automator)

> **Ordem do operador (05/08, verbatim):** *"esses erros que forem dando, temos que
> catalogar todo o log, pq quando implementarmos no automator, temos que estar blindados
> contra esses erros, e caso aconteça, ele tem que conseguir se corrigir sozinho, para não
> cairmos no que aconteceu no início do automator... te dei o GO, deixei gerando, saí,
> achei que quando eu chegasse os dois vídeos estariam prontos, cheguei e não tinha nenhum."*
>
> Este arquivo é o inventário VIVO: cada erro observado em produção real entra aqui com
> (a) como detectar, (b) como o sistema se corrige SOZINHO, (c) status do fix.
> O macro-princípio é um só: **processo vivo ≠ processo saudável. Todo passo longo precisa
> de batimento cardíaco mensurável, e silêncio prolongado é FALHA, não normalidade.**

---

## Incidente-mestre (05/08): "deixei gerando, voltei, nada pronto"

GO dado ~17:32. Runner rodou 106 min SEM travar e SEM produzir — lento e 100% cego
(stdout capturado, zero visibilidade). Operador voltou e nenhum vídeo existia. Composto
pelos erros E1+E3+E4 abaixo. É a reedição exata do trauma original do Automator.

---

## E1 · Gate de Vision PERMISSIVO — otimização que vira overhead

**O que aconteceu:** a 1ª versão do poster-gate aprovava quando o sujeito "PODE aparecer
no vídeo". O TOP-1 de reprovação real é `missing_subject` (208/341): o gate aprovava
exatamente o que o vet caro reprovaria. Resultado: +10-15s por query SEM economizar nada.
**Detecção:** taxa de corte do gate ≈ 0 em N queries seguidas + tempo/cena acima do baseline.
**Auto-correção:** kill-switch automático — gate com corte <10% em 20 lotes se DESLIGA
sozinho e loga (env `VISION_POSTER_GATE=0` já existe como mecanismo).
**Status:** prompt corrigido para DECISIVO (commit 9af73f3); kill-switch automático PENDENTE.

## E2 · Mesma query re-julgada N vezes (sem cache de veredicto) — AJUSTE FUTURO DO OPERADOR

**O que aconteceu:** "ringed seal arctic polar region" foi julgada pelo gate por 6+ cenas
diferentes, e DE NOVO na rodada de fotos de cada uma. Cada re-julgamento ≈ 6-9s + 4
downloads de poster.
**Ordem do operador (verbatim):** *"se ele já refutou query, ele já sabe o que tem nela,
então não deve pegar ela se ela CLARAMENTE NÃO FOR SERVIR."*
**Auto-correção:** cache de veredicto por `(query, kind, hash_do_contrato)` — query com
0 aprovados no gate NÃO re-entra na fila de nenhuma cena com o mesmo contrato; TTL por
rodada (a oferta do Pexels muda pouco numa rodada).
**Status:** ✅ CORRIGIDO (05/08, mesma noite — promovido de 'futuro' a 'agora' pelo operador): `gate_query_morta`/`gate_registrar` — query com corte total não re-busca nem re-julga.

## E3 · Cegueira operacional — capture_output sem streaming

**O que aconteceu:** `producao.run` capturava TODO o stdout; 106 min sem UMA linha
visível; diagnóstico só foi possível por forense externa (mtime de arquivos, processos).
**Auto-correção:** logs SEMPRE streamados para arquivo (`VM_LIVE_LOG`), e o supervisor
lê o tail — nunca mais um passo longo mudo. No Automator: mesmos logs viram o feed do
monitor/Telegram.
**Status:** ✅ CORRIGIDO (producao.run + VM_LIVE_LOG, commit 9af73f3).

## E4 · Processo vivo, progresso morto — sem watchdog de PROGRESSO

**O que aconteceu:** o resolver tinha timeout de 5h (proporcional ao roteiro) mas nenhum
batimento intermediário: 106 min "saudável" aos olhos do runner, improdutivo na real.
**Auto-correção:** heartbeat por unidade de trabalho — se NENHUMA cena resolve em X min
(ex.: 10), o supervisor age sozinho: loga diagnóstico, tenta degradação (desliga gate,
reduz fila) e só então aborta com estado salvo. Timeout total é a ÚLTIMA linha, não a única.
**Status:** ⏳ PENDENTE (essencial para o Automator).

## E5 · Exaustão de oferta — grind ilimitado quando o estoque bom acaba

**O que aconteceu:** o pool consumiu os 18 takes bons de urso/foca; as cenas restantes
garimparam o RESTO do Pexels (zoo, pedra, aquário) — corretamente reprovado, mas em loop:
cortes de poster NÃO contam no CAP de julgados, então a cena varre a fila inteira de
variantes (10+ lotes ≈ 2-4 min/cena) antes do hard-miss.
**Detecção:** N lotes seguidos com corte 4/4 na mesma cena.
**Auto-correção:** (a) CAP de lotes de poster por identidade (ex.: 6); (b) curto-circuito:
3 lotes 4/4 consecutivos → declara exaustão de oferta → hard-miss HONESTO imediato (a
degradação digna já existe: atmosfera + moldura de herança). Falhar rápido é qualidade.
**Status:** ✅ CORRIGIDO (05/08): teto de 8 lotes de poster por identidade + cache E2 (query morta não conta lote). Exaustão declarada cedo -> hard-miss honesto.

## E6 · Vocabulário de OUTRO bioma vazando pro contrato

**O que aconteceu:** queries "jaguar arctic polar region", "black caiman arctic",
"green anaconda arctic" num documentário do ÁRTICO — membros do vocabulário-semente
(Amazônia) entraram como allowed_members de classe. O gate matou barato, mas nem deveria
ter chegado à busca.
**Auto-correção:** contrato_visual filtra allowed_members pela REGIÃO do documentário
antes de gravar o contrato; membro de bioma incompatível nem vira query.
**Status:** ⏳ PENDENTE (fix no contrato_visual, não no resolver).

## E7 · Buffering de pipe — log de 0 bytes com processo saudável

**O que aconteceu:** `python | tee` sem `-u`: o stdout do Python fica em buffer de bloco;
o arquivo permaneceu 0 bytes por 106 min e o monitor não tinha o que ler.
**Auto-correção:** `python -u` + redirect direto para arquivo (sem pipe intermediário).
Regra: ferramenta de observação NUNCA depende de flush de terceiros.
**Status:** ✅ CORRIGIDO (relançamento com -u + VM_LIVE_LOG).

## E8 · Mesmo vídeo-fonte baixado dezenas de vezes sob nomes diferentes

**O que aconteceu:** registro do cache mostra o MESMO pexels_id baixado até **42×** —
o nome do arquivo é hash(query+id), então cada query nova re-baixa o mesmo MP4 de 20MB.
**Auto-correção:** cache canônico por `pexels_id` (arquivo único; nomes por query viram
links/aliases). Economia de banda E de vet (decisão por conteúdo, não por caminho).
**Status:** ✅ CORRIGIDO (05/08, ordem do operador: 'isso não pode acontecer NUNCA'): cache canônico `_cache_stock/pex/{id}` no pexels_download — id existente vira cópia local, zero rede.

## E9 · Pool com fontes duplicadas → re-resolve pós-hoc

**O que aconteceu (rodada atual):** cena resolvida pelo pool foi RE-resolvida depois —
dois arquivos do pool continham o MESMO vídeo-fonte (consequência direta do E8), e o
orçamento de repetição (por pexels_id) detectou e mandou substituir.
**Auto-correção:** dedup do pool POR pexels_id/file_hash na construção (_montar_pool_takes)
— dois arquivos com a mesma fonte = 1 entrada.
**Status:** ✅ CORRIGIDO (05/08): `_dedup_conteudo` no _montar_pool_takes (por pexels_id + file_hash); pools regenerados (predadores 63->51: 12 fontes duplicadas).

## E11 · Job morre junto com a sessão do agente (processo FILHO do shell)

**O que aconteceu (05/08, 2× na mesma noite):** o runner foi lançado como background do
shell do agente. Quando a sessão do Claude reiniciou/compactou, o shell morreu e **levou o
render junto** — 35 min de resolver perdidos, sem UM aviso. É o mesmo padrão já documentado
no Automator (memória `servidor_restart_scheduled_task`): processo lançado pelo agente é
filho do agente.
**Detecção:** processo do runner some sem linha de conclusão no log; live.log para de crescer.
**Auto-correção:** trabalho longo NUNCA nasce filho do agente/terminal — sempre
**Scheduled Task** (ou serviço). Aqui: `_render_v2.bat` + task `Vidmator Render v2`
(`Start-ScheduledTask`), log em D: (não em %TEMP%, que é volátil e virtualizado).
No Automator isso já é lei para o servidor; passa a ser lei para QUALQUER job de produção.
**Status:** ✅ CORRIGIDO (05/08): render relançado como Scheduled Task; regra vira Z8 do zelador.

## E12 · Aprendizado do gate morria no restart (cache só em memória)

**Pergunta do operador que expôs (05/08):** *"agora que reiniciou ele já sabe o que
recusou para não pegar mais ele e ter aquele ganho de 40%?"* — **não sabia.** O cache de
veredictos do E2 era um dict de processo; todo restart (E11) zerava o aprendizado.
**Auto-correção:** persistência em `_cache_stock/gate_veredictos.json` (TTL 7 dias, grava
em lotes de 10 + flush no fim do resolver). O que já foi refutado NÃO volta pra fila — nem
depois de restart, nem no job seguinte, nem amanhã.
**Status:** ✅ CORRIGIDO (05/08). Testado com restart simulado: veredicto sobrevive ao
processo novo; resultado MISTO continua fora do cache (não generaliza para a query).

## E13 · Knob que não está no .bat NÃO EXISTE no job real

**O que aconteceu (05/08, o erro mais caro da noite):** ao migrar o runner para Scheduled
Task (fix do E11) eu reescrevi a linha de comando e **esqueci `VM_LIVE_LOG`** — o job
voltou a rodar CEGO (E3 de volta, 10 min depois de "corrigido"). Pior: passei a analisar
o `live.log` da rodada MORTA achando que era da viva, e conclui que o E2 estava quebrado
quando na verdade a rodada analisada era ANTERIOR ao commit do E2.
**Lição dupla:** (a) trocar o mecanismo de lançamento re-abre TODOS os buracos de
ambiente; (b) **antes de concluir qualquer coisa de um log, conferir o mtime dele contra o
início do processo** — log velho é a pior fonte de diagnóstico, porque parece verdade.
**Auto-correção:** o `.bat` é o contrato de execução — todo knob (VM_LIVE_LOG, NICHO,
RENDER_CONCURRENCY, PYTHONUNBUFFERED) vive ali, e o zelador sobe junto no mesmo arquivo.
No Automator: um único ponto de verdade para o env do job, versionado no repo.
**Status:** ✅ CORRIGIDO (05/08): `_render_v2.bat` com env completo + zelador embutido.

## E14 · Rótulo de reuso construído a partir de `allowed_members` (semente ≠ conteúdo)

**O que aconteceu (05/08):** o pool dos PREDADORES (doc marinho) apareceu com `jaguar`,
`black caiman`, `green anaconda`. **O arquivo estava certo** — cena 13, query
`Orca hunting prey`, footage de recife, aprovada no vet completo (`ID-vid`,
`auto_accepted`). Errado era o RÓTULO: eu montei os "sujeitos" do pool somando
`canonical` + `allowed_members`, e o contrato dessa cena (canonical `predator`,
`class_member`) trazia fauna amazônica como membro — sintoma do **E6**, anterior ao pool.
**Por que é grave apesar do arquivo certo:** o rótulo é a CHAVE do reuso com zero Vision.
Rótulo mentindo = take entregue para a cena errada em outro roteiro, sem nada que pegue.
**Auto-correção:** rótulo = `canonical` APENAS (allowed_members é semente de busca, o
próprio resolver a declara "NOT an exhaustive whitelist" — nunca foi afirmação sobre o
conteúdo) + validador `_rotulo_valido` (4-40 chars, ≤3 palavras, sem fragmento verbal,
stoplist de genéricos). Medidos e barrados: `be`, `barely been revised`, `take sea lions`,
`water`, `animal`.
**Status:** ✅ CORRIGIDO (05/08). Pools regenerados: predadores agora só fauna marinha
(great white shark, killer whale, sperm whale, tiger shark, saltwater crocodile, seal);
urso só polar bear + seal. **Origem upstream (E6) segue PENDENTE** — o contrato_visual
continua podendo gerar membro de outro bioma e canonical-lixo.

## E15 · Heartbeat media CONCLUSÃO, não TRABALHO — watchdog matou pass saudável

**O que aconteceu (05/08 21:12, primeiro incidente REAL do zelador):** o resolver estava
trabalhando normalmente (garimpando as últimas cenas de urso/foca, ~7s por lote de
poster), mas o batimento só era emitido quando uma CENA FECHAVA. No fim do pass, com 8
workers todos no grind da cauda, ficaram 10 min sem nenhuma cena concluir — o zelador leu
silêncio e **matou um resolver saudável** aos 802s. Evidência no próprio log do zelador:
as 6 últimas linhas eram cortes de poster ACONTECENDO no momento da morte.
**A lição, que vale para qualquer watchdog:** progresso é **trabalho feito**, não entrega
concluída. Um heartbeat de granularidade grossa transforma "devagar" em "morto".
**Auto-correção:** `_bater_trabalho()` global no resolver — pulso a cada lote de poster e
a cada vet completo, não só a cada cena. Limites afrouxados junto (resolver 15, render 20,
narração 25) porque agora o pulso é fino e o limite não precisa cobrir a cauda.
**Status:** ✅ CORRIGIDO (05/08). Custo do incidente: 1 tentativa de resolver (13 min) —
mas a tentativa seguinte recuperou **40 veredictos do disco** (E12 funcionando), então o
retry saiu mais barato que o original.

## E16 · Anti-repetição REATIVA custa 5,6× o trabalho principal (decisão: VEO cobre o vão)

**Medido no urso v2 (05/08, tentativa 2 do resolver):**
```
LOOP das 82 cenas ..........   174s  ( 2,9 min)
PÓS-LOOP anti-repetição ....   973s  (16,2 min)  = 85% do pass
```
10 cenas violaram o orçamento; cada re-resolve refaz busca + poster-gate + download + vet
**até 4×** (39 lotes de gate só nessa fase). Resultado: **7 substituições e 3 repetições
mantidas** — 16 min gastos e o defeito persistindo em 3 cenas.

**Por que é caro por construção:** o desenho é REATIVO — deixa a colisão acontecer e
tenta desfazer depois, competindo consigo mesmo pelo estoque escasso que o próprio loop
já consumiu. Em bioma pobre de footage (Ártico) a busca simplesmente não tem o que achar.

**DECISÃO DO OPERADOR (05/08, implementar amanhã — NÃO implementado):** o que falhar por
repetição passa a ser **coberto por imagem/take GERADO no VEO**, por uma automação
autônoma. *"o tempo para gerar essas imagens e vídeos no VEO não levaria 5 min"* contra os
16 min de garimpo que ainda entrega repetição. Inverte a lógica: em vez de procurar mais
fundo num estoque que acabou, **cria** o material que falta.
**EVIDÊNCIA DE SUPORTE (predadores v2, testada à mão 05/08):** o 2º gatilho do VEO não é
repetição — é **HARD-MISS DE ESPÉCIE EXATA**. Cena 52 pede `blue whale` (exact) e a
narração diz "Blue whales are bigger and eat krill"; o Pexels devolveu 4 candidatos e o
gate reprovou os 4. Conferido por imagem: **são baleias reais, mas JUBARTES** (tubérculos
na cabeça), não baleias-azuis. O gate está CERTO — mostrar jubarte falando de baleia-azul
é erro factual. Espécie rara filmada quase só por pesquisador não existe em stock grátis:
9 hard-miss nos predadores contra 2 no urso, e ZERO por rigor do gate (48% dos lotes
deixam candidato passar). Nenhuma busca melhor resolve um vazio de acervo.

**Status:** 🔵 DECIDIDO, NÃO IMPLEMENTADO. Escopo p/ amanhã: DOIS gatilhos — (a) `pool esgotado, mantendo repetição` e (b) hard-miss de
espécie exata, geração VEO, ingestão no cache com
`pexels_id` sintético para o orçamento contabilizar, e vet do gerado antes de entrar.

## E17 · Supervisor com a IDENTIDADE do supervisionado HARDCODED

**O que aconteceu (06/08, 2º incidente real do zelador):** o `runner_vivo()` checava a
Scheduled Task **'Vidmator Render v2'** — nome fixo no código. Quando o job **v3** subiu,
o zelador olhou a task ERRADA (v2, parada = `Ready`), concluiu *"runner morreu"* e
**relançou o job antigo**. Resultado: dois pipelines completos escrevendo no MESMO
`timeline.json`, e o v2 chegou a rodar `montar_timeline` por cima do timeline resolvido
que o v3 estava usando. Pior: cada relançamento subia um zelador novo, também apontado
para v2 — três zeladores ativos, todos relançando.

**Por que passou na vistoria:** os testes do loop usaram processos-cobaia com PID; a
identidade da TASK nunca foi exercitada porque só existia UMA task no mundo do teste.
**Classe de bug:** acoplamento por nome — o supervisor sabendo quem supervisiona por
literal em vez de por parâmetro.

**Auto-correção:** o nome vem do env **`VM_TASK`** (o `.bat` que lança é o único que sabe
o próprio nome — coerente com o E13: o .bat é o contrato de execução). E **sem `VM_TASK`,
o Z8 é DESLIGADO** em vez de adivinhar: não relançar é sempre mais seguro que relançar o
alvo errado.
**Status:** ✅ CORRIGIDO (06/08). Ambos os `.bat` passaram a exportar `VM_TASK`.

## E18 · Workspace ÚNICO e compartilhado — job concorrente troca a narração do outro

**O que aconteceu (06/08, consequência direta do E17):** o zelador relançou o job antigo,
que copiou `narracao.mp3` / `roteiro_en.txt` / `words.json` do **URSO** por cima do
workspace enquanto o job dos **PREDADORES** rodava. O predadores seguiu com o roteiro
errado — a `abertura` chegou a resolver **'ARCTIC' num documentário de oceano**, e o render
teria saído com a **narração do urso**. É exatamente o incidente que arruinou o 1º urso
(áudio de outro vídeo), reencenado por outra porta.

**Fato que salva o diagnóstico:** o hash denunciou na hora — `narracao.mp3` estava
`297208468cc2` (urso) quando o job exigia `97d1692f0019` (predadores). Parado antes do
render; **nenhum vídeo contaminado foi produzido**.

**Causa estrutural:** `_workspace/` é um diretório ÚNICO. Todo job escreve nos MESMOS
nomes de arquivo. Não existe isolamento — dois jobs simultâneos são incompatíveis por
construção, e nada avisava.

**Auto-correção (implementada):** `_conferir_identidade()` no runner — tira a impressão
digital de narração/roteiro/words ao ENTRAR no job e reconfere **imediatamente antes do
`preparar`**. Divergiu = **ABORTA**. Render de 20 min com áudio errado é muito pior que
job interrompido, e a checagem custa milissegundos.
**Correção estrutural PENDENTE (para o Automator):** workspace POR JOB (`_workspace/<job>/`)
em vez de diretório único. Enquanto não existir, a guarda de identidade é o que segura.
**Status:** ✅ GUARDA ATIVA (06/08) · ⏳ isolamento por job pendente.

## E19 · Passe que NÃO LIMPA a própria saída — re-execução SOMA em cima da anterior

**O que aconteceu (06/08):** o coordenador (`edicao.py`) grava `overlay_take` nas cenas. O
timeline resolvido dos predadores ainda trazia **4 desses campos do run v3**, e o passe
novo simplesmente ACRESCENTOU os dele: resultado `quadro_filme` 3× (teto era 2) e o rodízio
saiu `b,q,q,f,b,q,f` — quadro repetido em sequência, exatamente o que o operador proibiu.

**Como foi visto:** contando os overlays do timeline pós-`edicao` e comparando com o log do
próprio passe. O log dizia uma coisa, o arquivo tinha outra — a diferença era o resíduo.

**Causa raiz:** todo o resto do cronograma é reescrito do zero na seção de ESCRITA (a
apresentação, a ilustração, o texto_impacto e o infográfico são REMOVIDOS quando não
aceitos, e `acervo_texto` é substituído inteiro). `overlay_take` era o único campo que só
ganhava valor e nunca perdia. **Passe que não limpa a própria saída não é re-executável** —
e este pipeline foi feito para re-rodar sobre timeline já resolvido, que é o caso normal.

**Auto-correção (implementada):** `for c in cenas: c.pop("overlay_take", None)` na entrada
do bloco. **Regra geral para passes novos:** antes de decidir, apagar tudo o que a decisão
anterior deste mesmo passe escreveu. A verificação barata é rodar o passe 2× no mesmo
arquivo — saída idêntica ou tem resíduo.
**Status:** ✅ CORRIGIDO (06/08).

## E20 · Loop do TTS sobrevive a qualquer re-render porque o áudio é REUSADO

**O que aconteceu:** o operador apontou frase duplicada no áudio na revisão do **v2** e de
novo na do **v3**. A guarda anti-loop foi implementada no `narrar.py` (passo H) e o defeito
**continuou no v5** — porque a guarda age na GERAÇÃO e todo render posterior REUSA o mesmo
MP3. `narracao_urso_polar.mp3` era de **05/08 12:01**; a guarda, de **06/08 09:38**. O
arquivo nunca foi tocado.

**Por que não bastava re-narrar:** áudio novo = palavras novas = fronteiras de cena novas =
re-resolver todos os takes, jogando fora dezenas de cenas já revisadas e aprovadas.

**Auto-correção (implementada):** `director/_deloop_audio.py` faz a CIRURGIA — corta o
trecho repetido do MP3 e desloca `words.json` + `timeline.json` junto, preservando cada
take. Dois detectores, porque são dois defeitos diferentes:
- **ciclo** (k>=3 palavras idênticas coladas): pega `«the largest animal,» ×3`;
- **âncora** (6-grama repetido em <40s, estendido para a frente): pega a cópia quase-igual
  (`«his skin is black...»` vs `«his skin is A black...»`), que o ciclo não vê.

O ciclo roda ANTES: numa sequência periódica a âncora casa índices deslocados e corta no
lugar errado (medido — ela levava embora o `the largest` da frase BOA).

**Armadilhas que custaram tempo:**
- as rodadas medem sobre as palavras SOBREVIVENTES, então devolvem faixas que se contêm
  (`[266, 267]` e `[266, 268.2]`); sem FUNDIR, o `atrim` sai com duração negativa e o
  deslocamento conta o mesmo trecho 2×;
- a emenda usa `atrim` + **`asetpts=N/SR/TB`** + `concat`, nunca `-c copy` — PTS duplicado
  emudece o player do Windows (incidente de 31/07).

**Controle de falso positivo:** rodado nos 3 roteiros ORIGINAIS (texto que o LLM escreveu,
livre de defeito de TTS por construção) → **0 detecções**. Nos 2 áudios com defeito → acerta
os 2. Medido: urso −5,22s, predadores −2,18s; prosa resultante conferida palavra a palavra.
**Status:** ✅ CORRIGIDO (06/08) · falta plugar como passe automático do pipeline.

## E21 · LLM devolve ZERO em silêncio e o vídeo sai com uma família inteira faltando

**O que aconteceu (06/08):** duas rodadas do MESMO roteiro do urso, com minutos de
diferença — uma deu `acervo: 28 momento(s) válidos`, a outra `acervo: 0`. Nada falhou,
nada foi logado como erro: o `detectar_v2` simplesmente devolveu lista vazia e o
coordenador seguiu com o que tinha. Resultado: a família **Overlay saiu de 8 elementos
para ZERO** e o `Texto01_Typewriter` foi chamado 6× para tapar o buraco.

**Por que é grave num batch noturno:** o vídeo termina, o portão não reprova por isso, e
o defeito só aparece quando alguém ASSISTE. É a definição de degradação silenciosa.

**Auto-correção (implementada):** zero num roteiro de 200+ palavras é falha de chamada,
não veredicto — **re-tenta até 3×**. Persistindo, imprime a degradação em vez de deixá-la
invisível. **Regra geral:** todo passe que consome LLM precisa de um piso de plausibilidade
(zero elementos num roteiro longo é implausível) e de retry antes de aceitar o vazio.
**Status:** ✅ CORRIGIDO (06/08).

## E22 · Valor cravado no runner ANULA a configuração do lançador

**O que aconteceu:** o render morreu em `delayRender timeout` a **118s de um teto de 120s**
carregando `mood_overlay.mp4` — com 6 overlays de take (eram 4) + overlay de capítulo + os
takes, a concorrência 8 satura o navegador. Baixei a concorrência no `.bat`… e não mudou
nada: `_rodar_pos_resolver.py` fazia `os.environ["RENDER_CONCURRENCY"] = "8"`, **atribuição
que sobrescreve o ambiente que o `.bat` acabou de montar**.

**Mesma família do E17** (identidade do supervisionado cravada no supervisor). O sintoma é
traiçoeiro: a configuração parece aplicada, o arquivo mostra o valor novo, e o
comportamento é o velho.

**Auto-correção (implementada):** `os.environ.setdefault(...)` em vez de atribuição — o
default continua valendo quando ninguém configura, e o lançador passa a mandar quando
configura. Junto: `RENDER_TIMEOUT_MS` para 300s e concorrência 6.
**Status:** ✅ CORRIGIDO (06/08).

## E23 · Campo nulo do LLM derruba o passe num f-string, DEPOIS do trabalho feito

**O que aconteceu:** o `detectar_mapas` resolveu o mapa inteiro (detectou o lugar, baixou a
imagem) e então morreu em
`TypeError: unsupported format string passed to NoneType.__format__` — porque o LLM
devolveu o lugar **sem `pais`** e a linha de log fazia `{m['pais']:<14}`. O runner degradou
("segue sem esse elemento") e o vídeo saiu **sem o mapa que já estava pronto**.

**A lição, que vale para todo o pipeline:** campo vindo de LLM é opcional por definição.
Formatar direto (`:<14`, `:>6.1f`) transforma um rótulo ausente em queda do passe — e o
prejuízo não é o log, é o TRABALHO já feito que vai junto.

**Auto-correção (implementada):** `{(m['pais'] or '—'):<14}`. **Regra:** nenhum campo de
origem LLM entra em f-string com especificador de formato sem fallback.

**⚠️ Recaída no MESMO dia, e a armadilha vale registrar:** consertei 2 prints e o passe
quebrou de novo em outros 3, escritos como **`m.get('pais', '')`** — que PARECE protegido
e não é: `dict.get(k, default)` só devolve o default quando a chave está **AUSENTE**. O
LLM devolve a chave PRESENTE com valor `None`, o `.get` entrega `None` e o `:<14` estoura
igual. A forma segura é `(m.get(k) or fallback)`.
**Status:** ✅ CORRIGIDO (06/08, 2 rodadas).

## E24 · Marcas por-cena de um passe PODADO viram estado fantasma — e o render CONFIA nelas

**O que aconteceu (06/08, urso v5 final):** 9 segundos de TELA PRETA (3:06→3:15). As
`enumeracoes` originais criaram 21 janelas de word-pop e marcaram `pop_bg_dono/membro`
nas cenas (grupos 1-20). O `edicao` do run final podou para 2 janelas — reescrevendo SÓ
`tl["enumeracoes"]`. As marcas POR CENA ficaram órfãs, incluindo uma cena que era **dona
do grupo 10 e membro do grupo 9 ao mesmo tempo** — e `BrollTest.tsx:793` devolve `null`
para qualquer membro, então a "dona" nunca renderizou a cobertura dela. Três cenas em
sequência devolveram null → preto.

**Agravante:** o portão D5 (frames pretos) rodou com **"0 amostras"** — não olhou nada e
aprovou. Detector que não amostra é detector desligado que finge estar ligado.

**Lições (2):** (a) mesma família do E19 — quem poda uma LISTA tem que limpar as marcas
POR ITEM que a lista gerou; (b) o renderizador nunca deve devolver `null` para cena com
mídia por confiança em metadado — degradar é renderizar normal, não sumir.
**Status:** espec S1 da `ESPEC-AJUSTES-V5.md` — aguardando GO.

## E25 · Calibração por MULTIPLICADOR com arquivos de loudness selvagem

**O que aconteceu:** operador reclamou de SFX altos em 4 pontos dos v5 finais. Auditoria:
todos os multiplicadores ≤0.20 e clampados (exceto UM `volume={0.7}` hardcoded,
BrollTest:1294). O problema real: os ARQUIVOS variam **18 LU** de loudness (ui_click
-10.9 LUFS vs counter_digital -29.3). 0.20 × riser (-12.3) soa ~7× mais alto que
0.20 × counter. A calibração de ouvido do operador foi feita POR ARQUIVO nos 2 primeiros;
os ~15 novos entraram crus com multiplicador genérico.

**Lição:** multiplicador uniforme só funciona sobre arquivos NORMALIZADOS. Calibrar o
acervo uma vez (loudnorm por pico/integrado) e o multiplicador volta a ter significado.
**Status:** espec S2 da `ESPEC-AJUSTES-V5.md` — aguardando GO.

## E10 · Reuso por manifesto chaveado por CENA — morre quando a estrutura muda

**O que aconteceu:** o `production mode` reusa por (idx, hash do texto da cena); o F1
mudou as fronteiras das cenas → 0% de reuso. Sem o pool por SUJEITO, toda a revisão
anterior teria sido jogada fora.
**Auto-correção:** reuso SEMPRE por identidade de CONTEÚDO (sujeito + doc), nunca por
estrutura (cena). Implementado no pool.
**Status:** ✅ CORRIGIDO (pool por sujeito, commit 854afbd).

---

## → Auto-correção SEM operador: ver [ESPEC-ZELADOR-PRODUCAO.md](ESPEC-ZELADOR-PRODUCAO.md)

## Princípios de blindagem para a integração no Automator

1. **Heartbeat mensurável em todo passo longo** (E4) — silêncio de X min = incidente.
2. **Logs streamados sempre** (E3/E7) — o supervisor lê o que o passo faz AGORA.
3. **Falhar rápido quando a oferta acabou** (E5) — degradação honesta cedo > grind tarde.
4. **Cache de veredicto e de conteúdo** (E2/E8/E9) — nunca julgar/baixar o mesmo material 2×.
5. **Otimização com kill-switch automático** (E1) — toda otimização mede o próprio ganho
   e se desliga quando vira custo.
6. **Notificação ativa** — no Automator, os avisos Telegram (ESPEC 07) recebem: início,
   heartbeat-perdido, degradações, hard-miss acima do limiar e conclusão com o veredito
   do portão de decupagem. O operador NUNCA descobre no dia seguinte.
7. **Identidade por conteúdo, não por estrutura** (E10) — reuso sobrevive a refactors.

*Criado 05/08/2026 durante o render final v2 (urso+predadores). Manter vivo: todo erro
novo de produção entra aqui ANTES do fix, com detecção + auto-correção.*
