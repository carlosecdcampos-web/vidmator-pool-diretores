# CURADORIA B — INVENTÁRIO DE LEIS NÃO-UNIVERSAIS (relatório do Codex, verbatim)

Ordem do operador 10/08: *"a inteligência de edição será unânime em TODOS OS DIRETORES...
edição PERFEITA é uma premissa de qualidade em QUALQUER vídeo, não só os que tiverem capítulo."*

Final output
Auditoria concluída nos 13 arquivos indicados, com rastreamento dos gates de preset e das validações posteriores. Foram confirmados 40 pontos em que uma lei de edição não é universal. Escolhas exclusivamente estéticas foram excluídas.

1. arquivo:linha: director/montar_timeline.py:168-210; director/presets.json:563
   - qual lei de edição está condicionada: toda mudança de sujeito visível dentro da fala deve criar um novo take exatamente quando o novo sujeito é mencionado.
   - qual é a condição que a prende: `b2_take_fala` precisa estar ativo — hoje só aparece em `doc_realista` —, a cena precisa ter pelo menos 4,6 s e 10 palavras, o LLM precisa responder, a palavra precisa casar verbatim e as duas metades precisam ter pelo menos 2,2 s.
   - o que acontece na tela num vídeo que não satisfaz a condição: o take do sujeito seguinte pode entrar antes de seu nome ou continuar mostrando o sujeito anterior depois que a narração já mudou.
   - severidade: crítico.

2. arquivo:linha: director/montar_timeline.py:245-267
   - qual lei de edição está condicionada: a cadência mínima deve atingir 12 cortes por minuto.
   - qual é a condição que a prende: o trecho final com menos de 15 s é excluído; nos demais minutos a correção para quando não existe cena com pelo menos 4,6 s divisível em duas partes de 2,2 s.
   - o que acontece na tela num vídeo que não satisfaz a condição: permanecem minutos ou encerramentos visualmente lentos, com takes longos e menos cortes que o piso declarado.
   - severidade: crítico.

3. arquivo:linha: director/edicao.py:1273-1300; director/legibilidade.py:39-79; director/legibilidade.py:122-138; director/presets.json:420
   - qual lei de edição está condicionada: todo elemento textual deve entrar na palavra correspondente da narração.
   - qual é a condição que a prende: `legibilidade` só está ligado em `doc_realista`; mesmo com ele ligado, depende de `words.json` e do casamento da frase. Em `edicao.py`, elementos `_d4`, `slot` e elementos cujo novo horário cairia em janela exclusiva são explicitamente pulados.
   - o que acontece na tela num vídeo que não satisfaz a condição: texto ou segundo infográfico entra no começo da cena, antes ou depois da fala, ou conserva um horário antigo desalinhado.
   - severidade: crítico.

4. arquivo:linha: director/legibilidade.py:77-84; director/legibilidade.py:137-138; remotion/src/compositions/BrollTest.tsx:1208-1216
   - qual lei de edição está condicionada: texto dinâmico precisa permanecer tempo suficiente para ser lido.
   - qual é a condição que a prende: depende do knob `legibilidade`; mesmo quando executado, `texto_ate` é limitado ao fim da cena e pode ficar abaixo de `texto_min_s`. Sem `texto_ate`, o componente usa somente o restante da cena.
   - o que acontece na tela num vídeo que não satisfaz a condição: o texto pode piscar por menos do que o mínimo de leitura e desaparecer no corte.
   - severidade: crítico.

5. arquivo:linha: director/legibilidade.py:144-157
   - qual lei de edição está condicionada: o infográfico numérico deve entrar quando o número é falado sem sacrificar sua legibilidade.
   - qual é a condição que a prende: quando restam menos de `infografico_min_s`, o código move `aparece_em` para `fim da cena - info_min`, mesmo que isso seja anterior à âncora falada.
   - o que acontece na tela num vídeo que não satisfaz a condição: o número surge antes de ser pronunciado.
   - severidade: crítico.

