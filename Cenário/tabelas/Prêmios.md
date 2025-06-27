---
tags:
  - objeto
entidade: operacional
mestre: true
objeto: tabela
---
**Definição**: Transações financeiras de recebimento de prêmios

**Atributos Principais:**

- `payment_id` (PK)
- `policy_id` (FK)
- `parcela_numero`
- `total_parcelas`
- `data_vencimento`
- `data_pagamento`
- `valor_parcela`
- `valor_juros`
- `valor_multa`
- `valor_pago`
- `forma_pagamento` - Boleto/Cartão/Débito
- `status_pagamento` - Pendente/Pago/Atrasado/Cancelado
- `dias_atraso`
- `tentativas_cobranca`
- `gateway_pagamento`
- `transaction_id`

**Relacionamentos:**

- [[Apólices]] (N:1)
- [[Comissões Geradas]] (1:N)
- [[Avisos de Cobrança]] (1:N)