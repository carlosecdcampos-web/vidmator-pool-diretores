# ESPEC — Sistema de Mapas Dinâmicos (Passo 8 do acervo)

> ⛔ **OBSOLETA — arquivada em 04/08/2026 por decisão do operador:** *"era a versão inicial
> da animação de mapas, essa espec já está obsoleta"*.
>
> **A espec vigente dos mapas é o §7.2 de
> [ESPEC-ACERVO-40-VARIANTES.md](ESPEC-ACERVO-40-VARIANTES.md)** — inventário dos 21
> componentes, seleção de 14 para produção, cor `#f59e0b`, filtro de elegibilidade e valores.
>
> ⚠️ **MAS o código descrito aqui EXISTE e funciona** — não jogar fora sem olhar. Em
> `acervo/mapas/` estão o engine (`map_core.js` + `map_render.py` Playwright), 12 receitas e
> **10 previews renderizados**. Ele cobre **8 modos de transporte** (avião, navio, trem,
> carro, bicicleta, cavalo, a pé, exército) com rota marítima A* e penalidade de costa —
> coisas que **nenhum** dos 21 componentes React da seleção faz.
>
> Ou seja: a espec como *plano* está obsoleta (a arquitetura escolhida foi a dos componentes
> React do `AcervoMapas.tsx`), mas o *artefato* é uma capacidade real e não coberta. Se um
> dia o Diretor precisar de "a expedição foi de navio do Brasil à Austrália", é aqui que isso
> existe. Registrado no §7.2.3 da espec vigente para não se perder.

## 0. As 4 decisões do operador (20/07)

1. **Protótipo NOSSO independente primeiro**, desenhado plug-and-play pro
   Remotion do Piter (estratégia da casa).
2. **Identidade dark universal** com cor de destaque parametrizada — ajuste
   fácil por canal no futuro.
3. **Rotas com TODOS os modos**: avião, navio, trem, carro, bicicleta,
   cavalo, exército (setas de guerra), a pé... versatilidade total de nicho.
4. **2D primeiro**; satélite na v2 renovando a pirâmide do Piter com NASA
   GIBS (domínio público).

## 1. O que já existe (base do Piter, em `_referencias\vidmator-director`)

- `MapAnimation.tsx` — zoom mundo→país + highlight + pin + legenda
  (d3-geo + world-atlas/Natural Earth, Remotion). SEM rotas, SEM estados.
- `SatelliteZoom.tsx` — zoom realista por pirâmide de 6 níveis de imagem
  (tiles pré-baixados) com HUD scanner.
- `detectar_mapas.py` — LLM acha lugares no roteiro + coord + timestamp da
  fala → escreve `mapas[]` no timeline.json. (A inteligência JÁ existe.)

## 2. Arquitetura v1 (nossa, independente, plugável)

```
Vidmator\acervo\mapas\
  _engine\
    map_render.py        ← driver Playwright frame-exact (padrão gl_render)
    map_page.html        ← página d3-geo (SÓ DOM/render)
    map_core.js          ← NÚCLEO PURO: projeção, câmera, arco, timing
                            (zero dependência de DOM — roda igual num
                            componente Remotion; é O arquivo do plug)
    tema.json            ← tokens do dark universal (ver §4)
    modos.json           ← modos de rota (ver §5)
  _dados\
    countries-110m.json / countries-50m.json   (world-atlas — NE, PD)
    admin1-10m.topojson                        (NE admin-1: ESTADOS, PD)
    capitais.json                              (NE populated places → so
                                                capitais: nome, país, [lng,lat])
  _previews\  ·  manifesto.json (MAP-NNNN pros clipes gerados)
```

**Plug-and-play com o Remotion (decisão 1):** três amarras que garantem
porte sem retrabalho:
- MESMO dado (world-atlas TopoJSON que o MapAnimation já importa);
- MESMO contrato de entrada: `mapa_job.json` (§6) espelha os props do
  MapAnimation (pais/coord/legenda/durFrames) e o estende (rotas/estados);
- Toda a matemática (câmera, easing, great-circle, progresso do traço) vive
  em `map_core.js` puro — o componente Remotion do Piter importa esse
  arquivo e só troca o "desenhar" (SVG React em vez de canvas nosso).

## 3. Os 3 blocos (a API)

1. `mapa_destaque` — mundo→país OU estado, preenchimento na cor de
   destaque com glow, legenda serif. Zoom lento easing cúbico (~0.15→0.49
   da duração, keyframes proporcionais — herdado do MapAnimation).
