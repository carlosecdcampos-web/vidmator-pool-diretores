# Diretor de Documentário Animal

O primeiro diretor do pool — e o MOLDE de todos os futuros (REGRA-MÃE: diretor
novo = clonar esta estrutura + adaptar só as especificações do nicho, nunca do
zero). Estado em 07/08/2026: **v7 PIRANHA entregue e aprovado** ("PARABÉNS PARA
NÓS, ficou muito muito bom"), **ESPEC-V8 executada e provada em bancada**, leis
do VEO validadas ao vivo pelo operador.

## Divisão de responsabilidade (decisão do operador, 07/08)

| repo | papel | o que vive aqui |
|---|---|---|
| **este (pool-diretores)** | o CÉREBRO do nicho | leis do nicho, ESPECs, relatórios, estrutura invisível |
| [`vidmator`](https://github.com/carlosecdcampos-web/vidmator) (branch `acervo-mapas-chapter`) | o CORPO | engine, passes do director, e a INTELIGÊNCIA DO FLOW (`_import/2026-08-06-veo-flow`): driver, ciclo, download linha-a-linha, coleção, curador, socorro, entrega |

Linha divisória: o MECANISMO (fiscal, rodízio, ciclo, download) é infra do
Vidmator e serve a todos os diretores; o CONTEÚDO das leis (espécie exata, bioma,
contemplativo, ângulos p/ animais...) é deste diretor.

**⚖️ DOUTRINA DO POOL (ver `../ARQUITETURA.md`)**: a INTELIGÊNCIA DE EDIÇÃO
pertence ao diretor — o take de abertura, o dinamismo de texto, o acervo de
variantes, o contrato visual: tudo aqui descreve O ESTILO DESTE DIRETOR. Outro
diretor pode não ter abertura, ter outra cadência, outras famílias de overlay.
*"O ajuste na edição de um diretor NÃO DEVE ALTERAR a direção de edição de outro
diretor, NUNCA"* — todo knob de estilo vira preset do diretor, nunca constante
no código do corpo.

## As leis do funcionário do VEO (nicho animal — validadas em produção no v7)

O GUIA completo vive em `vidmator/_import/2026-08-06-veo-flow/veo_pedido.py`
(autor = LLM; fiscal `_validar_prompt` reprova e devolve o motivo). Resumo:

1. **Espécie EXATA** — genérico proibido ("predator" virou TUBARÃO, "fish" virou NEMO)
2. **BIOMA em todo prompt** (cabeçalho SPECIES:/BIOME: no lote)
3. **1 sujeito por take**; interação entre espécies não vai pro VEO
4. **Só contemplativo** — ação/caçada vira o instante quieto da mesma história
5. Variedade obrigatória (dois takes confundíveis = falha)
6. **1-em-4 atmosfera** sem animal (o doc precisa respirar)
7. **Câmera em RODÍZIO** (10 movimentos documentais; nunca o mesmo 2× seguidos)
8. Estilo verbatim: "cinematic documentary look" + frase tranquila
9. "ambient natural sound only" (negação gera o negado)
10. Enquadramento sempre positivo
11. Uma frase fluida, 25-45 palavras
12. **ÂNGULO + DIREÇÃO em rodízio** (8 ângulos, 4 direções; nunca repetir em
    consecutivos — validada ao vivo: "as imagens estão lindas, ângulos diferentes,
    ponto de vista diferente")

Mais: **carimbo "CENA NNN" REVOGADO** (o gerador desenha o que lê — plaquinha
queimada em ~20% das imagens); ilustração gerada SÓ para anatomia (cutaway
semi-realista, rótulos ditados M1); medida é da lei dos números, não da ilustração.

## Fluxo VEO canônico (ordem do operador, palavra por palavra)

Projeto por canal → coleção por vídeo (escola do Piter) → modelo conferido NA
RAIZ → entrar na coleção PELO CARD → enviar em rajadas → download LINHA-A-LINHA
(par imagem+prompt da mesma linha, nome do arquivo = prompt, zero cliques) →
casamento prompt→cena OFFLINE → curadoria: *"usa tudo que está pronto; regenera
SÓ quem tem CENA+número na tela, uma vez; se vier queimada de novo, entra outra
pronta que sirva; ponto. Zero geração para buracos."*

## Documentos

**Na raiz (os vivos):**
- `ESPEC-V8.md` — as leis do veredito do v7 (cor pontual, abertura limpa,
  expurgo, lock, ângulo/direção), executadas e provadas em bancada
- `ESPEC-INCIDENTE-CICLOS-CONCORRENTES.md` — a noite das colisões e do download:
  cada causa com sua blindagem (leitura OBRIGATÓRIA antes de mexer no ciclo)
- `RELATORIO-V7-PIRANHA.md` — catálogo completo de erros da primeira produção fresh

**`especs/` — TODA a inteligência de DIREÇÃO/EDIÇÃO construída (30 ESPECs):**
o take de entrada e a abertura (`ESPEC-ABERTURA-E-TAKES`), o dinamismo de texto
(`ESPEC-DINAMISMO-TEXTO`, `ESPEC-DINAMISMO-E-CORRECOES`), o acervo de overlays
com 40 variantes e rodízio (`ESPEC-ACERVO-40-VARIANTES`), o contrato visual
reproduzível, a validação/barreira, o zelador de produção, os ajustes ditados
pelo operador rodada a rodada (V3→V7) — a memória dirigida completa do nicho.

**`relatorios/`** — relatórios de implementação, auditorias do acervo, estudos
de takes, catálogo de blindagem de produção, pendências e runbook.

## Pendências vivas

- Integrar o download linha-a-linha + casamento offline no `veo_ciclo` (a rota
  canônica já está provada; falta a costura no ciclo de produção)
- v8 fresh: aguardando o operador definir o animal
- Validação de ouvido do ASMR -30 (C4) no v8
- Personagem/avatar: gatilho armado e TRAVADO (VM_VEO_AVATAR_OK) até sinalização
  explícita do operador
