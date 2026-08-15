# Arquitetura do Pool de Diretores — a doutrina (operador, 07/08/2026)

## O princípio fundamental

**Há um único motor e um único caminho de execução no `vidmator`.** O pool não
contém runners, recuperadores ou pipelines alternativos. Cada diretor fornece
política/configuração ao mesmo compilador editorial: permissões, proibições,
densidade, linguagem visual e contratos do nicho. Qualidade, isolamento por job,
ledger, sourcing, reuso seguro, EditPlan, gates e render são mecanismos universais
do corpo e nunca são duplicados por diretor.

**CADA DIRETOR CARREGA JUNTO DE SI:**
1. as leis do nicho;
2. as ESPECs;
3. a estrutura invisível (o que muda de diretor pra diretor);
4. **e a INTELIGÊNCIA DA EDIÇÃO DO SEU VÍDEO** — o estilo de edição é DO diretor.

*"Cada diretor terá seu próprio estilo de edição. Terá diretores que não terão o
take de abertura. Nós refinaremos diretor por diretor... O ajuste na edição do
vídeo de um diretor NÃO DEVE ALTERAR A DIREÇÃO DE EDIÇÃO DE OUTRO DIRETOR,
NUNCA."* — operador, 07/08/2026.

O projeto deve permitir **diversos diretores diferentes com suas respectivas
inteligências de edição**, coexistindo sem contaminação.

## Divisão corpo × cérebro

| repo | papel |
|---|---|
| [`vidmator`](https://github.com/carlosecdcampos-web/vidmator) | o **CORPO**: engine, passes, inteligência do Flow (driver, ciclo, download linha-a-linha, coleção, curador) — o MOTOR genérico que todo diretor compartilha |
| `vidmator-pool-diretores` (este) | o **CÉREBRO** de cada nicho: leis + ESPECs + estrutura invisível + **estilo de edição** — uma pasta por diretor |

## Como o isolamento se materializa (direção de implementação)

O corpo já tem o embrião disso: `director/preset.py` carrega comportamento POR
NICHO (ex.: `banco_epoca` só liga em nichos que o declaram). A evolução
obrigatória, a ser feita ANTES do segundo diretor entrar em produção:

1. **Todo knob de estilo vira PRESET do diretor** (nunca constante no código):
   abertura ligada/desligada e sua duração, cadência-alvo do dinamismo, famílias
   de overlay permitidas e cotas, teto de look mono, uso de mapas/pessoas/datas,
   política de trilha/SFX, leis do VEO do nicho (GUIA), etc.
2. O corpo LÊ o preset do diretor ativo; o código dos passes e a rota operacional
   permanecem únicos e genéricos.
3. Ajustar o estilo de um diretor = editar o preset/ESPEC NA PASTA DELE no pool —
   nenhum outro diretor é tocado. Mudança no CÓDIGO do corpo só é legítima quando
   é mecanismo novo (disponível a todos via knob), nunca estilo hard-coded.
4. Regra de revisão: um diff no corpo que mude o VISUAL de um diretor existente
   sem knob novo = violação da doutrina; volta como preset.
5. Regra de execução: nenhum arquivo deste pool pode chamar produção, VEO ou render.
   O pool descreve decisões; somente o pipeline canônico do Vidmator as executa.

## Convenção de trabalho

- Uma pasta por diretor: `diretor-<nicho>/`.
- Operador edita um diretor e diz **"commit e push"** → o push vai na pasta
  daquele diretor, neste repo.
- Diretor novo = clonar a estrutura do `diretor-documentario-animal/` (o molde,
  validado em produção) e reescrever o cérebro do nicho + o estilo de edição.
