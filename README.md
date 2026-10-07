# Racha

> Nome provisório. Plataforma para dividir e pagar a conta de restaurante pela mesa, via QR Code ou NFC.

O cliente encosta o celular na peça da mesa (NFC) ou lê o QR Code e abre a comanda da mesa no navegador. Ele marca o que consumiu, ou divide igualmente, e paga a própria parte via Pix. A mesa fecha sozinha quando tudo estiver pago.

A conta chega ao sistema por uma camada de ingestão que aceita várias fontes:

- API aberta
- Conectores de PDV
- Agente de captura de impressão
- Painel manual

Todas as fontes são convertidas para um único formato canônico de comanda.

## Status

Fase de concepção: requisitos, modelagem e contratos. Ainda não há código de aplicação.

## Documentação

| Documento | Conteúdo |
|---|---|
| [docs/00-visao.md](docs/00-visao.md) | Problema, proposta de valor, personas, escopo do MVP |
| [docs/01-requisitos.md](docs/01-requisitos.md) | Requisitos funcionais, não funcionais e regras de negócio |
| [docs/02-casos-de-uso.md](docs/02-casos-de-uso.md) | Histórias de usuário com critérios de aceite |
| [docs/03-modelo-de-dominio.md](docs/03-modelo-de-dominio.md) | Entidades, ERD e máquinas de estado |
| [docs/04-arquitetura.md](docs/04-arquitetura.md) | Contexto, containers e camada de ingestão |
| [docs/05-contrato-ingestao.md](docs/05-contrato-ingestao.md) | Formato canônico, eventos e webhooks de saída |
| [docs/06-questoes-em-aberto.md](docs/06-questoes-em-aberto.md) | Decisões pendentes |
| [docs/adr/](docs/adr/) | Registros de decisão de arquitetura |
| [schemas/](schemas/) | JSON Schema da comanda canônica e dos eventos |
| [api/openapi.yaml](api/openapi.yaml) | Contrato da API de ingestão |
| [hardware/peca-mesa.md](hardware/peca-mesa.md) | Especificação da peça impressa em 3D |

## Estrutura

```
docs/       requisitos, modelagem, arquitetura, ADRs
schemas/    contratos de dados (JSON Schema)
api/        contratos HTTP (OpenAPI)
hardware/   peça de mesa (QR + NFC)
```