2. `mapa_rota` — pin origem (drop + ripple) → arco great-circle
   (d3.geoInterpolate) desenhando com tracejado animado → ícone do MODO
   viajando na ponta → pin destino. Convenção: cidade = capital do país
   (lookup offline em capitais.json), override por coord explícita.
3. `mapa_ponto` — pin + card de legenda/imagem num local exato (paridade
   com o que o Piter já tem).

## 4. Tema (decisão 2 — dark universal, cor plugável)

`tema.json` (tokens únicos, canal pode ter override futuro):
```json
{
  "bg": "#0b0e13", "oceano": "#0b0e13", "terra": "#1a212b",
  "borda": "#3a4656", "borda_peso": 0.7,
  "destaque": "#e0452e", "destaque_glow": 0.35,
  "rota": "#e8d9a0", "rota_tracejado": [8, 6],
  "pin": "#e0452e", "texto": "#e8e4da", "fonte": "serif",
  "grain": true, "vignette": true
}
```
- `destaque`/`rota`/`pin` são OS pontos de individualidade por canal
  (trocar 3 cores = identidade nova). Override: `tema.<canal>.json`.
- `grain`/`vignette` aplicados NO PÓS (FFmpeg) usando o NOSSO acervo
  (film_grain multiply + vinheta) — o look "documentário famoso".

## 5. Modos de rota (decisão 3 — todos)

`modos.json` — cada modo = ícone + estilo de linha + som do banco:
| modo | ícone | linha | sfx_par (banco) |
|---|---|---|---|
| aviao | plane | arco alto tracejado | machine_plane_01 |
| navio | ship | arco baixo tracejado | ambience_ocean_01 (curto) |
| trem | train | quase reto tracejado | machine_train_01 |
| carro | car | quase reto tracejado | machine_engine_01 |
| bicicleta | bike | quase reto pontilhado | foley_footsteps? (leve) |
| cavalo | horse | quase reto pontilhado | foley_footsteps_02 |
| a_pe | footprints | pontilhado curto | foley_footsteps_01 |
| exercito | seta de guerra | SETA SÓLIDA GROSSA crescendo (estilo mapa militar) | drone_dark / machine_marching |

- Ícones: **Lucide (licença ISC — sem atribuição)** cobre plane/ship/train/
  car/bike; o que faltar (cavalo, pegadas, seta militar) a gente GERA
  (SVG próprio — via validada 3x no acervo). Ícone anda pela curva com
  rotação tangente ao caminho.
- Curvatura por modo: aéreo = arco alto; marítimo = arco suave; terrestre =
  arco raso (convenção visual de documentário; rota por estradas reais =
  fora de escopo v1, anotado pra futuro com OSRM se um dia precisar).

## 6. Contrato de entrada (`mapa_job.json`) — o que o Director gerará

```json
{ "tipo": "rota", "modo": "aviao",
  "origem": {"pais": "Brazil"}, "destino": {"pais": "United States of America"},
  "legenda": "Rio de Janeiro -> Washington", "dur_s": 6.0,
  "tema_override": null }
```
(destaque: `{tipo, local:{pais|estado}, legenda, dur_s}` · ponto:
`{tipo, coord, legenda, imagem}`) — nomes de país no padrão do atlas EN
(mesma convenção do detectar_mapas.py do Piter → plug direto).

## 7. v2 — satélite (decisão 4)

- Script `gibs_pyramid.py`: dado [lng,lat], baixa N níveis de zoom da
  **NASA GIBS/Blue Marble (domínio público)** → alimenta o formato de
  pirâmide que o SatelliteZoom.tsx do Piter JÁ consome (renovação sem
  reescrever o componente).
- HUD scanner: manter o do Piter; tema nosso por cima.

## 8. Qualidade "documentário famoso" (o checklist estético)

Zoom lento com easing cúbico · paleta escura dessaturada · destaque com glow
discreto · tipografia serif pequena · grain 16mm + vignette do acervo ·
pins com drop+ripple · traço desenhando NO RITMO da narração (temos
timestamps por palavra via words.json/detectar_mapas) · SFX por modo.

## 9. Fora de escopo v1 (registrado)

- Rota por malha viária real (OSRM) — arco estilizado é a convenção.
- Globo 3D (CesiumJS/three.js) — candidato v3 pra abertura de docs.
- Labels automáticos de cidades vizinhas — poluição; só origem/destino.