6. arquivo:linha: director/enumeracoes.py:342-356; director/legibilidade.py:162-200; director/pre_render_report.py:167-175
   - qual lei de edição está condicionada: o word-pop deve ser o único dono da tela em sua janela.
   - qual é a condição que a prende: `enumeracoes.py` testa apenas se o início do overlay cai perto da janela, não a sobreposição integral; a guarda intervalar final só roda com `legibilidade`; além disso, `legibilidade.py` preserva explicitamente o infográfico colidente.
   - o que acontece na tela num vídeo que não satisfaz a condição: texto, ilustração ou infográfico pode aparecer simultaneamente com as palavras gigantes do word-pop.
   - severidade: crítico.

7. arquivo:linha: director/legibilidade.py:77-103
   - qual lei de edição está condicionada: metadados antigos de duração textual precisam ser limpos antes de recalcular a edição.
   - qual é a condição que a prende: a limpeza de todos os `texto_ate` está depois do early-return de `legibilidade`.
   - o que acontece na tela num vídeo reutilizado sem esse knob: um texto recriado ou remanescente pode conservar a duração de uma execução anterior e atravessar tempo indevido.
   - severidade: médio.

8. arquivo:linha: director/edicao.py:872-916
   - qual lei de edição está condicionada: cada minuto precisa cumprir o piso de entradas de texto.
   - qual é a condição que a prende: o minuto final só é contado se tiver pelo menos 30 s; dentro dos minutos considerados, o laço abandona a lei quando `_cands_m` fica vazio.
   - o que acontece na tela num vídeo que não satisfaz a condição: um minuto ou encerramento pode ficar visualmente vazio, sem as duas ou três entradas declaradas como requisito.
   - severidade: crítico.

9. arquivo:linha: director/edicao.py:251; director/edicao.py:1169-1233; remotion/src/compositions/BrollTest.tsx:1067-1075
   - qual lei de edição está condicionada: toda medida falada deve ganhar elemento numérico dinâmico.
   - qual é a condição que a prende: depende de `words.json`; é pulada se existir qualquer elemento a até 6 s que não termine na medida, se o instante estiver em word-pop ou se nenhuma das três posições passar `_cabe`. O segundo infográfico ainda cai se começar a menos de 1 s do fim da cena ou se houver capítulo.
   - o que acontece na tela num vídeo que não satisfaz a condição: peso, altura, distância, porcentagem ou segunda medida é apenas ouvida, sem animação numérica, ou é coberta por um elemento sem relação com o número.
   - severidade: crítico.

10. arquivo:linha: director/edicao.py:1307-1325
   - qual lei de edição está condicionada: nenhum fragmento ultracurto de take pode aparecer entre elementos.
   - qual é a condição que a prende: a busca considera somente elementos `tela_cheia` aceitos e capítulos previstos; mapas, pessoas, word-pops, abertura e outros donos não entram em `_donos`. Mesmo nos pares considerados, a correção é abandonada se outra janela exclusiva começa no vão.
   - o que acontece na tela num vídeo que não satisfaz a condição: sobra um flash de take de até 0,35 s entre cards ou outros elementos de tela cheia.
   - severidade: crítico.

11. arquivo:linha: director/enumeracoes.py:262-280; director/enumeracoes.py:359-390
   - qual lei de edição está condicionada: o word-pop deve ter um único take de fundo relacionado ao tema do roteiro.
   - qual é a condição que a prende: a substituição temática exige `micro_takes`, sucesso do classificador e `fundo_query`; sem essa string, o fundo único é marcado, mas a query original não é substituída.
   - o que acontece na tela num vídeo que não satisfaz a condição: o word-pop pode permanecer sobre um take contínuo, porém semanticamente errado, como colagem ou objeto sem relação com o tema.
   - severidade: crítico.

12. arquivo:linha: director/detectar_mapas.py:233-268; director/presets.json:492
   - qual lei de edição está condicionada: lugar explicitamente mencionado no hook deve ganhar mapa de apresentação na introdução.
   - qual é a condição que a prende: `if _Pm.get("mapa_hook_obrigatorio")`; o knob só está declarado no preset `doc_realista`.
   - o que acontece na tela num vídeo histórico, psicológico, chosen one ou estoico: a introdução pode falar claramente de um país ou localidade sem mostrar qualquer orientação geográfica.
   - severidade: crítico.

