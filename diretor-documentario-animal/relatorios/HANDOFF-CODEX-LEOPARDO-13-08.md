# Handoff — Leopardo E2E — 13/08/2026

## Sessão 14/08/2026 — Leopardo é o golden vigente; próximo E2E especificado

- Decisão explícita do operador: `_saida/leopardo_neves.mp4` substitui a Onça como
  golden visual canônico. O único ponteiro operacional é
  `director/golden/current.json`; o manifesto imutável é
  `director/golden/leopardo_neves_14_08.json`. A Onça permanece somente como
  fixture histórica.
- O MP4 não foi alterado: 322.331.302 bytes, SHA-256
  `d7ceb7af126a37f0d514c8bb2597367cb33064fe33d1181d30d02ab440d4cfe9`.
- As qualidades aprovadas do Leopardo são invariantes positivas. Ovl15 aplicado a
  quantidades, contador persistente, takes excessivamente longos e cauda preta são
  restrições negativas: não devem ser copiadas pelo próximo canário.
- Banco: causa confirmada dos 2,4 min foi multiplicidade de limites locais, não o
  teto global. `director/bank_policy.py` agora é o dono único: 9 min totais, 7 min
  por cena paralela, dez ondas, um download final por cena e alvo de 90% de
  cobertura. O próximo E2E mede o resultado; não há promessa artificial de que o
  acervo contém 90% das cenas.
- Render: as duas falhas antigas ocorreram na rota Offthread; o render válido usou
  `ReliableVideo`/`@remotion/media` e quatro chunks. O gate agora rejeita preflight
  de outro job ou timeline, decoder diferente, fallback Offthread e marcador sem
  frames testados. Meta operacional: passar na primeira tentativa.
- Critérios completos de vitória e ordem do canário:
  `ESPEC-CANARIO-POS-GOLDEN-LEOPARDO.md`.
- A otimização do VEO só começa após esse canário e fica isolada à futura BOX do
  gerente VEO. É proibido tocar no restante da esteira para otimizar esse funcionário.
- Testes offline e sintáticos passaram; nenhuma chamada externa ou render foi
  disparado.

## Sessão 14/08/2026 — Leopardo aprovado como MVP com exclusões; backend corrigido

- Decisão do operador: **não rerenderizar o Leopardo**. O artefato
  `_saida/leopardo_neves.mp4` é referência positiva de sincronia, punchlines,
  elementos de texto e linguagem editorial. Aprovação e exclusões estão
  documentadas em `director/jobs/leopardo_neves.json`, sem alterar o vídeo.
- Integridade preservada: 322.331.302 bytes, SHA-256
  `d7ceb7af126a37f0d514c8bb2597367cb33064fe33d1181d30d02ab440d4cfe9`.
- Causa raiz confirmada do Ovl15 incorreto: `datas.py` aceitava todo número
  interpretável entre 1000 e 2099 como ano. No roteiro real isso transformou
  `a thousand` e `between four and seven thousand animals` em ano 1000. O
  detector agora exige evidência de ano-calendário, rejeita unidades/magnitudes
  e transporta a prova até o contrato e o compilador final. `1970` continua
  reconhecido e é a ocorrência correta de referência.
- Causa raiz confirmada das cenas longas: L13 e o reconciliador isentavam imagens,
  embora o espectador veja duração de tela, não tipo de arquivo. A política agora
  vale para toda mídia visual. O checkpoint real dividiu 26 cenas com doadores
  válidos; a cena 38 permaneceu em 9,99 s por falta de doador identitariamente
  seguro e agora bloqueia honestamente o compilador, sem reuso mascarado.
- Fiscal pós-render recalibrado: combina cortes declarados e scene-detect,
  proporcionaliza o minuto final, recompõe `4` + `,000` + `meters` e só acusa
  repetição quando há proximidade capaz de produzir fadiga. As 26 acusações
  antigas caíram para duas exclusões técnicas conhecidas no MP4 atual: contador
  `40 meters` persistente e cauda preta de aproximadamente 0,25 s.
- Causa da cauda preta: concatenação dos chunks preservava pacotes AAC além do
  último frame da timeline. O renderer futuro limita a concatenação à duração
  assinada do plano. A correção foi validada sintaticamente; nenhum render foi
  disparado para este MVP.
- Bancadas aprovadas: datas/BOX, contrato do Marco Temporal, política temporal de
  mídias, checkpoint real do reconciliador, receita canônica, gates, contratos do
  refino MVP e fiscal recalibrado. `git diff --check` sem erros.
