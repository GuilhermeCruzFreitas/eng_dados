---
tags:
  - objeto
entidade: relacionamento
mestre: true
objeto: tabela
---
**Atributos:**

- `address_id` (PK)
- `entity_id` (FK) - Polimórfico
- `entity_type` - Cliente/Risco/Prestador
- `tipo_endereco` - Residencial/Comercial/Correspondência
- `cep`
- `logradouro`
- `numero`
- `complemento`
- `bairro`
- `cidade`
- `estado`
- `pais`
- `latitude`
- `longitude`