## 10. Plano de execução quando vier o GO

1. `_dados`: baixar/converter NE (countries + admin1 + capitais) — PD.
2. `map_core.js` + `map_page.html` + `map_render.py` (padrão gl_render).
3. Bloco 1 (destaque) → demo · Bloco 2 (rota, 8 modos) → demo · Bloco 3.
4. Pós FFmpeg (grain/vignette/LUT) + SFX pareado.
5. Demos sobre casos reais: Brasil→EUA avião · Egito destaque · Alemanha→
   França seta de guerra · travessia de bicicleta.
6. Checkpoint Piter: entregar `map_core.js` + contrato + dados.
```

---

## 11. V1 IMPLEMENTADO E APROVADO (20/07/2026) — contrato real

Motor completo em `acervo/mapas/_engine/` (catálogo: `acervo/mapas/manifesto.json`,
12 receitas MAP-0001..0012, previews em `acervo/mapas/_previews/`). Aprovado pelo
operador em 20/07: destaque hero, rota A→B, multi-ponto, ida-volta, marítima e
bateria dos 8 modos.

### Contrato do mapa_job (o que o Director/Piter manda)

```json
{ "tipo": "destaque",
  "local": { "pais": "Egypt", "nome": "Ancient Egypt" } }

{ "tipo": "rota", "modo": "aviao|navio|trem|carro|bicicleta|cavalo|a_pe|exercito",
  "pontos": [ { "pais": "Brazil", "nome": "Brazil" },
              { "pais": "United States of America", "nome": "United States" },
              { "pais": "Thailand", "nome": "Thailand" } ] }

{ "tipo": "ponto", "coord": [31.1342, 29.9792], "legenda": "Great Pyramid" }
```

- `nome` = SEMPRE no idioma do roteiro (motor não traduz; fallback = atlas EN).
- `origem`/`destino` seguem aceitos (compat A→B); `pontos[]` >= 2 é o caminho novo.
- `pontos[].lado` (-1|1) força o lado do label (opcional; auto = oposto ao arco).
- Duração é AUTO: plano de tempo + REGRA DO HOLD (fim do último elemento + 2s).
- `dur_s` do job é IGNORADO desde o plano time-based (20/07).

### Regras de direção aprovadas (fonte: modelos do operador)

1. Destaque: nome GIGANTE (132px) canto superior + linha fina de baixo do texto
   até o pin; linha revela do pin, nome DIGITADO (typewriter com cursor).
2. Rota: SEM linha de callout — nome (64px, único pros 2+, mínimo 34px encolhendo
   JUNTOS) digitado ao lado do pin, lado oposto ao arco, >=10px da borda, âncora
   sempre start (digita na ordem certa).
3. Câmera de rota FIXA preenchendo o quadro; vizinhos = zoom regional
   (scale 33000/spread, clamp [340, 3000]).
4. Arcos de avião em ESPAÇO DE TELA com liftMax 0.22*H (great-circle polar
   estourava o quadro — EUA→Tailândia).
5. A-B-A: volta com bojo no lado oposto (automático — perpendicular acompanha a
   direção); ponto repetido não re-pina.
6. NAVIO: só pelo mar — máscara de água 0.25° (1440x720, terra rasterizada do
   atlas + dilatação 1 célula que abre Øresund/Dover/Bósforo/canais + calotas
   bloqueadas 78N/60S) + A* com PENALIDADE DE COSTA (custo 1→2.3 colado na
   terra = navega mar aberto) + Chaikin; pin/nome no PORTO (snap litoral);
   5.5s/perna (`modos.json dur_perna_s`) vs 2.4s do avião; ícone sem rotação.
7. Exército: linha sólida vermelha grossa + PONTA de flecha (ícone seta_guerra
   rotacionado pela tangente) avançando e cravando no destino.
8. Ícone pousa em cada parada e fica no destino final até o fim do vídeo.

### Plug do Piter

`map_core.js` = matemática pura (zero DOM): easings, projeção, câmera fixa multi,
trajetória por distância, arcos tela, typewriter, plano de tempo, A* marítimo +
máscara/costa. `map_page.html` = camada SVG de referência (d3). No Remotion:
importar map_core, reimplementar só o desenho (React/SVG) — contrato idêntico.

### Pendências v2 (registradas no manifesto)

GIBS satélite · admin-1/estados · atlas 50m pra zoom alto · SFX de navio no
banco (hoje placeholder machine_train) · SFX integrado no render.
