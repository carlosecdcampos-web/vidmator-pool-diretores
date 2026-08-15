# ESPEC — CLOSER: o último momento de saída

> Ordem do operador (13/08, depois de assistir a onça re-renderizada): *"o corte
> seco no final mata um pouco da imersão. DEPOIS QUE O ÁUDIO DA NARRAÇÃO ACABOU,
> colocar um take fixo, de 7s... a função desse take é o último momento de saída."*
>
> Posição na cadeia: **é uma inteligência de CLOSER — entra depois de TUDO que já
> foi otimizado.** Não negocia com nenhum passe existente, não altera fronteira de
> cena, não mexe em elemento. Só acrescenta cauda.
>
> Ordem de execução: implementar **depois** dos itens 1-5 da lista de otimização, e
> **antes** do E2E de espécie nova.

---

## 1. A LINHA DO TEMPO

`T` = instante em que o áudio da narração termina (hoje, o fim do último `fim` de
cena falada e também o fim da trilha).

```
    T ─────────────────────── T+5s ──────── T+7s
    │  take do closer, cheio   │  fade out  │
    │  trilha continua, cheia  │   de 2s    │
    └──────────────────────────┴────────────┘
                          vídeo E trilha somem juntos
```

- **T → T+7s**: um único take fixo, duplicata de um já usado no vídeo.
- **T → T+5s**: vídeo e trilha em volume/opacidade plenos.
- **T+5s → T+7s**: fade out de 2s, **vídeo e trilha juntos**, terminando em 0.
- Duração total da cauda: **7s** (knob `VM_CLOSER_S`, fade `VM_CLOSER_FADE_S`).

## 2. QUAL TAKE (decisão do operador)

> *"mais paisagem, qualquer um que for mais paisagem, footage de paisagem, só não
> pegar dentro do T − 10 cenas"*

Critério, na ordem:
1. **Universo**: takes já usados no vídeo, **excluindo as 10 últimas cenas**
   (o closer não pode ecoar o que o espectador acabou de ver).
2. **Preferência**: o mais PAISAGEM. Sinais disponíveis sem Vision nova:
   `nivel` de atmosfera (`ATM-*`, `L3v-mix`, `L5-*`), ausência de
   `required_subjects` no contrato da cena de origem, e `real_query` com termos de
   paisagem. **Vision só se nenhum sinal decidir** — o closer não pode custar lote
   do orçamento (R1) por padrão.
3. **Desempate**: cena mais próxima do início (é o arco visual fechando).
4. **Fallback declarado**: sem candidato, o closer NÃO nasce e o log diz por quê —
   nunca um take qualquer, nunca tela preta.

## 3. O CLOSER É ISENTO — E DECLARADO (decisão do operador: "SIM")

A cena do closer é uma duplicata **deliberada**. Ela nasce marcada:

```json
{"_closer": true, "_isento_reuso": "closer", "_copia_de": <idx da origem>}
```

- `verificador_reuso` e `_viola_reuso` **ignoram** cena com `_closer` — isenção
  explícita, nunca por acidente. Sem isso o portão acusaria a duplicata legítima.
- O closer **não entra** no livro-caixa (`usos`) — ele não pode gastar a cota de
  aparições do material nem empurrar outra cena para violação.

## 4. O CLOSER É PURO (ordem do operador)

> *"nunca terá nenhum elemento de apresentação, overlay, somente o próprio take,
> puro e cru"*

A cena do closer nasce e permanece **sem**: `presentacao`, `overlay_take`,
`texto_impacto`, `infografico`, `infografico2`, `ilustracao`, `entrada_texto`,
`palavra_chave`, `aparece_em`, `texto_ate`, `sfx`, `mascote`, legenda.
`transicao="corte"`, `fade=0` na ENTRADA (o fade é só na saída, em T+5s).

**Onde isso é garantido**: o closer é criado **depois** de `reconciliar_cenas` e
depois do `verificador_reuso` — ou seja, depois de todo passe que atribui elemento.
Nenhum passe posterior olha para ele. É o que torna a mudança segura.

## 5. A TRILHA (decisão do operador)

> *"a trilha termina com a narração hoje, ela deve ser estendida para T + 7s, com o
> fade out começando em T + 5s"*

Este é o **único ponto que toca algo existente**. A trilha hoje é cortada no fim da
narração; passa a ser estendida por 7s e a receber o mesmo fade de 2s a partir de
T+5s. Se a faixa de trilha não tiver 7s de sobra, ela faz loop ou o closer encurta
para o que houver — decisão declarada no log, nunca silêncio abrupto.

## 6. O QUE PODE QUEBRAR (e o guard de cada um)

| risco | guard |
|---|---|
| duração total do vídeo muda (+7s) | o `preparar_render` deriva a duração da timeline; conferir que nada usa a duração da NARRAÇÃO como fim do vídeo |
| legenda/SRT tentando cobrir o closer | o closer não tem fala: garantir que o gerador de legenda pare em T |
| leis (L01/L04/L06/L13) julgando a cena nova | o closer entra DEPOIS do portão; se algum passe rodar depois, ele precisa da isenção `_closer` |
| verificador acusando a duplicata | isenção do §3 |
| render assumindo que a última cena tem áudio | o mix precisa aceitar trecho só-trilha |

## 7. ACEITE

1. Vídeo sai com `dur_narração + 7s`.
2. O último take é duplicata de um take de paisagem fora das 10 últimas cenas.
3. Nos 7s finais: sem texto, sem overlay, sem moldura, sem legenda, sem SFX.
4. Trilha audível até T+5s e em fade até T+7s; vídeo idem.
5. `verificador_reuso` passa **sem** contar a duplicata do closer.
6. Nenhum outro tempo do vídeo muda (diff de fronteiras = zero até T).
