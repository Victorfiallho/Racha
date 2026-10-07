# 02 — Casos de uso e histórias

Formato: história de usuário + critérios de aceite (Dado / Quando / Então). Os IDs entre colchetes referenciam [01-requisitos.md](01-requisitos.md).

## UC-01 — Restaurante envia comanda via API  [RF-01, RF-02, RF-04]

**Como** integrador, **quero** enviar a comanda de uma mesa para a plataforma **para que** os clientes possam dividi-la.

**Critérios de aceite:**

- Envio de `comanda.aberta` com uma chave de API válida e uma mesa cadastrada: a comanda é criada com status `aberta` e a resposta é `202`.
- Reenvio do mesmo evento com a mesma `idempotency_key`: nada é duplicado e a resposta é `200` com o resultado original.
- Envio de um snapshot com itens diferentes dos atuais: os itens são reconciliados por `external_item_id`, e as atribuições dos itens que continuam existindo são preservadas.
- Envio de um evento para uma mesa que já tem outra comanda aberta: a resposta é `409 conflito_comanda_aberta` (RN-01).

## UC-02 — Operador lança comanda manualmente  [RF-05]

**Como** garçom, **quero** lançar os itens da mesa no painel **para que** a mesa use o app mesmo sem integração.

**Critérios de aceite:**

- Ao abrir uma comanda para uma mesa livre no painel, ela passa a aparecer para quem acessar a URL da mesa.
- O lançamento manual gera internamente os mesmos eventos canônicos da API.

## UC-03 — Cliente acessa a mesa  [RF-10..RF-13]

**Como** cliente, **quero** encostar o celular na peça ou ler o QR Code **para** ver a conta da minha mesa.

**Critérios de aceite:**

- Mesa com comanda aberta: o cliente informa o código de verificação e um apelido, entra como participante e vê os itens.
- Mesa sem comanda aberta: o cliente vê "Ainda não há conta aberta nesta mesa".
- URL com identificador inexistente: o cliente vê uma página de erro genérica, sem revelar se o identificador existe.

## UC-04 — Cliente assume itens  [RF-20..RF-22, RF-25]

**Como** cliente, **quero** marcar o que consumi **para** pagar só a minha parte.

**Critérios de aceite:**

- Item de R$ 30,00 assumido por A e B: cada um deve R$ 15,00, e todos os participantes veem a mudança em menos de 1 s.
- Item de R$ 10,00 dividido entre 3 pessoas: as partes ficam em 334, 333 e 333 centavos, na ordem de entrada (RN-03).
- Item "3× cerveja" e A assume 2 unidades: A deve 2/3 do valor do item, e 1 unidade fica não atribuída.

## UC-05 — Divisão igual  [RF-23]

**Critérios de aceite:**

- Com divisão igual ativada e 4 participantes, cada um deve `total/4`, com o resto conforme a RN-05.
- Com divisão igual ativada, as atribuições por item ficam desabilitadas até a opção ser desativada.

## UC-06 — Cliente paga a sua parte  [RF-30..RF-33]

**Como** cliente, **quero** pagar via Pix **para** sair sem depender do garçom.

**Critérios de aceite:**

- Um participante com valor devido > 0 que toca em "Pagar" recebe um QR Code e um código copia-e-cola com o valor congelado (RN-06).
- Quando o provedor confirma o pagamento por webhook, ele fica como `confirmado` e todos veem "A pagou".
- Se a parte do participante muda enquanto há uma cobrança pendente, a cobrança expira e o participante é avisado para gerar outra.
- Quando o último pagamento confirmado iguala o total, a comanda vai para `quitada` e o restaurante é notificado (RF-34).

## UC-07 — Pagar o restante  [RF-26]

**Critérios de aceite:**

- Há saldo não atribuído e um participante escolhe "pagar o restante": o saldo é somado à parte dele antes de gerar a cobrança.

## UC-08 — Restaurante acompanha mesas  [RF-42]

**Critérios de aceite:**

- O painel mostra cada mesa com status (livre, aberta, em pagamento, quitada), valor pago e total, atualizado em tempo real.

## UC-09 — Gestor configura integração  [RF-43]

**Critérios de aceite:**

- O gestor pode gerar uma chave de API. Ela é exibida uma única vez e armazenada apenas como hash.
- Ao cadastrar uma URL de webhook, o sistema envia um evento de teste assinado com HMAC.
