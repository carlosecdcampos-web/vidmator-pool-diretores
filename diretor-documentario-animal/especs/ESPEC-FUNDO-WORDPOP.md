# ESPEC — Fundo do word-pop temático + furo do re-vet + volumes de SFX

**Criada:** 03/08/2026 · **Status:** aguardando OK EXPLÍCITO · **Escopo:** SÓ o apontado
pelo operador — o fundo do word-pop fora de contexto (arte de recortes de papel sob
CANOPY/NOISE). *"Não precisamos fazer ele mudar o restante dos pensamentos"* — o resto
do render foi aprovado e NÃO será tocado.

---

## 1. A cadeia do bug (evidência, linha a linha)

O fundo da janela do word-pop (cena 15, 47,3–55,6s) virou um vídeo de colagem de papel
(arte suprematista) assim:

1. A query da cena era até BOA: `"Amazon river flowing dense jungle"`.
2. O resolver escolheu um vídeo; o **vet do diretor julgou "none" e reprovou** — o vet
   FUNCIONOU (log: `[15] vídeo 'none' -> trocou p/ outro vídeo (L5-fallback)`).
3. O re-resolve com `evitar` desceu a cascata até o **L5-fallback** — o ÚNICO nível que
   restou **sem nenhum gate**: ele pega o "melhor resto" do pool do Pexels
   (`best_video`) ou uma query AMPLA (`" ".join(query.split()[-2:])` = "dense jungle").
4. **O substituto não é re-julgado por ninguém** → a colagem entrou.

Dois defeitos independentes: (a) o fundo do pop competia com a frase da enumeração
(que gera pool ruim); (b) o replacement do vet passa sem vet.

## 2. R1 — Fundo do word-pop é SEMPRE footage do TEMA (regra do operador)

*"O fundo do word-pop deve ser um footage do que o roteiro se trata — floresta se
Amazônia, deserto se deserto, mar se mar."*

**Implementação (aditiva, só nas janelas de pop):**
- O **classificador do `enumeracoes.py`** (mesma chamada Gemini que já decide pop×takes)
  devolve um campo a mais: `fundo_query` — footage ATMOSFÉRICO do tema macro do trecho
  (ex.: `"amazon rainforest canopy aerial"`; num roteiro de deserto: `"sahara desert
  dunes aerial"`). Custo: zero chamadas extras.
- Para janelas modo **"pop"**, o pass grava nas cenas sobrepostas:
  `stock_query_override` = fundo_query + `prefer_video_forcado` = true (fundo de pop é
  FOOTAGE, nunca foto parada, salvo a cascata não achar vídeo aprovado).
- O **resolver** usa o override quando presente (com o MESMO vet do diretor). Fallback
  se o Gemini não devolver fundo_query: mantém a query original (degrada como hoje).
- Cenas fora de janelas de pop: **intocadas** (zero mudança no resto do pensamento).

## 3. R2 — O substituto do vet é RE-JULGADO (fecha o furo que causou ESTE take)

No `#5b` (vet de vídeo pós-resolução):
- O vídeo substituto passa pelo `vision_video_ok` também — até **2 trocas**; se nenhuma
  atingir o rigor do diretor, cai para **imagem com vet** (imagem certa > vídeo errado).
- O nível **L5** (fallback/amplo), quando o diretor tem vet ativo, julga o vídeo UMA vez:
  aceita até "weak" (é o último recurso), mas mata "none" (colagem, gente cortando papel,
  qualquer coisa claramente alheia). Sem diretor rigoroso (nichos do Piter): L5 continua
  exatamente como é.

## 3.5 R3 — Volumes de SFX parametrizados (apontamento do operador, de fone)

**Diagnóstico:** os SFX "muito altos" são herança do preset do Piter (o `doc_realista`
clonou o bloco `sfx` do `documentario` dele) com volumes HARDCODED no BrollTest.

**⚖️ POLÍTICA DE VOLUME DO OPERADOR (03/08 — permanente, "até segunda ordem"):**
> **NUNCA, NENHUM SFX acima de 0.20.** Os calibrados a 0.15 ficam SOMENTE em 0.15.
> Todos os demais: 0.20. Se algum incomodar mesmo a 0.20, o operador aponta e
> abaixa-se SÓ aquele.

| Papel | Arquivo | Vol atual | Toca | Vol FINAL (operador) |
|---|---|---|---|---|
| transição de cena | click_camera/click_mech (Piter) | **0.80** hardcoded | ~19× | **0.15 — fixo** |
| entrada de texto | ding_bright/chime_sparkle (Piter) | **0.50** hardcoded | ~6× | **0.20** |
| flash dos micro-takes | whoosh_fast_02 (nosso) | 0.35 | 4× | **0.20** |
| clique do word-pop | ui_mouse_04 (calibrado) | **0.15 — fixo** | 7× | intocado |
| riser/glitch/first/typing/paper/cta (quando ativos) | — | 0.5/0.45/0.7/0.09/0.1/0.4 | — | knobs; **teto 0.20** |

**Implementação:**
- Bloco `sfx_vols` no preset + **`sfx_vol_max: 0.20`** (TETO com clamp na leitura — pega
  qualquer SFX, inclusive papéis futuros/esquecidos; à prova de regressão de volume).
- `preparar_render` injeta no render json; BrollTest lê `vol(papel)` = 
  `min(sfx_vols[papel] ?? valor_atual, sfx_vol_max ?? ∞)`. Nada mais hardcoded.
- Defaults SEM o bloco = valores atuais do Piter → nichos dele byte a byte iguais.
- A narração e a TRILHA (bed musical) NÃO são SFX — ficam fora do teto.

## 4. Gates de validação (novo render, NOVO)

- [ ] Fundo da janela do word-pop (47–53s) = footage da FLORESTA (tema macro), em vídeo
- [ ] Word-pop em si intacto (caixa alta, clique 0.15, timestamps)
- [ ] Micro-takes dos animais (60–67s) intactos (aprovados — não re-resolver à toa;
      cache desta rodada é MANTIDO: a regra do zero foi da rodada anterior e já cumpriu
      seu papel; aqui só a janela do pop re-resolve por causa do override novo)
- [ ] Nenhuma cena fora das janelas de pop muda de take
- [ ] SFX de transição/entrada audivelmente mais discretos (0.22/0.16); clique intocado
- [ ] Regressão: NICHO=documentario → no-op (overrides e sfx_vols só existem no doc_realista;
      sem o bloco, o BrollTest usa os valores atuais do Piter byte a byte)
