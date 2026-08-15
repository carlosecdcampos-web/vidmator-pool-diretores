# ESPEC — canário pós-golden do Leopardo

**Golden vigente:** `director/golden/current.json` → `leopardo_neves_14_08`
**Objetivo:** provar que toda a esteira fora do VEO está estável, mensurável e
automática antes de otimizar o gerente de produção VEO.

## 1. Vitória

Esta etapa termina com **VITÓRIA** somente quando um novo E2E real, disparado pelo
executor canônico, provar simultaneamente:

1. qualidade igual ou superior aos `approved_invariants` do Leopardo;
2. zero ocorrência dos `negative_constraints` do manifesto do golden;
3. banco com alvo de 90% das cenas originalmente atribuídas a ele, usando até
   aproximadamente 8 minutos de busca útil;
4. no máximo um download final materializado por cena e zero lixo rejeitado
   persistente no workspace do job;
5. preflight e render aprovados na primeira tentativa;
6. nenhum caminho legado, fallback em série ou reexecução de checkpoint válido;
7. telemetria separando banco, VEO, edição, preparar, preflight, render e fiscal.

Se a oferta pública não alcançar 90%, o E2E pode continuar com realocação honesta
ao VEO para permitir a inspeção do vídeo final, mas a métrica fica
`BANK_TARGET_MISS` e a etapa ainda não recebe o selo de vitória.

## 2. Evidência do Leopardo

### Banco

- 32 cenas atribuídas ao banco;
- 13 entregas válidas: 40,6% de cobertura;
- 19 realocações ao VEO;
- tempo de parede do ramo: 142,375 s;
- 89 buscas, 75 chamadas Vision e 12 operações de download na tentativa final;
- 58 candidatos de identidade julgados, 11 hard misses;
- gate de qualidade aprovado, zero reuso forçado e zero esgotamento operacional;
- limite final de um download por cena respeitado.

O teto global era 720 s, mas não foi o limitador. O ramo encerrava primeiro pelos
120 s por cena e por dois tetos contraditórios de lotes: seis no runner e três
dentro de `resolver_identidade`. Portanto, apenas aumentar o teto global não
mudaria o resultado.

### Render

- tentativa 1: falhou em 17 s, frame 26;
- tentativa 2: falhou em 44 s, frame 31;
- ambas falharam ao extrair quadro de `abertura_take.mp4` pela rota antiga do
  `OffthreadVideo`;
- tentativa 3: `ReliableVideo`/`@remotion/media`, fallback antigo proibido,
  preflight de 223 quadros aprovado e render em quatro chunks;
- render aprovado: 11.864 frames em 1.237 s (~20min38s), sem contar preparação;
- a cauda preta de ~0,25 s veio da concatenação preservar pacotes AAC além da
  duração editorial; o concat futuro usa `-t timeline.duracao`.

## 3. Política canônica do banco para o próximo E2E

A fonte única é `director/bank_policy.py`:

- alvo de cobertura: 90%;
- teto do resolver: 540 s (9 min);
- orçamento de cena difícil: 420 s, executado em paralelo;
- até 10 lotes de posters por cena;
- no máximo um download final materializado por cena.

O tempo adicional é gasto em buscas e pré-julgamento de posters. Material
reprovado no poster não é baixado. Cena resolvida encerra imediatamente; somente
as pendentes continuam consumindo orçamento. O alvo de oito minutos não autoriza
espera artificial: se 90% forem alcançados antes, o ramo termina antes.

Telemetria obrigatória do ramo:

- tempo de parede total e por faixa de 120 s;
- cobertura acumulada e novas cenas resolvidas por faixa;
- buscas, lotes Vision, imagens julgadas e latência p50/p95;
- downloads, bytes, origem cache/rede e arquivos finais persistidos;
- hard miss, falha operacional e realocação ao VEO;
- `TARGET_MET` ou `TARGET_MISS` no manifesto.

## 4. Contrato de primeira tentativa do render

Antes de iniciar o encode caro:

1. `preparar_render` materializa todos os vídeos pelo protocolo único;
2. `preflight-broll.mjs` exercita abertura, meio de cada cena e, para vídeos,
   entrada, +1 s e saída;
3. o marcador deve declarar o job correto, `@remotion/media`, fallback antigo
   desligado e SHA-256 igual à timeline que será renderizada;
4. qualquer divergência bloqueia antes do render;
5. o render usa quatro chunks retomáveis e concatena exatamente até a duração do
   EditPlan assinado.

Aceite do canário:

- `render.attempt == 1`;
- zero `No frame found at position`;
- zero normalização posterior ao preflight;
- 100% dos chunks concluídos na mesma tentativa;
- MP4 com vídeo+áudio, duração editorial e fiscal executado;
- tempo de render comparado ao baseline do Leopardo (1.237 s para 395,5 s).

## 5. Ordem da execução

1. validar `director/golden/current.json` e o hash do Leopardo;
2. rodar bancadas offline do pipeline, banco, contratos e render preflight;
3. criar novo `job_id`; nunca reutilizar o estado do Leopardo;
4. executar o pipeline canônico fresh com telemetria;
5. acompanhar banco e VEO em paralelo sem intervir manualmente;
6. exigir todos os gates pré-render;
7. exigir preflight vinculado à timeline exata;
8. renderizar uma única vez;
9. executar decupagem/fiscal;
10. comparar o resultado com invariantes e restrições do golden;
11. emitir placar final de vitória ou pendências, sem maquiar falhas.

## 6. Isolamento da futura otimização do VEO

A otimização do gerente VEO começa **somente depois** deste canário. Seu escopo
será formalizado numa BOX própria de política/prompt/seleção do VEO.

Essa BOX poderá mudar como o gerente cria, prioriza e acompanha pedidos VEO. Ela
não poderá alterar:

- `pipeline_canonico.py` ou a ordem da receita;
- alocação/reuso do banco;
- Maestro, contratos, leis ou EditPlan;
- gates, preflight, renderer ou fiscal;
- limites universais de duração, distância, cópia segura e elementos.

Qualquer necessidade de tocar essas fronteiras é mudança de arquitetura fora da
BOX e deve falhar revisão, não entrar como “otimização do VEO”.
