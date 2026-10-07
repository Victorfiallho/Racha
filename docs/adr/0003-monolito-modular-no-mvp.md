# ADR-0003 — Monólito modular no MVP

- **Status:** proposto
- **Data:** 2026-10-07

## Contexto

A arquitetura alvo tem serviços separados: ingestão, comandas, pagamentos e despachante de webhooks. No MVP, porém, a equipe é de uma pessoa e o volume é de um restaurante piloto.

## Decisão

Começar com um único deploy organizado em módulos com fronteiras explícitas. Cada módulo tem sua própria pasta, sua interface pública e suas tabelas, e nenhum módulo acessa as tabelas de outro.

Um módulo vira serviço separado quando houver motivo concreto:

- Escala diferente.
- Necessidade de deploy independente.
- Isolamento de falhas, como no caso dos webhooks Pix.

**Exceção:** o agente de impressão é um executável separado desde o início, porque roda no computador do restaurante.

## Consequências

- Entrega mais rápida e infraestrutura mais simples.
- Exige disciplina nas fronteiras dos módulos, que pode ser reforçada com regras de lint de import.
- Separar um módulo depois sai barato, desde que as fronteiras tenham sido respeitadas.
