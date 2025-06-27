---
tags:
  - objeto
entidade: processo
mestre: true
objeto: tabela
---
**Definição**: Contrato efetivo de seguro em vigor

**Atributos Principais:**

- `policy_id` (PK)
- `policy_number` - Número único da apólice
- `proposal_id` (FK)
- `customer_id` (FK)
- `product_id` (FK)
- `data_emissao`
- `data_inicio_vigencia`
- `data_fim_vigencia`
- `data_renovacao`
- `valor_premio_total`
- `valor_franquia`
- `limite_indenizacao`
- `tipo_renovacao` - Automática/Manual
- `apolice_anterior_id` (FK) - Para renovações
- `status_apolice` - Ativa/Suspensa/Cancelada/Expirada
- `motivo_cancelamento`
- `broker_id` (FK)
- `comissao_percentual`

**Relacionamentos:**

- [[Propostas]] (1:1)
- [[Roteiros/Discussões guiadas/mesh/tabelas/Clientes]] (N:1)
- [[Riscos Segurados]] (1:N)
- [[Coberturas Contratadas]] (1:N)
- [[Endossos]] (1:N)
- [[Sinistros]] (1:N)
- [[Pagamentos]] (1:N)