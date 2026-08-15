# Análise da pasta do Piter e adaptação segura ao Vidmator

Data: 15/08/2026
Escopo: `C:\Users\lenovo\Downloads\Pasta do Piter` + contraste com o E2E `tigre_bengala` e o pipeline canônico atual.

## Conclusão executiva

Há respostas úteis e reaproveitáveis na arquitetura do Piter, mas o ganho não vem de um único script nem apenas de abrir quatro contas do Flow. O resultado documentado de 1h06 combina cinco decisões:

1. decisão visual por beat antes de buscar ou gerar mídia;
2. uma frota de perfis isolados, com um navegador por perfil e fusão por ID;
3. checkpoints por item e por bloco, preservando tudo que já passou;
4. especialistas com propriedade exclusiva sobre cada tipo de decisão;
5. render em blocos retomáveis.

Nosso Vidmator já possui parte importante dessas garantias: pipeline canônico único, workspaces por job, lock do sourcing, manifestos de ramo, ledger VEO por cena, contabilidade física, uma coleção por vídeo, gates e render em quatro chunks retomáveis. Não devemos substituir isso.

O maior ganho seguro é adaptar a frota ao `veo_gerar.py` que já existe, como configuração do mesmo funcionário VEO. Não deve nascer `veo_paralelo.py` como segundo executor operacional.

## Estado da adaptação em 15/08

### Implementado e validado offline

- `verificador_reuso.py` deixou de escrever `apres:*`: ele pode trocar somente a
  mídia por cópia segura e preserva a roupa editorial escolhida pelo Maestro.
- `Texto05_BoxedKicker` e `Texto07_StampImpact` agora têm teto cognitivo de 3,0 s.
  O teto também reconhece a família `Texto`, impedindo que um domínio cadastrado
  incorretamente como `overlay` faça um componente de tela cheia escapar.
- `veo_frota.py` foi integrado ao mesmo `veo_gerar.py`: N=1 mantém a execução atual;
  N>1 divide o lote por `scene_id`, fixa personagem/avatar no perfil-casa, usa pasta
  privada por worker e consolida somente o arquivo declarado.
- O lock do ciclo, a porta CDP e a limpeza do Chrome passaram a ser por perfil. Um
  worker não pode navegar ou matar o navegador de outro.
- A configuração canônica declara apenas o perfil atual, já validado, na porta 9223.
  Nenhuma conta adicional foi ativada por presunção.

Bancadas verdes: compilação dos módulos alterados, distribuição 40/4 = 10 por
worker, afinidade do perfil-casa, validação do registro vivo, receita canônica e E2E
offline com provedores dublês. Nenhuma chamada real ao Flow e nenhum render foram
disparados nesta adaptação.

### Ainda pendente antes de ativar N>1

- login e registro explícitos de cada conta/perfil adicional, com projeto e porta
  CDP exclusivos;
- ensaio controlado de dois perfis com lote pequeno e telemetria por worker;
- transformar os misses do banco em anexos da mesma fila viva, eliminando a segunda
  execução completa de pedido/entrega/curadoria;
- prefilter CLIP e alocação por resolubilidade; cache do preparo e heartbeat de chunk.

Estado observado dos perfis: `chrome_profile` e `chrome_profile_conta1` possuem
sessão detectada; `conta2` a `conta6` ainda não. Somente `chrome_profile` possui hoje
canal/projeto/coleção contratados no registro vivo e, por isso, é o único worker
habilitado. Sessão detectada não basta para ativar `conta1`: ainda é necessário
confirmar a conta correta, criar/registrar seu projeto e fixar uma porta CDP própria.

## Qualidade da evidência

### Fatos diretamente documentados

- O caso BC2 declara vídeo final de 26min43s, 72 necessidades visuais, seis blocos em três workers, montagem do corpo em 10min45s, retomada final em 12min15s e ciclo completo com correções ao vivo em 1h06 (`HANDOFF_VIDMATOR.md`, linhas 162–183).
- A frota divide itens em round-robin, usa um driver por perfil, proíbe dois drivers no mesmo perfil e estima vazão aproximadamente proporcional ao número de perfis porque a espera remota domina a CPU (`docs/FROTA.md`, linhas 73–87).
- A decisão operacional é usar a frota inteira para um canal por vez, com fila em disco, lock global, afinidade do avatar ao perfil-casa e fusão por ID (`docs/FROTA.md`, linhas 135–151).
- O sourcing V5 reúne metadados de várias fontes sem download, faz um gate Vision em lote e só baixa o vencedor (`docs/V5_ESTADO.md`, linhas 33–42).
- O Diretor define estratégia, tipo de mídia e fallback antes da resolução (`docs/ARQUITETURA_DIRETOR.md`, linhas 19–23 e 48–88).
- O Animador é o único dono de orçamento de texto, quotas e reuso; quando falta material, ele reporta o buraco ao responsável em vez de improvisar (`docs/ARQUITETURA_FUNCIONARIOS.md`, linhas 25–37).

