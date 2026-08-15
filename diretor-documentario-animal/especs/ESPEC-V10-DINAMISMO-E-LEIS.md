# ESPEC-V10 — O REFINO DO DINAMISMO (leis ditadas pelo operador, 08/08 tarde)

Contexto: v9 saiu "pobre de elementos" ("uma sequência de slides de PowerPoint").
Causa raiz 1: o passe `edicao` (Diretor de Edição) e `legibilidade` NÃO estavam na
lista PASSES do producao.py — a v9 rodou SEM o maestro (acervo_texto=0). Corrigido:
ambos agora na cadeia canônica. Causa raiz 2: mesmo com o maestro, os knobs eram
conservadores (66 recusas vs 40 aceitos). Este é o aperto, número a número, do dono.

## L1 · LEI DO TETO DE 6s DO TAKE VEO (sem congelar, JAMAIS)
Palavras do operador: "NENHUM MP4 vindo do VEO deve ter duração na edição de mais
de 6s (o vídeo completo tem 8s; passou disso fica rodando em loop, fica feio).
Pode ficar menos; é OBRIGATÓRIO nenhum ultrapassar. NÃO deve congelar NUNCA — a
DIVISÃO DE CENA já deve fazer o cálculo para passar a quantidade EXATA de vídeos
que o VEO deve preparar."
- Cena veo_video com janela >6s ⇒ dividir o visual em ⌈janela/6⌉ sub-takes
  (cada um ≤6s; prompts distintos respeitando a Lei do Ângulo).
- O PEDIDO nasce com a conta exata (a alocação/divisão informa n_takes por cena).
- v9 atual: 13 cenas violam (janelas 6,0-7,1s) ⇒ +13 takes a gerar.
- PROIBIDO: freeze/congelamento de frame como remédio (vetado explicitamente).

## L2 · LEI DAS APRESENTAÇÕES DOS 3 PRIMEIROS MINUTOS (molduras film/polaroid/quadro)
- Minuto 1 (obrigatório): 1-2 film + 1 polaroid (sem quadro). Rodízio: F→P→F.
- Minuto 2 (mínimo): 1-2 film + 1 polaroid + 1 quadro.
- Minuto 3: 2 quadros + 1 polaroid (sem film).
- ADJACÊNCIA PROIBIDA NA SEQUÊNCIA (nos DOIS sentidos): film↔film e film↔quadro.
  ⚠️ REGISTRO DE VERGONHA: o operador plantou "quadro–film" num exemplo como TESTE
  e o agente engoliu. A polaroid é o separador universal: F→P→Q→P→F→...
- NUNCA apresentações em cenas subsequentes: ≥1 cena de respiro entre aparições.
- Pós-3min: SEM piso por minuto (lei ≥1/min RETIRADA pelo operador). A presença é
  guiada pela densidade global (L3.3).

## L3 · KNOBS DITADOS
1. Orçamento (D5): 1 elemento / **20 palavras** (era 35). ~75 elementos no vídeo de 8min.
2. Respiro: calibrar para o nível LOGO ABAIXO da distribuição definida — "não pode
   retirar NENHUM elemento da distribuição correta; o que passar dela, pega".
   (Implementação: respiro_s reduzido até recusas-por-respiro ≈ 0 na bancada,
   mantendo-se como guarda contra excesso.)
3. Molduras: densidade **1-1,5/min** (acima do benchmark do Piter 0,8-1,2), teto 2/min.
4. Cotas por família: são FRAÇÕES do orçamento (escalam automaticamente com o D5
   novo) — dúvida do operador respondida; sem mudança estrutural.
5. Infográficos: "quero SEMPRE, não remova." Investigar os 9 removidos (I16, campo
   faltante) e garantir presença.
6. Legibilidade: ligar o knob no preset do nicho (estava em no-op).

## L4 · NORMALIZAÇÃO DE ASSETS — PLANO C (aprovado)
- B como lei: PORTA DO POOL normaliza (30fps CFR h264 yuv420p seekável) na entrada
  (veo_entrega + resolver/cache) — cada asset conformado 1× na vida.
- Normalizar os 55 não-conformes existentes agora (43 VEO 24fps + 3×25fps + 1×60fps
  + 8 flags).
- A como barreira: conferência no preparar fica (custo zero quando tudo conforme).

## Bancada obrigatória antes do render
Re-rodar o maestro e apresentar: texto/min por minuto · molduras/min e sequência
(ledger) · recusas por respiro (~0) · infográficos presentes · zero take VEO >6s.

## L5 · LEI DA SUBSTITUIÇÃO DE IDENTIDADE (ditada 08/08 ~18h, após a 1ª quarentena)
Palavras do operador: "ao invés de dar erro por causa de 11 takes num vídeo grande
desse, ele pode colocar no lugar outro take da MESMA IDENTIDADE que deveria ser,
que tenha nos assets — DESDE QUE NÃO SEJA CENAS SUBSEQUENTES: se faltar a cena 70,
pode pegar outra que seja da mesma família, desde que NÃO seja a 69 ou a 71 — OU
footage de PAISAGEM."
- Cena reprovada no portão por IDENTIDADE (sujeito obrigatório resolvido por
  caminho fraco) não quarentena o vídeo: recebe SUBSTITUIÇÃO —
  1º) take da MESMA família/espécie já nos assets (pool/timeline), NUNCA das
      cenas adjacentes (±1);
  2º) fallback: footage de paisagem/habitat (atmosfera).
- A quarentena continua existindo para o que a substituição não cobrir.
- Status: REGISTRADA; implementação no portão/preparar na próxima sessão de
  código (o render v10 de hoje saiu em quarentena pela regra antiga).

## DECISÕES DO OPERADOR (09/08 — arbitragem pós-review do Codex)
- **Word-pop × infográfico: O INFOGRÁFICO VENCE.** A legibilidade não pode apagar
  infográfico por colisão com word-pop (era o achado #6 do Codex).
- **Quota das molduras 3min que não cabe: OPÇÃO A com esforço máximo.** "Vídeo sai
  com a cota parcial, aviso no log, NUNCA TRAVA PRODUÇÃO — mas a lei não é
  sugestão: é máximo-esforço; se não der, não deu." O passe esgota todas as cenas
  elegíveis e combinações do rodízio antes de aceitar parcial.
- Densidade pós-3min alvo 1-1,5/min (ditada 08/08) SUPERSEDE a frase "lei ≥1/min
  retirada" da L2 — o achado #2 do Codex era leitura da camada histórica; texto
  consolidado: 3 primeiros minutos = quotas obrigatórias; pós-3min = alvo suave
  por minuto, sem trava.
