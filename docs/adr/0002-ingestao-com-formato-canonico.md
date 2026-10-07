# ADR-0002 — Camada de ingestão com formato canônico único

- **Status:** aceito
- **Data:** 2026-10-07

## Contexto

Os restaurantes operam de formas diferentes:

- Alguns têm PDV com API.
- Outros têm PDV fechado.
- Outros só têm impressora térmica ou nem isso.

O produto precisa funcionar em todos esses cenários.

## Decisão

Todas as fontes (API aberta, conectores de PDV, agente de impressão e painel manual) passam por um normalizador. Ele converte a entrada em eventos canônicos ([05-contrato-ingestao.md](../05-contrato-ingestao.md)) e garante a idempotência. Nenhuma fonte grava diretamente no estado da comanda.

São aceitos dois modos:

- **Evento**, para fontes em tempo real.
- **Snapshot**, para fontes que só conhecem o estado final.

## Consequências

- **Positivas:**
  - Adicionar uma nova fonte significa escrever apenas um tradutor.
  - A divisão, o pagamento e o PWA não sabem de onde os dados vieram.
  - O payload bruto fica salvo, então dá para reprocessar quando um parser for corrigido.
- **Negativas:**
  - A reconciliação de snapshots sem `external_item_id` é heurística.
  - É preciso manter um contrato público versionado.