### Limitação da prova

A pasta entregue contém documentação e um grafo exportado, não o código completo nem os logs brutos da execução BC2. Portanto, 1h06 é uma medição documentada, mas não reproduzível apenas com esse pacote. Há checkouts locais antigos do Piter que confirmam parte do desenho, porém o grafo fornecido descreve uma evolução mais recente (`frota_fila.py`) que não está integralmente naquele checkout.

Não é correto prometer que copiar a frota produzirá imediatamente o mesmo tempo. Hardware, quantidade de necessidades visuais, contas, créditos, UI do Flow, componentes e rigor dos gates diferem.

## Comparação justa com o Tigre

| Medição | Tigre-de-bengala | Piter BC2 |
|---|---:|---:|
| Duração do vídeo | 8min19s | 26min43s |
| Janela total observada | 2h57min15s | 1h06 documentado |
| Caminho final preservado do Tigre | 1h53min08s | não separado no pacote |
| Necessidades externas | 99 cenas de sourcing; 86 atendidas pelo VEO no final | 72 necessidades visuais |
| Sourcing | 1h03min16s | não discriminado |
| Render | 22min30s | corpo ~10min45s com 3 workers |

As 2h57 do Tigre incluem 1h04 de diagnóstico, correções e esperas que não pertencem ao caminho final. Isso continua sendo inaceitável para produção autônoma, mas a base comparável para otimização é 1h53, não três horas de trabalho inevitável.

O gargalo confirmado do Tigre foi o sourcing: 63min16s. O banco terminou em 3min18s, resolveu somente 13/40 e empurrou 27 cenas para uma segunda passagem VEO. Essa segunda passagem repetiu autoria de prompts, casamento, entrega e curadoria. O render técnico passou na primeira tentativa.

## O que o Piter esclarece sobre nossos problemas

### 1. A alocação atual decide por proporção, não por probabilidade real

O alocador importado fixa, por padrão, 40% banco, 36% VEO vídeo e 24% VEO imagem (`_import/2026-08-06-veo-flow/veo_alocacao.py`, linhas 4–5 e 38). No Tigre isso atribuiu 40 cenas ao banco, mas apenas 13 eram resolvíveis sob os gates atuais.

**Causa raiz confirmada:** a cota de 40% mede composição desejada, não a capacidade do banco para cada beat. A cobertura de 32,5% não foi causada por falta de tempo: o ramo terminou muito antes do teto de nove minutos.

**Prevenção:** o plano visual precisa classificar cada beat por `external_video`, `external_image`, `self_rendered` e cadeia de fallback antes de alocar. A composição desejada vira restrição do Diretor, não seleção cega de cenas.

### 2. Nosso VEO possui uma cadeia única, mas apenas um perfil

`director/veo_gerar.py` chama pedido → ciclo → entrega → socorro → curador. O ciclo atual abre o perfil pinado do canal (`_import/.../veo_ciclo.py`, linhas 270–274). Ele já usa uma coleção por vídeo e um ledger compartilhado por cena.

**Causa raiz confirmada do tempo:** 86 cenas acabaram atendidas pelo VEO; um único perfil submeteu e colheu os lotes. Além disso, os 27 misses do banco abriram um segundo consumidor completo em `_rodar_paralelo.py`, linha 537.

**Adaptação:** o mesmo `veo_gerar.py` deve delegar itens a um executor de frota configurável. Com um perfil, o comportamento continua equivalente ao atual. Com N perfis saudáveis, o lote é fragmentado em round-robin, cada shard recebe coleção/output/lease próprios e todos retornam ao mesmo entregador por `scene_id` e `generation_id`.

### 3. O segundo VEO precisa virar uma fila viva, não outra passada

Hoje o VEO inicial e o banco começam em paralelo. Quando o banco encerra com misses, `_rodar_paralelo.py` chama `veo_gerar.py` outra vez. No Tigre isso também zerou o prompt-store por fingerprint diferente, permitiu prompts duplicados e reinspecionou takes já aprovados.