13. arquivo:linha: director/detectar_mapas.py:270-304
   - qual lei de edição está condicionada: o mapa deve entrar cedo na unidade narrativa, e o mapa único deve aparecer antes do minuto 4.
   - qual é a condição que a prende: o reposicionamento só ocorre para capítulos previstos e para menção até 12 s do card, com antecipação máxima de 5 s. A regra do minuto 4 só testa `len(limpos) == 1` e apenas imprime aviso, sem corrigir ou reprovar.
   - o que acontece na tela num vídeo que não satisfaz a condição: em roteiro sem capítulos o mapa pode aparecer no meio do assunto; um ou vários mapas podem surgir apenas muito tarde, depois de minutos sem orientação.
   - severidade: crítico.

14. arquivo:linha: director/detectar_mapas.py:306-327; director/presets.json:462
   - qual lei de edição está condicionada: mapa nunca pode cobrir a cena cujo sujeito precisa estar visível.
   - qual é a condição que a prende: `_ids` só contém cenas com `contrato.required_subjects`; a geração do contrato visual está habilitada especificamente em `doc_realista`.
   - o que acontece na tela num vídeo sem contratos: o mapa pode continuar sobre a entrada do personagem, animal ou objeto que deveria estar na tela, escondendo-o.
   - severidade: crítico.

15. arquivo:linha: director/detectar_mapas.py:196-214
   - qual lei de edição está condicionada: o mapa deve ser ancorado na menção falada do lugar.
   - qual é a condição que a prende: depende da presença de `words.json` e de `localizar()` encontrar o trecho; `ini is None` elimina silenciosamente o candidato.
   - o que acontece na tela num vídeo que não satisfaz a condição: um lugar detectado no roteiro não recebe mapa algum.
   - severidade: crítico.

16. arquivo:linha: director/pessoas.py:159-177; remotion/preparar_render.py:772-787
   - qual lei de edição está condicionada: pessoa citada deve ganhar seu card no instante do nome, com retrato correto.
   - qual é a condição que a prende: depende de `words.json` localizar o trecho e da existência do retrato público ou gerado; o preparador suprime o card se o retrato pendente não existir.
   - o que acontece na tela num vídeo que não satisfaz a condição: a pessoa é mencionada sem qualquer identificação visual.
   - severidade: médio.

17. arquivo:linha: director/pre_render_report.py:78-83; remotion/preparar_render.py:149-177; director/presets.json:463
   - qual lei de edição está condicionada: todo vídeo deve ter plano validado e aprovado antes de materializar o render.
   - qual é a condição que a prende: tanto o relatório quanto a exigência do lock dependem de `pre_render_lock`, presente somente em `doc_realista`.
   - o que acontece na tela num vídeo que não satisfaz a condição: ele pode ser renderizado mesmo contendo erros que o relatório detectaria, sem aprovação ou verificação de que a timeline não mudou.
   - severidade: crítico.

18. arquivo:linha: director/pre_render_report.py:81-83; director/pre_render_report.py:140-166
   - qual lei de edição está condicionada: o mesmo take não pode aparecer adjacente, mais de duas vezes ou com menos de sete cenas de distância.
   - qual é a condição que a prende: o juiz inteiro só roda com `pre_render_lock`.
   - o que acontece na tela nos demais nichos: repetições de take podem chegar ao render sem serem bloqueadas pelo relatório.
   - severidade: crítico.

19. arquivo:linha: director/pre_render_report.py:81-83; director/pre_render_report.py:167-184
   - qual lei de edição está condicionada: word-pop deve ter fundo contínuo e não pode dividir a tela com outro overlay.
   - qual é a condição que a prende: os checks N3/N4/N8 só existem após o gate `pre_render_lock`; o check de colisão ainda observa somente `aparece_em`.
   - o que acontece na tela nos demais nichos: pode haver corte no fundo entre palavras ou dois elementos textuais simultâneos sem bloqueio pré-render.
   - severidade: crítico.

