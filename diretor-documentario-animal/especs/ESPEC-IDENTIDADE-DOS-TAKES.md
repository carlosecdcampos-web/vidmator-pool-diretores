# ESPEC — IDENTIDADE DOS TAKES **v2** (pós-auditoria externa GPT-5.6-sol)

**v2 em:** 03/08/2026 · **Status:** EM EXECUÇÃO (GO do operador; render aguarda GO próprio)
**O que mudou da v1:** a auditoria externa concluiu — corretamente — que a v1 não devia
ser implementada como estava. Três defeitos de desenho acatados na íntegra:
1. A `entidade` era declarada pelo MESMO LLM/chamada que acabara de omiti-la (falha
   correlacionada) → agora existe um **compilador de contrato visual** em etapa separada,
   com prompt próprio.
2. O fallback "primeiro substantivo + 1 modificador" era heurística de string frágil
   ("pink river dolphins" quebra) → agora **variantes de query derivadas do contrato**,
   com checagem determinística de que o sujeito canônico sobrevive em todas.
3. "Criatura genérica → espécie concreta" arriscava **falsidade documental** ("monsters"
   metafórico virando jaguar afirmado) → agora `match_mode`: espécie nomeada = `exact`;
   classe real ("predators") = `class_member` com membros plausíveis do bioma; metáfora/
   negação/comparação = NÃO obriga sujeito (atmosfera).

## 1. Arquitetura da v2 (a cadeia nova, só no diretor `doc_realista`)

```
fala da cena ─► [CONTRATO VISUAL] (compilador próprio, 1 chamada batch, cacheado no timeline)
                 required_subjects{canonical, match_mode, representation, aliases,
                 allowed_members}, forbidden, visual_intent, ambiguity
                       │
     cena SEM sujeito obrigatório ──► cascata atual (atmosfera — intocada)
     cena COM sujeito obrigatório ──► [RESOLVER DE IDENTIDADE] (caminho novo):
        variantes de query (todas preservam o canonical):
          Q0 query original (sujeito garantido por guardrail determinístico)
          Q1 canonical + modificadores   Q2 canonical + bioma   Q3 canonical   Q4 alias
        por variante: candidatos de VÍDEO → [VET DE CONTRATO] multi-frame →
        PASS aceita · FAIL/UNCERTAIN nunca entra → esgotou vídeo: IMAGEM da entidade
        (mesmo vet) → esgotou tudo: ABSTENÇÃO (hard_miss registrado)
                       │
              [MANIFESTO] decision_manifest.json: contrato, queries, cada candidato
              julgado (decisão+violações), escolha ou hard_miss — nasce o registro
              auditável (modos replay/surgical = próxima espec)
```

**Pós-condição (invariante, não disciplina):** no caminho de identidade, só entra mídia
com julgamento PASS registrado; candidato reprovado NUNCA é reaproveitado; UNCERTAIN
não passa (rigor do diretor).

## 2. O vet de contrato (substitui o {ok:bool} de frame único)

- **Multi-frame:** 3 frames do SUBCLIP efetivamente usado (~20/50/80% da janela), numa
  única chamada Vision (payload multi-imagem; frame único era cego a sujeito ausente
  no trecho usado). Imagem estática: 1 frame.
- **Saída estruturada:** `decision PASS|FAIL|UNCERTAIN` + `violations[]`
  (missing_subject, wrong_species, wrong_representation, forbidden_human, illustration,
  statue, book_or_document, unrelated_city, insufficient_evidence...) + evidência curta.
- **Regras herdadas do prompt auditado:** exact exige A espécie (habitat genérico, cidade,
  estátua, desenho, taxidermia = FAIL); class_member aceita membro da classe SEM exigir
  espécie particular; representation live_real_animal mata depiction; mood não compensa
  sujeito ausente; evidência insuficiente = UNCERTAIN (que reprova).

## 3. Decisões da v2 onde ADAPTEI a auditoria (com justificativa)

- **Abertura:** acatado que "roubar footage de outra cena" viola semântica → a abertura
  sem vídeo aprovado agora busca um **establishing shot do tema** (query do bioma, com
  vet). O roubo NÃO morreu: vira último-último recurso com log forte — porque a lei do
  operador ("NUNCA abrir com imagem") prevalece sobre a elegância. Hierarquia: establishing
  aprovado > roubo logado > imagem (nunca).
- **Hard-miss de CENA (não micro-take):** abstenção total deixaria tela preta — pior que
  o erro. Degradação honesta: cena sem sujeito encontrável cai para **atmosfera do bioma
  com vet de adequação** + `hard_miss` no manifesto (não mente espécie; não fura a
  continuidade). Micro-take sem mídia → take OMITIDO (cena pai validada continua) — aqui
  a abstenção pura é acatada.
- **Vocabulário de bioma:** nasce `biomas.json` local mínimo (amazon: predators = jaguar,
  black caiman, green anaconda, harpy eagle) usado como allowed_members preferencial;
  o Gemini complementa quando o bioma não está no arquivo. Revisão/expansão = próxima espec.
- **Contradição "NEVER objects":** broll_diretrizes refinada para "unrelated objects/
  books/studio props" — objeto documental exigido por contrato é legítimo.
- **L5-amplo:** morre para cenas de identidade (nunca chegam lá — o caminho novo não
  passa por ele); para cenas de atmosfera ganha o gate espelho do L5-fallback (mata none).

## 4. O que fica para a PRÓXIMA espec (estrutural, proposta §6 da auditoria — registrada)

`ESPEC-CONTRATO-VISUAL-EXECUCAO-REPRODUTIVEL.md` (backlog): ValidatedResolver completo
para TODAS as cenas · decision manifest com modos replay/surgical/fresh/production ·
P1–P18 como suíte de fixtures automatizada · fault injection/testes de propriedade ·
auditoria final da composição (preview 360p) · política de proveniência (SearXNG por
licença, não por watermark) · métricas operacionais · seleção comparativa multi-fonte
com contact sheets · avaliação de mixagem final. Nesta v2 nasce só o EMBRIÃO do manifesto.

## 5. Critérios de aceite da v2 (inclui controles NEGATIVOS)

- [ ] Contrato da cena "hides monsters, canopy hides predators" = `class_member`
      (predador do bioma), NUNCA uma espécie inventada afirmada
- [ ] Contrato de fala com espécie nomeada = `exact` com canonical/alias
- [ ] **Controle negativo REAL:** o clip da CIDADE (o próprio arquivo que entrou no take
      do jaguar) submetido ao vet de contrato do jaguar → FAIL com violação coerente
- [ ] Toda variante de query gerada contém o canonical/alias (checagem determinística)
- [ ] Candidato FAIL/UNCERTAIN ausente da timeline final (manifesto prova)
- [ ] Micro-take sem mídia aprovada → omitido (hard_miss), nunca o reprovado
- [ ] decision_manifest.json existe com contrato/candidatos/decisões por cena de identidade
- [ ] Regressão: NICHO=documentario → nenhum comportamento novo (tudo gated)
- [ ] Gemini dentro do free tier no vídeo de referência