**Correção arquitetural proposta:** uma única fila VEO permanece aberta durante a janela do banco. Os itens originalmente VEO entram imediatamente. Ao final do banco, somente os misses são anexados à mesma fila/coleção/job. Entrega, socorro e curador rodam uma vez, quando produtores e banco fecharem. Não existe “VEO inicial” e “R4 VEO” como duas receitas.

### 4. Já fazemos parte do sourcing eficiente; falta a peneira local

Nosso resolver já possui busca multibanco, gate de posters em lote e no máximo um download final materializado por cena. Isso não deve ser refeito.

O ponto novo útil do Piter é o CLIP local antes do Vision: ele documenta 19 candidatos reduzidos a 8 em seis segundos. CLIP decide afinidade semântica; Vision continua sendo o único gate de segurança/qualidade.

**Adaptação:** inserir um prefilter local opcional entre coleta de metadados/posters e `POSTER_BATCH`. Medir recall contra o golden e nunca substituir vetos Vision. Mais threads não são a solução: o próprio Piter mediu degradação ao duplicar workers por contenção de APIs.

### 5. Nosso pipeline ainda busca mídia antes de terminar a edição

No pipeline canônico, `source_parallel` ocorre nas linhas 63–64; decisões editoriais como mapas, pessoas, efeitos, ilustrações, apresentações e edição vêm nas linhas 67–88. O Piter planeja estratégia/tipo/fallback antes da resolução.

**Risco atual:** produzimos mídia externa para cenas que depois podem virar elemento autoral, apresentação, collage, mapa ou outra composição. Isso aumenta o número de pedidos e obriga reconciliações posteriores.

**Adaptação:** antecipar somente o `VisualIntentPlan` necessário ao sourcing. Não mover todos os escritores editoriais para antes da mídia. O plano declara necessidade e fallback; o compilador editorial único continua resolvendo duração, exclusividade e frequência depois.

### 6. O problema dos elementos repetidos confirma falta de proprietário exclusivo

A decupagem do Tigre acusou `apres:spotlight` 14 vezes. Treze foram adicionadas pelo `verificador_reuso`; só uma era decisão original da edição. Um validador alterou linguagem editorial para resolver reuso.

Isso viola exatamente a regra útil dos três funcionários do Piter: o verificador deveria reportar o conflito, e somente o Maestro/compilador deveria escolher tratamento.

**Correção de raiz implementada:** `verificador_reuso` pode escolher mídia/cópia segura,
mas não atribui mais `apres:*`, texto, efeito ou duração. Se não houver doador legal,
ele reprova e entrega o conflito ao gate; não improvisa linguagem editorial.

Os dois prints enviados também foram identificados no plano final:

- `Texto05_BoxedKicker`, “LESS THAN TWENTY METRES”: início 16,75s, duração 9,21s — defeito temporal real.
- `Texto07_StampImpact`, “A HUNDRED METRES”: início 154,94s, duração 3,47s — não viola o teto global atual, mas excede a janela estética desejada pelo operador para essa variante.

Ambos receberam duração contratual de 3,0 s por componente. A âncora da fala pode
escolher o momento; não pode alongar o componente além de seu teto visual.

### 7. Render: copiar três workers agora seria regressão

Nosso renderer já produz quatro chunks atômicos e retomáveis (`remotion/render-broll.mjs`, linhas 49–111) e o Tigre concluiu em 22min30s na primeira tentativa.

No laptop atual, dois workers concorrentes já foram medidos como 6–10 vezes mais lentos no pre-compose, com risco de OOM e contenção do único NVDEC. Portanto:

- manter `render_workers=1` no laptop;
- preservar quatro chunks retomáveis;
- adicionar scheduler N-worker como configuração da mesma rota, desligado por padrão;
- habilitar 2/3 workers somente no servidor novo após benchmark solo × pareado e gates idênticos.

O reaproveitamento seguro imediato é cachear normalizações por hash+spec: a primeira preparação do Tigre normalizou 60 entradas e, após um gate falhar, a segunda repetiu o trabalho.

## Arquitetura-alvo sem segunda fonte da verdade

```text
Pipeline canônico
  → VisualIntentPlan (tipo de necessidade + fallback; imutável para sourcing)
  → Banco e fila VEO única
       Banco: coleta sem download → CLIP opcional → Vision → 1 vencedor
       VEO: fila persistente → leases por item → N perfis configurados
       Miss do banco: anexado à mesma fila VEO, sem reiniciar a receita
  → entrega única por scene_id/generation_id
  → Maestro/compilador editorial único
       resolve texto, apresentação, duração, frequência, transição e exclusividade
  → validadores somente leitores
  → EditPlan assinado
  → preparo/cache → preflight → render em chunks → fiscal
```

