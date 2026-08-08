# ESPEC — iNaturalist como fonte de mídia (tier 2/3) + tratamento obrigatório

> **Status:** 🟢 **DECIDIDA E PRONTA PARA IMPLEMENTAR (04/08)** — o operador travou as
> decisões: **só CC0 + CC-BY** ("as duas mais safes") · **fotos bastam, para começar**
> (sons fora do escopo por ora). Deve estar **OPERANTE antes do render de teste das
> aberturas** — os testes já puxam material daqui também.
>
> **Regra-mãe:** aditivo e gated. O `resolver_cascata` atual não muda o comportamento
> validado; o tier novo entra gated por preset.

---

## 1. 🚨 O RISCO PRINCIPAL: LICENÇA (nossos vídeos são MONETIZADOS)

> *"devemos nos certificar de pegar imagens/vídeos TOTALMENTE FREE, pois nossos vídeos
> serão monetizados, então não podemos correr risco de copyright."*

**Verificado na API real (04/08).** O iNaturalist tem material sob licenças bem diferentes,
e **a maioria NÃO serve para vídeo monetizado**:

| Licença | Uso comercial? | Exige atribuição? | Serve? |
|---|---|---|---|
| **CC0** | ✅ livre | não | ✅ **ideal** |
| **CC-BY** | ✅ sim | **sim** | ✅ com crédito |
| **CC-BY-SA** | ✅ sim | sim + *share-alike* | ⚠️ o *share-alike* pode contaminar a obra derivada — **evitar** |
| **CC-BY-NC** (e todas com `NC`) | ❌ **NÃO** | — | 🔴 **PROIBIDO** — "NC" = non-commercial |
| **CC-BY-ND** (e `ND`) | — | — | 🔴 **PROIBIDO** — "ND" = no derivatives (não pode editar/recortar) |
| *sem licença / all rights reserved* | ❌ | — | 🔴 **PROIBIDO** |

### ⚠️ Achado que muda o cálculo: o material seguro é a MINORIA

Medição real (espécie: *Crocodylus porosus*, grau research):

| Licença | Observações |
|---|---|
| CC0 | 89 |
| CC-BY | 429 |
| CC-BY-SA | 59 |
| **CC-BY-NC** (proibida) | **6.135** |

→ **CC-BY-NC tem ~11× mais material que todas as comerciais somadas.** Qualquer integração
que não filtre licença explicitamente vai, por probabilidade, **puxar material proibido na
maioria das vezes**. O filtro não é detalhe: é a funcionalidade.

**Filtro obrigatório na API:** `photo_license=cc0,cc-by` (**nunca** aceitar o default).
Também gravar `attribution` de cada foto — CC-BY exige crédito, e sem ele o uso é
irregular mesmo com a licença certa.

Volume para espécies do nosso corpus (só licenças comerciais): leão **1.723** ·
hipopótamo **1.466** · sucuri **81**. Suficiente para as espécies comuns; magro para as
raras — mais um motivo para ser **tier 2/3**, nunca a fonte principal.

## 2. 🚨 SEGUNDO ACHADO: o iNaturalist NÃO tem vídeo

Verificado na API: os campos de mídia de uma observação são **`photos`** e **`sounds`** —
**não existe vídeo**. O operador falou em *"imagens/vídeos do iNaturalist"*; na prática:

- ✅ **fotos** — até ~2048px de lado (`original.jpg`), muitas em alta qualidade;
- ✅ **sons** — gravações de campo (canto, vocalização) — pode ser um ativo à parte, para
  ambiência real da espécie;
- ❌ **vídeo** — não existe na plataforma.

**Consequência de arquitetura:** o iNaturalist entra como fonte de **IMAGEM**, o que casa
exatamente com o tratamento que o operador propôs (moldura/apresentação) e com a mecânica
que o projeto já tem para imagem (Ken Burns, apresentações). Não substitui footage de vídeo.

## 3. O TRATAMENTO OBRIGATÓRIO (pedido do operador)

> *"essas imagens e vídeos do iNaturalist OBRIGATORIAMENTE deverão estar numa apresentação,
> ou com um overlay por cima… podemos usar nosso grid, redimensionar o vídeo, colocar ele
> dentro de uma 'moldura', aplicar aquele efeito de tela que aplicamos quando fica com o
> mood vermelho, sem texto, como se fosse uma TV mesmo — mas a moldura não deve ser de TV,
> e sim uma borda."*

