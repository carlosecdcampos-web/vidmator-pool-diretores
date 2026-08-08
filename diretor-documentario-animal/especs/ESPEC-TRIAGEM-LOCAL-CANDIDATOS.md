# ESPEC — TRIAGEM LOCAL DE CANDIDATOS v1 (Bloco 3)

**Status:** ESPEC APROVADA EM ESCOPO (auditoria 3 + parecer do GPT sobre as ferramentas
open source, 03/08/2026). **NÃO IMPLEMENTADA** — aguarda GO explícito do operador.

## Problema

O resolver atual manda o lixo bruto das APIs direto para o Gemini Vision (caro, quota
limitada, lento) e descobre candidato ruim tarde. A triagem local elimina o lixo ANTES,
de graça, offline — e o Gemini vira o juiz editorial FINAL de 2-3 finalistas fortes,
não o primeiro filtro de 20 candidatos.

**Princípio (do parecer):** os modelos locais são TRIAGEM, nunca juízes finais. Criar
uma torre de validadores correlacionados (CLIP→YOLO→DINO→BioCLIP→Gemini) daria falsa
confiança — vários modelos podem falhar pela mesma razão visual.

## Escopo v1 (estrito — nada além disto)

1. **Pixabay Video** como segunda fonte no pool (key já existe no acervo; padrão de
   adapter = copiar do fontes5 do Piter).
2. **OpenCLIP ViT-B/32** (ou SigLIP Base 224 — decidir por benchmark; UM só) como
   ranqueador local de thumbnails/previews.
3. **MegaDetector** (variante compacta) como gate animal/pessoa/veículo — NUNCA espécie.
4. **Gemini Vision** somente nos 2-3 finalistas.
5. **Banco local** recebendo todo SEGMENTO aprovado (schema abaixo).
6. **PRE-RENDER REPORT obrigatório** (Bloco 1 — já implementado).
7. **SEM** YOLO-World, Grounding DINO ou BioCLIP nesta implantação.
   (v2, após benchmark: BioCLIP com hard negatives e margem calibrada.)

## Pipeline do resolver com triagem

```
1. CONSULTAR BANCO LOCAL (contratos compatíveis por espécie/classe/ação/setting)
2. BUSCAR POOL EXTERNO (Pexels Video + Pixabay Video + Commons/imagens)
3. FILTROS DE METADADOS (licença, resolução, duração, aspect, termos proibidos em título/tags)
4. BAIXAR APENAS THUMBNAILS/PREVIEWS (leves)
5. RANKING OPENCLIP/SIGLIP  →  top 6
6. BAIXAR PREVIEWS DOS TOP 6
7. EXTRAIR O SUBCLIP CANDIDATO (validar o trecho USADO, nunca o arquivo inteiro)
8. GATES LOCAIS: MegaDetector (animal/pessoa/veículo) nos frames 10/30/50/70/90% do subclip
9. GEMINI VISION: somente top 2-3 (vet_contrato v2.2, inalterado)
10. ESCOLHA FINAL: PASS | DEGRADED explícito | HARD_MISS
11. PRE-RENDER REPORT → aprovação → LOCK → RENDER ÚNICO
```

## Ranking CLIP — scoring com negativos (obrigatório)

CLIP mede proximidade semântica global: estátua/ilustração/capa de livro/thumbnail com
texto "jaguar" pontuam ALTO na frase positiva. Sem negativos, o ranker herda A1/A6/A8.

```
POSITIVE: real live jaguar in rainforest · jaguar wildlife footage · jaguar walking outdoors
NEGATIVE: city · human · book cover · drawing · painting · cartoon · statue · toy ·
          taxidermy · computer screen · zoo sign
score_final = 0.70 * max(sim_positive) - 0.30 * max(sim_negative)
```

Pesos 0.70/0.30 são INICIAIS — calibrar no corpus A1-A9. Positivos/negativos derivam do
CONTRATO-EDITORIAL.md (grupo A) + contrato da cena (forbidden), não de listas soltas.

## Gate MegaDetector — semântica exata

