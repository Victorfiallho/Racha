# 01 — Requisitos

Prioridade: **M** = obrigatório no MVP, **S** = desejável no MVP, **C** = fase posterior.

## Requisitos funcionais

### Ingestão de comandas

| ID | Requisito | Prio |
|---|---|---|
| RF-01 | O sistema deve receber comandas via API aberta em modo **evento** (abertura, item adicionado/alterado/cancelado, ajuste, fechamento). | M |
| RF-02 | O sistema deve receber comandas via API aberta em modo **snapshot** (estado completo da comanda substitui o anterior). | M |
| RF-03 | Toda fonte de ingestão deve produzir o mesmo formato canônico de comanda. | M |
| RF-04 | O sistema deve aceitar o mesmo evento mais de uma vez sem duplicar dados (idempotência por `idempotency_key`). | M |
| RF-05 | O operador deve conseguir abrir, editar e fechar comandas manualmente pelo painel. | M |
| RF-06 | O sistema deve guardar o payload bruto de cada ingestão para auditoria e reprocessamento. | M |
| RF-07 | O sistema deve integrar com PDVs específicos por meio de conectores (pull). | C |
| RF-08 | O sistema deve capturar a pré-conta impressa (ESC/POS) por meio de um agente local. | C |

### Mesa e acesso

| ID | Requisito | Prio |
|---|---|---|
| RF-10 | Cada mesa deve ter uma URL pública fixa, com identificador não sequencial, gravada no QR Code e na tag NFC. | M |
| RF-11 | Ao acessar a URL, o cliente deve ver a comanda **aberta** daquela mesa; se não houver, uma tela de "mesa sem conta aberta". | M |
| RF-12 | O acesso à comanda deve exigir um código curto de verificação exibido ao cliente pelo restaurante (ver Q-03). | S |
| RF-13 | O cliente deve entrar como participante informando apenas um apelido, sem cadastro. | M |

### Divisão

| ID | Requisito | Prio |
|---|---|---|
| RF-20 | O participante deve poder assumir um item inteiro. | M |
| RF-21 | Vários participantes devem poder assumir o mesmo item, que é dividido entre eles. | M |
| RF-22 | Em itens com quantidade > 1, o participante deve poder assumir unidades específicas. | M |
| RF-23 | A mesa deve poder optar por divisão igual entre todos os participantes. | M |
| RF-24 | A taxa de serviço deve ser rateada proporcionalmente ao consumo de cada participante. | M |
| RF-25 | Todos devem ver em tempo real o que cada um assumiu e o saldo não atribuído. | M |
| RF-26 | Um participante deve poder "pagar o restante" (todo o saldo não atribuído). | M |
| RF-27 | Um participante deve poder pagar a parte de outro. | S |

### Pagamento

| ID | Requisito | Prio |
|---|---|---|
| RF-30 | O sistema deve gerar uma cobrança Pix dinâmica com o valor devido pelo participante. | M |
| RF-31 | O pagamento deve ser confirmado por webhook do provedor Pix, nunca por ação do cliente. | M |
| RF-32 | A cobrança deve ter validade; se a comanda mudar antes do pagamento, a cobrança é invalidada e regerada. | M |
| RF-33 | A comanda deve mudar para quitada quando a soma dos pagamentos confirmados igualar o total. | M |
| RF-34 | O sistema deve notificar a origem (webhook de saída / painel) quando a comanda for quitada. | M |
| RF-35 | O sistema deve tratar pagamento acima do devido após alteração da comanda (ver Q-05). | S |

### Painel do restaurante

| ID | Requisito | Prio |
|---|---|---|
| RF-40 | O gestor deve cadastrar mesas e gerar o identificador público de cada peça. | M |
| RF-41 | O gestor deve configurar a taxa de serviço e se ela é opcional para o cliente. | M |
| RF-42 | O operador deve ver o status de todas as mesas em tempo real. | M |
| RF-43 | O gestor deve gerar chaves de API e configurar webhooks de saída. | M |
| RF-44 | O gestor deve consultar o relatório de recebimentos por período, para conciliação. | S |

## Requisitos não funcionais

| ID | Categoria | Requisito |
|---|---|---|
| RNF-01 | Desempenho | O PWA deve abrir a comanda em até 2 s numa conexão 4G e propagar as alterações aos outros participantes em menos de 1 s. |
| RNF-02 | Leveza | O carregamento inicial do PWA deve ficar abaixo de 200 KB (gzip), sem instalação. |
| RNF-03 | Multi-tenant | Os dados de cada restaurante devem ser isolados no banco por política de acesso (RLS ou equivalente). |
| RNF-04 | Consistência | Os valores devem ser armazenados em centavos inteiros. A soma das partes deve ser exatamente igual ao total, nunca com diferença de ±1 centavo. |
| RNF-05 | Idempotência | Toda escrita externa (ingestão e webhooks Pix) deve ser idempotente. |
| RNF-06 | Auditoria | Toda mudança em comanda, atribuição e pagamento deve ser registrada em log imutável (event log). |
| RNF-07 | Segurança | Os identificadores de mesa devem ser não enumeráveis. A API deve ser autenticada por chave de restaurante, os webhooks assinados com HMAC e todo o tráfego trafegar por HTTPS. |
| RNF-08 | Privacidade (LGPD) | Coletar apenas o mínimo: apelido e sessão do dispositivo. Não pedir CPF. Os dados do pagador ficam com o provedor Pix. |
| RNF-09 | Disponibilidade | Meta de 99,5% no horário de funcionamento. Se a plataforma cair, o restaurante continua operando pelo fluxo tradicional. |
| RNF-10 | Observabilidade | Logs estruturados, métricas de ingestão (taxa de erro por fonte) e rastreio dos pagamentos de ponta a ponta. |
| RNF-11 | Acessibilidade | O PWA deve seguir o WCAG 2.1 AA, com textos grandes e alto contraste para leitura em ambiente com pouca luz. |

## Regras de negócio

| ID | Regra |
|---|---|
| RN-01 | Uma mesa tem no máximo **uma** comanda aberta por vez. |
| RN-02 | O valor de cada item é `quantidade × preço_unitário − descontos do item`, sempre em centavos. |
| RN-03 | Um item compartilhado é dividido pelo **peso** de cada participante (padrão 1). O resto da divisão inteira é distribuído de 1 em 1 centavo, na ordem de entrada dos participantes. |
| RN-04 | A taxa de serviço de cada participante é `taxa% × subtotal_assumido`, aplicando a mesma regra de resto da RN-03. |
| RN-05 | Na divisão igual, `total ÷ nº de participantes`, com o resto distribuído conforme a RN-03. |
| RN-06 | O valor de uma cobrança Pix fica **congelado** no momento em que é gerada. Se a parte devida mudar, a cobrança pendente expira e outra é gerada. |
| RN-07 | Um item cancelado na origem sai da divisão. Se já tiver sido pago, aplica-se a política da Q-05. |
| RN-08 | A comanda só é quitada quando `soma(pagamentos confirmados) ≥ total`. Não existe quitação parcial manual pelo cliente. |
| RN-09 | Novos participantes só entram enquanto a comanda estiver `aberta` ou `em_pagamento`. |
| RN-10 | Depois de quitada, a comanda fica somente leitura para os clientes. |