20. arquivo:linha: director/pre_render_report.py:81-83; director/pre_render_report.py:185-232
   - qual lei de edição está condicionada: cenas com identidade obrigatória devem passar pelo caminho de identidade e respeitar região, sujeito e contexto.
   - qual é a condição que a prende: o relatório depende de `pre_render_lock`; internamente, o check também depende de `contrato.required_subjects` ou de violações registradas no manifesto.
   - o que acontece na tela num vídeo que não satisfaz a condição: pode entrar animal, pessoa ou paisagem de região errada, ou uma cena genérica que não mostra o sujeito falado.
   - severidade: crítico.

21. arquivo:linha: director/pre_render_report.py:81-83; director/pre_render_report.py:234-284
   - qual lei de edição está condicionada: cenas, micro-takes e textos precisam respeitar o estado mínimo de tela e o tempo mínimo de leitura.
   - qual é a condição que a prende: o relatório só roda com `pre_render_lock`; o check textual só existe quando `texto_ate` foi produzido.
   - o que acontece na tela nos demais nichos: cenas abaixo de 1 s, sobras de cena-pai e textos ilegíveis podem seguir para o render sem bloqueio.
   - severidade: crítico.

22. arquivo:linha: director/pre_render_report.py:81-83; director/pre_render_report.py:362-391
   - qual lei de edição está condicionada: mapa não pode esconder sujeito e sua legenda deve ser o termo geográfico realmente falado.
   - qual é a condição que a prende: o relatório depende de `pre_render_lock`; a proteção de sujeito depende de `required_subjects`, e a conferência verbatim é pulada quando `_fala` está vazio.
   - o que acontece na tela num vídeo que não satisfaz a condição: mapa pode cobrir o sujeito ou exibir uma legenda geográfica não pronunciada.
   - severidade: crítico.

23. arquivo:linha: director/pre_render_report.py:81-83; director/pre_render_report.py:246-277
   - qual lei de edição está condicionada: toda cena deve ter mídia legível e decisão semanticamente válida, e a cena-pai de enumeração não pode contradizer seus takes.
   - qual é a condição que a prende: todo esse conjunto de verificações só roda com `pre_render_lock`; a contradição pai/take depende dos hashes opcionais do manifesto.
   - o que acontece na tela nos demais nichos: mídia ausente, ilegível ou contraditória pode ser descoberta apenas no render ou aparecer como troca editorial errada.
   - severidade: crítico.

24. arquivo:linha: remotion/preparar_render.py:1398-1460; remotion/preparar_render.py:1514-1532
   - qual lei de edição está condicionada: transição ou animação de entrada só pode existir quando há troca visual real.
   - qual é a condição que a prende: a marcação não é julgada se não coincidir com duas fronteiras de cena, se a apresentação mudar ou se `_ahash` não conseguir ler algum dos clipes.
   - o que acontece na tela num vídeo que não satisfaz a condição: whip, slide ou clarão pode acontecer no meio de uma cena ou revelar exatamente a mesma imagem.
   - severidade: crítico.

25. arquivo:linha: remotion/preparar_render.py:72-87; remotion/preparar_render.py:1159-1169
   - qual lei de edição está condicionada: todo vídeo usado no Remotion precisa estar normalizado para reprodução navegável e estável.
   - qual é a condição que a prende: se o ffmpeg falha, `_copiar_video` usa a cópia bruta; o take da abertura sempre é copiado diretamente e nem passa por `_copiar_video`.
   - o que acontece na tela num vídeo que não satisfaz a condição: o clip ou abertura pode congelar, buscar frames incorretos, variar a cadência ou falhar durante o render.
   - severidade: crítico.

26. arquivo:linha: remotion/preparar_render.py:508-528
   - qual lei de edição está condicionada: a recuperação de hard-miss não pode criar takes consecutivos iguais nem deixar tempo sem mídia.
   - qual é a condição que a prende: quando não existe doador não adjacente, o código usa explicitamente o vizinho colado; quando não há doador algum, apenas executa `continue`.
   - o que acontece na tela num vídeo que não satisfaz a condição: o mesmo take aparece em sequência ou a cena fica sem materialização visual.
   - severidade: crítico.

