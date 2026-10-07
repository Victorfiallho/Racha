# ADR-0004 — Pix com recebimento direto ao restaurante

- **Status:** proposto (depende da Q-02)
- **Data:** 2026-10-07

## Contexto

Se a plataforma receber o dinheiro e repassar ao restaurante depois, ela passa a custodiar recursos de terceiros. Isso pode enquadrá-la como subcredenciadora ou instituição de pagamento, com exigências regulatórias do Banco Central, além de criar risco operacional e de conciliação. A validação jurídica fica a cargo de um especialista.

## Decisão

O valor pago pelo cliente deve cair direto na conta do restaurante, por um destes caminhos:

- Usando as credenciais Pix do próprio restaurante no PSP escolhido.
- Usando um PSP com modelo de marketplace e split nativo, em que a plataforma recebe apenas a própria tarifa.

## Consequências

- A plataforma não toca no dinheiro do restaurante.
- O onboarding exige que o restaurante conecte ou crie a conta no PSP.
- A escolha do PSP define taxas, a API de estorno e os prazos.
