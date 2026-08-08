# ESPEC ESTRUTURAL — Contrato Visual e Execução Reprodutível

**Criada:** 03/08/2026 · **Origem:** seção 6 da auditoria externa (GPT-5.6-sol), adaptada
ao que JÁ existe pós-IDENTIDADE-v2 · **Status:** EM EXECUÇÃO (decisão do operador: sem
render agora; implementar → nova auditoria externa → só então decidir o render)

**Objetivo (nas palavras da auditoria):** transformar decisões probabilísticas em planos
auditáveis, impor invariantes em todos os caminhos, e permitir testar mudanças **sem
ressortear o vídeo inteiro** — tornar classes de erro impossíveis ou imediatamente
observáveis, em vez de corrigir instância por instância.

## O que JÁ existe (da IDENTIDADE v2) e o que esta espec ADICIONA

| Peça | v2 entregou | Esta espec completa |
|---|---|---|
| Contrato visual | compilador separado, match_mode, filtro por tipo, vocabulário de bioma, segunda chance | versionamento do contrato no manifesto (hash) |
| Vet | estruturado, multi-frame, UNCERTAIN reprova | registro de TODA decisão de vet no manifesto (também cenas de atmosfera) |
| Pós-condição | no caminho de identidade | **registro universal**: toda cena sai com decisão registrada (aceita com validação OU degradação declarada) |
| Manifesto | embrião (só identidade) | **completo + MODOS de execução** (o coração desta espec) |
| Testes | controles manuais no turno | **suíte executável** (fixtures dos casos históricos) |
| Proveniência | — | origem/licença por asset; SearXNG marcado como licença-desconhecida com flag de bloqueio p/ produção |
| Métricas | — | contadores por rodada no manifesto |
| Auditoria de composição | — | script pós-render (frames por cena + Vision) pronto p/ o próximo render |

## 1. DECISION MANIFEST completo (`_workspace/decision_manifest.json`)

Por rodada: `meta` (hash do roteiro, hash do áudio, preset/nicho, bioma, timestamp da
narração, versão dos prompts-chave) · por CENA: texto, hash-de-invalidação (texto+query+
contrato), decisão (nível, arquivo, hash do arquivo, mídia, vet aplicado e veredito,
origem/licença), tentativas rejeitadas (candidato+violações) · por TAKE: idem · métricas.

**Invalidação por dependência:** a decisão de uma cena só é reutilizável se o
hash-de-invalidação bater (texto da cena, query, contrato e preset iguais) E o arquivo
local existir com o mesmo hash. Mudou qualquer um → stale → re-resolve.

## 2. MODOS de execução (env `VM_MODO`, default `fresh`)

| Modo | O que faz | Para quê |
|---|---|---|
| `fresh` | ignora manifesto anterior (comportamento atual) | 1ª rodada; teste de generalização |
| `production` | reusa TODA decisão válida do manifesto; resolve só o que falta/stale | produção real: convergência, custo mínimo |
| `surgical:3,15` | re-resolve SÓ as cenas/takes apontados; resto preservado | correção pontual sem ressortear as outras 20 cenas (o pedido do doc 03) |
| `replay` | zero chamadas (LLM/fontes); reconstrói timeline só do manifesto+assets | reproduzir um render byte-consistente; debug |

Estados de decisão no manifesto: `auto_accepted` (vet PASS) · `degraded` (atmosfera por
hard_miss, declarado) · `hard_miss` · `stale` (invalidada). `operator_approved/rejected`
= campos previstos (a UI de aprovação é futura; a estrutura já os comporta).

## 3. Suíte de testes executável (`director/_testes/`)

`test_diretor.py` — fixtures dos casos históricos que NÃO dependem de rede (rodam sempre):
detector de enumeração (preâmbulo EN/PT, falso positivo, núcleo nominal) · variantes
preservam sujeito (pink river dolphin) · mood map EN→PT · guardrail de query (sujeito
ausente → prepend) · manifesto: rejeitado não aparece como escolha · filtro por tipo do
contrato (lugar/elemento nunca é required).
`test_diretor_online.py` — os que chamam Gemini/Vision (rodar sob demanda): controle
negativo cidade-vs-jaguar · positivo golfinhos · segunda chance de criaturas.

## 4. Proveniência e SearXNG

Todo asset aceito registra `origem` (fonte, id, url quando houver, licença). Pexels =
"Pexels License". SearXNG = `licenca: "DESCONHECIDA (web aberta)"` + flag
`searxng_producao` (default `false`): em produção o nível Lsx é PULADO com aviso —
watermark não é licença (auditoria §2.5). Em teste/PoC continua ativo e REGISTRADO.

## 5. Auditoria de composição (`auditoria_composicao.py`, roda pós-render)

Extrai 1 frame por cena do MP4 final → contact sheet → Vision em batch: sujeito
obrigatório do contrato visível? tela preta? livro/ilustração/mídia alheia? overlay
encobrindo o sujeito? → relatório JSON + saída humana. Pega interferência de passes
POSTERIORES ao resolver (classe P11). Entra no fluxo no próximo render.

## 6. Fora desta espec (registrado, não implementado agora)

Seleção comparativa multi-fonte com contact sheets no RESOLVER (custo/latência — avaliar
após métricas) · validação formal de mixagem (loudness/true peak do render) · UI de
aprovação por cena (operator_approved) · vocabulário de biomas expandido com revisão.

## 7. Critérios de aceite

- [ ] Manifesto completo gerado com meta+cenas+takes+métricas e hashes
- [ ] `surgical:N` re-resolve só a cena N (provado com o manifesto da PoC)
- [ ] `production` com nada stale = zero chamadas de resolução novas
- [ ] `replay` = zero chamadas externas
- [ ] Suíte offline verde; suíte online verde sob demanda
- [ ] Asset sem origem registrada não existe no manifesto
- [ ] `searxng_producao=false` pula o Lsx com aviso
- [ ] Regressão: NICHO=documentario → nada disso executa
