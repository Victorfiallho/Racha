# 05 — Contrato de ingestão

Toda fonte (API, conector, agente de impressão ou painel) produz uma das duas formas de entrada abaixo. A especificação formal está em [`schemas/`](../schemas) e [`api/openapi.yaml`](../api/openapi.yaml).

## Modo evento (recomendado para PDVs com integração em tempo real)

```json
{
  "idempotency_key": "pdv-x:8841:item-3:v2",
  "tipo": "item.adicionado",
  "ocorrido_em": "2026-10-07T20:15:03-03:00",
  "mesa_ref": "12",
  "comanda_external_id": "8841",
  "dados": {
    "external_item_id": "item-3",
    "descricao": "Chopp 500ml",
    "quantidade": 2,
    "preco_unitario_centavos": 1490,
    "aplica_taxa": true
  }
}
```

| Tipo | Efeito |
|---|---|
| `comanda.aberta` | Cria a comanda para a mesa (falha se já houver outra aberta) |
| `item.adicionado` | Adiciona um item |
| `item.alterado` | Altera quantidade, preço ou descrição de um item existente |
| `item.cancelado` | Marca o item como cancelado |
| `ajuste.aplicado` | Aplica desconto, couvert ou acréscimo no nível da comanda |
| `comanda.fechamento_solicitado` | A mesa pediu a conta (opcional, usado para notificação) |
| `comanda.cancelada` | Cancela a comanda |

## Modo snapshot (para fontes que só conhecem o estado final, como o agente de impressão)

`PUT /v1/comandas/{comanda_external_id}` com o documento completo do schema [`comanda-canonica`](../schemas/comanda-canonica.schema.json). O normalizador compara o snapshot com o estado atual por `external_item_id` e gera internamente os eventos equivalentes. Se a fonte não tiver identificadores de item, o normalizador usa um hash de `descricao + preco_unitario + posição`.

## Regras de processamento

- O `idempotency_key` é único por fonte. Um evento repetido retorna o resultado original.
- Eventos fora de ordem são aceitos dentro da mesma comanda. A ordem é dada por `ocorrido_em` e, no empate, pela ordem de chegada.
- Eventos rejeitados ficam salvos com o motivo, para que o integrador possa depurar (`GET /v1/eventos/{id}`).
- Todos os valores são em centavos (inteiros) e em BRL.

## Webhooks de saída

| Evento | Quando |
|---|---|
| `pagamento.confirmado` | Uma parte foi paga |
| `comanda.quitada` | O total foi coberto (a origem deve liberar a mesa) |
| `ingestao.rejeitada` | Um evento da fonte falhou na validação |

O corpo segue o envelope `{ "id", "tipo", "criado_em", "dados" }`, com o header `X-Racha-Signature: sha256=<hmac>`.
