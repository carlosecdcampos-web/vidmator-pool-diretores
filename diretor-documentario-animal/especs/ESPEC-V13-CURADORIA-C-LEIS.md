# CURADORIA C — INVENTÁRIO DAS LEIS DO OPERADOR (ancorado no código real)

Objetivo (ordem do operador 10/08): *"prevenir é melhor que remediar... senão daqui
a pouco estaremos na v30 com erros absurdos que já deveriam ter sido corrigidos, ou
pior, que foram corrigidos e apareceram depois."*

Método: para cada lei já ditada, três colunas — **onde ela mora hoje**, **quem a
verifica automaticamente**, e **como ela pode morrer em silêncio**. Lei sem
verificador é lei que depende do olho do operador; foi assim que todas as três
regressões da v11 chegaram ao vídeo final.

## O que o portão verifica HOJE (`_decupagem_video.py`, roda só no MP4)
F2 cortes/min · D5 tela preta · F3/F4 elementos/min · F12 teto de variante ·
F13 lei dos números · F15 take repetido · F16 conteúdo duplicado · W3 word-pop ·
I16 elemento vazio · V10-L2 adjacência de moldura.
**São 10 leis verificadas — e todas SÓ depois de 25-30 min de render.**

## LEIS SEM NENHUM VERIFICADOR (o buraco real)
| # | Lei do operador | Mora hoje em | Como morre em silêncio |
|---|---|---|---|
| L1 | Elemento tela-cheia é CENA: nenhum take por baixo | `preparar_render` R2/R2b (pós-lock) | passe posterior (V7) desfaz; `if _caps` desliga |
| L2 | Duração = tempo falado, nunca fixa | `legibilidade` (só `texto_impacto`) | `acervo_texto` e cards seguem 4,0/4,5/3,0s fixos |
| L3 | Entra na 1ª palavra, sai na última | `legibilidade.janela_falada` | casa ocorrência errada se a frase repete |
| L4 | Take VEO nunca ≥7,5s (loop) | `resolver_cascata` (alvo 6s, cede com aviso) | merge/absorção alonga cena depois do resolver |
| L5 | Nunca take repetido em cenas consecutivas | guard V12-R1 (dentro do `if _caps`) | vídeo sem capítulo não roda o guard |
| L6 | Nenhum fragmento de cena <2s | `montar_timeline` merge | merge desiste antes de marcador; sobra o toco |
| L7 | SFX banidos (apito) | trava V12 no `preparar` | preset novo/clone pode reintroduzir noutro campo |
| L8 | ASMR dos takes a −33 M-max | `veo_entrega` | só na entrada do pool; take de fora não passa |
| L9 | Cota de moldura dos 3 primeiros minutos | `edicao` (pré-pass) | vira AVISO, nunca reprova (opção A) |
| L10 | Palavra não se parte na quebra de linha | `AcervoTexto` LetterCascade | os outros ~30 componentes não foram auditados |
| L11 | Elemento não repete o assunto de outro | V12-R3 (dentro do `if _caps`) | idem L5 |
| L12 | Transição/projetor/shutter fora da janela do elemento | **ninguém** | nunca foi implementado (o operador ouviu em 0:17) |

## BLINDAGEM PROPOSTA — `director/leis.py`
1. **Registro único**: cada lei vira `LEI(id, data, quem_pediu, texto, verificar())`.
   O texto é a frase do operador, verbatim — a lei não se perde na tradução.
2. **Dois momentos obrigatórios**: `leis.verificar(timeline)` antes do lock (barato,
   segundos) e `leis.verificar(render_json, mp4)` depois do render. A primeira
   execução é a que mata o ciclo de 30 min: o defeito é pego ANTES de renderizar.
3. **Lei nunca some**: revogação exige `revogada_em` + motivo + quem autorizou.
4. **Lei nova nasce com verificador** — sem verificador, a lei não entra no
   registro (é só comentário, e comentário já provou que não segura nada).
5. **O portão passa a ser a execução do registro**, e o relatório sai nomeando a
   lei violada com a frase original do operador.

## CURADORIA A — o reconciliador ancorado no código (fato verificado)
Varredura de quem escreve estrutura de cena hoje:
- **`montar_timeline.py:418` é o ÚNICO escritor de `cena.inicio/fim`** (agrupamento
  → merge → B2 semântico → E1 cadência). Todos os demais passes (`detectar_mapas`,
  `enumeracoes`, `topicos`, `trilha`) escrevem `inicio/fim` **dos objetos DELES**
  (mapa, janela de pop, tópico, segmento de trilha), nunca da cena.
  ⇒ **O reconciliador será o 2º escritor estrutural da história do pipeline.** Por
  isso ele precisa ser explícito, único, logado linha a linha e rodar num ponto só.
- **`aparece_em` tem 3 escritores**: `edicao`, `ilustrar`, `legibilidade` — nessa
  ordem. O último a falar é o `legibilidade`, que é onde o relógio-palavra vive
  agora. Confirma que o reconciliador tem que rodar DEPOIS dele.
- Ponto de inserção definido: `_rodar_v11_serpentes.py` entre `legibilidade` e
  `pre_render_report` — e, na cadeia canônica, em `producao.py` PASSES, na mesma
  posição (senão o runner de produção fica sem a lei, repetindo o erro do v9 em
  que `edicao`+`legibilidade` faltavam na lista canônica).

## Por que isso ataca a causa e não o sintoma
As três regressões da v11 (D1 rodando a lei v8 morta; `legibilidade` apagando o
relógio-palavra do B1; meu bloco de curiosidade re-atando o `elif` dos capítulos)
têm o mesmo mecanismo: **a lei existia só como código dentro de um passe, e outro
passe passou por cima**. Com o registro, qualquer um dos três teria falhado na
verificação pré-lock, em segundos, sem render e sem depender do olho do operador.
