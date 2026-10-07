# CLAUDE.md

Contexto para sessões do Claude Code neste repositório.

## Projeto

Racha (nome provisório): o cliente lê o QR Code ou encosta o celular no NFC da mesa, abre a comanda no navegador, divide a conta e paga a própria parte via Pix. Leia `README.md` e depois `docs/` em ordem numérica.

## Status

Fase de concepção. Ainda não há código de aplicação. As decisões pendentes estão em `docs/06-questoes-em-aberto.md`. As mais urgentes são a Q-02 (PSP e modelo de recebimento Pix) e a Q-07 (stack).

## Convenções

- Documentação, commits e nomes de domínio em português (`comanda`, `participante`, `cobranca`).
- Commits no estilo Conventional Commits (`docs:`, `feat:`, `fix:`), sem trailers de coautoria do Claude (`Co-Authored-By`, `Claude-Session`) e sem atribuição ao Claude em PRs.
- Toda decisão de arquitetura vira um ADR em `docs/adr/NNNN-titulo.md` (ver ADR-0001), e os requisitos afetados são atualizados no mesmo commit.
- Diagramas em Mermaid, dentro do próprio Markdown.

## Invariantes do domínio (não quebrar)

- Dinheiro sempre em **centavos inteiros**. Nunca usar ponto flutuante.
- Resto da divisão distribuído de 1 em 1 centavo, na ordem de entrada dos participantes (RN-03). Exemplo: R$ 10,00 / 3 = 334, 333, 333.
- `soma(partes) + pago + não atribuído == total`, sempre. Este é o teste central do motor de divisão.
- O valor da cobrança Pix é congelado na geração (RN-06). Um Pix recebido para uma cobrança expirada nunca é descartado.
- `quitada` e `fechada` são estados distintos da comanda.

## Arquitetura

Monólito modular no MVP (ADR-0003): módulos com fronteiras explícitas, e nenhum módulo acessa as tabelas de outro. A exceção é o agente de impressão, que é um executável separado. Todas as fontes de ingestão convertem para a comanda canônica (`schemas/comanda-canonica.schema.json`, ADR-0002).

## Validação

Quando alterar os contratos, valide os JSON Schemas e o `api/openapi.yaml` antes de commitar.