Referência: print do operador — mídia reduzida, dentro de moldura de **borda arredondada
clara**, sobre **fundo grid**, com textura de varredura por cima.

**Composição proposta (peças que JÁ existem no projeto):**

| Camada | O que usar | Já existe? |
|---|---|---|
| Fundo | **grid do tema do canal** (claro/escuro) | ✅ implementado (`fundoRel`) |
| Moldura | borda arredondada clara + sombra — **não** moldura de TV | ➕ novo (simples) |
| Mídia | foto redimensionada dentro da moldura | ✅ |
| Textura | o **overlay "Linhas de TV"** do pacote só-filtro, **sem texto** | ✅ existe (`mood_semtexto`) |

> 💡 Isso é praticamente uma **variante nova de apresentação** (`presentacao.tipo`), no
> mesmo molde do `film`/`polaroid` que já foram refinados — reaproveita o `Presentacao.tsx`
> e o grid do tema. Nome sugerido: **`quadro`**.

**Efeito colateral desejável:** o tratamento resolve dois problemas de uma vez —
diferencia visualmente a foto de acervo científico (que não tem cara de stock) e **cria
distância da imagem original**, o que é saudável do ponto de vista de direitos.

### 3.1 🚦 FREQUÊNCIA DO `quadro` (regra do operador, 04/08)

> *"NUNCA entrar com menos de 7 cenas de distância uma da outra — não deve ser algo para
> ser usado toda hora, senão vai cansar, sempre o mesmo efeito. Essa distância me referindo
> ao hook, que é mais dinâmico; depois do hook pode ter um intervalo bem maior, para não
> ficar cansativo."*

| Janela | Distância mínima entre dois `quadro` |
|---|---|
| **Dentro do hook** (janela do §1.4 da ESPEC-ABERTURA) | **≥ 7 cenas** |
| **Depois do hook** | **bem maior** — default proposto **≥ 14 cenas** (2×), a confirmar no refino |

- A distância conta em **cenas**, não em segundos, e vale entre **usos do `quadro`**
  (independente de qual foto está dentro).
- Knob no preset (gated, I1) junto dos demais cooldowns; o `pre_render_report` acusa
  violação (I15).

## ✅ IMPLEMENTADO (04/08, pós-GO) — `director/inaturalist.py`

Pass novo, gated pelo knob `inaturalist`, rodando **depois** de `apresentar` (a
apresentação `quadro` é obrigatória e não pode ser sobrescrita) e do resolver (usa
`nivel`/`hard_miss` para saber o que ficou fraco). Créditos saem em `creditos.txt`.

### 🚨 Armadilha encontrada na execução (e fechada): ESPÉCIE ERRADA

A busca do iNaturalist **ordena por número de observações, não por qualidade do nome**.
Medido ao vivo:

| Pedido | O que a API devolvia em 1º |
|---|---|
| `lion` | *Taraxacum officinale* — **dente-de-leão** |
| `jaguar` | *Coccinellidae* — **joaninhas** |
| `leopard` | *Limax maximus* — **lesma-leopardo** |

Colocar uma lesma onde o roteiro diz leopardo violaria o Contrato Editorial de forma
grosseira — e passaria despercebido porque a foto *é* de acervo científico legítimo.
**Trava implementada:** `autocomplete` (ranqueia por nome) + regra `_casa`: só aceita
igualdade exata **ou** nome comum TERMINANDO no termo pedido (*Common Hippopotamus* vale;
*Leopard Slug* não — a cabeça do nome é que manda a espécie). Nada casou = **sem foto**,
e está certo: tier 2/3 pode voltar vazio, nunca voltar errado.
Verificado 11/11 (incluindo `predator`/`creature`/`monster` → nada, como deve ser).

## 4. TIER 2/3 NA CASCATA

> *"usar estratégias de camada de tier 2 e tier 3 nos vídeos do iNaturalist pode ajudar a
> ter takes melhores, mais específicos também."*

O ganho real: **especificidade taxonômica**. O Pexels tem "crocodilo"; o iNaturalist tem
*Crocodylus porosus* **identificado por especialistas** (`quality_grade=research`) e
**georreferenciado** — casa com o gate `wrong_region`/`wrong_species` do Contrato Editorial,
que hoje precisa julgar por Vision.

