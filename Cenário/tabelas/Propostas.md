---
tags:
  - objeto
entidade: processo
mestre: true
objeto: tabela
---
**Definição**: Formalização da intenção de contratar com dados completos

**Atributos Principais:**

- `proposal_id` (PK)
- `proposal_number`
- `quote_id` (FK) - Referência à cotação original
- `customer_id` (FK)
- `data_proposta`
- `data_validade_proposta`
- `valor_premio_proposto`
- `forma_pagamento_escolhida`
- `dia_vencimento`
- `dados_completos_risco` (JSON)
- `documentos_recebidos` (JSON)
- `status_analise` - Pendente/Em análise/Aprovada/Recusada
- `motivo_recusa`
- `underwriter_id` (FK)
- `parecer_tecnico`

**Relacionamentos:**

- [[Cotações]] (N:1)
- [[Roteiros/Discussões guiadas/mesh/tabelas/Clientes]] (N:1)
- [[Apólices]] (1:1 quando aprovada)
- [[Documentos Anexados]] (1:N)