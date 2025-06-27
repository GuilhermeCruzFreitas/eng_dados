---
tags:
  - objeto
entidade: processo
objeto: tabela
mestre: true
---
**Definição**: Simulação de preço e condições para um possível seguro

**Atributos Principais:**

- `quote_id` (PK)
- `quote_number` - Número da cotação
- `customer_id` (FK)
- `product_id` (FK)
- `data_cotacao`
- `data_validade`
- `valor_premio_calculado`
- `desconto_aplicado`
- `valor_premio_final`
- `forma_pagamento_simulada`
- `canal_origem` - Online/Corretor/CallCenter
- `ip_origem`
- `dados_risco` (JSON) - Específicos por produto
- `score_risco_calculado`
- `status_cotacao` - Aberta/Convertida/Expirada/Cancelada

**Relacionamentos:**

- [[Roteiros/Discussões guiadas/mesh/tabelas/Clientes]] (N:1)
- [[Produtos]] (N:1)
- [[Propostas]] (1:1 opcional)
- [[Itens]] Cotados (1:N)