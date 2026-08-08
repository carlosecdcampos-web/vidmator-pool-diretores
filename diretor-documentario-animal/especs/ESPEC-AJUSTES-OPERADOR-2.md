# ESPEC — Ajustes do operador pós-render staged (03/08, noite)

**Status:** EM EXECUÇÃO (GO) · 4 regras do operador, diretas, sem auditoria externa.
Execução em modo SURGICAL — o que ele já aprovou (word-pop, jaguar/águia/boto nos takes,
cor, volumes) NÃO é re-sorteado. + VARREDURA de conformidade retroativa (regra 4 nova
aplicada aos clips já escolhidos; só os reprovados re-resolvem).

| # | Regra do operador (literal) | Implementação |
|---|---|---|
| R1 | "Texto atrelado à cena; o vídeo de fundo deve durar enquanto o texto durar; o texto dura o tempo que estiver sendo falado. Simples assim." | `legibilidade.py` v2: texto vai de `cena.inicio+0,3s` até `cena.fim` (a cena É a unidade da frase falada — o montar corta por frase). NUNCA atravessa cortes. Substitui o "2s/palavra atravessa cenas" (aprovado antes, reprovado agora pelo efeito órfão: "SERPENT SO ABSURD" sobreviveu à cena vermelha e ao desenho) |
| R2 | "No mapa deveria aparecer a localidade falada — 'South America'" | `detectar_mapas`: legenda = o termo geográfico FALADO (verbatim da narração); o país do atlas vira só o polígono de destaque aproximado. Correção imediata do mapa atual: legenda "South America" |
| R3 | "Boto na tela enquanto fala birds and butterflies" | (a) takes fundidos priorizam variantes POR MEMBRO do vocabulário (macaw/toucan primeiro — a query fundida gigante era a razão do hard miss); (b) take ainda omitido → a CENA PAI da janela re-resolve com atmosfera NEUTRA do bioma (nunca fica mostrando outro animal que contradiga o áudio) |
| R4 | "Documentário não tem take de desenho. NUNCA." | (a) `vet_regras` +ilustração/desenho/cartoon/animação/pintura/3D/IA = REPROVA (o vet de vídeo não tinha o veto — assimetria com o de foto; foi por aí que o desenho entrou na cena 8); (b) `broll_diretrizes` proíbe 'illustration/drawing/animation/art' nas queries; (c) guardrail determinístico remove esses termos de QUALQUER query antes da busca; (d) VARREDURA retroativa: o vet novo julga 1 frame de cada clip atual — reprovado re-resolve |

**Nota de custo/tempo (pergunta do operador):** o >1h de hoje foi OBRA (Bloco 1 + 3 bugs
de integração), não operação. Pipeline limpo do 93s ≈ 16 min. Vídeo de 25-30 min exige
espec própria de escala (batch/paralelismo/caps por minuto) ANTES de produção real —
registrado como pendência de 1ª linha junto com o replay total da fila.