27. arquivo:linha: remotion/preparar_render.py:474-496
   - qual lei de edição está condicionada: molduras não podem repetir nem formar a adjacência proibida `film`–`quadro`.
   - qual é a condição que a prende: se todos os tetos e alternativas do rodízio forem esgotados, `_prox_moldura` devolve `quadro` mesmo sabendo que a regra foi violada.
   - o que acontece na tela num vídeo que não satisfaz a condição: uma moldura repetida ou um par visualmente incompatível é inserido no fallback de hard-miss.
   - severidade: crítico.

28. arquivo:linha: remotion/preparar_render.py:581-583; remotion/preparar_render.py:694-709
   - qual lei de edição está condicionada: personagem recortado deve permanecer associado à cena correta.
   - qual é a condição que a prende: cenas irrecuperáveis são omitidas de `render["cenas"]`, mas os personagens são transferidos posteriormente por índice original `i`, não por `idx` ou horário.
   - o que acontece na tela quando uma cena anterior foi omitida: o personagem pode aparecer sobre a cena seguinte, cuja fala e take não lhe pertencem.
   - severidade: crítico.

29. arquivo:linha: remotion/preparar_render.py:1538-1540; remotion/preparar_render.py:1649-1695
   - qual lei de edição está condicionada: nunca pode haver take idêntico em cenas consecutivas, inclusive dentro de moldura ou overlay.
   - qual é a condição que a prende: o guard final V12-R1 está indentado dentro de `if _caps`; sem card de capítulo ele não roda. Mesmo com capítulos, todas as enumerações são tratadas como exceção e, sem doador legal, a repetição é mantida.
   - o que acontece na tela num vídeo que não satisfaz a condição: o mesmo take continua atravessando dois cortes consecutivos, às vezes acompanhado de transição que finge uma mudança inexistente.
   - severidade: crítico.

30. arquivo:linha: remotion/preparar_render.py:1538-1540; remotion/preparar_render.py:1697-1778
   - qual lei de edição está condicionada: elemento de tela cheia deve ser a própria cena, sem take nascendo ou mudando por baixo e sem SFX de corte interno.
   - qual é a condição que a prende: V12-R2/R2b está dentro de `if _caps`.
   - o que acontece na tela num vídeo sem capítulos: o card pode começar ou terminar entre cortes, revelar fragmentos de take, e um novo take, whip ou SFX pode nascer escondido no meio da animação.
   - severidade: crítico.

31. arquivo:linha: remotion/preparar_render.py:1538-1540; remotion/preparar_render.py:1780-1846
   - qual lei de edição está condicionada: sistemas diferentes não podem mostrar simultaneamente ou em sequência próxima o mesmo conteúdo.
   - qual é a condição que a prende: o dedup V12-R3 está dentro de `if _caps`.
   - o que acontece na tela num vídeo sem capítulos: data, ticker, contador, infográfico e texto podem repetir juntos a mesma informação, queimando dois elementos na mesma fala.
   - severidade: crítico.

32. arquivo:linha: remotion/src/compositions/BrollTest.tsx:798-810
   - qual lei de edição está condicionada: uma cena deve usar uma única arma visual, sem empilhar o pacote vermelho sobre outro dono de tela.
   - qual é a condição que a prende: `overlapsElemento` só reconhece mapas, imagens e enumerações; pessoas, acervo de tela cheia, capítulos, abertura e outros sistemas ficam fora do predicado.
   - o que acontece na tela num vídeo que não satisfaz a condição: wash vermelho, linhas de TV ou glitch pode permanecer combinado com card ou elemento que já domina a tela.
   - severidade: crítico.

