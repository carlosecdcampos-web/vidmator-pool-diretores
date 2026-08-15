# Plano operacional — otimização antes do próximo E2E

**Decisão do operador:** 15/08/2026
**Estado:** blocos 1–4 implementados e verdes offline; frota é o próximo gate
**Golden vigente:** Leopardo-das-neves

## Resultado dos blocos 1–4 — 15/08/2026

- fila viva geracional: um `veo_gerar.py`, misses anexados antes do único ciclo,
  sem relançamento pós-merge;
- alocação por `source_resolvability`: fração é preferência do diretor, não cota;
- CLIP antes do Vision em OFF/SHADOW/ENFORCE; permanece OFF porque o venv isolado
  `torch/open_clip` ainda não foi instalado/provado neste laptop;
- cache de normalização por conteúdo+contrato e heartbeat do render independente
  de newline;
- 52 bancadas do Director verdes, incluindo E2E offline; nenhum VEO/render real.

## Ordem obrigatória

1. Implementar e provar os blocos 1–4 no corpo canônico do Vidmator:
   - uma única fila VEO viva para alocação inicial e misses do banco;
   - alocação por resolubilidade real do beat, sem cota cega de 40%;
   - ensaio do prefilter CLIP local antes do Vision;
   - cache de normalização e heartbeat real do render.
2. Ensaiar a frota VEO:
   - registrar perfil, conta, porta CDP e diretório isolado;
   - testar duas contas com 8–12 pedidos;
   - validar divisão, locks, retomada, isolamento e merge;
   - expandir para quatro contas somente após aprovação;
   - provar 40 takes como 10 por conta, sem cruzamento.
3. Executar ensaio integral sem custo:
   - replay do banco;
   - VEO simulado;
   - golden do Leopardo;
   - uma única fonte de timeline;
   - um único EditPlan;
   - nenhuma rota histórica;
   - todos os gates verdes.
4. Somente depois, autorizar um novo E2E real.

## Critérios de vitória do novo E2E

- banco em até 9 minutos;
- no máximo um download final materializado por cena;
- uma única passagem VEO;
- quatro perfis trabalhando paralelamente, se autenticados;
- zero reenvio depois do teto;
- preflight aprovado na primeira passagem;
- render aprovado na primeira tentativa;
- retomada somente da etapa que falhou;
- telemetria completa por etapa.

## Limite do pool

Este documento guarda decisão e política. A implementação pertence ao único
caminho executável do repositório `vidmator`; nenhum runner deve nascer neste pool.