```
Contrato exige animal:  nenhum frame com animal ≥ threshold → REJECT_LOCAL_MISSING_ANIMAL
Contrato proíbe humano: humano em qualquer frame ≥ threshold → REJECT_LOCAL_FORBIDDEN_HUMAN
Animal pequeno/camuflado/oculto sem detecção → LOW_CONFIDENCE (segue p/ Gemini), NUNCA rejeição
```

MegaDetector foi treinado em câmera-trap (domain shift p/ stock com drone/subaquático/
close): ausência de detecção NÃO é prova de ausência.

## Banco local — a unidade é o SEGMENTO aprovado por FACETAS

"Aprovado uma vez" ≠ "aprovado para sempre": o clip do jaguar descansando serve para
`jaguar resting` e falha para `jaguar swimming`. Schema mínimo:

```json
{
  "asset_id": "wildlife_000184",
  "file_hash": "sha256...",
  "source": "pixabay", "source_id": "123456", "source_url": "...",
  "creator": "...", "license": "Pixabay Content License", "downloaded_at": "2026-08-03",
  "media_type": "video", "duration": 18.4, "resolution": "1920x1080",
  "segments": [{
    "start": 2.1, "end": 6.8,
    "species": ["jaguar"], "animal_class": ["predator", "mammal"],
    "actions": ["walking"], "settings": ["rainforest", "riverbank"],
    "time_of_day": ["day"], "shot_types": ["medium"],
    "human_present": false, "vehicle_present": false,
    "representation": "live_real_animal", "operator_approved": true
  }],
  "clip_embedding": "...",
  "model_versions": {"clip": "...", "megadetector": "..."}
}
```

Fontes para POPULAR o banco (garimpo humano, nunca fallback automático ao vivo):
YouTube CC via yt-dlp (revisão de licença/proveniência obrigatória), NASA/NOAA/USGS
(adapters especializados por tipo de contrato), Openverse (só IMAGEM — a API de vídeo
não existe operacionalmente hoje).

## Benchmark — corpus adversarial A1-A9

As mídias erradas das rodadas passadas (estátua, cidade, desenho, colagem, livro, peixes,
humano) ainda estão em `_cache_stock/` e registradas nos manifests → catalogar como
corpus rotulado ANTES de codar o ranker. É nele que se calibram pesos e thresholds.

## Critérios de aceite (verbatim do parecer — contrato de aceitação da v1)

1. O candidato correto, quando existente no pool, aparece no top 3 em ≥95% do benchmark.
2. Cidade, livro, desenho, estátua e humano proibido nunca chegam ao Gemini como top 3
   nos casos conhecidos, ou são obrigatoriamente rejeitados pelos gates posteriores.
3. Chamadas Gemini Vision caem ≥50% contra o resolver atual.
4. Nenhum candidato é aceito somente pelo score do CLIP.
5. MegaDetector nunca é usado para afirmar espécie.
6. Falha ou baixa confiança local não vira PASS.
7. O subclip real, e não o arquivo inteiro, é validado.
8. Todo asset final aparece no PRE-RENDER REPORT.
9. O render só é liberado depois de: hard_miss revisados, degraded revisados, zero
   candidatos rejeitados na timeline, aprovação explícita do relatório.
10. O MP4 é renderizado uma única vez.

## Hardware

RTX 4050 6GB: OpenCLIP ViT-B/32 (~150MB) e MegaDetector compacto rodam com folga.
NÃO usar variantes Large/So400m na v1. Atenção à licença GPL-3.0 do YOLO-World se
algum dia entrar (v2+ decide entre Grounding DINO Tiny Apache-2.0 e YOLO-World-S).

## Relação com a escala 25-30min

Esta espec é pré-requisito da arquitetura "resolver por ENTIDADE" (auditoria 3):
1 resolução por entidade → pool de 3-6 clips aprovados no banco → rotação entre cenas
(nunca o mesmo clip 14×). A espec de escala (batch de contratos em lotes de 5-8,
resolver por entidade, aprovação incremental) é documento separado, escrito quando
esta v1 estiver aceita no benchmark.
