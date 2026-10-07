# 00 — Visão do produto

## Problema

Dividir a conta em restaurante é lento e sujeito a erro. Hoje acontece assim:

- O garçom traz uma única conta.
- Alguém calcula no celular.
- As pessoas pagam valores aproximados, e a taxa de serviço é rateada "no olho".
- Várias maquininhas circulam pela mesa.

Para o restaurante, isso tem custos concretos:

- A mesa fica ocupada por mais tempo.
- O garçom perde tempo no fechamento.
- Há divergência de caixa.

## Proposta de valor

**Para o cliente:** ver a conta da mesa no próprio celular, sem instalar nada, assumir o que consumiu e pagar só a sua parte via Pix.

**Para o restaurante:**

- A mesa é fechada mais rápido e com o valor exato.
- Há menos trabalho no fechamento.
- Funciona com o PDV que o restaurante já usa, por qualquer uma das fontes de ingestão.

## Personas

| Persona | Descrição | Objetivo principal |
|---|---|---|
| Cliente da mesa | Qualquer pessoa sentada à mesa, sem cadastro | Pagar exatamente a sua parte, rápido |
| Organizador | Cliente que abre a comanda no app primeiro | Garantir que a conta feche sem sobrar valor |
| Garçom / caixa | Operador do restaurante | Lançar a conta e acompanhar o pagamento das mesas |
| Gestor do restaurante | Dono ou gerente | Configurar mesas, taxa, integrações e conciliar recebimentos |
| Integrador | Desenvolvedor de PDV ou de rede | Enviar comandas via API sem fricção |

## Escopo do MVP

**Dentro:**

- Peça de mesa com QR Code e NFC apontando para a URL da mesa.
- PWA do cliente com:
  - Visualização da comanda em tempo real.
  - Divisão por item (inclusive item compartilhado e por unidade).
  - Divisão igual.
  - Taxa de serviço proporcional.
- Pagamento via Pix dinâmico com confirmação por webhook.
- Painel do restaurante com:
  - Cadastro de mesas.
  - Lançamento manual de comanda.
  - Status das mesas em tempo real.
- API de ingestão aberta, nos modos evento e snapshot.
- Webhook de saída avisando que a comanda foi quitada.

**Fora (próximas fases):**

- Conectores de PDV específicos.
- Agente de captura de impressão.
- Cartão de crédito e Apple Pay ou Google Pay.
- Gorjeta individual para o garçom.
- Fidelidade, cardápio e pedidos pelo app.
- App nativo.

## Métricas de sucesso do piloto

- Tempo médio entre o pedido da conta e a mesa quitada.
- Percentual de mesas que usam o app quando ele está disponível.
- Percentual de comandas quitadas sem intervenção do garçom.
- Divergência de caixa (meta: zero).