- Próximo E2E deve herdar as correções. Aprovação excepcional do Leopardo não
  afrouxa leis globais: falta de mídia segura deve parar antes do render.

## Sessão 14/08/2026 — canário editorial pausado no gate (render NÃO iniciado)

- Operador autorizou o render e depois pediu pausa imediata ao sair da
  concessionária. Auditoria: nenhum processo do pipeline/Remotion do Leopardo e
  nenhum MP4 final criado. Estado atual: `QUARANTINED` em `aprovar_render`.
- A origem validada (80/80 mídias) foi adotada pelo executor canônico; banco,
  VEO e autoria já aprovados não foram repetidos. Os checkpoints 07–26 estão em
  `_workspace/_jobs/leopardo_neves/estado/checkpoints`.
- Causa confirmada do primeiro bloqueio: dois infográficos concorrentes moviam
  as bordas de takes `Ovl15_MarcoTemporal`. O contrato da BOX agora reserva seus
  5,5 s (overlay em +1,3 s por 4,2 s), devolve concorrentes e é imutável nas
  operações de casamento/absorção/costura. Os 3 Marcos passaram L14.
- `tapar_buracos` era chamado pela receita sem a timeline obrigatória. `Step.args`
  passou a declarar `("{timeline}", "--so-buracos")`; a etapa passou em 0,47 s
  sem realocar decisões válidas.
- O reuso 19↔38 tinha 90,32 s na origem, mas caiu a 87,63 s após a edição final.
  O verificador agora reutiliza a regra única do alocador, proíbe doador de vídeo
  menor que a janela receptora, escolhe cópia semanticamente segura e verifica
  tudo novamente. Na timeline real, cena 38 recebeu a imagem da cena 3
  (snow leopard stalking blue sheep), preservando fronteiras/parallax; verificador
  terminou com zero violações de reuso.
- Passaram: `reconciliar_cenas`, `tapar_buracos`, `verificador_reuso`, `closer` e
  `pre_render_report`. O gate bloqueou corretamente antes de preparar/renderizar.
- Pendências reais do gate para a retomada: 7 erros fora da lista branca
  (identidade nas cenas 38/48; frame extraction 26/36/53/73; mapa 1 ilegível) e
  12 violações de leis: L04=2, L06=2, L09=3, L11=1, L13=4. L01/L02/L03/L05/L07/
  L08/L10/L12/L14 passaram. Investigar causa antes de corrigir; não afrouxar gate.
- Decisão arquitetural aprovada pelo operador: detectores só propõem; um único
  Maestro/compilador resolve e assina `EditPlan`; passes posteriores são leitores;
  renderer só materializa; diretores mudam política, não o caminho. Migrar após o
  canário, escritor por escritor e contra golden. Se o Leopardo for aprovado,
  torna-se o novo golden canônico; a Onça permanece fixture histórica.

## Sessão 14/08/2026 — retomada canônica concluída (sem render)

- Timeline principal fechou em **80/80 cenas com mídia**, SHA-256
  `16547460a258ff5666bb4a6633b09958d766f2eddeedf5d5ac92898e79fb05a8`.
- Mailbox oficial `_coord/leopardo_neves`: **20/20 DONE**, zero pendentes e zero
  esgotados. Zero processo Python do pipeline/render ao encerrar.
- Banco válido: **2,4 min**, 13/32 cenas resolvidas no banco e 19/32 realocadas
  honestamente ao VEO; zero reuso forçado e zero esgotamento operacional. Cobertura
  de 40,6% é insuficiente: próximo E2E deve perseguir >=90% em ondas até 8–9 min.
- Causa da caixa VEO invisível: `resolver_cascata` derivava `_coord` do workspace
  mutável do ramo. `coord_job_dir()` agora é a fonte única; o merge também reprova
  qualquer buraco pós-consumidor.
- Incidente VEO: a retomada direta perdeu o contexto do job e caiu nos defaults
  `ENSAIO/pipeline`, abrindo o projeto do urso e criando a coleção `pipeline`.
  Três prompts foram enviados antes da interrupção: cenas 19, 23 e 28. A coleção e
  seus três resultados permanecem no Flow; não apagar sem autorização do operador.
