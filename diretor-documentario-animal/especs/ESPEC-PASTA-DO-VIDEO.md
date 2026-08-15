# ESPEC — A PASTA DO VÍDEO É A FRONTEIRA

Desenho do operador (11/08), para ser aplicado ANTES de plugar o Vidmator no
Automator. Adota a convenção que **já funciona** no Automator hoje, em vez de
inventar uma nova.

> "vou desenhar o fluxo, VAMOS UTILIZAR O QUE FUNCIONA, vamos ter como exemplo o
> que roda hoje no automator... dentro da pasta do vídeo, onde estará áudio,
> roteiro, txt, legenda, dá para ter uma subpasta com todos os asset do vídeo
> (os baixados da internet e os produzidos no flow) tudo lá dentro, seria uma
> subpasta por job, que estaria na PASTA DO VÍDEO, seria impossível contaminar
> assim... assim eu posso colocar 200 vídeos na esteira que eles nunca se
> misturariam, isso vai possibilitar produção em massa horizontalmente,
> produzindo imagens e vídeos para 5 canais diferentes ao mesmo tempo, sem que um
> download vá para a pasta do outro."

## A estrutura (o que JÁ existe + o que a Vidmator acrescenta)

```
Canais/
  [CANAL]/
    [DD-MM-YYYY]{_seq} - [Titulo_Sanitizado]/     ← A PASTA DO VÍDEO = O JOB
        1_roteiro.txt                              (já existe)
        2_narracao.mp3 · .voz.json                 (já existe)
        video_final.mp4                            (já existe)
        thumbnail.jpg                              (já existe)
        SEO.json  ← dispara a publicação           (já existe)
        SEO.txt · status.json                      (já existe)
        legenda / words                            (Vidmator)
        _assets/                                   ← ACRÉSCIMO DA VIDMATOR
            stock/    baixados dos bancos (Pexels, Archive, Commons, SearXNG…)
            veo/      produzidos no Flow (pool privado + staging sNNN)
            _tmp/     temporários do resolver (keyframes, checagens)
            estado/   index_cascata, pexels_ids, gate_veredictos
```

A pasta é criada por `orchestrator._obter_pasta_producao(canal, data, titulo,
seq, multi)` (AC-Automator/orchestrator.py:79). O Vidmator recebe esse caminho
pronto — não o inventa.

## As três regras

**R1 — A pasta do vídeo é a fronteira.** Tudo que é produzido ou baixado PARA
aquele vídeo mora dentro dela. Quem baixa do VEO recebe o caminho exato e cria a
subpasta lá. O montador recebe o mesmo caminho e conhece as subpastas. Nada fora
dela é enxergado.

**R1b — "Asset do job" é TUDO que foi produzido ou baixado PARA aquele vídeo.**
Decisão do operador (11/08): *"a narração FAZ PARTE DO JOB ENTÃO DEVE ESTAR NA
PASTA DO JOB, como o automator faz hoje com os vídeos que produz"*. Portanto:
roteiro, narração, words/legenda, stock, VEO, thumbnail — todos dentro. Se serve
a um vídeo só, mora na pasta dele. A L11 fiscaliza todos, não só mídia visual.

**R2 — A única exceção é o material de FUNDO do canal.** Ele vive na pasta de
material do canal, é compartilhado por construção e não tem dono de vídeo. Vale
o mesmo para o que não tem dono nenhum: overlays, SFX, trilha, molduras.

**R3 — Isolamento é por CONSTRUÇÃO, não por faxina.** Não existe balde comum de
material com dono. Não se limpa entre vídeos porque nunca se misturou.

## Por que isto resolve o que a subpasta em `_workspace` não resolvia

- **200 vídeos na esteira nunca se misturam** — cada um escreve na própria pasta.
- **Produção horizontal**: 5 canais ao mesmo tempo, e um download não tem como
  cair na pasta do outro.
- **A publicação já sabe ler essa pasta** — o `SEO.json` que dispara o upload
  mora ali; não há encanamento novo.
- **A limpeza vira trivial**: apagar `_assets/` apaga tudo que aquele vídeo
  consumiu, sem risco de levar junto material de outro.

## Decisão em aberto (do operador)

> "após o vídeo final PRONTO, ele pode excluir a pasta de cache/assets, ou depois
> que o vídeo for publicado no yt, vamos definir como funcionará essa limpeza."

Duas opções, com o trade-off:
- **após o render** — libera disco antes; se o vídeo precisar ser refeito, tudo
  volta a ser baixado.
- **após publicar no YouTube** — mantém o material enquanto o vídeo não está no
  ar; re-render/correção sai de graça. Custa disco por mais tempo.

Recomendação: **após publicar**. O período entre render e publicação é
justamente quando o operador assiste e pode pedir correção — e é quando o
material ainda vale.

## O que muda em relação ao que foi implementado hoje (L11)

Hoje o isolamento foi feito em `_workspace/_jobs/<job>/`. A raiz passa a ser a
PASTA DO VÍDEO. O mecanismo (`dir_job()`, os três baldes, o pool privado do VEO)
está correto e é reaproveitado — muda apenas de onde a raiz vem: em vez de
`TESTE/_jobs/<VM_JOB>`, vem de um caminho recebido (`VM_PASTA_VIDEO`), que no
Automator é a pasta de produção e no uso avulso é declarado na mão.
