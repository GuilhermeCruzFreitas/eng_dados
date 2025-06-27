---
tags:
  - objeto
entidade: raiz
objeto: tabela
mestre: true
---
**Definição**: Tipos de seguro oferecidos pela seguradora

**Atributos Principais:**

- `product_id` (PK)
- `product_code` - Código único do produto
- `nome_produto`
- `linha_negocio` - Auto/Residencial/Vida/Saúde
- `tipo_produto` - Individual/Empresarial
- `categoria` - Compreensivo/Básico/Premium
- `data_lancamento`
- `data_descontinuacao`
- `status_comercializacao`
- `idade_minima` / `idade_maxima`
- `requisitos_elegibilidade` (JSON)
- `coberturas_padrao` (JSON)
- `coberturas_opcionais` (JSON)

**Relacionamentos:**

- [[Coberturas]] (1:N)
- [[Fatores de Rating]] (1:N)
- [[Regras de Aceitação]] (1:N)
- [[Documentos Necessários]] (1:N)