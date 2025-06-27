---
tags:
  - objeto
entidade: suporte
mestre: true
objeto: tabela
---
**Definição**: Proteções específicas contratadas

**Atributos Principais:**

- `coverage_id` (PK)
- `policy_id` (FK)
- `tipo_cobertura` - Básica/Adicional
- `codigo_cobertura`
- `nome_cobertura`
- `limite_indenizacao`
- `franquia_especifica`
- `carencia_dias`
- `ambito_territorial`
- `clausulas_especiais`
- `exclusoes` (JSON)

**Relacionamentos:**

- [[Apólices]] (N:1)
- [[Sinistros]] (N:N)