Diretores continuam mudando política: frações desejadas, componentes banidos, linguagem visual, uso de avatar e fallback. Não ganham executor próprio.

## Ordem de implementação proposta antes do próximo E2E

### Bloco A — invariantes e regressões

1. Congelar métricas do Tigre e golden vigente.
2. Teste que proíbe qualquer chamador operacional de VEO fora de `source_parallel → veo_gerar`.
3. Teste que validadores não escrevem `apres`, `acervo_texto`, duração ou transição.
4. Contratos temporais de `Texto05_BoxedKicker` e `Texto07_StampImpact`.

### Bloco B — uma única fila VEO

1. Criar registry de perfis como dados, com owner, profile path, conta, saúde, login, capacidade e perfil-casa.
2. Criar lease atômico por `scene_id/generation_id/profile_id` no estado do job.
3. Generalizar `veo_gerar` para N produtores; N=1 preserva o comportamento atual.
4. Shards round-robin e diretórios/coleções isolados por perfil.
5. Fusão somente por IDs; nunca por ordem, nome de arquivo ou prompt.
6. Refileirar apenas leases falhos; não repetir shards aprovados.
7. Fechar entrega/socorro/curador uma vez.

### Bloco C — eliminar desperdício do banco

1. Substituir a cota cega por score de resolubilidade do beat.
2. Coletar candidatos sem download.
3. Adicionar prefilter CLIP local, com feature flag e medição de recall.
4. Manter Vision e um download final por cena.
5. Usar até 8–9 minutos somente quando há progresso/candidatos; ausência estrutural termina cedo.
6. Anexar misses à fila VEO viva, sem uma segunda receita.

### Bloco D — propriedade editorial única

1. Remover escolha de `spotlight` do `verificador_reuso`.
2. Representar “precisa diferenciação” como pedido declarativo.
3. Fazer o compilador resolver apresentação/frequência/cooldown uma vez.
4. Recalibrar F3: o operador aprovou quatro elementos no primeiro minuto; o piso automático de seis não pode reprovar um vídeo visualmente aprovado sem evidência de defeito.

### Bloco E — preparo e render

1. Cache de normalização por conteúdo+spec.
2. Heartbeat baseado em progresso do chunk, não em linhas com newline.
3. Scheduler de chunks parametrizável, default 1.
4. Benchmark no servidor: 1, 2 e 3 workers; promover somente o vencedor estável.

## Metas mensuráveis para o próximo canário

| Área | Critério mínimo |
|---|---|
| Caminho | uma única entrada canônica e uma única fila VEO |
| Banco | ≤9min; ≥90% somente entre cenas classificadas como realmente bancáveis |
| Download | no máximo um arquivo final materializado por cena |
| VEO | nenhum perfil com dois drivers; nenhum shard aprovado repetido |
| Recuperação | falha de perfil redistribui só seus leases pendentes |
| Curadoria | cada asset novo inspecionado uma vez |
| Editorial | validadores não escolhem elemento; BoxedKicker/StampImpact dentro do contrato |
| Preparo | normalização aprovada reaproveitada após retry |
| Render | primeira tentativa técnica; chunks retomáveis; worker count medido |
| Telemetria | wall time por etapa, tempo de espera remota, trabalho repetido e perfil produtor |

## Projeção — hipótese, não promessa

Com quatro perfis VEO saudáveis, fila única e sem segunda curadoria, é plausível reduzir o sourcing de 63 minutos para algo na faixa de 15–25 minutos. No laptop, mantendo render de 22–23 minutos, um próximo E2E limpo pode se aproximar de 50–70 minutos. No servidor, se o benchmark autorizar chunks concorrentes, a faixa pode cair mais.

Essas faixas precisam ser provadas por telemetria. O objetivo da implementação não é reproduzir cegamente “1h06”, mas tornar cada minuto explicável, retomável e herdado pela produção oficial.

## Grafo da análise

- `D:\AC Creator HUB\Vidmator\_analise_piter\graphify-out\graph.html`
- `D:\AC Creator HUB\Vidmator\_analise_piter\graphify-out\graph.json`
- `D:\AC Creator HUB\Vidmator\_analise_piter\graphify-out\GRAPH_REPORT.md`

Saúde do grafo: 77 nós, 76 arestas, 11 comunidades; zero endpoints ausentes ou pendentes. Duas relações direcionais colapsam no modo não direcionado e estão registradas como aviso de integridade, sem perda de nós. Extração sem API: custo registrado de 0 tokens de provedor.