33. arquivo:linha: director/_decupagem_video.py:128-151
   - qual lei de edição está condicionada: a auditoria deve comprovar 12 mudanças visuais por minuto.
   - qual é a condição que a prende: quando `rj["cenas"]` existe, cada início de cena é contado como corte sem conferir se o frame realmente mudou; o último minuto com menos de 30 s também é excluído.
   - o que acontece na tela num vídeo que não satisfaz a condição: uma sequência quase estática ou com takes perceptualmente iguais pode ser aprovada como tendo 12 cortes, e um encerramento lento não é julgado.
   - severidade: crítico.

34. arquivo:linha: director/_decupagem_video.py:166-177
   - qual lei de edição está condicionada: o piso de elementos deve valer durante todo o vídeo.
   - qual é a condição que a prende: o minuto final é deliberadamente excluído por `range(3, n_min)`, e uma fração final abaixo de 30 s nem entra em `n_min`.
   - o que acontece na tela num vídeo que não satisfaz a condição: o encerramento pode ficar sem qualquer elemento ou reforço visual e ainda passar no F3/F4.
   - severidade: crítico.

35. arquivo:linha: director/_decupagem_video.py:184-217
   - qual lei de edição está condicionada: o portão pós-render deve reprovar toda medida falada sem elemento dinâmico.
   - qual é a condição que a prende: depende de encontrar e ler um `words.json`; o check só executa dentro de `if _medidas`, portanto ausência ou leitura inválida desliga F13 silenciosamente.
   - o que acontece na tela num vídeo que não satisfaz a condição: medidas sem animação numérica são aprovadas porque o portão conclui que não havia nada para conferir.
   - severidade: crítico.

36. arquivo:linha: director/_decupagem_video.py:266-282
   - qual lei de edição está condicionada: takes idênticos consecutivos devem sempre reprovar.
   - qual é a condição que a prende: `_pops_f15` inclui todas as enumerações, não apenas `modo == "pop"`; qualquer par inteiramente dentro de uma enumeração é isento.
   - o que acontece na tela num vídeo que não satisfaz a condição: em enumeração de micro-takes, o mesmo fundo pode reaparecer entre entradas ou atravessar cenas consecutivas sem que F15 reprove.
   - severidade: crítico.

37. arquivo:linha: director/_decupagem_video.py:284-323
   - qual lei de edição está condicionada: conteúdo duplicado simultâneo deve ser detectado independentemente do sistema que o produziu.
   - qual é a condição que a prende: F16 inventaria apenas datas, acervo, `texto_impacto` e primeiro infográfico; mapas, pessoas, enumerações, ilustrações, segundo infográfico, capítulos e outros cards não participam.
   - o que acontece na tela num vídeo que não satisfaz a condição: dois sistemas omitidos podem exibir a mesma informação ao mesmo tempo e o portão declarar zero duplicações.
   - severidade: crítico.

38. arquivo:linha: director/_decupagem_video.py:339-359
   - qual lei de edição está condicionada: vídeo reprovado não pode permanecer na pasta de entrega.
   - qual é a condição que a prende: a retirada só acontece quando a CLI recebe `--quarentena`; sem a flag, o processo retorna erro, mas o MP4 fica no local publicável. Se a movimentação falha, ele também permanece.
   - o que acontece na tela num vídeo posteriormente confundido com entrega válida: todos os defeitos que reprovaram a decupagem continuam disponíveis para publicação.
   - severidade: médio.

39. arquivo:linha: remotion/preparar_render.py:1088-1099
   - qual lei de edição está condicionada: o pico do SFX deve coincidir com a entrada visual do elemento.
   - qual é a condição que a prende: depende de carregar `catalogo_sfx_calibrado.json`; qualquer erro define `_sfx_cat = {}` e desliga S9.
   - o que acontece na tela/áudio num vídeo que não satisfaz a condição: a animação entra primeiro e o impacto sonoro chega atrasado.
   - severidade: médio.

40. arquivo:linha: remotion/src/compositions/BrollTest.tsx:1325-1336
   - qual lei de edição está condicionada: todo elemento planejado pelo diretor precisa materializar conteúdo visível válido.
   - qual é a condição que a prende: variante desconhecida, texto vazio ou gráfico sem `valores` resulta em `return null`, sem substituição por outra variante válida.
   - o que acontece na tela num vídeo que não satisfaz a condição: o elemento desaparece e deixa um trecho de take nu, reduzindo a densidade decidida pelo diretor.
   - severidade: médio.

