# ESPEC — Ajustes pré-rodada 3 (aprovações da rodada 2)

**Criada:** 03/08/2026 · **Status:** EM EXECUÇÃO (GO dado; rodada 3 só após OK EXPLÍCITO do operador)
**Contexto:** rodada 2 aprovada ("MUITO MELHOR"). Quatro regras novas do operador, todas
permanentes no Director, todas aditivas/gated. Rodada 3 (SearXNG) espera o OK.

## Regras (decisões fechadas do operador, 03/08)

| # | Regra | Status |
|---|---|---|
| R1 | **NUNCA abrir vídeo com imagem estática** — o 1º beat é SEMPRE footage. A rodada 1 abriu com imagem+movimento (reprovado); a rodada 2 abriu com footage (aprovado — "É EXATAMENTE ASSIM") | esta espec implementa |
| R2 | **Word-pop SEMPRE em CAIXA ALTA** — só nessa animação; nenhum outro texto muda | ✅ já implementado no teste de calibração (BrollTest, EnumWord, comentário da regra no código) |
| R3 | **SFX oficial do word-pop = "Clique do mouse" do operador** — catalogado como **SFX-0127** (`acervo/sfx/ui/ui_mouse_04.mp3`, trim + pico −3 dBFS); regra apontada no manifesto do acervo E no `vidmator.config.json` (`sfx.enum_click`). O `ui_pop_01` foi reprovado de ouvido | ✅ já implementado |
| R4 | **Volume do clique = 0.15** — escolhido de ouvido na escada 0.15/0.20/0.25/0.40/0.55 (5 renders de calibração) | esta espec trava |

## R1 — implementação (resolver_cascata.py, gated)

Knob de preset: `"abre_com_video": true` — SÓ nos presets nossos (`natureza`, futuros).
Sem o knob (nichos do Piter): zero mudança.

Duas camadas de garantia:
1. **Preventiva**: `prefer[beat0] = True` — o 1º beat entra na cascata mirando vídeo
   (L3v-mix primeiro), independente do sorteio Bresenham do `video_frac`.
2. **Pós-check** (após resolução + reuse-fill): se o 1º beat mesmo assim saiu `imagem`
   → re-resolve com `evitar={path}` e `prefer_video=True`; se AINDA imagem → **rouba o
   footage de outra cena já resolvida** (`abre-swap` — reuso é aceitável na abertura,
   imagem não); só se o timeline INTEIRO não tiver nenhum vídeo, loga AVISO e desiste
   (caso teórico; o L5-fallback amplo sempre acha vídeo).

## R4 — implementação (volume na cadeia de config)

`vidmator.config.json` → `sfx.enum_click_vol: 0.15` → `vmconfig.ENUM_CLICK_VOL` →
`preparar_render` escreve `enum_click_vol` no render json (preset pode sobrepor por nicho:
`_preset.get("enum_click_vol")` vence) → `BrollTest` usa `timeline.enum_click_vol ?? 0.15`.
Nada hardcoded no componente.

## Validação (gates desta espec)

- [ ] Resolver re-rodado sob `natureza`: 1º beat = vídeo (era vídeo na rodada 2 — a regra
      confirma e protege; o log mostra a regra ativa)
- [ ] Diff do timeline pós-resolver: footage das demais cenas INTACTO (cache idempotente)
- [ ] Render completo final: caixa alta + SFX-0127 a 0.15 + cor + dinamismo → **OK
      explícito do operador** = destrava a rodada 3
- [ ] Gate de não-regressão: sem o knob `abre_com_video`, o resolver não executa nenhuma
      das 2 camadas (código inspecionável; nichos do Piter imunes)

## Fora desta espec (registrado)
- Outras correções visuais que o operador citou sem detalhar: ele acredita que o SearXNG
  (rodada 3) resolve — reavaliar após a rodada 3.
