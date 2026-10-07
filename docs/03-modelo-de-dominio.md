# 03 — Modelo de domínio

## Contextos delimitados

| Contexto | Responsabilidade | Entidades principais |
|---|---|---|
| **Ingestão** | Receber dados de fontes externas e traduzi-los para o formato canônico | FonteIntegracao, EventoIngestao |
| **Comanda** | Estado da conta da mesa (fonte da verdade dos itens) | Restaurante, Mesa, PecaMesa, Comanda, ItemComanda, Ajuste |
| **Divisão** | Quem assumiu o quê e quanto cada um deve | Participante, Atribuicao |
| **Pagamento** | Cobranças, confirmações e conciliação | Cobranca, Pagamento |
| **Identidade** | Operadores do restaurante e permissões | UsuarioRestaurante |

## Diagrama de entidades

```mermaid
erDiagram
  RESTAURANTE ||--o{ MESA : possui
  RESTAURANTE ||--o{ FONTE_INTEGRACAO : configura
  RESTAURANTE ||--o{ USUARIO_RESTAURANTE : tem
  MESA ||--o{ PECA_MESA : "identificada por"
  MESA ||--o{ COMANDA : recebe
  FONTE_INTEGRACAO ||--o{ EVENTO_INGESTAO : gera
  EVENTO_INGESTAO }o--|| COMANDA : altera
  COMANDA ||--o{ ITEM_COMANDA : contem
  COMANDA ||--o{ AJUSTE : aplica
  COMANDA ||--o{ PARTICIPANTE : tem
  PARTICIPANTE ||--o{ ATRIBUICAO : faz
  ITEM_COMANDA ||--o{ ATRIBUICAO : "dividido em"
  PARTICIPANTE ||--o{ COBRANCA : solicita
  COBRANCA ||--o| PAGAMENTO : "resulta em"

  RESTAURANTE {
    uuid id PK
    string nome
    string cnpj
    int taxa_servico_bp "basis points, 1000 = 10%"
    bool taxa_opcional
    string psp_conta_ref "conta Pix recebedora"
  }
  MESA {
    uuid id PK
    uuid restaurante_id FK
    string rotulo "ex.: Mesa 12"
    string external_ref "id da mesa no PDV"
    bool ativa
  }
  PECA_MESA {
    uuid id PK
    uuid mesa_id FK
    string slug_publico UK "não sequencial"
    string nfc_uid
    timestamp revogada_em
  }
  COMANDA {
    uuid id PK
    uuid mesa_id FK
    string external_id "id na origem"
    string origem "api|pdv|impressao|manual"
    string status
    string modo_divisao "por_item|igual"
    string codigo_verificacao
    int versao "otimistic lock"
    timestamp aberta_em
    timestamp quitada_em
  }
  ITEM_COMANDA {
    uuid id PK
    uuid comanda_id FK
    string external_item_id
    string descricao
    int quantidade
    int preco_unitario_centavos
    int desconto_centavos
    bool aplica_taxa
    string status "ativo|cancelado"
  }
  AJUSTE {
    uuid id PK
    uuid comanda_id FK
    string tipo "desconto|couvert|acrescimo"
    int valor_centavos
    string rateio "proporcional|igual"
  }
  PARTICIPANTE {
    uuid id PK
    uuid comanda_id FK
    string apelido
    string sessao_dispositivo
    int ordem_entrada
  }
  ATRIBUICAO {
    uuid id PK
    uuid item_id FK
    uuid participante_id FK
    int unidades "null = item inteiro compartilhado"
    int peso
  }
  COBRANCA {
    uuid id PK
    uuid participante_id FK
    int valor_centavos "congelado"
    int versao_comanda
    string psp_txid UK
    string status
    timestamp expira_em
  }
  PAGAMENTO {
    uuid id PK
    uuid cobranca_id FK
    int valor_centavos
    string e2e_id UK "id fim a fim Pix"
    timestamp confirmado_em
  }
  FONTE_INTEGRACAO {
    uuid id PK
    uuid restaurante_id FK
    string tipo
    string api_key_hash
    string webhook_url
    string webhook_secret
  }
  EVENTO_INGESTAO {
    uuid id PK
    uuid fonte_id FK
    string idempotency_key UK
    string tipo
    json payload_bruto
    string status "recebido|aplicado|rejeitado"
    string erro
  }
  USUARIO_RESTAURANTE {
    uuid id PK
    uuid restaurante_id FK
    string email
    string papel "gestor|operador"
  }
```

## Máquina de estados — Comanda

```mermaid
stateDiagram-v2
  [*] --> aberta: comanda.aberta
  aberta --> em_pagamento: primeira cobrança gerada
  em_pagamento --> aberta: itens adicionados e nenhum pagamento confirmado
  em_pagamento --> quitada: soma confirmada ≥ total
  aberta --> cancelada: comanda.cancelada
  em_pagamento --> cancelada: comanda.cancelada (exige tratar pagamentos)
  quitada --> fechada: origem confirma fechamento / timeout
  cancelada --> [*]
  fechada --> [*]
```

`quitada` significa que todo o valor foi pago. `fechada` significa que a mesa foi liberada na origem (PDV ou operador). Os dois estados ficam separados porque a origem pode demorar a reconhecer a quitação.

## Máquina de estados — Cobrança

```mermaid
stateDiagram-v2
  [*] --> pendente
  pendente --> confirmada: webhook Pix
  pendente --> expirada: prazo vencido
  pendente --> invalidada: parte devida mudou (RN-06)
  confirmada --> estornada: devolução
  expirada --> [*]
  invalidada --> [*]
```

Um webhook de confirmação que chega para uma cobrança `invalidada` ou `expirada` **não é descartado**. O dinheiro entrou de fato, então o pagamento é registrado e o excedente segue a política da Q-05.

## Cálculo da divisão

O cálculo é uma **função pura**: recebe a comanda, as atribuições e as configurações, e devolve o valor devido por participante. Ele é sempre recalculado do zero, nunca de forma incremental, o que facilita testar e auditar.

```
para cada item ativo:
  valor_item = quantidade × preco_unitario − desconto
  se há atribuições por unidade:
     valor por unidade = valor_item / quantidade (resto pela RN-03)
  senão:
     divide valor_item pelos pesos dos participantes (resto pela RN-03)
subtotal[p] = soma das partes de p
taxa[p]     = subtotal_taxavel[p] × taxa_bp / 10000 (resto pela RN-03)
ajustes     = rateados conforme o campo rateio
devido[p]   = subtotal[p] + taxa[p] + ajustes[p] − pagos_confirmados[p]
nao_atribuido = total − soma(subtotal + taxa + ajustes)
```

**Invariante testável:** `soma(devido) + soma(pago) + nao_atribuido == total`, para qualquer entrada.