## Leis já universais hoje

- Teto estrutural de cena: até 4 s no hook e 6 s no restante, aplicado durante toda construção — director/montar_timeline.py:26-31 e 99-110.
- Marcador “Number/Número + item” cria corte obrigatório antes da revelação, sem gate de nicho — director/montar_timeline.py:84-108.
- Datas detectadas entram pelo timestamp da palavra, com deduplicação e espaçamento de 2,5 s, sem knob de preset — director/datas.py:32-60.
- Word-pop com qualquer cena sem mídia é descartado, impedindo palavra gigante sobre fundo preto — remotion/preparar_render.py:563-580.
- Adjacência cronológica de molduras é reconciliada no estado final, fora do bloco de capítulos — remotion/preparar_render.py:1181-1226.
- Overlay de take nunca permanece sobre outra moldura/apresentação, e filmadora nunca permanece sobre imagem — remotion/preparar_render.py:1465-1495.
- Mood vermelho é retirado de toda cena com apresentação especial — remotion/preparar_render.py:1497-1512.
- Look monocromático é limitado a 10% das cenas, fora de qualquer gate de capítulo ou nicho — remotion/preparar_render.py:1848-1861.
- Lugar sem país, mas com coordenada, recebe obrigatoriamente variante de ponto independentemente de `mapas_variantes` — director/detectar_mapas.py:381-391.
- Overlay de take entra e sai exatamente com a cena/take — remotion/src/compositions/BrollTest.tsx:1038-1060.
- Detecção pós-render de tela preta não admite cota: qualquer amostra preta reprova, e detector cego também reprova — director/_decupagem_video.py:153-164.
- `apresentar.py` impede duas apresentações especiais consecutivas considerando imagem e vídeo na mesma ordem temporal — director/apresentar.py:135-146.

## Early-returns / no-ops que desligam um passe inteiro

- director/edicao.py:242-245 — condição: `if not P.get("edicao"): return`. Perde-se todo o cronograma absoluto: respiro, dono de tela, dedup, alinhamento, piso textual, lei dos números, anti-fragmento e overlays de take. O `default` atual contém `edicao` em director/presets.json:94-103, portanto os presets normais o herdam hoje, mas o desligamento estrutural continua existindo.
- director/legibilidade.py:77-79 — condição: `if not p.get("legibilidade"): return`. Perdem-se relógio por palavra, duração legível, limpeza de `texto_ate` e guarda final de word-pop. Hoje isso atinge todos os presets que não sejam `doc_realista`.
- director/enumeracoes.py:248-250 — condição: `if not p.get("enumeracoes"): return`. Perdem-se word-pops, micro-takes, fundo único, supressão de colisões e alinhamento dos itens. O `default` atual define `enumeracoes: true` em director/presets.json:57.
- director/pre_render_report.py:81-83 — condição: `if not p.get("pre_render_lock"): return`. Perde-se integralmente o relatório e todos os gates pré-render; hoje somente `doc_realista` ativa o knob.
- director/apresentar.py:88-90 — condição: ausência simultânea de cenas de imagem e vídeo. O passe termina; neste caso não há conteúdo visual ao qual aplicar apresentações.
- director/detectar_mapas.py:186-194 — condição: qualquer argumento adicional na CLI ativa modo de teste. O passe retorna antes de ler ou atualizar a timeline; nenhum mapa é gravado.
- director/pessoas.py:147-157 — condição: qualquer argumento adicional na CLI ativa modo de teste. O passe retorna antes de gerar `timeline["pessoas"]`.
- director/_decupagem_video.py:115-118 — condição: ausência do caminho do MP4. O portão pós-render encerra com código 2 e nenhuma lei é medida.
- remotion/src/compositions/BrollTest.tsx:777-780 — condição: `if (!timeline) return null`. A composição inteira vira tela vazia quando o objeto de timeline não chega ao componente.