**Posição na cascata (proposta):** entra **depois** das fontes de vídeo, como fonte de
IMAGEM para cenas de identidade — quando o sujeito obrigatório é uma espécie nomeada e o
tier atual falhou ou trouxe candidato duvidoso. Nunca como fonte primária.

### 4.1 FLUXO DA BUSCA (como deverá acontecer, passo a passo)

**Gatilho:** cena cujo contrato exige uma **espécie nomeada** (o `real_query`/sujeito
obrigatório já identifica isso hoje) **e** os tiers de vídeo falharam ou trouxeram
candidato duvidoso no gate de identidade.

```
1. TAXON    GET /v1/taxa?q={nome da espécie}            -> taxon_id
            (aceitar só rank species/subspecies; sem taxon_id -> desiste do tier)

2. BUSCA    GET /v1/observations?taxon_id={id}
              &photo_license=cc0,cc-by      <- OBRIGATÓRIO, nunca o default (§1)
              &quality_grade=research       <- identificado por especialistas
              &order_by=votes               <- melhores observações primeiro
              &per_page=30

3. ESCOLHA  rankear fotos por tamanho original + votos; descartar observação sem
            foto >= 1200px de lado (moldura `quadro` reduz, mas não salva miniatura)

4. DOWNLOAD original.jpg -> cache com identidade própria (inat_{observation_id})
            entra no MESMO orçamento de takes do item A2 (identidade por fonte)

5. REGISTRO gravar NO TIMELINE, por uso: observation_id · license_code ·
            attribution (texto pronto da API) · url da observação
            -> sem attribution gravada, o uso é INVÁLIDO (validador rejeita, I15)

6. RENDER   a foto NUNCA aparece crua: sempre dentro da apresentação `quadro`
            (grid do tema + moldura de borda + linhas de TV sem texto, §3)

7. CRÉDITO  o texto de attribution alimenta o mecanismo de crédito decidido na
            pendência 2 (créditos do fim / descrição do YouTube / FootnotePill)
```

- **Vision continua no circuito:** a foto passa pelo mesmo gate de identidade das demais
  fontes (georreferência e taxonomia ajudam, mas não substituem o olho).
- **Cache e re-render:** `observation_id` é a identidade — o mesmo material nunca conta
  como dois takes (mesma regra do A2/pexels_id).
- **Sons (se entrarem no escopo — pendência 4):** mesmo fluxo com `sounds` no lugar de
  `photos`, mesmo filtro de licença, mesmo registro de attribution.

## 5. ✅ DECISÕES TRAVADAS (operador, 04/08)

1. **Política de licença: só `CC0 + CC-BY`** — *"vamos usar somente as duas mais safes"*.
   CC-BY-SA **excluída** (share-alike). O filtro `photo_license=cc0,cc-by` é lei.
2. **Crédito/atribuição (CC-BY):** default operacional = **bloco de créditos na DESCRIÇÃO
   do YouTube**, gerado a partir das attributions gravadas no timeline (§4.1 passo 5).
   Zero custo visual no vídeo; o timeline carrega tudo que a descrição precisa.
   *(Default definido para destravar a implementação — operador pode vetar/mudar no GO.)*
3. **Fotos bastam, para começar** — confirmado.
4. **Sons: FORA do escopo por ora** — *"fotos bastam, para começar"*. O fluxo §4.1 já
   descreve como entrariam num futuro escopo.
5. **Apresentação `quadro`** — valores DEFAULT para a primeira implementação (calibrar
   visualmente no teste das aberturas, método da casa):

   | Parâmetro | Default |
   |---|---|
   | fundo | grid do tema do canal (claro/escuro) — reaproveita `fundoRel` |
   | mídia | **62% da largura**, centrada, proporção original |
   | moldura | borda **6px** branco 0.90 · raio **18px** · sombra 0.35 |
   | textura | linhas de TV do `mood_semtexto`, **sem texto**, opacidade do pacote |
   | frequência | **§3.1**: ≥7 cenas no hook · ≥14 pós-hook |

---

*Criada em 04/08/2026. API verificada ao vivo (`api.inaturalist.org/v1/observations`).*