- Prevenção: jobs híbridos agora exigem `veo_channel`, `veo_project_id` e
  `veo_collection`; runner cruza os três com `_veo_flow/projetos.json` antes da UI,
  resume exige coleção registrada, conflitos de ambiente falham fechado e os
  defaults genéricos foram removidos de `veo_gerar.py`.
- Colheita sem envio (`veo_ciclo.py --somente-colher`) encontrou material antigo
  para 13, 19 e 47. O entregador provou que eram duplicatas físicas e recusou; o
  socorro encerrou essas três como `fonte=veo_reuso`, com distância de 97–109s.
  Resultado honesto do lote: **16 takes novos + 3 reusos declarados**.
- Cena 41: envios em 10:58:24.520, 11:02:45.490 e 11:05:40.930; arquivo local em
  11:07:41.616. Total **9min17,1s**; atraso incremental sobre os demais vídeos
  **5min59,4s** (cerca de 27% dos 22,2 min da retomada).
- Nenhum Remotion/render foi disparado e nenhum MP4 final novo foi criado.

### Pendências após esta sessão

1. Próximo E2E: resolver em ondas progressivas, alvo >=90% de cobertura do banco,
   teto total 8–9 min e um único arquivo persistente por cena.
2. VEO: retries de exceções não devem bloquear entrega/gate do lote inteiro; cena 41
   mostrou quase 6 min de atraso incremental.
3. Resume: reaproveitar prompt-store sem pagar novamente ~3 min de autoria Codex.
4. Classificar com o operador os 3 `veo_reuso` antes dos Blocos 6/7, se a exigência
   for take inédito por cena no MVP.
5. Limpeza de artefatos pesados continua separada; não misturar com o pipeline.

## Sessao 14/08/2026

- Operador conectado ao Wi-Fi da concessionaria. Timeouts/oscilacoes de chamadas
  externas nesta sessao devem ser primeiro classificados como variavel de rede.
- GO autorizado para continuar a cascata, sem disparar render.

## Pausa solicitada

- O operador solicitou a interrupção para usar CPU/GPU no Automator.
- Nenhum render do Leopardo foi disparado.
- Testes em execução foram encerrados.
- Auditoria final: zero workers ativos de Vidmator, FFmpeg, resolver ou Remotion.
- Não existe MP4 final novo do Leopardo.

## Estado importante

- O replay rápido do banco terminou em cerca de 2 minutos e respeitou 32 downloads para 32 cenas, mas é **inválido para qualidade**.
- Apenas 8 clipes foram aceitos; 24 cenas sofreram `frame_extract_error` e foram preenchidas por reutilização forçada.
- A timeline principal atual recebeu esse merge inválido. **Não editar nem renderizar essa timeline.**
- Causa confirmada: o processo usava FFmpeg incompatível do Miniconda e lançava o resolver com o Python errado, sem Pillow.
- Runtime correto: `D:\AC Creator HUB\Vidmator\_venv-director\Scripts\python.exe`.
- FFmpeg correto: `C:\Users\lenovo\AppData\Local\Microsoft\WinGet\Packages\Gyan.FFmpeg_Microsoft.Winget.Source_8wekyb3d8bbwe\ffmpeg-8.1.1-full_build\bin\ffmpeg.exe`.
- O código foi encaminhado para usar esses runtimes e ganhou preflight, mas a validação foi interrompida antes de terminar.

## Retomada obrigatória

1. Reexecutar testes de runtime/preflight e extração de frames, sem render.
2. Criar portão de qualidade: reutilização forçada ou esgotamento operacional não podem produzir sucesso, manifest válido ou merge.
3. Colocar em quarentena o banco rápido inválido e a timeline mesclada atual.
4. Restaurar a timeline-base somente de uma cópia com SHA-256 `3d0ad15bae8bae2cd011b6aa2db82ada06eda55e0a6910e22a251912044490f7`.
5. Confirmar no resume-check: VEO `SKIP_VALIDATED`, banco `RUN`.
6. Reexecutar somente o banco, validar qualidade, máximo de um download materializado por cena e teto de 720 segundos.
7. Não disparar render sem ordem expressa do operador.

## Artefatos preservados

- Banco antigo duplo: `_workspace\_jobs\leopardo_neves\quarantine\banco_duplo_20260813_2134`.
- Incidente: `director\incidents\leopardo_banco_duplo_13_08.json`.
- Cópias conhecidas da timeline-base: `_ws_banco\timeline.json` e a quarentena acima; verificar o hash antes de restaurar.
